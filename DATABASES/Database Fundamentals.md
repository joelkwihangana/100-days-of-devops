# PART 1: DATABASES - FIRST PRINCIPLES

## Why Databases Exist

Before we talk about PostgreSQL, ClickHouse, or vector databases, understand this:

**The Core Problem:** You need to store data that multiple people can read/write simultaneously, find specific data quickly, and ensure data follows rules.

### What Happens Without a Database?

**Attempt 1: Text Files**

```
users.txt:
joel,joel@email.com,password123
jane,jane@email.com,password456
```

**What Breaks:**

1. **Concurrent Access:**
   ```
   User A reads file → adds new user → writes file
   User B reads file at same time → adds different user → writes file
   Result: One user's data disappears (file overwritten)
   ```

2. **Finding Data:**
   ```
   To find joel@email.com in 1 million users:
   - Read line 1: not joel
   - Read line 2: not joel
   - ... check every single line ...
   - Read line 1,000,000: FOUND!
   
   Time: 45 seconds (unacceptable for login)
   ```

3. **Data Integrity:**
   ```
   No way to enforce:
   - Email must be unique
   - Password can't be empty
   - Email must contain @
   
   You could save: "joel,,notanemail" → Broken data
   ```

### What a Database Does

A database is **software** that solves these three problems:

1. **Concurrency Control:** Locks data during writes so others wait
2. **Indexing:** Creates lookup tables to find data instantly
3. **Constraints:** Enforces rules before saving data

---

## PostgreSQL Fundamentals (OLTP Database)

### What PostgreSQL Actually Is

PostgreSQL is a **process** (running program) on a server that:
- Listens on port 5432
- Receives SQL commands
- Manages data files on disk
- Enforces rules and provides fast access

When your FastAPI app connects to PostgreSQL, it's making a network connection to this process.

### What Happens: INSERT Command

When you run this:

```python
# Your FastAPI code
db.execute("INSERT INTO users (email, password) VALUES (?, ?)", 
           "joel@email.com", "hashed_pass")
```

**PostgreSQL's Internal Process (8 Steps):**

**Step 1: Receive Command**
```
PostgreSQL process receives: 
"INSERT INTO users (email, password) VALUES ('joel@email.com', 'hashed_pass')"
```

**Step 2: Parse SQL**
```
Parse the command to understand:
- Action: INSERT
- Table: users
- Columns: email, password
- Values: joel@email.com, hashed_pass
```

**Step 3: Check Constraints**
```
For each constraint on the table:
- UNIQUE(email): Does joel@email.com already exist?
  → Check existing data
  → If exists: REJECT with error "duplicate key value"
  
- NOT NULL(password): Is password empty?
  → Check value
  → If empty: REJECT with error "null value in column"
  
All checks pass ✅
```

**Step 4: Acquire Lock**
```
PostgreSQL says: "I'm writing to users table now"
Sets a lock so other writes to this row must wait
(Prevents concurrent write corruption)
```

**Step 5: Write to WAL (Write-Ahead Log)**
```
BEFORE touching data file, write to log:
/var/lib/postgresql/data/pg_wal/000000010000000000000001

Log entry: "INSERT users: joel@email.com, hashed_pass"

Why first? If server crashes during write, this log lets
PostgreSQL replay the operation on restart = data recovery
```

**Step 6: Write to Data File**
```
Write actual data to table file:
/var/lib/postgresql/data/base/16384/24576

Data is organized in 8KB "pages" (like pages in a notebook)
PostgreSQL finds free space, writes the row
```

**Step 7: Update Index**
```
If there's an index on email column:

CREATE INDEX idx_users_email ON users(email);

PostgreSQL updates the index file:
Index entry: "joel@email.com" → Page 42, Row 15

This makes future lookups instant
```

**Step 8: Commit & Release Lock**
```
Mark transaction as complete in WAL
Release the lock
Return to FastAPI: "1 row inserted"
```

**Total time: 0.001 - 0.005 seconds**

### What Happens: SELECT Command

When you run:

```python
user = db.execute("SELECT * FROM users WHERE email = ?", "joel@email.com")
```

**Without Index (Sequential Scan):**
```
PostgreSQL must check every row:

1. Read Page 1 (contains rows 1-100)
   Check each row: email = joel@email.com? No
   
2. Read Page 2 (contains rows 101-200)
   Check each row: email = joel@email.com? No
   
3. Continue through all pages...

42. Read Page 42 (contains rows 4101-4200)
    Check row 4115: email = joel@email.com? YES! ✅
    
Return row 4115

Time for 1 million rows: ~45 seconds
```

**With Index (Index Scan):**
```
1. Look up "joel@email.com" in index
   Index says: "Page 42, Row 15"
   
2. Jump directly to Page 42
   
3. Read Row 15
   
4. Return data

Time for 1 million rows: ~0.001 seconds
```

**This is why indexes are critical.**

### Understanding Indexes Deeply

**What an Index Actually Is:**

When you create:
```sql
CREATE INDEX idx_users_email ON users(email);
```

PostgreSQL creates a **B-tree data structure** (balanced tree) stored in a separate file.

**Think of it like a book's index:**

```
Book (users table): 1000 pages
Without index: To find "PostgreSQL", read all 1000 pages
With index: Look in back → "PostgreSQL: page 847" → jump there

Index is a sorted lookup table:
┌──────────────────┬─────────────────┐
│ Email (sorted)   │ Location        │
├──────────────────┼─────────────────┤
│ alice@email.com  │ Page 5, Row 3   │
│ bob@email.com    │ Page 12, Row 7  │
│ jane@email.com   │ Page 38, Row 2  │
│ joel@email.com   │ Page 42, Row 15 │
│ zara@email.com   │ Page 99, Row 44 │
└──────────────────┴─────────────────┘

Sorted = Fast binary search
```

