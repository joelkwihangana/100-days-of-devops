# Claude Code: Zero to Expert Guide
**Your DevOps Engineering Companion**

---

## Table of Contents
1. [What is Claude Code? (The "Why" Before the "What")](#what-is-claude-code)
2. [The Problem Claude Code Solves](#the-problem-it-solves)
3. [How Claude Code Actually Works](#how-it-works)
4. [Installation on Ubuntu (The Right Way)](#installation)
5. [Core Concepts You Must Understand](#core-concepts)
6. [Your First Session (Learning By Doing)](#first-session)
7. [Advanced Workflows for DevOps](#devops-workflows)
8. [MCP Servers: Extending Claude Code's Abilities](#mcp-servers)
9. [Integration with VS Code](#vscode-integration)
10. [Real Production Scenarios](#production-scenarios)
11. [How to Debug When Things Break](#debugging)
12. [Mental Models for Effective Use](#mental-models)
13. [Cost & Subscription Management](#cost-management)
14. [Comparison with Other Tools](#comparison)
15. [Best Practices & Pitfalls to Avoid](#best-practices)

---

## What is Claude Code?

### The First Principles Answer

Claude Code is an **agentic coding tool** - not just a chatbot that writes code snippets, but an autonomous agent that can:
- Read your entire codebase
- Edit multiple files across your project
- Run commands in your terminal
- Execute tests and verify changes
- Interact with external services via MCP (Model Context Protocol)
- Handle git workflows (commits, branches, PRs)

**Key distinction**: Traditional AI coding assistants (like GitHub Copilot) **suggest** code. Claude Code **executes** code changes autonomously.

### Why Does Claude Code Exist?

**The problem**: As a DevOps engineer, you spend 60-70% of your time on:
- Boilerplate code (Dockerfiles, CI/CD configs, infrastructure as code)
- Reading documentation and figuring out API changes
- Debugging obscure errors across microservices
- Writing tests for untested code
- Updating dependencies and fixing breaking changes

**The solution**: Claude Code acts as a pair programmer who:
- Knows your entire codebase context
- Can read official documentation in real-time (via MCP)
- Executes routine tasks while you focus on architecture
- Never gets tired of writing tests or fixing linter errors

---

## The Problem It Solves

### Before Claude Code (Your Current Workflow)
```
1. You get a bug report: "API endpoint /users fails with 500 error"
2. You search through logs → find stack trace
3. You grep codebase to find where error originates
4. You read 5 different files to understand the flow
5. You Google the error message
6. You read Stack Overflow
7. You make changes to 3 files
8. You run tests manually
9. You write a commit message
10. You create a PR

Time: 2-3 hours
```

### With Claude Code
```
You: "API endpoint /users is failing with 500 error. Trace the issue, fix it, write tests, and create a PR"

Claude Code:
1. Searches codebase for /users endpoint
2. Finds error in database query (missing null check)
3. Reads your test patterns, writes matching tests
4. Verifies fix works locally
5. Stages changes, writes descriptive commit
6. Opens PR with context

Time: 15-20 minutes (mostly verification)
```

**What you saved**: Not just time, but mental context switching. You stayed in "architect mode" instead of dropping into "debugger mode."

---

## How It Works

### Architecture (Mental Model)

```
┌─────────────────────────────────────────────┐
│          You (Natural Language)             │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│         Claude Code Agent                   │
│  (Sonnet 4.5 model running locally)        │
│                                             │
│  - Understands your project context         │
│  - Plans multi-step tasks                   │
│  - Makes autonomous decisions               │
└──────────┬──────────────────────────────────┘
           │
           ├──► Built-in Tools:
           │    • File editing (str_replace)
           │    • Bash commands
           │    • File viewing
           │    • Git operations
           │
           └──► MCP Servers (external integrations):
                • Web search (real-time docs)
                • GitHub API
                • Slack
                • Google Drive
                • Custom tools you build
```

### How Claude Code Sees Your Project

When you run `claude` in a project directory:

1. **Initial Scan**: Reads your CLAUDE.md file (if exists) for project-specific context
2. **Context Building**: Analyzes:
   - File structure (respects .gitignore)
   - Package dependencies (package.json, requirements.txt, go.mod, etc.)
   - Configuration files (docker-compose.yml, .env.example, etc.)
   - Recent git history
3. **Tool Selection**: Decides which tools it needs for your request
4. **Execution Loop**:
   ```
   Plan → Execute → Verify → Adjust → Repeat
   ```

**Critical insight**: Claude Code doesn't just write code and dump it. It verifies its changes work before considering the task complete.

---

## Installation

### Prerequisites Check

Before installing, verify your system:

```bash
# Check Ubuntu version (20.04+ required)
lsb_release -a

# Check you have curl
which curl

# Check git is installed
git --version
```

### The Right Installation Method

**DO NOT use npm installation** - it's deprecated and causes permission headaches.

**Use the native installer** (recommended):

```bash
# One-line install (handles everything)
curl -fsSL https://claude.ai/install.sh | bash
```

**What this does**:
1. Downloads the correct binary for your system (x86_64 or ARM)
2. Installs to `~/.claude/local/claude`
3. Adds to your PATH automatically
4. Sets up auto-updates
5. No Node.js dependency required

### Verify Installation

```bash
# Check version
claude --version
# Should show: 2.0.x (Claude Code)

# Run diagnostic
claude doctor
```

**Expected output**:
```
Version: 2.0.30 or higher
Config install method: native
Auto-updates: enabled
Search: OK (bundled)
```

### If Installation Fails

**Problem**: `command not found: claude`

**Why**: PATH not updated in current shell session

**Fix**:
```bash
# Reload shell config
source ~/.bashrc  # or ~/.zshrc

# Verify PATH contains claude
echo $PATH | grep -o '[^:]*claude[^:]*'
```

**Problem**: `Permission denied`

**Why**: Binary not executable

**Fix**:
```bash
chmod +x ~/.claude/local/claude
```

---

## Core Concepts

### 1. Authentication (How You Pay)

Claude Code requires authentication. You have three options:

#### Option A: Claude Pro/Max Subscription (Recommended for Learning)
- **Cost**: $20/month (Pro) or $100/month (Max)
- **Best for**: Personal use, learning, side projects
- **Setup**:
  ```bash
  claude
  # On first run, browser opens → sign in with Claude.ai account
  ```

#### Option B: Anthropic API Key (Pay-per-use)
- **Cost**: ~$3 per million input tokens, ~$15 per million output tokens
- **Best for**: Production environments, team use
- **Setup**:
  ```bash
  # Get API key from console.anthropic.com
  export ANTHROPIC_API_KEY="sk-ant-..."
  
  # Make permanent
  echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc
  ```

#### Option C: Cloud Providers (AWS Bedrock, GCP Vertex AI, Azure)
- **Best for**: Enterprise with existing cloud contracts
- **Setup varies** - see official docs

**Which should you use?**
- **Learning DevOps (you)**: Claude Pro ($20/month) - unlimited code generation, worth it
- **Production team project**: API key with usage limits
- **Enterprise**: Cloud provider integration

### 2. Scopes (Where Claude Code Can See Your Server)

**Think of scopes as permission levels:**

```
Global Scope (~/.claude.json)
└─ Available in ALL projects
   └─ Use for: General tools (linters, formatters)

Project Scope (/your-project/.mcp.json)
└─ Available ONLY in this project
   └─ Use for: Project-specific databases, APIs

Local Scope (current directory only)
└─ Temporary, not saved
   └─ Use for: Quick tests, one-off tasks
```

**Real example**:
```bash
# Global MCP server (available everywhere)
claude mcp add --global brave-search -- npx -y @modelcontextprotocol/server-brave-search

# Project MCP server (only in current directory's projects)
cd ~/projects/my-api
claude mcp add postgres-dev -- npx -y @modelcontextprotocol/server-postgres
```

### 3. The CLAUDE.md File (Teaching Claude About Your Project)

**What it is**: A markdown file in your project root that Claude reads on startup.

**Why it matters**: Claude Code is smart, but it doesn't know your project's conventions, architecture decisions, or "gotchas." CLAUDE.md is how you tell it.

**Example for a FastAPI + React + Docker project**:

```markdown
# Project: DevOps Learning Platform

## Architecture
- Backend: FastAPI (Python 3.11) in `/backend`
- Frontend: React 18 + Tailwind in `/frontend`
- Database: PostgreSQL 15 (via Docker Compose)
- Deployment: Docker containers on Hostinger VPS

## Development Workflow
1. Never edit `main` branch directly
2. Always create feature branches: `feature/description`
3. Run tests before committing: `pytest` (backend), `npm test` (frontend)
4. Use conventional commits: `feat:`, `fix:`, `chore:`

## Code Patterns
- Backend: All API routes in `/backend/app/routes/`
- Frontend: Components in `/frontend/src/components/`
- Shared types: Keep TypeScript interfaces in `/frontend/src/types/`
- Environment vars: NEVER commit `.env`, use `.env.example` as template

## Testing
- Backend tests: `pytest tests/` (must pass before commit)
- Frontend tests: `npm run test` (must pass before commit)
- Integration tests: `docker-compose -f docker-compose.test.yml up`

## Common Issues
- If Docker build fails, check `docker-compose.yml` port conflicts
- PostgreSQL migrations: Use Alembic, never manual SQL
- React build errors: Clear `node_modules` and reinstall

## Tools Available
- Linter: `ruff` for Python, `eslint` for JavaScript
- Formatter: `black` for Python, `prettier` for JavaScript
- Type checker: `mypy` for Python, `tsc` for TypeScript

## Deployment
- Staging: Push to `develop` branch → Auto-deploys to staging.example.com
- Production: Create PR to `main` → Manual approval → Deploys to example.com
```

**How to use it**:
```bash
# Create CLAUDE.md in project root
cd ~/projects/my-app
touch CLAUDE.md
# Edit with your project info

# Claude reads it automatically on startup
claude
```

---

## First Session

### Scenario: You're starting a new FastAPI project

```bash
# Create project directory
mkdir ~/projects/fastapi-learning
cd ~/projects/fastapi-learning

# Start Claude Code
claude
```

**Your first conversation**:

```
You: I'm starting a FastAPI project for learning DevOps. 
     Create a basic project structure with:
     - FastAPI app with health check endpoint
     - Docker setup
     - Requirements.txt
     - README with setup instructions
     
     Explain each decision you make.
```

**What Claude Code does**:

1. **Plans the structure**:
   ```
   fastapi-learning/
   ├── app/
   │   ├── __init__.py
   │   ├── main.py
   │   └── routes/
   │       └── health.py
   ├── Dockerfile
   ├── docker-compose.yml
   ├── requirements.txt
   ├── .dockerignore
   ├── .gitignore
   └── README.md
   ```

2. **Creates each file** with explanations:
   - Shows you the content
   - Explains WHY each configuration option exists
   - Asks for approval before writing

3. **Verifies it works**:
   ```bash
   docker-compose build
   docker-compose up -d
   curl http://localhost:8000/health
   ```

4. **Stages changes for git**:
   ```bash
   git add .
   git commit -m "feat: initial FastAPI project structure with Docker"
   ```

### Key Commands During Session

```bash
# See available slash commands
/help

# View current context (what Claude knows)
/context

# Clear conversation, keep files
/clear

# Check MCP servers
/mcp

# Run terminal command (if you want control)
/terminal docker ps

# Exit Claude Code
/exit
```

---

## DevOps Workflows

### Workflow 1: Dockerizing a FastAPI App

**Scenario**: You have a working FastAPI app, need production-ready Docker setup.

```
You: I have a FastAPI app in app/main.py. Create a multi-stage Dockerfile 
     optimized for production with:
     - Small image size
     - Non-root user
     - Health checks
     - Poetry for dependencies
     
     Then create docker-compose.yml for local development with hot reload.
```

**What Claude Code delivers**:

1. **Multi-stage Dockerfile**:
   - Build stage (installs dependencies)
   - Runtime stage (copies only what's needed)
   - Health check configuration
   - Non-root user setup

2. **docker-compose.yml** with:
   - Volume mounts for hot reload
   - Environment variables from .env
   - PostgreSQL service
   - Network configuration

3. **Verification**:
   - Builds image
   - Runs containers
   - Checks health endpoint
   - Tests hot reload

**You learn**: Not just the files, but WHY each Docker layer matters for production.

### Workflow 2: Setting Up CI/CD Pipeline

```
You: Create a GitHub Actions workflow that:
     - Runs on push to main and PRs
     - Lints Python code with ruff
     - Runs pytest
     - Builds Docker image
     - Pushes to Docker Hub if tests pass (main branch only)
     
     Add necessary secrets documentation.
```

**Claude Code output**:
- `.github/workflows/ci-cd.yml` with full pipeline
- README section documenting required GitHub secrets
- `.github/workflows/pr-checks.yml` for PR-only checks

**Mental model building**: You see how professional CI/CD pipelines handle different branch strategies.

### Workflow 3: Debugging Production Issues

```
You: Our /api/users endpoint is returning 500 errors in production. 
     Docker logs show "connection pool exhausted".
     
     Find the issue, explain the root cause, and fix it with proper 
     connection pooling.
```

**Claude Code process**:
1. Reads database connection code
2. Identifies missing connection pool configuration
3. Explains why default pool size (10) is insufficient
4. Shows you how to configure SQLAlchemy pool
5. Adds monitoring for future debugging
6. Updates CLAUDE.md with this lesson

---

## MCP Servers

### What Are MCP Servers?

**Mental model**: Think of MCP servers as "plugins" that give Claude Code superpowers.

**Without MCP**: Claude knows only what's in your codebase (and its training data up to Jan 2025).

**With MCP**: Claude can:
- Search the web for current documentation
- Access your GitHub repos
- Query your databases
- Read your Google Drive
- Interact with Slack
- Use ANY API you connect

### How MCP Works (Architecture)

```
┌─────────────────────────────────────────┐
│         Claude Code (MCP Host)          │
│  "I need to check React 18.3 docs"      │
└──────────────┬──────────────────────────┘
               │
               │ (Calls tool via MCP)
               ▼
┌─────────────────────────────────────────┐
│    MCP Server (e.g., Context7)          │
│  "Fetching React 18.3.0 docs..."        │
│  Returns: Latest API documentation      │
└──────────────────────────────────────────┘
```

### Essential MCP Servers for DevOps

#### 1. Web Search (Brave Search)

**Why**: Get current documentation, not outdated training data.

**Installation**:
```bash
# Global scope (available everywhere)
claude mcp add --global brave-search \
  -e BRAVE_API_KEY="your-key" \
  -- npx -y @modelcontextprotocol/server-brave-search
```

**Get API key**: https://brave.com/search/api/

**Use case**:
```
You: What's the correct way to use React.lazy() with error boundaries 
     in React 18.3? Search official docs.
```

#### 2. Context7 (Real-time Documentation)

**Why**: Version-specific docs from official sources (npm, PyPI, crates.io).

**Installation**:
```bash
claude mcp add --global context7 \
  -e CONTEXT7_API_KEY="your-key" \
  -- npx -y @context7/mcp
```

**Get API key**: https://context7.com

**Use case**:
```
You: Show me the FastAPI 0.115.0 documentation for WebSocket handling
```

#### 3. GitHub Integration

**Why**: Create issues, review PRs, manage repositories.

**Installation**:
```bash
# Project scope (specific repo)
claude mcp add github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN="ghp_..." \
  -- npx -y @modelcontextprotocol/server-github
```

**Use case**:
```
You: Review the open PRs in this repo and summarize what needs review
```

#### 4. PostgreSQL (Database Access)

**Why**: Query databases, generate migrations, analyze schema.

**Installation**:
```bash
# Project scope (don't expose globally!)
cd ~/projects/my-app
claude mcp add postgres \
  -e DATABASE_URL="postgresql://user:pass@localhost:5432/db" \
  -- npx -y @modelcontextprotocol/server-postgres
```

**Use case**:
```
You: Show me all users created in the last 7 days and their activity count
```

### Managing MCP Servers

```bash
# List all configured servers
claude mcp list

# Check server status
/mcp

# Remove a server
claude mcp remove server-name

# Test server connection
You: /mcp test brave-search
```

### Building Your Own MCP Server

**When to build custom MCP**:
- You have internal APIs Claude should access
- You want to automate repetitive tasks specific to your company
- You need to integrate with proprietary tools

**Simple example** (Python MCP server for custom API):

```python
# my_api_mcp.py
import asyncio
from mcp.server import Server
from mcp.types import Tool, TextContent
import httpx

server = Server("my-api-mcp")

@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="fetch_user",
            description="Fetch user data from internal API",
            inputSchema={
                "type": "object",
                "properties": {
                    "user_id": {"type": "string"}
                },
                "required": ["user_id"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "fetch_user":
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"https://api.mycompany.com/users/{arguments['user_id']}"
            )
            return [TextContent(text=response.text)]

async def main():
    from mcp.server.stdio import stdio_server
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream)

if __name__ == "__main__":
    asyncio.run(main())
```

**Register it**:
```bash
claude mcp add my-api -- python ~/mcp-servers/my_api_mcp.py
```

---

## VS Code Integration

### Why Use Claude Code in VS Code?

**Terminal Claude Code**: Great for autonomous tasks (scaffolding, refactoring, testing)

**VS Code Extension**: Better for interactive development (inline suggestions, quick fixes)

### Installation

1. Install VS Code extension: "Claude Code"
2. Authenticate (same as terminal)
3. Configure keyboard shortcuts

### Key Features

**1. Inline Chat** (⌘ + I):
```
Select code → Cmd+I → "Add error handling"
```

**2. Terminal Integration**:
- Run `claude` in VS Code terminal
- Same experience as standalone terminal

**3. Diff View**:
- See proposed changes before accepting
- Accept/reject individual hunks

### When to Use Which?

| Task | Terminal Claude | VS Code Extension |
|------|----------------|-------------------|
| Scaffolding new project | ✅ Better | ❌ |
| Writing tests | ✅ Better | ❌ |
| Quick inline fix | ❌ | ✅ Better |
| Refactoring | ✅ Better | ❌ |
| Learning by reading code | ❌ | ✅ Better |
| CI/CD pipeline setup | ✅ Better | ❌ |

---

## Production Scenarios

### Scenario 1: Migrating Legacy Code to Docker

**Context**: You have a Python 2.7 app running on bare metal. Need to containerize for Kubernetes.

**Conversation**:
```
You: I have a legacy Python 2.7 Flask app in /app. It uses:
     - MySQL database
     - Redis for caching
     - Celery for background tasks
     
     Create a migration plan to:
     1. Upgrade to Python 3.11
     2. Dockerize all services
     3. Create docker-compose for local dev
     4. Create Kubernetes manifests for production
     
     Show me the breaking changes and migration path for each step.
```

**What you get**:
1. **Analysis report** of breaking changes Python 2→3
2. **Migration script** to update syntax
3. **Dockerfiles** for each service (app, celery worker)
4. **docker-compose.yml** with all services
5. **k8s/** directory with:
   - Deployments
   - Services
   - ConfigMaps
   - Secrets (templates)
6. **Migration guide** with rollback plan

### Scenario 2: Implementing Observability

```
You: Add comprehensive observability to this FastAPI app:
     - Prometheus metrics (request count, latency, errors)
     - Structured logging with correlation IDs
     - OpenTelemetry tracing
     - Grafana dashboard configuration
     
     Integrate with existing Docker Compose setup.
```

**Output**:
- Updated `app/main.py` with Prometheus middleware
- `app/logging.py` with structured logging
- `app/tracing.py` with OpenTelemetry config
- `docker-compose.yml` adds Prometheus, Grafana, Jaeger
- `grafana/dashboards/` with API dashboard JSON
- `prometheus/` with scrape configs

### Scenario 3: Security Hardening

```
You: Perform a security audit of this Docker setup and fix issues:
     - Check for running as root
     - Scan for exposed secrets
     - Verify minimal attack surface
     - Add security scanning to CI/CD
     
     Document each finding and fix.
```

**Deliverables**:
- Audit report (markdown)
- Updated Dockerfile (non-root user, minimal base image)
- `.trivyignore` for false positives
- GitHub Actions workflow with Trivy scanning
- Updated README with security best practices

---

## Debugging

### When Claude Code Makes Mistakes

**It will happen.** Claude is powerful but not perfect. Here's how to debug:

#### 1. Check the Diff Before Accepting

```bash
# Always review changes
You: Show me a diff of what you changed

# Or use git
git diff
```

#### 2. Verify Generated Code Runs

```bash
# Don't trust, verify
You: Run the tests to confirm this works

# Manual verification
docker-compose up -d
curl http://localhost:8000/api/test
```

#### 3. Ask for Explanation

```bash
You: Why did you choose this approach over [alternative]?
```

#### 4. Course Correct

```bash
You: That implementation has a race condition. Fix it using proper locking.
```

### Common Issues & Fixes

#### Issue: "Claude keeps modifying the same file incorrectly"

**Why**: Context got confused, needs reset.

**Fix**:
```bash
/clear
You: Let me give you fresh context. [Re-explain the task]
```

#### Issue: "Generated code doesn't match our style guide"

**Why**: No style guide in CLAUDE.md

**Fix**:
```bash
# Add to CLAUDE.md
## Code Style
- Python: Follow Black + Ruff
- TypeScript: Airbnb style guide
- Max line length: 88 characters
- Use type hints everywhere
```

#### Issue: "Claude suggests deprecated packages"

**Why**: Training data is older (Jan 2025 cutoff)

**Fix**:
```bash
# Use MCP server for current info
You: Search for the latest stable version of [package] and use that
```

#### Issue: "Proposed changes break tests"

**Why**: Didn't run tests before presenting

**Fix**:
```bash
You: Before showing me changes, always run the test suite and 
     fix any failures.
```

---

## Mental Models

### Model 1: Claude Code as a Junior Engineer

**Think of Claude as**: A smart junior engineer who:
- ✅ Knows syntax and patterns
- ✅ Can implement features from clear specs
- ✅ Writes tests when told to
- ❌ Doesn't know your business logic
- ❌ Needs architectural guidance
- ❌ Can miss edge cases

**Your role**: Senior engineer providing:
- Clear requirements
- Architectural decisions
- Code review
- Edge case identification

### Model 2: The Iteration Loop

```
Your Request (High-level)
    ↓
Claude Plans Approach (Shows you)
    ↓
You Approve/Adjust
    ↓
Claude Executes
    ↓
Verifies (Runs tests)
    ↓
Shows Results
    ↓
You Review & Accept/Reject
    ↓
[Repeat if needed]
```

### Model 3: Context is King

**Claude Code works best when you provide**:
- **Historical context**: "This project uses FastAPI because..."
- **Constraint context**: "We can't use X library because of Y license"
- **Pattern context**: "All API responses follow this format..."
- **Error context**: "Here's the full stack trace..."

**Bad request** (no context):
```
You: Fix the API
```

**Good request** (rich context):
```
You: The /api/users endpoint returns 500 when querying users created 
     before 2020. Stack trace shows NoneType error in serialization. 
     We use Pydantic models. Find the bug and add validation to prevent 
     this in the future.
```

---

## Cost Management

### Understanding Usage

**Claude Pro/Max**: Flat monthly fee, unlimited usage (within rate limits)

**API Key**: Pay per token
- Input: ~$3 per million tokens (~750K words)
- Output: ~$15 per million tokens

### Estimating Costs

**Typical request sizes**:
- Simple task (lint file): ~1K tokens input, ~500 tokens output = $0.01
- Medium task (write tests): ~5K tokens input, ~3K tokens output = $0.06
- Large task (refactor module): ~20K tokens input, ~10K tokens output = $0.21

**Monthly estimation**:
- Light use (5 requests/day): ~$10-15/month
- Medium use (20 requests/day): ~$40-60/month
- Heavy use (50+ requests/day): $100+/month → **Get Pro subscription instead**

### Cost Optimization

**1. Use CLAUDE.md** to reduce repeated context:
```markdown
# Instead of explaining your setup every time
## Tech Stack
- React 18.2, TypeScript 5.1, Tailwind 3.4
- FastAPI 0.115, Python 3.11
- PostgreSQL 15, Redis 7
```

**2. Be specific** (fewer iteration loops):
```
❌ "Make it better"  → Vague, requires multiple rounds
✅ "Add error handling for network timeouts" → One round
```

**3. Use local tools first**:
```
❌ You: Run prettier on all files
✅ You: [Run prettier locally] → Then ask Claude to fix remaining issues
```

**4. Check context usage**:
```bash
/context
# Shows how many tokens used
```

---

## Comparison

### Claude Code vs. GitHub Copilot

| Feature | Claude Code | Copilot |
|---------|-------------|---------|
| **Model** | Claude Sonnet 4.5 | GPT-4 Turbo |
| **Autonomy** | High (edits files, runs commands) | Low (suggestions only) |
| **Context** | Entire codebase | Current file + nearby files |
| **Verification** | Runs tests automatically | No execution |
| **Best for** | Refactoring, scaffolding, DevOps | Inline autocomplete |
| **Learning curve** | Steeper | Gentler |
| **Cost** | $20-100/mo subscription | $10-19/mo |

### Claude Code vs. Cursor

| Feature | Claude Code | Cursor |
|---------|-------------|---------|
| **Interface** | Terminal + VS Code plugin | Full IDE |
| **Model** | Claude Sonnet 4.5 | Multiple (Claude, GPT-4) |
| **Git integration** | Native | Via extensions |
| **Codebase indexing** | Via MCP | Built-in |
| **Best for** | DevOps workflows, automation | Full-stack dev |

### Claude Code vs. Windsurf

| Feature | Claude Code | Windsurf |
|---------|-------------|---------|
| **Provider** | Anthropic | Codeium |
| **Focus** | General coding | AI pair programming |
| **Extensibility** | MCP servers | Limited |
| **Documentation** | Excellent | Growing |

**When to use Claude Code**: DevOps automation, infrastructure as code, testing, refactoring at scale

**When to use alternatives**: Real-time autocomplete (Copilot), integrated IDE experience (Cursor)

---

## Best Practices

### 1. Start Small, Build Trust

```bash
# First week: Simple tasks
You: Add logging to this function
You: Write tests for this class
You: Fix linter errors

# Second week: Medium tasks
You: Refactor this module to use async/await
You: Add authentication middleware

# Third week+: Complex tasks
You: Implement caching layer with Redis
You: Set up CI/CD pipeline
```

### 2. Review Everything

```bash
# Never blind-merge Claude's changes
git diff
# Read and understand before accepting
```

### 3. Use CLAUDE.md as Documentation

**Your CLAUDE.md should grow** as you learn project-specific lessons:

```markdown
## Lessons Learned
- 2025-02-15: Database connection pooling config in app/db.py
- 2025-02-10: Docker layer caching requires careful COPY order
- 2025-02-08: Celery tasks must be idempotent (retry logic)
```

### 4. Combine Tools

**The power move**:
```bash
# Use Claude Code for structure
You: Create FastAPI project with auth, CRUD, tests

# Use Copilot for inline refinement
[Edit function logic with Copilot suggestions]

# Back to Claude Code for verification
You: Run all tests and fix any failures
```

### 5. Teach Claude Your Patterns

```bash
# Instead of:
You: Write an API endpoint

# Do:
You: Write an API endpoint following our pattern:
     - Use @router.post decorator
     - Pydantic models for request/response
     - Dependency injection for DB session
     - Raise HTTPException for errors
     - Return 201 for creation, 200 for update
```

---

## Pitfalls to Avoid

### ❌ Don't: Treat Claude Like Google

```bash
❌ You: What is Docker?
# Use actual documentation for learning fundamentals
```

### ❌ Don't: Use for One-Liners

```bash
❌ You: Print hello world
# Faster to just type it yourself
```

### ❌ Don't: Assume it Knows Your Business Logic

```bash
❌ You: Implement the user payment flow
# Too vague, Claude will make assumptions

✅ You: Implement user payment flow:
       1. Validate payment method (Stripe API)
       2. Check user balance
       3. Create transaction record
       4. Update user credits
       5. Send confirmation email (SendGrid)
```

### ❌ Don't: Skip Testing Claude's Code

```bash
❌ git add . && git commit -m "Claude did it"
# Always verify, always test
```

### ❌ Don't: Use Sensitive Production Credentials

```bash
❌ claude mcp add prod-db \
     -e DATABASE_URL="postgresql://admin:real_prod_password@prod.db.com/main"

✅ # Use development/staging credentials only
✅ # Or use role-based access with temporary tokens
```

---

## Next Steps

### Week 1: Get Comfortable
- [ ] Install Claude Code
- [ ] Complete "First Session" tutorial
- [ ] Create CLAUDE.md for one project
- [ ] Run 5 simple tasks (lint, format, write tests)

### Week 2: Build Complexity
- [ ] Install 2-3 MCP servers (Brave Search, Context7)
- [ ] Use Claude to scaffold a full FastAPI + React project
- [ ] Set up Docker Compose with Claude's help
- [ ] Practice git workflows (branch, commit, PR)

### Week 3: Real DevOps Work
- [ ] Implement CI/CD pipeline with GitHub Actions
- [ ] Add monitoring (Prometheus + Grafana)
- [ ] Deploy to your Hostinger VPS
- [ ] Document everything in CLAUDE.md

### Week 4: Advanced Integration
- [ ] Build a custom MCP server for your internal API
- [ ] Use Claude to refactor a legacy project
- [ ] Set up comprehensive testing (unit, integration, e2e)
- [ ] Implement blue-green deployment strategy

---

## Resources

### Official Documentation
- Claude Code Docs: https://code.claude.com/docs
- MCP Documentation: https://modelcontextprotocol.io
- Anthropic API: https://docs.anthropic.com

### Community
- Discord: Claude Developers
- GitHub: https://github.com/anthropics/claude-code
- r/ClaudeAI on Reddit

### Learning Path for DevOps
1. **Docker**: Use Claude to create Dockerfiles, understand multi-stage builds
2. **Kubernetes**: Generate manifests, learn resource limits, ConfigMaps
3. **CI/CD**: Build GitHub Actions workflows, understand pipeline stages
4. **Monitoring**: Implement Prometheus metrics, Grafana dashboards
5. **IaC**: Generate Terraform configs, learn state management

---

## Final Mental Model

**Claude Code is not magic.** It's a tool that:
- ✅ Accelerates routine work (boilerplate, tests, configs)
- ✅ Helps you learn by showing best practices
- ✅ Catches mistakes you might miss (linting, type errors)
- ❌ Doesn't replace architectural thinking
- ❌ Doesn't understand your business context without guidance
- ❌ Can make mistakes (always verify)

**Your growth as a DevOps engineer** comes from:
1. **Using Claude to build** → Learn patterns by seeing them in action
2. **Reviewing Claude's code** → Understand WHY choices were made
3. **Asking "why" constantly** → Force Claude to explain reasoning
4. **Experimenting** → Break things intentionally to learn how they work
5. **Teaching Claude** → Write CLAUDE.md files that crystallize your knowledge

**The ultimate goal**: You should be able to do everything Claude does, but faster with Claude as your assistant.

---
