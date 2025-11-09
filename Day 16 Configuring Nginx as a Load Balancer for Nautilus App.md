# Nginx Fundamentals & Load Balancing - Complete Learning Guide

## Table of Contents
1. [What is Nginx?](#what-is-nginx)
2. [Understanding Ports](#understanding-ports)
3. [Web Servers vs Reverse Proxy](#web-servers-vs-reverse-proxy)
4. [Load Balancing Concepts](#load-balancing-concepts)
5. [Nginx Architecture](#nginx-architecture)
6. [Configuration File Structure](#configuration-file-structure)
7. [Upstream & Proxy Pass](#upstream--proxy-pass)
8. [Common Commands](#common-commands)
9. [Troubleshooting Tips](#troubleshooting-tips)

---

## What is Nginx?

### Definition
**Nginx** (pronounced "Engine-X") is a powerful, open-source software that can function as:
- **Web Server** - Serves static files (HTML, CSS, images)
- **Reverse Proxy** - Forwards client requests to backend servers
- **Load Balancer** - Distributes traffic across multiple servers
- **HTTP Cache** - Stores responses to speed up delivery

### Why Use Nginx?
- **High Performance**: Handles thousands of connections simultaneously
- **Low Memory Usage**: Very efficient with system resources
- **Flexibility**: Can be configured for many different use cases
- **Reliability**: Stable and battle-tested in production environments

### Real-World Analogy
Think of Nginx as a **restaurant host**:
- Customers (clients) come to the restaurant
- The host (Nginx) greets them at the door
- The host directs customers to available tables (backend servers)
- The host ensures no table gets too crowded (load balancing)

---

## Understanding Ports

### What is a Port?
A **port** is like a numbered door on a building (your server). Different services listen on different ports to handle specific types of traffic.

### Port Basics
- Ports are numbered from **0 to 65535**
- Ports 0-1023 are **well-known ports** (require root/admin access)
- Ports 1024-49151 are **registered ports** (commonly used services)
- Ports 49152-65535 are **dynamic/private ports**

### Common Ports You Should Know

| Port | Service | Purpose |
|------|---------|---------|
| **80** | HTTP | Standard web traffic (unencrypted) |
| **443** | HTTPS | Secure web traffic (encrypted) |
| **22** | SSH | Secure remote server access |
| **3306** | MySQL | Database connections |
| **8080** | Alt HTTP | Alternative web port (development) |
| **3004** | Custom | Application-specific (like in our example) |

### Port in Action
```
http://example.com:80        # Explicitly using port 80 (usually omitted)
http://example.com           # Port 80 is assumed
http://example.com:3004      # Must specify non-standard port
```

### Why Ports Matter
- Only **one service** can listen on a port at a time
- If Apache is on port 3004, nothing else can use 3004
- Clients need to know which port to connect to
- Firewalls control which ports are open/closed

---

## Web Servers vs Reverse Proxy

### Web Server Mode
Nginx serves files directly to clients.

```
Client → Nginx (serves file.html) → Client gets file
```

**Use Case**: Serving static websites, images, CSS files

### Reverse Proxy Mode
Nginx forwards requests to backend servers and returns their responses.

```
Client → Nginx → Backend Server (Apache/Tomcat) → Nginx → Client
```

**Use Case**: Protecting backend servers, load balancing, SSL termination

### Key Difference
- **Web Server**: Nginx IS the final destination
- **Reverse Proxy**: Nginx is a MIDDLEMAN between client and real server

### Why Use a Reverse Proxy?
1. **Security**: Backend servers hidden from internet
2. **Load Distribution**: Spread traffic across many servers
3. **SSL Termination**: Handle HTTPS encryption in one place
4. **Caching**: Store responses to reduce backend load
5. **Compression**: Reduce bandwidth usage

---

## Load Balancing Concepts

### What is Load Balancing?
**Load balancing** distributes incoming traffic across multiple servers to:
- Prevent any single server from being overwhelmed
- Improve response time
- Ensure high availability (if one server fails, others continue)

### Real-World Analogy
Imagine a grocery store with multiple checkout lanes:
- If everyone used one lane (one server), there'd be a huge queue
- With multiple lanes (multiple servers), checkout is faster
- If one lane closes (server fails), others handle the customers

### Load Balancing Methods

#### 1. Round Robin (Default)
Requests are distributed **evenly in order**.

```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A (cycles back)
```

**Pros**: Simple, fair distribution  
**Cons**: Doesn't consider server load or capability

#### 2. Least Connections
Sends requests to the server with **fewest active connections**.

```nginx
upstream myapp {
    least_conn;
    server stapp01:3004;
    server stapp02:3004;
}
```

**Use Case**: When requests have varying processing times

#### 3. IP Hash
The same client IP always goes to the **same server**.

```nginx
upstream myapp {
    ip_hash;
    server stapp01:3004;
    server stapp02:3004;
}
```

**Use Case**: When you need session persistence (sticky sessions)

#### 4. Weighted Round Robin
Servers with higher weight get **more requests**.

```nginx
upstream myapp {
    server stapp01:3004 weight=3;  # Gets 3x more traffic
    server stapp02:3004 weight=1;
}
```

**Use Case**: When servers have different capacities

---

## Nginx Architecture

### Process Model
Nginx uses a **master-worker** architecture:

```
Master Process (nginx)
  ├── Worker Process 1 (handles connections)
  ├── Worker Process 2 (handles connections)
  └── Worker Process N (handles connections)
```

### How It Works
1. **Master Process**: Reads config, manages workers, doesn't handle traffic
2. **Worker Processes**: Handle actual client connections (non-blocking)
3. Each worker can handle **thousands** of connections simultaneously

### Key Advantage
Unlike Apache (one process per connection), Nginx uses **event-driven** architecture:
- Uses **less memory**
- Handles **more concurrent connections**
- Better for **high-traffic** scenarios

---

## Configuration File Structure

### Main Configuration File
**Location**: `/etc/nginx/nginx.conf`

### Basic Structure
```nginx
# Global settings
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;

# Events block (connection handling)
events {
    worker_connections 1024;  # Max connections per worker
}

# HTTP block (web traffic settings)
http {
    # Server block (virtual host)
    server {
        listen 80;              # Port to listen on
        server_name example.com; # Domain name
        
        # Location block (URL routing)
        location / {
            proxy_pass http://backend;
        }
    }
}
```

### Configuration Hierarchy
```
nginx.conf (main)
├── events { }        # Connection processing
└── http { }          # HTTP-specific settings
    ├── upstream { }  # Backend server groups
    └── server { }    # Virtual hosts
        └── location { }  # URL routing rules
```

### Directive Types

#### Simple Directive
```nginx
worker_processes 2;  # Directive name + value + semicolon
```

#### Block Directive
```nginx
server {
    # Multiple directives inside
    listen 80;
    server_name example.com;
}
```

### Important Directives Explained

| Directive | Purpose | Example |
|-----------|---------|---------|
| `listen` | Port to listen on | `listen 80;` |
| `server_name` | Domain name(s) | `server_name example.com;` |
| `location` | URL path matching | `location /api { }` |
| `proxy_pass` | Forward to backend | `proxy_pass http://backend;` |
| `root` | Document root | `root /var/www/html;` |

---

## Upstream & Proxy Pass

### Upstream Block
Defines a **group of backend servers** for load balancing.

```nginx
upstream myapp {
    server stapp01:3004;  # Backend server 1
    server stapp02:3004;  # Backend server 2
    server stapp03:3004;  # Backend server 3
}
```

### What Upstream Does
1. Creates a named pool of servers
2. Nginx will distribute requests among them
3. Can include health checks and weights

### Proxy Pass Directive
**Forwards requests** from Nginx to backend servers.

```nginx
location / {
    proxy_pass http://myapp;  # "myapp" refers to upstream block
}
```

### Complete Example
```nginx
http {
    # Define backend server pool
    upstream backend_servers {
        server 172.16.238.10:3004;
        server 172.16.238.11:3004;
        server 172.16.238.12:3004;
    }
    
    # Define how to handle incoming requests
    server {
        listen 80;
        server_name _;  # Accept any hostname
        
        location / {
            proxy_pass http://backend_servers;  # Route to upstream
        }
    }
}
```

### Request Flow
```
1. Client sends: http://loadbalancer.com/page.html
2. Nginx receives on port 80
3. Matches location / rule
4. proxy_pass forwards to http://backend_servers
5. Upstream selects a backend (e.g., server1:3004)
6. Backend processes request
7. Response flows back through Nginx to client
```

### Additional Proxy Directives
```nginx
location / {
    proxy_pass http://backend;
    proxy_set_header Host $host;              # Pass original host
    proxy_set_header X-Real-IP $remote_addr;  # Pass client IP
    proxy_connect_timeout 10s;                # Connection timeout
    proxy_read_timeout 30s;                   # Read timeout
}
```

---

## Common Commands

### Installation
```bash
# CentOS/RHEL
sudo yum install nginx -y

# Ubuntu/Debian
sudo apt install nginx -y
```

### Service Management
```bash
# Start Nginx
sudo systemctl start nginx

# Stop Nginx
sudo systemctl stop nginx

# Restart Nginx (stops then starts)
sudo systemctl restart nginx

# Reload config without dropping connections
sudo systemctl reload nginx

# Enable auto-start on boot
sudo systemctl enable nginx

# Check service status
sudo systemctl status nginx
```

### Configuration Testing
```bash
# Test configuration syntax
sudo nginx -t

# Test and show configuration details
sudo nginx -T
```

**⚠️ ALWAYS test before restarting!**

### Checking Processes
```bash
# See running nginx processes
ps aux | grep nginx

# Check which ports nginx is listening on
sudo ss -tulnp | grep nginx

# Or using netstat
sudo netstat -tulnp | grep nginx
```

### Log Files
```bash
# View access log (all requests)
sudo tail -f /var/log/nginx/access.log

# View error log (problems only)
sudo tail -f /var/log/nginx/error.log
```

---

## Troubleshooting Tips

### Problem: Nginx won't start

**Check 1**: Test configuration
```bash
sudo nginx -t
```
Look for syntax errors like missing semicolons or braces.

**Check 2**: Port already in use
```bash
sudo ss -tulnp | grep :80
```
If another service is on port 80, stop it or change nginx port.

**Check 3**: Check logs
```bash
sudo tail -20 /var/log/nginx/error.log
```

### Problem: 502 Bad Gateway

**Meaning**: Nginx can't reach backend servers

**Solutions**:
```bash
# 1. Check if backend is running
sudo systemctl status httpd  # or apache2

# 2. Verify backend port
sudo ss -tulnp | grep 3004

# 3. Test backend directly
curl http://stapp01:3004

# 4. Check firewall
sudo firewall-cmd --list-all
```

### Problem: 404 Not Found

**Meaning**: Nginx is running but can't find the resource

**Check**:
- Is `proxy_pass` pointing to correct upstream?
- Is backend server configured correctly?
- Are location blocks matching URLs correctly?

### Problem: Configuration changes not taking effect

**Solution**: Reload nginx
```bash
sudo nginx -t           # Test first!
sudo systemctl reload nginx  # Then reload
```

### Useful Debugging Commands
```bash
# Check nginx version
nginx -v

# Check compiled modules
nginx -V

# View current configuration
sudo nginx -T

# Check SELinux issues (CentOS/RHEL)
sudo ausearch -m avc -ts recent

# Test backend connectivity
curl -I http://backend-server:3004

# Follow logs in real-time
sudo tail -f /var/log/nginx/access.log /var/log/nginx/error.log
```

---

## Quick Reference: Your Day 16 Challenge

### Scenario Recap
- **3 App Servers** running Apache on port 3004
- **1 Load Balancer** (Nginx) on port 80
- Goal: Distribute traffic evenly across all app servers

### Key Configuration
```nginx
http {
    upstream myapp {
        server stapp01:3004;
        server stapp02:3004;
        server stapp03:3004;
    }
    
    server {
        listen 80;
        server_name _;
        
        location / {
            proxy_pass http://myapp;
        }
    }
}
```

### Why This Works
1. **Client** connects to load balancer on port 80
2. **Nginx** receives request
3. **Upstream** selects one of three backends (round-robin)
4. **Proxy_pass** forwards request to selected backend's port 3004
5. **Backend** processes and responds
6. **Nginx** returns response to client

### Critical Points
- ✅ Apache stays on port 3004 (unchanged)
- ✅ Nginx listens on port 80 (standard HTTP)
- ✅ All three servers in upstream for load distribution
- ✅ No need to modify Apache configuration

---

## Learning Path Recommendations

### Beginner
1. Understand ports and how services listen
2. Learn basic nginx server blocks
3. Practice starting/stopping/reloading nginx
4. Read and understand nginx.conf structure

### Intermediate
1. Configure reverse proxy
2. Set up upstream blocks
3. Experiment with load balancing methods
4. Add custom headers and timeouts

### Advanced
1. SSL/TLS certificate configuration
2. Caching strategies
3. Rate limiting
4. Advanced security headers

---

## Additional Resources

### Official Documentation
- [Nginx Beginner's Guide](http://nginx.org/en/docs/beginners_guide.html)
- [Nginx Admin Guide](https://docs.nginx.com/nginx/admin-guide/)

### Practice Ideas
1. Set up nginx on a local VM
2. Configure multiple server blocks
3. Test different load balancing methods
4. Break things intentionally and fix them!

### Next Steps
- Learn about HTTPS/SSL certificates
- Explore nginx caching
- Study nginx security best practices
- Practice with Docker containers

---

## Summary

### Key Takeaways
1. **Nginx** is a powerful, efficient web server and reverse proxy
2. **Ports** are numbered doors for network services
3. **Load balancing** distributes traffic across multiple servers
4. **Upstream** defines backend server pools
5. **Proxy_pass** forwards requests to backends
6. **Always test** configuration before restarting

### The Power of Nginx
With just a few lines of configuration, you can:
- Handle thousands of concurrent connections
- Distribute load across many servers
- Protect backend servers from direct exposure
- Improve application performance and reliability

---

**Remember**: Configuration is an iterative process. Test small changes, understand each directive, and build complexity gradually. Good luck with your learning journey! 🚀