**The Trade-off:**

✅ **Benefit:** SELECT queries 1000x faster  
❌ **Cost:** INSERT/UPDATE/DELETE slower (must update index)  
❌ **Cost:** More disk space (index file is separate)

**When to Index:**
- Columns in WHERE clauses (WHERE email = ...)
- Columns in JOIN operations (JOIN ON user_id = ...)
- Columns you search frequently

**When NOT to Index:**
- Columns you never search
- Small tables (< 1000 rows - sequential scan is fast enough)
- Tables with heavy writes, rare reads

### PostgreSQL Architecture (How It Works Internally)

```
Your Application (FastAPI)
        ↓
PostgreSQL Process (port 5432)
        ↓
┌───────────────────────────┐
│   Query Parser            │ ← Understands SQL
├───────────────────────────┤
│   Query Planner           │ ← Decides: use index or scan?
├───────────────────────────┤
│   Transaction Manager     │ ← Handles locks, WAL
├───────────────────────────┤
│   Buffer Cache (RAM)      │ ← Caches frequently-used pages
├───────────────────────────┤
│   Storage Engine          │ ← Writes to disk
└───────────────────────────┘
        ↓
Disk Files:
  - Data files (/base/...)
  - WAL files (/pg_wal/...)
  - Index files
```

**Key Concepts:**

**Buffer Cache:**
- PostgreSQL caches frequently-accessed data in RAM
- If data is in cache → 0.0001 sec access
- If data not in cache → must read from disk → 0.001 sec access
- You configure cache size: `shared_buffers = 256MB`

**Query Planner:**
- Analyzes your SQL query
- Decides: Should I use an index or scan the whole table?
- Chooses fastest approach based on statistics
- You can see the plan: `EXPLAIN SELECT ...`

---

## Why PostgreSQL Alone Isn't Enough (The Inkomoko Problem)

PostgreSQL is an **OLTP** database (Online Transaction Processing).

**What OLTP Means:**
- Handles individual transactions fast
- INSERT a loan application → 0.001 sec ✅
- SELECT a user by ID → 0.001 sec ✅
- UPDATE a single record → 0.001 sec ✅

**What OLTP Struggles With:**
- Analyzing millions of records
- Aggregations across large datasets
- Complex reporting queries

**Real Inkomoko Scenario:**

```sql
-- OLTP query (PostgreSQL is great):
SELECT * FROM loans WHERE loan_id = 12345;
-- Returns 1 row
-- Time: 0.001 seconds ✅

-- OLAP query (PostgreSQL struggles):
SELECT 
  country,
  YEAR(disbursement_date) as year,
  MONTH(disbursement_date) as month,
  COUNT(*) as total_loans,
  SUM(amount) as total_amount,
  AVG(amount) as avg_loan_size
FROM loans
WHERE disbursement_date >= '2023-01-01'
GROUP BY country, year, month
ORDER BY country, year, month;

-- Must scan 10 million rows, aggregate them
-- Time: 45-60 seconds ❌
-- This is too slow for a dashboard
```

**Why is this slow?**

PostgreSQL stores data **row-by-row**:

```
Disk layout (simplified):
Row 1: [loan_id: 1, country: Rwanda, date: 2023-01-15, amount: 5000, ...]
Row 2: [loan_id: 2, country: Kenya, date: 2023-01-20, amount: 3000, ...]
Row 3: [loan_id: 3, country: Rwanda, date: 2023-02-10, amount: 7000, ...]
...

To calculate SUM(amount) by country:
- Read entire Row 1 → Extract amount (5000) → Discard other columns
- Read entire Row 2 → Extract amount (3000) → Discard other columns
- Read entire Row 3 → Extract amount (7000) → Discard other columns
- ... repeat 10 million times ...

You're reading ALL columns for ALL rows, but only using a few columns
This is wasteful for analytics
```

**This is why Inkomoko needs ClickHouse.**

---

# PART 2: OLAP DATABASES (ClickHouse)

## The Fundamental Difference: Row-Based vs Column-Based Storage

### Row-Based Storage (PostgreSQL)

**How data is stored on disk:**
```
[Row 1: all columns] [Row 2: all columns] [Row 3: all columns]

Physical layout:
Page 1: loan_id=1, country=Rwanda, amount=5000, date=2023-01-15
Page 1: loan_id=2, country=Kenya, amount=3000, date=2023-01-20
Page 2: loan_id=3, country=Rwanda, amount=7000, date=2023-02-10
...
```

**Great for:** Getting one complete record  
**Bad for:** Aggregating one column across millions of records

### Column-Based Storage (ClickHouse)

**How data is stored on disk:**
```
[All loan_ids] [All countries] [All amounts] [All dates]

Physical layout:
File 1 (loan_id): 1, 2, 3, 4, 5, 6, 7, ...
File 2 (country): Rwanda, Kenya, Rwanda, Ethiopia, Kenya, ...
File 3 (amount): 5000, 3000, 7000, 2000, 4500, ...
File 4 (date): 2023-01-15, 2023-01-20, 2023-02-10, ...
```

**Great for:** Reading specific columns for aggregation  
**Bad for:** Getting one complete record

### Why ClickHouse is Faster for Analytics

**Query:** Calculate total loan amount by country

**PostgreSQL (row-based):**
```
Must read: loan_id, country, amount, date, status, officer_id, ...
           (all columns for all rows)
           
Disk reads: 10 million rows × 8 columns = 80 million column reads
Time: 45 seconds
```

**ClickHouse (column-based):**
```
Must read: country, amount (only 2 columns needed)

Disk reads: 10 million × 2 columns = 20 million column reads
Plus: Columns compress better (same value repeated)
Plus: CPU can process column data faster (SIMD instructions)

Time: 0.3 seconds
```

**That's 150x faster.**

### When to Use Each

