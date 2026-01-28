# WordPress Mastery: Phase 1 — Foundation (Deep Understanding Edition)

You're absolutely right. Let's do this properly. I'll explain **every single concept** so deeply that you can teach it to others and never forget it.

---

## 🎯 Phase 1 Learning Objectives

By the end of Phase 1, you will understand:
- **What a web server is and why it exists**
- **How HTTP works at a protocol level**
- **What PHP is, why WordPress needs it, and how it executes**
- **What MySQL is, how databases work, and why WordPress stores data there**
- **Linux file permissions and why they matter for security**

---

## 📖 Part 1: Understanding the Web Stack (The Foundation)

### **What is a Web Stack?**

**Analogy:** Think of a restaurant:
- **Linux (Ubuntu)** = The building (foundation)
- **Apache** = The waiter (takes orders, delivers food)
- **MySQL** = The kitchen storage (ingredients/data)
- **PHP** = The chef (processes requests, cooks food)

When a customer (browser) orders food (web page):
1. **Waiter (Apache)** takes the order
2. **Chef (PHP)** cooks using ingredients from **storage (MySQL)**
3. **Waiter (Apache)** delivers the finished dish (HTML page)

This is called the **LAMP stack** (Linux, Apache, MySQL, PHP).

---

## 🌐 Part 2: How the Web Actually Works (HTTP Protocol)

### **What is HTTP?**

**HTTP** = **H**yper**T**ext **T**ransfer **P**rotocol

It's the **language** browsers and servers use to communicate.

### **Real-World Example:**

When you type `www.google.com` in your browser:

```
1. Your browser: "Hey, give me the homepage"
   (HTTP Request: GET / HTTP/1.1)

2. Google's server: "Here's the HTML code for the homepage"
   (HTTP Response: 200 OK + HTML content)

3. Your browser renders the HTML into the page you see
```

### **Understanding HTTP Requests**

Every HTTP request has:
1. **Method** — What you want to do (GET = retrieve, POST = send data)
2. **URL** — Which resource you want
3. **Headers** — Extra information (your browser type, cookies, etc.)
4. **Body** — Data you're sending (for POST requests, like form submissions)

### **Example HTTP Request:**

```http
GET /about-us HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Ubuntu; Linux x86_64)
Accept: text/html
Cookie: session_id=abc123
```