**Use PostgreSQL (OLTP) when:**
- Need to INSERT/UPDATE/DELETE individual records frequently
- Need ACID transactions (money transfers, user updates)
- Need to fetch complete records (user profile, loan application)
- Working with < 1 million rows

**Use ClickHouse (OLAP) when:**
- Need to aggregate millions/billions of rows
- Building dashboards and reports
- Running analytics queries (SUM, COUNT, AVG across large datasets)
- Data is append-mostly (rare updates)

### Inkomoko's Architecture (Why Both Databases)

```
Day-to-Day Operations (OLTP)
    ↓
PostgreSQL
  - Loan applications
  - User management
  - Daily transactions
  - Fast reads/writes
    ↓
  Change Data Capture (CDC)
    ↓
ClickHouse
  - Analytics
  - Dashboards
  - Reports
  - Historical trends
```

**Why this architecture?**

1. **PostgreSQL:** Staff submit loan applications, update records → Fast ✅
2. **CDC (Change Data Capture):** Every change in PostgreSQL streams to ClickHouse in real-time
3. **ClickHouse:** Leadership views dashboard "Total loans by country" → Queries 10M rows in 0.3 sec ✅

**This is a standard pattern: Operational database (OLTP) + Analytical database (OLAP)**

### What is CDC (Change Data Capture)?

**The Problem:**
- PostgreSQL has operational data
- ClickHouse needs that data for analytics
- How do you keep them in sync?

**Bad Solution:** Batch ETL
```
Every hour:
1. SELECT all new data from PostgreSQL
2. INSERT into ClickHouse
3. Wait 1 hour for next sync

Problem: Data is 1 hour stale
Problem: Heavy load on PostgreSQL every hour
```

**Good Solution:** CDC (Streaming)
```
Real-time:
1. PostgreSQL writes data
2. CDC tool (Debezium) captures the write from WAL
3. Streams change to Kafka
4. ClickHouse consumes from Kafka
5. Data appears in ClickHouse in < 1 second

Benefit: Near real-time analytics
Benefit: Minimal load on PostgreSQL (just reading WAL)
```

**CDC Tools:** Debezium, Airbyte, AWS DMS

---

# PART 3: VECTOR DATABASES (AI-Powered Search)

## The Problem Traditional Databases Can't Solve

**Traditional Search (Exact Match):**

```sql
SELECT * FROM documents WHERE title = 'loan application';
-- Returns only documents with EXACT title "loan application"
-- Misses: "microfinance application", "credit request", "funding form"
```

**Traditional Search (Keyword Match):**

```sql
SELECT * FROM documents WHERE content LIKE '%loan%';
-- Returns documents containing word "loan"
-- Misses: Documents about "credit", "microfinance", "funding"
-- Even though they're semantically related
```

**The Need:** Find documents by **meaning**, not exact words.

## How Vector Databases Work (From First Principles)

### Step 1: Understanding Embeddings

**An embedding is a mathematical representation of meaning.**

Think of it like coordinates on a map:
- London → Latitude: 51.5, Longitude: -0.1
- Paris → Latitude: 48.9, Longitude: 2.4

Cities close together on the map are close in real life.

**Similarly, words/sentences can be represented as coordinates:**

```
"loan" → [0.8, 0.2, 0.5, 0.1, ... ] (1536 numbers)
"credit" → [0.7, 0.3, 0.4, 0.2, ... ] (1536 numbers)
"banana" → [0.1, 0.0, 0.9, 0.8, ... ] (1536 numbers)

"loan" and "credit" are CLOSE in this math space (similar meaning)
"loan" and "banana" are FAR in this math space (different meaning)
```

**How do we get these numbers?**

AI models (like OpenAI's text-embedding-ada-002) analyze billions of sentences and learn:
- "loan" and "credit" appear in similar contexts → similar vectors
- "banana" appears in different contexts → different vector

### Step 2: Storing Embeddings

**You use an AI API to generate embeddings:**

```python
import openai

text = "How to apply for a business loan"
response = openai.Embedding.create(
    input=text,
    model="text-embedding-ada-002"
)
embedding = response['data'][0]['embedding']
# embedding is a list of 1536 numbers: [0.2, 0.8, ...]
```

**Then store in a vector database:**

```sql
-- Using pgvector (PostgreSQL extension)
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    embedding vector(1536)  -- Array of 1536 numbers
);

INSERT INTO documents (title, content, embedding)
VALUES (
    'Loan Application Guide',
    'How to apply for a business loan...',
    '[0.2, 0.8, 0.5, ...]'  -- The 1536 numbers
);
```

### Step 3: Searching by Similarity

**User searches:** "How do I get business funding?"

```python
# 1. Convert search query to embedding
query = "How do I get business funding?"
query_embedding = openai.Embedding.create(input=query)['data'][0]['embedding']

# 2. Find similar documents
results = db.execute("""
    SELECT title, content,
           1 - (embedding <=> %s) AS similarity
    FROM documents
    ORDER BY embedding <=> %s
    LIMIT 5
""", [query_embedding, query_embedding])

# <=> is "cosine distance" operator
# Smaller distance = more similar
```

**Results:**
```
1. "Loan Application Guide" (similarity: 0.95)
2. "Microfinance Eligibility" (similarity: 0.89)
3. "Business Credit Requirements" (similarity: 0.85)
4. "Funding Options for Startups" (similarity: 0.82)
5. "Grant Application Process" (similarity: 0.78)
```

**Notice:** None of these titles contain "funding", but they're semantically related!

### Vector Database Options

**pgvector (PostgreSQL extension):**
- Pros: Works inside PostgreSQL (no new database)
- Pros: Good for < 1 million embeddings
- Cons: Slower than specialized vector DBs at scale
- Use when: Starting small, want simplicity

**Milvus (Open source):**
- Pros: Purpose-built for vectors, very fast
- Pros: Handles billions of embeddings
- Cons: Separate infrastructure to manage
- Use when: Scaling to millions of documents

**Pinecone (Managed service):**
- Pros: Fully managed, no infrastructure
- Pros: Easy to use, fast
- Cons: Costs money, vendor lock-in
- Use when: Want simplicity, don't want to manage infrastructure

### Inkomoko Use Case for Vector Search

**Scenario:** Inkomoko has 50,000 documents:
- Training materials
- Loan policies
- Best practices
- Case studies
- Support articles

**Without Vector Search:**
```
Staff member searches: "refugee loan eligibility"
Keyword search returns: 3 documents with exact words "refugee loan eligibility"
Misses: 47 related documents using different terms
```

**With Vector Search:**
```
Staff member searches: "refugee loan eligibility"
Vector search finds:
- "Displaced persons microfinance" ✅
- "Asylum seeker business funding" ✅
- "Emergency loan programs" ✅
- "Documentation requirements for non-citizens" ✅
- All semantically related content

Returns: 50 relevant documents
```

**This is AI-powered semantic search.**

### Creating the Index (How Vector Search is Fast)

**Without index:**
```
To find similar embeddings among 1 million:
- Compare query embedding to embedding 1 → Calculate distance
- Compare query embedding to embedding 2 → Calculate distance
- ... 1 million comparisons ...

Time: 30 seconds
```

**With index (IVF - Inverted File Index):**
```
1. Divide 1 million embeddings into 1000 clusters
2. Each cluster has ~1000 similar embeddings
3. Find which cluster query belongs to → 1 comparison
4. Search within that cluster → 1000 comparisons

Time: 0.1 seconds
```

**With index (HNSW - Hierarchical Navigable Small World):**
```
Creates a graph where similar embeddings are connected
Navigate the graph to find nearest neighbors quickly

More accurate than IVF, slightly slower to build

Time: 0.05 seconds
```

**Trade-off:**
- IVF: Faster to build, less accurate, good for massive scale
- HNSW: Slower to build, more accurate, great for quality search

---

# PART 4: EVENT-DRIVEN ARCHITECTURE (Kafka)

## The Problem with Synchronous Systems

**Your Current Approach (FastAPI):**

```python
@app.post("/loan/apply")
def apply_for_loan(loan: LoanApplication):
    # Do everything in sequence
    save_to_database(loan)          # 0.1 sec
    send_email(loan.entrepreneur)   # 2.0 sec (email service slow)
    notify_loan_officer(loan)       # 0.5 sec
    check_fraud_rules(loan)         # 3.0 sec (fraud check slow)
    update_analytics(loan)          # 0.2 sec
    
    return {"status": "success"}
    
    # Total time: 5.8 seconds
    # User waits 5.8 seconds for response
```

**What breaks:**

1. **Slow response:** User waits 5.8 seconds (bad UX)
2. **Cascading failure:** If email service is down, entire request fails
3. **Coupling:** Fraud check and analytics shouldn't block loan submission
4. **No retry:** If fraud check fails, it's lost (no automatic retry)

## Event-Driven Architecture Solution

**The Pattern:**

```python
@app.post("/loan/apply")
def apply_for_loan(loan: LoanApplication):
    # Only do critical work
    save_to_database(loan)  # 0.1 sec
    
    # Publish event to Kafka
    kafka.publish("loan-applications", {
        "loan_id": loan.id,
        "entrepreneur": loan.entrepreneur,
        "amount": loan.amount,
        "timestamp": now()
    })
    
    return {"status": "accepted"}
    
    # Total time: 0.15 seconds
    # User gets instant response
```

**Separate services consume events:**

```python
# Email Service (runs independently)
def email_consumer():
    for event in kafka.consume("loan-applications"):
        send_email(event['entrepreneur'])
        # If this fails, Kafka keeps the event for retry

# Fraud Check Service (runs independently)
def fraud_consumer():
    for event in kafka.consume("loan-applications"):
        check_fraud_rules(event)
        # Runs in background, doesn't block submission

# Analytics Service (runs independently)
def analytics_consumer():
    for event in kafka.consume("loan-applications"):
        update_dashboard(event)
        # Processes at its own pace
```

**Benefits:**

1. **Fast response:** User gets response in 0.15 sec (not 5.8 sec)
2. **Resilience:** If email service is down, loan still accepted
3. **Decoupling:** Each service is independent
4. **Reliability:** Kafka stores events, failed services can replay
5. **Scalability:** Can run multiple instances of each consumer

## How Kafka Works (Simplified)

**Kafka is a distributed log (append-only event stream).**

```
Kafka Topic: "loan-applications"

Partition 0: [Event1] [Event2] [Event3] [Event4] ...
Partition 1: [Event5] [Event6] [Event7] [Event8] ...
Partition 2: [Event9] [Event10] [Event11] ...

Events are appended, never deleted (until retention policy)
Consumers read from their last position (offset)
```

**Key Concepts:**

**Topic:** Category of events (like "loan-applications", "user-signups")

**Partition:** Topic is split into partitions for parallelism
- Events with same key go to same partition
- Each partition is ordered
- Multiple consumers can read different partitions in parallel

**Producer:** Service that publishes events to Kafka

**Consumer:** Service that reads events from Kafka

**Consumer Group:** Multiple instances of same service
- Kafka distributes partitions among group members
- If one consumer dies, others take over its partitions

**Offset:** Position in the log
- Each consumer tracks its offset
- Can replay from any offset (go back in time)

### Kafka vs Traditional Message Queue

**Traditional Queue (RabbitMQ):**
```
Producer → Queue → Consumer
           [Msg1]
           [Msg2]
           [Msg3]

Consumer reads Msg1 → Message deleted from queue
Only one consumer can process each message
Can't replay
```

**Kafka (Event Log):**
```
Producer → Kafka Topic → Consumer A
           [Event1]      Consumer B
           [Event2]      Consumer C
           [Event3]      (All can read same events)

Events stay in log (configurable retention)
Multiple consumers can process same event
Can replay from any point
```

### Real Inkomoko Scenario

**Event:** Loan is disbursed (money sent to entrepreneur)

**What needs to happen:**
1. Update accounting system (critical)
2. Send SMS to entrepreneur (important)
3. Notify loan officer (important)
4. Update analytics dashboard (non-critical)
5. Log for audit trail (critical)
6. Check for fraud patterns (background)

**Without Kafka (Synchronous):**
```python
def disburse_loan(loan_id):
    transfer_money(loan_id)         # 2 sec
    update_accounting(loan_id)      # 1 sec
    send_sms(loan_id)               # 3 sec (SMS gateway slow)
    notify_officer(loan_id)         # 0.5 sec
    update_dashboard(loan_id)       # 0.2 sec
    audit_log(loan_id)              # 0.1 sec
    fraud_check(loan_id)            # 4 sec
    
    # If SMS gateway is down, entire operation fails
    # Staff member waits 10.8 seconds
```

**With Kafka (Event-Driven):**
```python
def disburse_loan(loan_id):
    transfer_money(loan_id)  # 2 sec (critical)
    
    kafka.publish("loan-disbursed", {
        "loan_id": loan_id,
        "amount": amount,
        "timestamp": now()
    })
    
    # Staff member gets response in 2.1 seconds
    # Everything else happens in background

# Independent consumers:
# - Accounting consumer → updates accounting
# - SMS consumer → sends SMS (retries if fails)
# - Analytics consumer → updates dashboard
# - Audit consumer → logs event
# - Fraud consumer → checks patterns
```

**Resilience:**
- If SMS gateway is down, loan still disbursed
- SMS consumer retries automatically when gateway recovers
- Other services unaffected

---

# PART 5: DOCKER - MENTAL MODEL

## What Problem Does Docker Solve?

**The Classic Problem: "Works on My Machine"**

```
Developer's laptop:
- Ubuntu 20.04
- Python 3.9
- PostgreSQL 13
- App works perfectly ✅

Production server:
- Ubuntu 22.04
- Python 3.11
- PostgreSQL 15
- App crashes ❌

Why? Different versions, different dependencies
```

**Docker's Solution: Package the entire environment**

## What Docker Actually Is

**At its core, Docker uses Linux features to isolate processes.**

**Container ≠ Virtual Machine**

```
Virtual Machine:
┌─────────────────┐
│ App             │
│ OS (Ubuntu)     │ ← Full operating system
│ Hypervisor      │
│ Hardware        │
└─────────────────┘
Heavy, slow to start (minutes)

Container:
┌─────────────────┐
│ App             │
│ Libraries       │
│ Docker Engine   │ ← Shares host kernel
│ Host OS         │
│ Hardware        │
└─────────────────┘
Lightweight, fast to start (seconds)
```

**Key difference:** Container shares the host's Linux kernel, just isolated.

## Core Docker Concepts

### Image vs Container

**Image:** Template, blueprint, recipe
- Read-only
- Contains: OS files, libraries, application code
- Stored in layers
- You can't "run" an image directly

**Container:** Running instance of an image
- Like an object from a class (if you know programming)
- Writable
- Isolated process
- Can have multiple containers from one image

**Analogy:**
```
Image = Recipe for chocolate cake
Container = Actual cake you baked from that recipe

You can bake multiple cakes (containers) from one recipe (image)
Each cake is independent - eating one doesn't affect the others
```

### Dockerfile: How Images Are Built

**A Dockerfile is instructions to build an image:**

```dockerfile
# Start from a base image
FROM python:3.11

# Set working directory
WORKDIR /app

# Copy files
COPY requirements.txt .

# Install dependencies
RUN pip install -r requirements.txt

# Copy application code
COPY . .

# Define startup command
CMD ["python", "app.py"]
```

**What happens when you run `docker build`:**

```
Step 1: Download python:3.11 image
Step 2: Create layer with /app directory
Step 3: Create layer with requirements.txt
Step 4: Create layer with installed packages
Step 5: Create layer with application code
Step 6: Save final image with startup command

Result: Image "my-app:latest"
```

**Layers are stacked:**
```
┌─────────────────────┐
│ CMD python app.py   │ Layer 5
├─────────────────────┤
│ Application code    │ Layer 4
├─────────────────────┤
│ Installed packages  │ Layer 3
├─────────────────────┤
│ requirements.txt    │ Layer 2
├─────────────────────┤
│ python:3.11 base    │ Layer 1
└─────────────────────┘
```

**Why layers matter:**
- Layers are cached
- If Layer 3 hasn't changed, Docker reuses it (faster builds)
- If you change app code (Layer 4), only rebuilds Layer 4-5

### Running Containers

**Basic command:**
```bash
docker run -p 8000:8000 my-app:latest
```

**What this does:**

1. **docker run:** Create and start a container
2. **-p 8000:8000:** Map port 8000 in container to port 8000 on host
3. **my-app:latest:** Use this image

**Behind the scenes:**
```
1. Docker creates isolated namespace (container ID: abc123)
2. Sets up network interface for container
3. Mounts filesystem layers (read-only image + writable layer)
4. Starts process: python app.py
5. Process thinks it's running on its own machine
6. Actually: isolated process on host machine
```

**Port mapping explained:**
```
Host machine:
  localhost:8000 → Container port 8000
  
Container:
  App listens on 0.0.0.0:8000 inside container
  
User accesses: localhost:8000
Request goes to: Container's port 8000
```

### Docker Compose: Multiple Containers

**The problem:**
```
App needs:
- FastAPI application
- PostgreSQL database
- Redis cache

Starting manually:
docker run postgres
docker run redis
docker run my-app
(Must configure networking, environment variables, volumes...)
```

**Docker Compose solution:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:5432/mydb
    depends_on:
      - db
      - redis
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
  
  redis:
    image: redis:7

volumes:
  pgdata:
```

**One command:**
```bash
docker-compose up
```

**What happens:**
1. Creates network for all services
2. Starts postgres container
3. Starts redis container
4. Builds and starts app container
5. Configures DNS (app can reach db by hostname "db")
6. All containers can communicate

### Common Docker Commands

```bash
# Images
docker images                    # List images
docker build -t myapp:v1 .      # Build image
docker pull postgres:15         # Download image
docker rmi myapp:v1             # Delete image

# Containers
docker ps                       # List running containers
docker ps -a                    # List all containers (including stopped)
docker run -d -p 8000:8000 myapp  # Run container in background (-d)
docker stop <container_id>      # Stop container
docker rm <container_id>        # Delete container
docker logs <container_id>      # View container logs
docker exec -it <id> /bin/bash  # Open shell inside container

# Docker Compose
docker-compose up               # Start all services
docker-compose down             # Stop and remove all services
docker-compose logs app         # View logs for specific service
```

### Understanding Volumes (Persistent Data)

**The problem:**
```
Container filesystem is temporary
When container stops, data is lost

Example:
1. Start PostgreSQL container
2. Insert data into database
3. Stop container
4. Start new PostgreSQL container
5. Data is gone! ❌
```

**Solution: Volumes**

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
      # Maps host volume "pgdata" to container path

volumes:
  pgdata:  # Docker manages this on host filesystem
```

**How it works:**
```
Host machine:
  /var/lib/docker/volumes/pgdata → Persistent storage

Container:
  /var/lib/postgresql/data → Mounted from host volume

Data written to /var/lib/postgresql/data in container
Actually saved to /var/lib/docker/volumes/pgdata on host
Survives container restarts ✅
```

### Docker for the IMS Admin Role

**Why Inkomoko uses Docker:**

1. **Consistency:** Same environment dev → staging → production
2. **Isolation:** ClickHouse, PostgreSQL, Kafka each in containers
3. **Easy deployment:** `docker-compose up` on any server
4. **Version control:** Dockerfile in git = reproducible environment

**Example Inkomoko setup:**
```yaml
services:
  postgres:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
  
  clickhouse:
    image: clickhouse/clickhouse-server:latest
    volumes:
      - clickhouse-data:/var/lib/clickhouse
  
  kafka:
    image: confluentinc/cp-kafka:latest
    
  app:
    build: ./app
    depends_on:
      - postgres
      - clickhouse
      - kafka
```

---

# PART 6: INFRASTRUCTURE AS CODE (Terraform Basics)

## The Problem with Manual Infrastructure

**Traditional approach:**
```
1. Log into AWS console
2. Click "Launch EC2 instance"
3. Select instance type, configure security groups
4. Click "Create RDS database"
5. Configure networking, backups
6. Repeat for 5 countries...

Problems:
- Time-consuming
- Error-prone (clicked wrong option)
- Not reproducible (what did I click?)
- Not version controlled (can't track changes)
- Not documented (new engineer doesn't know how it was set up)
```

## Infrastructure as Code (Terraform)

**The concept:** Define infrastructure in code files

```hcl
# main.tf
resource "aws_db_instance" "inkomoko_db" {
  identifier        = "inkomoko-rwanda-db"
  engine            = "postgres"
  instance_class    = "db.t3.large"
  allocated_storage = 100
  
  username = "admin"
  password = var.db_password
  
  backup_retention_period = 30
  multi_az                = true
}
```

**Benefits:**
- **Reproducible:** Run same code = get same infrastructure
- **Version controlled:** Infrastructure changes tracked in git
- **Documented:** Code IS documentation
- **Testable:** Can test infrastructure changes before applying
- **Collaborative:** Team can review infrastructure changes

**Terraform workflow:**
```bash
# 1. Write infrastructure code
vim main.tf

# 2. Initialize Terraform
terraform init
# Downloads provider plugins (AWS, Azure, etc.)

# 3. Preview changes
terraform plan
# Shows: "I will create 1 database, 2 security groups"

# 4. Apply changes
terraform apply
# Creates actual infrastructure in AWS

# 5. Infrastructure exists and is managed
# To make changes: edit main.tf, run terraform apply again
```

**For Inkomoko:**
- Define database infrastructure once
- Deploy to 5 countries: `terraform apply -var="country=rwanda"`
- Consistent setup across all regions
- Easy to audit and change

---

# PART 7: MONITORING & RELIABILITY

## Why Monitoring Matters

**Scenario:**
```
2:00 AM: Database crashes
2:01 AM: Users can't log in
2:02 AM: Users frustrated
2:05 AM: Users calling support
6:00 AM: You wake up, check email
6:01 AM: Panic - system has been down 4 hours

Lost: 4 hours of uptime, user trust, potential revenue
```

**With monitoring:**
```
2:00 AM: Database crashes
2:00 AM: Monitoring detects, sends alert to phone
2:01 AM: You wake up, start investigating
2:05 AM: Database restarted, system back up

Lost: 5 minutes of uptime
```

## The Three Pillars of Observability

### 1. Metrics (Numbers Over Time)

**What to monitor:**
```
Database:
- CPU usage: 45% (normal), 95% (problem)
- Memory usage: 60% (normal), 98% (problem)
- Query latency: 5ms (normal), 500ms (problem)
- Connection count: 50/200 (normal), 195/200 (problem)
- Disk usage: 40% (normal), 95% (problem)

Application:
- Request rate: 100 req/sec
- Error rate: 0.1% (normal), 5% (problem)
- Response time: 50ms (normal), 2000ms (problem)
```

**Tools:** Prometheus (collects metrics), Grafana (visualizes)

**Example Grafana dashboard:**
```
┌─────────────────────────────────────┐
│ PostgreSQL Performance              │
├─────────────────────────────────────┤
│ CPU Usage:    [====    ] 45%        │
│ Memory:       [=====   ] 60%        │
│ Connections:  [==      ] 50/200     │
│ Query Latency: 5ms avg              │
│                                      │
│ [Graph: Query latency over 24h]     │
│     ^                                │
│ 10ms│     ╱╲                        │
│  5ms│────╱  ╲─────                  │
│  0ms│                                │
│     └────────────────────>           │
│      00:00  06:00  12:00  18:00     │
└─────────────────────────────────────┘
```

### 2. Logs (What Happened)

**What to log:**
```
Application logs:
2024-02-17 14:32:01 INFO User joel@email.com logged in
2024-02-17 14:32:15 INFO Loan application submitted: ID 12345
2024-02-17 14:32:20 ERROR Failed to send email: SMTP timeout

Database logs:
2024-02-17 14:30:00 LOG checkpoint starting
2024-02-17 14:35:22 ERROR deadlock detected
2024-02-17 14:35:22 DETAIL Process 1234 waits for process 5678
```

**Tools:** ELK Stack (Elasticsearch, Logstash, Kibana)

**Centralized logging:**
```
All servers send logs to Elasticsearch
Kibana provides search interface:
"Show me all errors in the last hour containing 'database'"
```

### 3. Tracing (Request Flow)

**For distributed systems:**

```
User request: "Submit loan application"
    ↓
API Gateway (10ms)
    ↓
Loan Service (50ms)
    ↓ (parallel requests)
    ├─> Credit Check Service (200ms) ← SLOW!
    ├─> Fraud Check Service (30ms)
    └─> Email Service (100ms)
    ↓
Total: 260ms

Trace shows: Credit check is bottleneck
```

**Tools:** Jaeger, Zipkin

## Setting Up Alerts

**Bad alert:**
```
CPU > 50%
→ Fires 1000 times per day
→ Team ignores alerts
→ Real problem missed
```

**Good alert:**
```
CPU > 90% for 5 minutes
AND
Query latency > 1 second

→ Fires only when real problem
→ Team takes seriously
```

**Alert channels:**
- Email (low urgency)
- Slack (medium urgency)
- PagerDuty/phone call (high urgency - wakes you up)

---

# PART 8: YOUR INTERVIEW STRATEGY

## What They're Actually Assessing

**NOT:**
- ❌ Do you know every technology already?
- ❌ Can you recite commands from memory?
- ❌ Have you used ClickHouse in production?

**YES:**
- ✅ Can you think systematically?
- ✅ Do you understand WHY technologies exist?
- ✅ Can you learn quickly?
- ✅ Are you honest about gaps?
- ✅ Do you care about the mission?

## The Framework for Any Technical Question

When asked ANY technical question, use this structure:

**1. Acknowledge your experience level**
```
"I haven't worked with X in production yet, but..."
```

**2. Demonstrate conceptual understanding**
```
"I understand it solves [problem]..."
"The core concept is [explain]..."
```

**3. Connect to what you know**
```
"In my projects with PostgreSQL, I've dealt with [related concept]..."
"This is similar to [thing you know], except..."
```

**4. Apply to their context**
```
"For Inkomoko's use case of [specific scenario]..."
"Given you're operating across 5 countries..."
```

**5. Show learning approach**
```
"I'd start by [practical first step]..."
"I'd test this in a sandbox environment first..."
"I'd consult the team's experience with..."
```

## Example Answers to Likely Questions

### Q: "Do you have experience with ClickHouse?"

**Your answer:**

"I haven't worked with ClickHouse in production yet, but I understand why it exists and the problem it solves.

In my projects, I've used PostgreSQL, which is a row-based OLTP database - great for transactional operations like user login or creating records. But when you need to aggregate millions of records for analytics - like 'total loans by country for 2025' - row-based databases become slow because they read entire rows even when you only need a few columns.

ClickHouse is a columnar OLAP database that stores data by columns instead of rows. For aggregations, this is dramatically faster because you only read the columns you need. A query that might take 45 seconds in PostgreSQL can run in under a second in ClickHouse.

For Inkomoko's use case - analyzing loan data across 5 countries with millions of records - I can see why you'd want operational data in PostgreSQL for fast transactions, then stream changes via CDC to ClickHouse for real-time dashboards and reporting.

I'd approach learning ClickHouse by setting up a test instance with Docker, loading sample loan data, and benchmarking query performance compared to PostgreSQL to understand the trade-offs firsthand."

**What you showed:**
- Honesty about experience
- Deep understanding of the problem
- Connection to your PostgreSQL knowledge
- Application to Inkomoko's needs
- Practical learning approach

### Q: "How would you troubleshoot a slow query?"

**Your answer:**

"I'd approach this systematically.

First, identify which query is slow. In PostgreSQL, I'd enable pg_stat_statements to track query execution times. I'd look for queries with high average execution time OR queries that run frequently even if individually fast - a 1-second query that runs 1000 times per hour is a bigger problem than a 5-second query that runs once per day.

Second, analyze the execution plan. I'd run EXPLAIN ANALYZE on the slow query to see how PostgreSQL is executing it. I'm looking for sequential scans on large tables - that means it's reading every row instead of using an index. I'd also check for expensive JOIN operations or unnecessary sorting.

Third, optimize based on findings. If indexes are missing, I'd create them on columns used in WHERE clauses and JOIN conditions. If the query is too complex, I might break it into smaller queries or add filtering earlier. For reporting queries, I'd consider if they should run against a read replica or even an analytical database like ClickHouse instead of the production OLTP database.

Finally, I'd monitor after the fix to ensure it works and doesn't cause new issues - for example, an index that speeds up reads might slow down writes if the table has heavy inserts.

I'd also want to understand the business context - is this query for a live dashboard or a monthly report? That changes whether we optimize for the query itself or move it to a different system."

**What you showed:**
- Systematic debugging process
- Real PostgreSQL knowledge
- Understanding of trade-offs
- Business awareness

### Q: "What would you do if a database failed at 3 AM?"

**Your answer:**

"First, I need to understand what failed. Is it the database process crashed, a disk failure, network issue, or data corruption? I'd check monitoring dashboards first - they should show when metrics (CPU, memory, disk) went abnormal.

If it's a complete database failure and we have read replicas, I'd fail over to a replica immediately to restore service. This minimizes downtime while I investigate the root cause.

For investigation, I'd check logs - PostgreSQL logs, system logs, disk health. I'd look for patterns: Did we run out of disk space? Out of memory? Was there a hardware failure? A problematic query that crashed the server?

For recovery, the approach depends on the failure type. If it's a process crash, I'd try restarting PostgreSQL after confirming logs show no corruption. If there's data corruption, I'd restore from the most recent backup using point-in-time recovery from WAL archives to minimize data loss.

The key metrics I'd track are RTO (Recovery Time Objective) and RPO (Recovery Point Objective) - how fast can we recover and how much data can we afford to lose. These should be defined beforehand based on business requirements.

After recovery, I'd do a post-mortem: document what happened, why it happened, and how we prevent it. Maybe we need better alerting for disk space, automated failover to replicas, or more aggressive backup strategies.

I'd also communicate with stakeholders throughout - they need to know the system is down, what we're doing, and when we expect recovery."

**What you showed:**
- Incident response thinking
- Knowledge of key concepts (replicas, WAL, RTO/RPO)
- Systematic approach
- Communication awareness
- Learning from failure

### Q: "Tell us about yourself"

**Your answer:**

"Thank you for having me. I'm Joel, and I'm deliberately building a career as a DevOps engineer who understands systems deeply, not just runs commands.

My foundation comes from building full-stack applications with React and FastAPI backed by PostgreSQL. I've handled the complete lifecycle - designing database schemas, writing queries, containerizing with Docker, and deploying to production on VPS. Through this, I've learned database fundamentals, Linux server administration, and the challenges of reliability and performance.

I'm honest about where I am - I'm early in my DevOps journey. I haven't worked with production-scale systems or specialized technologies like ClickHouse, Kafka, or vector databases. But I've researched this role thoroughly, and I understand why these technologies exist, the problems they solve, and how they fit together.

What excites me about Inkomoko is the combination of technical complexity and meaningful impact. Managing data infrastructure across 5 countries for mission-critical financial services - ensuring reliability for systems that help entrepreneurs succeed and communities thrive - that's exactly the kind of challenging, purposeful work I want to master.

I learn by doing. I dig deep into concepts. I ask why, not just how. And I'm ready to grow into this role while contributing from day one. I'm looking for an environment where I can learn from experienced engineers like yourselves, face real production challenges, and build expertise that lasts a career."

**What you showed:**
- Authentic story
- Real technical foundation
- Honesty about skill level
- Research and preparation
- Mission alignment
- Growth mindset

## Questions You Should Ask Them

**To Senior IT Infrastructure Admin:**
1. "What are the biggest infrastructure challenges you're facing right now?"
2. "What does the on-call rotation look like, and how does the team handle incidents?"
3. "What would success look like for this role in the first 90 days?"

**To Senior Software Developer:**
4. "How do you currently handle schema changes and data migrations?"
5. "What's the workflow when you need database performance optimization?"

**To Senior Data Engineer:**
6. "What are the biggest challenges with data pipeline reliability right now?"
7. "How do you ensure data quality across the different data systems?"

**General:**
8. "How does Inkomoko's mission influence technical decisions around security and reliability?"
9. "What opportunities are there for learning and mentorship in this role?"
10. "What's the team's approach to learning new technologies and keeping skills current?"

---

# PART 9: FINAL PREPARATION PLAN

## Tuesday Evening (Tonight) - 3 Hours

### Hour 1: Deep Read (This Document)
- Read this entire document once
- Focus on understanding concepts, not memorizing
- Make notes on parts you don't understand

### Hour 2: Watch Core Concepts (Videos)

**Watch these 3 videos only:**

1. **"Database Indexing Explained"** - Hussein Nasser (15 min)
   https://www.youtube.com/watch?v=HubezKbFL7E
   
2. **"What is a Vector Database?"** - IBM Technology (10 min)
   https://www.youtube.com/watch?v=klTvEwg3oJ4
   
3. **"Event-Driven Architecture"** - ByteByteGo (8 min)
   https://www.youtube.com/watch?v=STKCRSUsyP0

# WHAT YOU NOW UNDERSTAND

After reading this document, you understand:

✅ **Databases from first principles:** Why they exist, what they do  
✅ **PostgreSQL internals:** INSERT, SELECT, indexes, WAL  
✅ **OLTP vs OLAP:** Row-based vs columnar, when to use each  
✅ **ClickHouse:** Why it exists, how it fits Inkomoko's needs  
✅ **Vector databases:** Embeddings, semantic search, use cases  
✅ **Event-driven architecture:** Kafka, decoupling, resilience  
✅ **Docker:** Images, containers, volumes, why it matters  
✅ **Infrastructure as Code:** Terraform basics, reproducibility  
✅ **Monitoring:** Metrics, logs, tracing, reliability  

**More importantly, you understand:**
✅ How to think through technical questions  
✅ How to be honest while showing capability  
✅ How to connect concepts to Inkomoko's needs  
✅ How to demonstrate learning mindset  

---