**Breaking it down:**
- `GET` — I want to retrieve something (not sending data)
- `/about-us` — The specific page I want
- `HTTP/1.1` — The version of HTTP protocol
- `Host:` — Which website (important because one server can host multiple sites)
- `User-Agent:` — What browser/OS I'm using
- `Accept:` — I want HTML back (not JSON or XML)
- `Cookie:` — My session identifier (so server knows I'm logged in)

### **HTTP Response:**

```http
HTTP/1.1 200 OK
Date: Wed, 28 Jan 2026 10:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Content-Length: 1234

<!DOCTYPE html>
<html>
<head><title>About Us</title></head>
<body><h1>Welcome!</h1></body>
</html>
```

**Breaking it down:**
- `HTTP/1.1 200 OK` — Request successful (200 = success code)
- `Server: Apache` — The web server software
- `Content-Type: text/html` — I'm sending HTML
- Then the actual HTML content

### **HTTP Status Codes (You Must Know These):**

- **200 OK** — Success
- **301 Moved Permanently** — Page moved (redirect)
- **302 Found** — Temporary redirect
- **400 Bad Request** — Client sent invalid request
- **401 Unauthorized** — Need to log in
- **403 Forbidden** — Logged in but don't have permission
- **404 Not Found** — Page doesn't exist
- **500 Internal Server Error** — Server crashed/PHP error
- **502 Bad Gateway** — Server behind proxy is down
- **503 Service Unavailable** — Server overloaded

**In WordPress context:**
- 404 = Post/page doesn't exist
- 500 = PHP fatal error (check debug.log)
- 503 = Your server is down or in maintenance mode

### **📚 Deep Dive Resources:**

1. **[MDN HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)** — Read this completely
2. **[HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)** — Memorize common ones
3. **[How HTTPS Works](https://howhttps.works/)** — Comic-style visual explanation

### **Hands-On Exercise (Before Installing Anything):**

```bash
# Install curl (command-line tool to make HTTP requests)
sudo apt update
sudo apt install curl

# Make a real HTTP request to Google
curl -v http://www.google.com
```

**What `-v` (verbose) shows you:**
```
* Trying 142.250.185.68:80...    ← DNS resolved google.com to this IP
* Connected to www.google.com    ← TCP connection established
> GET / HTTP/1.1                 ← Your request (> means outgoing)
> Host: www.google.com
> User-Agent: curl/7.81.0
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently ← Google's response (< means incoming)
< Location: http://www.google.com/
< Content-Type: text/html
< 
[HTML content here]
```

**Try these commands:**

```bash
# See just the response headers
curl -I http://www.google.com

# See headers for Wikipedia
curl -I https://en.wikipedia.org/wiki/Web_server

# Make a POST request (simulating form submission)
curl -X POST -d "username=test&password=test123" http://httpbin.org/post
```

**Understanding:** You just manually did what browsers do automatically thousands of times per day.

---

## 🖥️ Part 3: What is Apache? (The Web Server)

### **What Apache Does:**

Apache is a **web server** — software that:
1. **Listens** on port 80 (HTTP) or 443 (HTTPS)
2. **Receives** HTTP requests from browsers
3. **Decides** what to do with the request
4. **Returns** HTTP responses

### **Why We Need Apache for WordPress:**

WordPress is **not a standalone application**. It's PHP files that need:
- Something to receive HTTP requests (Apache)
- Something to execute PHP code (PHP interpreter)
- Something to store data (MySQL)

Without Apache, you'd have to manually run PHP files from command line (no web interface).

### **How Apache Works (Step-by-Step):**

```
1. Browser requests: http://localhost/index.php
   ↓
2. Apache receives request on port 80
   ↓
3. Apache checks: "Is there a file at /var/www/html/index.php?"
   ↓
4. If YES and it's a .php file:
   - Apache calls PHP interpreter
   - PHP executes the code
   - PHP returns output (HTML)
   - Apache sends HTML to browser
   ↓
5. If YES but it's .html/.jpg/.css:
   - Apache sends file directly (no PHP needed)
   ↓
6. If NO (file doesn't exist):
   - Apache returns 404 Not Found
```

### **Installing Apache (With Full Explanation):**

```bash
# Update package list (like refreshing app store)
sudo apt update
```

**What this does:**
- `sudo` — Run as superuser (admin)
- `apt` — Ubuntu's package manager (like App Store)
- `update` — Refresh list of available software versions

```bash
# Install Apache
sudo apt install apache2
```

**What happens during installation:**
1. Downloads Apache binary files
2. Creates `/var/www/html/` directory (document root)
3. Creates Apache user `www-data` (non-login user for security)
4. Sets up systemd service (so Apache starts on boot)
5. Configures firewall rules (if ufw is active)

```bash
# Check if Apache is running
sudo systemctl status apache2
```

**What `systemctl` does:**
- `systemctl` — Controls system services (start/stop/restart)
- `status` — Show if service is active/inactive
- `apache2` — The service name

**You should see:**
```
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2026-01-28 10:00:00 UTC; 2min ago
```

**Breaking down the output:**
- `Loaded: enabled` — Apache will start automatically on boot
- `Active: active (running)` — Apache is currently running
- `Main PID: 1234` — Process ID (useful for debugging)

### **Understanding Apache Processes:**

```bash
# See Apache processes
ps aux | grep apache2
```

**Output:**
```
root      1234  0.0  0.5  apache2 -k start
www-data  1235  0.0  0.3  apache2 -k start
www-data  1236  0.0  0.3  apache2 -k start
```

**What this means:**
- **One master process (root)** — Manages everything
- **Multiple worker processes (www-data)** — Handle actual requests
- **Why www-data?** Security — if hacked, attacker doesn't have root access

### **Apache Configuration Files:**

```bash
# Main configuration
/etc/apache2/apache2.conf       # Global settings

# Sites configuration
/etc/apache2/sites-available/   # Available site configs
/etc/apache2/sites-enabled/     # Active site configs (symlinks)

# Modules
/etc/apache2/mods-available/    # Available modules
/etc/apache2/mods-enabled/      # Active modules

# Document root (where your files go)
/var/www/html/                  # Default website location

# Logs
/var/log/apache2/access.log     # Every request
/var/log/apache2/error.log      # Errors only
```

### **Hands-On: Understanding Apache Logs**

```bash
# Watch access log in real-time
sudo tail -f /var/log/apache2/access.log
```

**Now open browser and visit:** `http://localhost`

**You'll see in the log:**
```
127.0.0.1 - - [28/Jan/2026:10:05:00 +0000] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0"
```

**Breaking it down:**
- `127.0.0.1` — Client IP (localhost = your own machine)
- `[28/Jan/2026:10:05:00 +0000]` — Timestamp
- `"GET / HTTP/1.1"` — Request method and protocol
- `200` — Status code (success)
- `3380` — Response size in bytes
- `"Mozilla/5.0"` — User agent (browser)

**Try this:**

```bash
# Visit a non-existent page
curl http://localhost/nonexistent

# Check access log
sudo tail -n 1 /var/log/apache2/access.log
```

You'll see `404` status code.

```bash
# Check error log
sudo tail /var/log/apache2/error.log
```

**Why logs matter in production:**
- Debug "site not loading" issues
- Track traffic patterns
- Identify attacks (repeated 404s = someone scanning for vulnerabilities)
- Monitor performance (slow requests)

### **Apache Commands You Must Know:**

```bash
# Start Apache
sudo systemctl start apache2

# Stop Apache
sudo systemctl stop apache2

# Restart Apache (stops then starts)
sudo systemctl restart apache2

# Reload configuration (no downtime)
sudo systemctl reload apache2

# Enable Apache on boot
sudo systemctl enable apache2

# Disable Apache on boot
sudo systemctl disable apache2

# Check configuration syntax
sudo apache2ctl configtest
```

**When to use each:**
- `restart` — After major config changes
- `reload` — After minor config changes (faster, no downtime)
- `configtest` — **ALWAYS** before restart (catches syntax errors)

### **📚 Deep Dive Resources:**

1. **[Apache Official Documentation](https://httpd.apache.org/docs/2.4/)** — Bookmark this
2. **[DigitalOcean Apache Guide](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04)** — Excellent tutorials
3. **[Apache Security Best Practices](https://geekflare.com/apache-web-server-hardening-security/)** — Must read

### **Hands-On Exercise:**

```bash
# Create a simple HTML file
echo '<h1>Hello from Apache!</h1>' | sudo tee /var/www/html/test.html

# Visit http://localhost/test.html in browser

# Check the log
sudo tail -n 1 /var/log/apache2/access.log
```

---

## 🐘 Part 4: What is PHP? (The Programming Language)

### **What PHP Is:**

**PHP** = **P**HP: **H**ypertext **P**reprocessor (recursive acronym)

It's a **server-side scripting language** designed for web development.

### **Client-Side vs Server-Side:**

**Client-Side (JavaScript):**
```
Browser requests page
  ↓
Server sends HTML + JavaScript
  ↓
JavaScript runs IN YOUR BROWSER
  ↓
You see dynamic effects (dropdown menus, animations)
```

**Server-Side (PHP):**
```
Browser requests page
  ↓
Server EXECUTES PHP code
  ↓
PHP generates HTML
  ↓
Server sends ONLY HTML to browser
  ↓
Browser never sees PHP code (security)
```

### **Why WordPress Uses PHP:**

1. **Dynamic Content** — Blog posts aren't static HTML files; they're generated from database
2. **User Authentication** — PHP checks if you're logged in
3. **Database Interaction** — PHP queries MySQL
4. **Template System** — One PHP template generates thousands of pages

### **Installing PHP:**

```bash
# Install PHP and essential extensions
sudo apt install php libapache2-mod-php php-mysql php-cli php-curl php-gd php-mbstring php-xml php-xmlrpc php-zip
```

**Breaking down what we installed:**

1. **php** — Core PHP interpreter
2. **libapache2-mod-php** — Connects Apache to PHP (so Apache can execute .php files)
3. **php-mysql** — Lets PHP talk to MySQL databases (ESSENTIAL for WordPress)
4. **php-cli** — Lets you run PHP from command line
5. **php-curl** — Lets PHP make HTTP requests (used by plugins to call APIs)
6. **php-gd** — Image manipulation (resizing, cropping thumbnails)
7. **php-mbstring** — Multi-byte string handling (for non-English characters)
8. **php-xml** — Parse XML (used by feeds, APIs)
9. **php-xmlrpc** — Remote procedure calls (some plugins need this)
10. **php-zip** — Extract/create ZIP files (plugin/theme uploads)

**Without these extensions, WordPress will either:**
- Not install at all
- Show errors like "PHP extension mbstring is missing"
- Have broken features (can't upload images without php-gd)

```bash
# Check PHP version
php -v
```

**Output:**
```
PHP 8.1.2 (cli) (built: Jan 15 2026 10:30:00)
Copyright (c) The PHP Group
Zend Engine v4.1.2
```

**PHP version matters:**
- WordPress requires **PHP 7.4+** (minimum)
- **PHP 8.0+** recommended for performance
- Older PHP versions have security vulnerabilities

```bash
# See ALL PHP configuration
php -i
```

This shows **phpinfo()** — every setting, every loaded extension.

```bash
# Find specific settings
php -i | grep memory_limit
php -i | grep upload_max_filesize
php -i | grep post_max_size
```

### **Creating Your First PHP Script:**

```bash
# Create test file
sudo nano /var/www/html/test.php
```

**Type this:**

```php
<?php
// This is a comment
echo "Hello, World!";
?>
```

**Save** (Ctrl+O, Enter, Ctrl+X)

**Visit:** `http://localhost/test.php`

**What happened:**
1. Browser requested `/test.php`
2. Apache saw `.php` extension
3. Apache called PHP interpreter
4. PHP executed `echo "Hello, World!";`
5. PHP returned string "Hello, World!"
6. Apache sent it to browser

**Now try this:**

```php
<?php
// Show all PHP info
phpinfo();
?>
```

Save as `/var/www/html/info.php` and visit `http://localhost/info.php`

**You'll see:**
- PHP version
- Loaded extensions
- Configuration settings
- Server information

**SECURITY WARNING:** Delete `info.php` in production! It reveals too much about your server.

```bash
sudo rm /var/www/html/info.php
```

### **Understanding PHP Basics (Critical Foundation):**

#### **1. Variables**

```php
<?php
// Variables start with $
$name = "John";
$age = 30;
$price = 19.99;
$is_admin = true;

// Concatenation (joining strings)
echo "Hello, " . $name;
echo "You are " . $age . " years old";
?>
```

#### **2. Arrays**

```php
<?php
// Indexed array
$fruits = array("Apple", "Banana", "Orange");
echo $fruits[0];  // Apple

// Associative array (key => value)
$user = array(
    "name" => "John",
    "email" => "john@example.com",
    "age" => 30
);
echo $user["name"];  // John
?>
```

**In WordPress context:**
```php
// WordPress returns post data as associative array
$post = array(
    'ID' => 123,
    'post_title' => 'My Blog Post',
    'post_content' => 'Post content here...',
    'post_author' => 1
);
```

#### **3. Control Structures**

```php
<?php
// If statement
$age = 18;
if ($age >= 18) {
    echo "Adult";
} else {
    echo "Minor";
}

// Loop
for ($i = 1; $i <= 5; $i++) {
    echo "Number: " . $i . "<br>";
}

// While loop
$count = 1;
while ($count <= 3) {
    echo "Count: " . $count . "<br>";
    $count++;
}

// Foreach (most used in WordPress)
$posts = array("Post 1", "Post 2", "Post 3");
foreach ($posts as $post) {
    echo $post . "<br>";
}
?>
```

#### **4. Functions**

```php
<?php
// Define function
function greet($name) {
    return "Hello, " . $name;
}

// Call function
echo greet("Alice");  // Hello, Alice

// Function with multiple parameters
function add_numbers($a, $b) {
    return $a + $b;
}

echo add_numbers(5, 3);  // 8
?>
```

**In WordPress:**
```php
// WordPress is FULL of functions
the_title();           // Outputs post title
get_the_title();       // Returns post title (doesn't output)
the_permalink();       // Outputs post URL
wp_enqueue_script();   // Loads JavaScript file
```

### **📚 PHP Learning Resources:**

1. **[PHP Official Manual](https://www.php.net/manual/en/)** — Your bible
2. **[PHP The Right Way](https://phptherightway.com/)** — Modern best practices
3. **[W3Schools PHP Tutorial](https://www.w3schools.com/php/)** — Interactive learning
4. **[PHP: The Wrong Way](https://www.phpthewrongway.com/)** — What NOT to do (important!)

### **Hands-On Exercise:**

Create `/var/www/html/exercise.php`:

```php
<?php
// Exercise: Display your system info
echo "<h1>My Server Info</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";
echo "<p>Server Software: " . $_SERVER['SERVER_SOFTWARE'] . "</p>";
echo "<p>Your IP: " . $_SERVER['REMOTE_ADDR'] . "</p>";
echo "<p>Current Time: " . date('Y-m-d H:i:s') . "</p>";

// Display loaded extensions
$extensions = get_loaded_extensions();
echo "<h2>Loaded PHP Extensions:</h2>";
echo "<ul>";
foreach ($extensions as $ext) {
    echo "<li>" . $ext . "</li>";
}
echo "</ul>";
?>
```

Visit `http://localhost/exercise.php`

---

## 📊 Part 5: MySQL Database (Data Storage)

### **What is a Database?**

**Analogy:** A filing cabinet.
- **Database** = The entire cabinet
- **Tables** = Individual drawers
- **Rows** = Files in a drawer
- **Columns** = Fields on each file (Name, Date, etc.)

### **Why WordPress Needs a Database:**

WordPress stores:
- **Posts/Pages** — Title, content, author, date
- **Users** — Username, email, password (hashed)
- **Comments** — Comment text, author, date
- **Settings** — Site URL, timezone, permalink structure
- **Plugin Data** — Custom tables for complex plugins

**Without database:** Every page would be a static HTML file. No dynamic content, no search, no user accounts.

### **Installing MySQL:**

```bash
# Install MySQL server
sudo apt install mysql-server
```

**What gets installed:**
- MySQL server (the database engine)
- MySQL client (command-line tool)
- Default configuration files

```bash
# Secure the installation
sudo mysql_secure_installation
```

**This wizard asks:**
1. **Set root password?** YES (strong password!)
2. **Remove anonymous users?** YES (security)
3. **Disallow root login remotely?** YES (only localhost)
4. **Remove test database?** YES (not needed)
5. **Reload privilege tables?** YES (apply changes)

### **Accessing MySQL:**

```bash
# Log in as root
sudo mysql -u root -p
```

**Breaking down the command:**
- `mysql` — MySQL client program
- `-u root` — Username (root = admin)
- `-p` — Prompt for password

**You'll see:**
```
mysql>
```

This is the MySQL command prompt.

### **Understanding Databases (Hands-On):**

```sql
-- Show all databases
SHOW DATABASES;
```

**Output:**
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

**What these are:**
- `information_schema` — Metadata about all databases
- `mysql` — User accounts, permissions
- `performance_schema` — Performance monitoring
- `sys` — System information

```sql
-- Create a new database for testing
CREATE DATABASE test_db;

-- Show databases again
SHOW DATABASES;
```

You'll now see `test_db` in the list.

```sql
-- Use the database
USE test_db;

-- Create a table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Breaking down the table structure:**
- `id INT AUTO_INCREMENT PRIMARY KEY`
  - `INT` — Integer data type
  - `AUTO_INCREMENT` — Automatically assigns next number (1, 2, 3...)
  - `PRIMARY KEY` — Unique identifier for each row
  
- `username VARCHAR(50) NOT NULL`
  - `VARCHAR(50)` — Variable-length string (max 50 characters)
  - `NOT NULL` — This field is required
  
- `email VARCHAR(100) NOT NULL`
  - Same as username but longer
  
- `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP`
  - `TIMESTAMP` — Date + time
  - `DEFAULT CURRENT_TIMESTAMP` — Auto-fills with current time

```sql
-- Show table structure
DESCRIBE users;
```

**Output:**
```
+------------+--------------+------+-----+-------------------+
| Field      | Type         | Null | Key | Default           |
+------------+--------------+------+-----+-------------------+
| id         | int          | NO   | PRI | NULL              |
| username   | varchar(50)  | NO   |     | NULL              |
| email      | varchar(100) | NO   |     | NULL              |
| created_at | timestamp    | YES  |     | CURRENT_TIMESTAMP |
+------------+--------------+------+-----+-------------------+
```

### **CRUD Operations (Create, Read, Update, Delete):**

#### **CREATE (Insert Data):**

```sql
-- Insert a user
INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com');

-- Insert multiple users
INSERT INTO users (username, email) 
VALUES 
    ('jane_smith', 'jane@example.com'),
    ('bob_jones', 'bob@example.com');
```

#### **READ (Retrieve Data):**

```sql
-- Get all users
SELECT * FROM users;

-- Get specific columns
SELECT username, email FROM users;

-- Filter results
SELECT * FROM users WHERE username = 'john_doe';

-- Sort results
SELECT * FROM users ORDER BY created_at DESC;

-- Limit results
SELECT * FROM users LIMIT 2;
```

#### **UPDATE (Modify Data):**

```sql
-- Update a user's email
UPDATE users 
SET email = 'newemail@example.com' 
WHERE username = 'john_doe';

-- Update multiple fields
UPDATE users 
SET username = 'johnny', email = 'johnny@example.com' 
WHERE id = 1;
```

**CRITICAL:** Always use `WHERE` clause! Without it, ALL rows get updated.

#### **DELETE (Remove Data):**

```sql
-- Delete specific user
DELETE FROM users WHERE id = 1;

-- Delete users matching criteria
DELETE FROM users WHERE created_at < '2025-01-01';
```

**CRITICAL:** Always use `WHERE` clause! Without it, ALL rows get deleted.

```sql
-- Delete all users (DANGEROUS)
DELETE FROM users;  -- This deletes EVERYTHING

-- Drop entire table
DROP TABLE users;   -- Table no longer exists
```

### **Understanding MySQL Users & Permissions:**

```sql
-- Create a database for WordPress
CREATE DATABASE wordpress_dev;

-- Create a user
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'strong_password_here';

-- Grant privileges
GRANT ALL PRIVILEGES ON wordpress_dev.* TO 'wpuser'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

-- Exit MySQL
EXIT;
```

**Breaking down the grant:**
- `GRANT ALL PRIVILEGES` — Give all permissions
- `ON wordpress_dev.*` — On database `wordpress_dev`, all tables (*)
- `TO 'wpuser'@'localhost'` — To user `wpuser` connecting from localhost

**Why not use root?**
- **Security:** If WordPress gets hacked, attacker doesn't have root MySQL access
- **Best practice:** Each application has its own database user

### **Testing the New User:**

```bash
# Log in as wpuser
mysql -u wpuser -p

# Enter password when prompted
```

```sql
-- Show accessible databases
SHOW DATABASES;
```

You should ONLY see:
- `information_schema`
- `wordpress_dev`

You won't see `mysql` or other databases (good security).

### **📚 MySQL Learning Resources:**

1. **[MySQL Official Documentation](https://dev.mysql.com/doc/)** — Comprehensive
2. **[SQLBolt Interactive Tutorial](https://sqlbolt.com/)** — Learn by doing
3. **[W3Schools SQL Tutorial](https://www.w3schools.com/sql/)** — Quick reference
4. **[Use The Index, Luke!](https://use-the-index-luke.com/)** — Advanced optimization

### **Hands-On Exercise:**

Create a table that mimics WordPress posts:

```sql
USE test_db;

CREATE TABLE posts (
    ID BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    post_title VARCHAR(200) NOT NULL,
    post_content TEXT,
    post_author BIGINT UNSIGNED NOT NULL,
    post_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    post_status VARCHAR(20) DEFAULT 'draft',
    INDEX (post_author),
    INDEX (post_status)
);

-- Insert sample posts
INSERT INTO posts (post_title, post_content, post_author, post_status) 
VALUES 
    ('My First Post', 'This is the content of my first post.', 1, 'publish'),
    ('Draft Post', 'This is a draft.', 1, 'draft'),
    ('Another Post', 'More content here.', 2, 'publish');

-- Query published posts
SELECT post_title, post_date 
FROM posts 
WHERE post_status = 'publish' 
ORDER BY post_date DESC;
```

---

## 🔐 Part 6: Linux File Permissions (Security Foundation)

### **Why Permissions Matter:**

In Linux, **everything is a file**:
- Directories are files
- Devices are files (/dev/sda)
- Processes can access files

**Without permissions:** Any user could:
- Delete system files
- Read other users' private data
- Execute malicious code

### **Understanding File Ownership:**

Every file has:
1. **Owner** — User who owns the file
2. **Group** — Group that owns the file
3. **Others** — Everyone else

### **Viewing Permissions:**

```bash
ls -la /var/www/html/
```

**Output:**
```
drwxr-xr-x 2 www-data www-data 4096 Jan 28 10:00 .
drwxr-xr-x 3 root     root     4096 Jan 28 09:00 ..
-rw-r--r-- 1 www-data www-data  291 Jan 28 10:05 index.html
```

**Breaking down the first line:**

```
d rwx r-x r-x  2  www-data  www-data  4096  Jan 28 10:00  .
│ │   │   │    │     │         │        │       │         │
│ │   │   │    │     │         │        │       │         └─ Filename
│ │   │   │    │     │         │        │       └─ Date modified
│ │   │   │    │     │         │        └─ File size
│ │   │   │    │     │         └─ Group owner
│ │   │   │    │     └─ File owner
│ │   │   │    └─ Number of hard links
│ │   │   └─ Others permissions
│ │   └─ Group permissions
│ └─ Owner permissions
└─ File type (d = directory, - = regular file)
```

### **Understanding Permission Notation:**

```
rwx rwx rwx
│   │   │
│   │   └─ Others (everyone else)
│   └─ Group
└─ Owner
```

**What each letter means:**
- `r` = **Read** (can view file content)
- `w` = **Write** (can modify file)
- `x` = **Execute** (can run file as program)

**For directories:**
- `r` = Can list files in directory
- `w` = Can create/delete files in directory
- `x` = Can enter directory (cd into it)

### **Numeric Permissions (Critical to Understand):**

```
r = 4
w = 2
x = 1
```

**Examples:**
```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

**Common permission combinations:**

```
755 = rwxr-xr-x
      Owner: read+write+execute (7)
      Group: read+execute (5)
      Others: read+execute (5)

644 = rw-r--r--
      Owner: read+write (6)
      Group: read (4)
      Others: read (4)

600 = rw-------
      Owner: read+write (6)
      Group: none (0)
      Others: none (0)

777 = rwxrwxrwx (DANGEROUS - everyone can do everything)
```

### **WordPress Specific Permissions:**

```bash
# Correct ownership (www-data = Apache user)
sudo chown -R www-data:www-data /var/www/html/

# Directories: 755
sudo find /var/www/html/ -type d -exec chmod 755 {} \;

# Files: 644
sudo find /var/www/html/ -type f -exec chmod 644 {} \;

# wp-config.php: 600 (only owner can read/write)
sudo chmod 600 /var/www/html/wordpress/wp-config.php
```

**Breaking down `chown`:**
```bash
sudo chown -R www-data:www-data /var/www/html/
│     │     │  │         │        └─ Path
│     │     │  │         └─ Group (www-data)
│     │     │  └─ Owner (www-data)
│     │     └─ Recursive (include subdirectories)
│     └─ Change ownership
└─ Run as superuser
```

**Why www-data?**
- Apache runs as user `www-data`
- WordPress needs to write files (uploads, theme changes)
- If files owned by `root`, Apache can't write → errors

### **Common Permission Issues in WordPress:**

**Problem:** "Can't upload images"

```bash
# Check uploads directory
ls -la /var/www/html/wordpress/wp-content/uploads/

# If owned by root or wrong user:
sudo chown -R www-data:www-data /var/www/html/wordpress/wp-content/uploads/
sudo chmod -R 755 /var/www/html/wordpress/wp-content/uploads/
```

**Problem:** "Can't install plugins"

```bash
# Check wp-content directory
sudo chown -R www-data:www-data /var/www/html/wordpress/wp-content/
```

**Problem:** "Can't update WordPress"

WordPress needs write access to its own directory during updates.

### **Security Best Practices:**

```bash
# Never do this:
chmod 777 /var/www/html/  # Everyone can do everything (hackable)

# Do this:
chmod 755 /var/www/html/  # Owner full, others read-only

# Sensitive files:
chmod 600 wp-config.php   # Only owner can read
```

### **📚 File Permissions Resources:**

1. **[Linux File Permissions Explained](https://www.linux.com/training-tutorials/understanding-linux-file-permissions/)** — Comprehensive guide
2. **[Digital Ocean Permissions Tutorial](https://www.digitalocean.com/community/tutorials/linux-permissions-basics-and-how-to-use-umask-on-a-vps)** — Practical examples

### **Hands-On Exercise:**

```bash
# Create test directory
mkdir ~/permission-test
cd ~/permission-test

# Create test file
echo "Secret data" > secret.txt

# View permissions
ls -l secret.txt

# Remove all permissions
chmod 000 secret.txt

# Try to read it
cat secret.txt  # Permission denied

# Add read permission for owner
chmod 400 secret.txt

# Now you can read it
cat secret.txt

# Try to write to it
echo "More data" >> secret.txt  # Permission denied

# Add write permission
chmod 600 secret.txt

# Now you can write
echo "More data" >> secret.txt
cat secret.txt
```

---

## 🎯 Phase 1 Summary & Checkpoint

Before moving to Phase 2, you should be able to:

### **✅ Can you answer these?**

1. **What happens when you type a URL in the browser?**
2. **What is the difference between Apache and PHP?**
3. **Why does WordPress need MySQL?**
4. **What does chmod 755 mean?**
5. **What's the difference between GET and POST requests?**
6. **Why can't root user run Apache?**
7. **What's in an HTTP response header?**
8. **How does PHP connect to MySQL?**

### **✅ Can you do these?**

```bash
# 1. Install LAMP stack from scratch
# 2. Create a PHP file that connects to MySQL
# 3. Fix file permission errors
# 4. Read Apache logs to debug 404 errors
# 5. Create a MySQL user with limited permissions
```

---

## 📝 Phase 1 Final Exercise

**Task:** Create a simple PHP application that:
1. Connects to MySQL
2. Displays a list of blog posts from database
3. Allows adding new posts via HTML form

**Create this at `/var/www/html/blog-app/`**
