# Day 20: Configure Nginx + PHP-FPM Using Unix Socket - Deep Dive

**Challenge Completed ✅** | **Understanding Level: Expert**

---

## 📋 Table of Contents
1. [Challenge Overview](#challenge-overview)
2. [Infrastructure & Requirements](#infrastructure--requirements)
3. [Core Concepts You Must Understand](#core-concepts-you-must-understand)
4. [Step-by-Step Implementation with Explanations](#step-by-step-implementation-with-explanations)
5. [Troubleshooting Guide](#troubleshooting-guide)
6. [Interview Questions & Answers](#interview-questions--answers)
7. [Real-World Applications](#real-world-applications)

---

## Challenge Overview

**Goal:** Deploy a PHP application on App Server 3 using Nginx as the web server and PHP-FPM for processing PHP code, communicating via a Unix socket.

**Why This Matters:** This is the standard production setup for PHP applications (WordPress, Laravel, etc.). Understanding this makes you deployment-ready for real projects.

---

## Infrastructure & Requirements

| Component | Details |
|-----------|---------|
| **Server** | stapp03 (172.16.238.12) |
| **User** | banner |
| **Web Server** | Nginx (port 8096) |
| **PHP Version** | 8.1 via PHP-FPM |
| **Communication** | Unix socket at `/var/run/php-fpm/default.sock` |
| **Document Root** | `/var/www/html` |

---

## Core Concepts You Must Understand

### 1. **What is Nginx?**
- **Type:** Web server (like Apache, but lighter and faster)
- **Role:** Receives HTTP requests from browsers/clients and serves files
- **Limitation:** Cannot execute PHP code directly (it's not a PHP interpreter)
- **Solution:** Needs PHP-FPM to handle PHP

**Real-World Analogy:** Nginx is like a restaurant waiter - takes orders but doesn't cook. PHP-FPM is the chef.

---

### 2. **What is PHP-FPM?**
- **Full Name:** FastCGI Process Manager
- **Purpose:** Executes PHP code and returns results
- **Why "FPM"?** It manages multiple PHP worker processes efficiently
- **Communication:** Can talk to Nginx via:
  - **TCP socket** (e.g., `127.0.0.1:9000`) - network-based
  - **Unix socket** (e.g., `/var/run/php-fpm/default.sock`) - file-based ⚡ **FASTER**

**Key Point:** PHP-FPM is a separate service from Nginx. They work together but are independent.

---

### 3. **Unix Socket vs TCP Socket**

| Feature | Unix Socket | TCP Socket |
|---------|-------------|------------|
| **Speed** | ⚡ Faster (no network overhead) | Slower (network stack) |
| **Use Case** | Same machine communication | Remote or multi-server |
| **Security** | File permissions control access | Network firewalls needed |
| **Path Example** | `/var/run/php-fpm/default.sock` | `127.0.0.1:9000` |

**Why We Use Unix Socket Here:** Both Nginx and PHP-FPM are on the same server, so Unix socket is optimal.

---

### 4. **The Request Flow**

```
Browser → Nginx (Port 8096) → Checks file type
                               ↓
                        Is it .php?
                               ↓
                         YES → Forward to PHP-FPM via Unix Socket
                               ↓
                        PHP-FPM executes code
                               ↓
                        Returns HTML output to Nginx
                               ↓
                        Nginx sends response to Browser
```

---

## Step-by-Step Implementation with Explanations

### **Step 1: Connect to Server**

```bash
ssh banner@stapp03
```

**Why?** You need to work on App Server 3 where the setup will happen.

---

### **Step 2: Install EPEL and Remi Repositories**

```bash
sudo dnf install -y epel-release
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

**What's Happening:**
- **EPEL** (Extra Packages for Enterprise Linux) - Adds community packages
- **Remi** - Provides newer PHP versions (CentOS default repos have old PHP)

**Why Needed:** Default CentOS/RHEL repos don't have PHP 8.1. Remi specializes in PHP packages.

**Real-World:** In production, you always need updated PHP for security and features.

---

### **Step 3: Enable PHP 8.1 Module**

```bash
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.1 -y
```

**Breakdown:**
- **`module reset php`** - Clears any previous PHP module settings (prevents conflicts)
- **`module enable php:remi-8.1`** - Activates PHP 8.1 from Remi repo

**Why This Step?** RHEL/CentOS 9 uses "Application Streams" - you must explicitly enable the version you want.

---

### **Step 4: Install PHP-FPM and Extensions**

```bash
sudo dnf install -y php php-fpm php-cli php-common php-mysqlnd php-json php-opcache
php-fpm -v
```

**Packages Explained:**
- **php** - Core PHP
- **php-fpm** - FastCGI Process Manager (the star of the show)
- **php-cli** - Command-line PHP (for running scripts)
- **php-mysqlnd** - MySQL database driver
- **php-json** - JSON handling
- **php-opcache** - Speeds up PHP by caching compiled code

**Verify Installation:** `php-fpm -v` shows version (should be 8.1.x)

---

### **Step 5: Configure PHP-FPM to Use Unix Socket**

**Edit the pool configuration file:**

```bash
sudo vi /etc/php-fpm.d/www.conf
```

**Find and modify these lines:**

```ini
listen = /var/run/php-fpm/default.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
user = nginx
group = nginx
```

**Line-by-Line Explanation:**

| Line | Meaning |
|------|---------|
| `listen = /var/run/php-fpm/default.sock` | Create Unix socket at this path |
| `listen.owner = nginx` | Nginx user owns the socket file |
| `listen.group = nginx` | Nginx group can access it |
| `listen.mode = 0660` | Permissions: owner/group can read+write, others blocked |
| `user = nginx` | PHP-FPM worker processes run as nginx user |
| `group = nginx` | PHP-FPM worker processes run as nginx group |

**Why These Settings?**
- **Security:** Only Nginx can talk to PHP-FPM (not other users)
- **Compatibility:** Both services run as same user to avoid permission issues

---

**Create the socket directory:**

```bash
sudo mkdir -p /var/run/php-fpm
sudo chown nginx:nginx /var/run/php-fpm
```

**Why?** The socket file needs a parent directory with correct ownership.

---

### **Step 6: Configure Nginx**

**Create/edit Nginx configuration:**

```bash
sudo vi /etc/nginx/conf.d/default.conf
```

**Add this server block:**

```nginx
server {
    listen 8096;
    server_name _;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
}
```

**Configuration Breakdown:**

```nginx
listen 8096;
```
→ Nginx listens on port 8096 (not default 80)

```nginx
server_name _;
```
→ Accept requests for any hostname (wildcard)

```nginx
root /var/www/html;
```
→ Where website files are stored

```nginx
index index.php index.html;
```
→ Default files to serve if no filename specified

```nginx
location / {
    try_files $uri $uri/ =404;
}
```
→ Try to serve the requested file; if not found, return 404

```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php-fpm/default.sock;
    fastcgi_index index.php;
    include fastcgi.conf;
}
```
→ **THE MAGIC PART:**
- `location ~ \.php$` - Matches any URL ending with `.php`
- `fastcgi_pass` - Send request to PHP-FPM via Unix socket
- `include fastcgi.conf` - Adds required environment variables (like SCRIPT_FILENAME)

---

### **Step 7: Start and Enable Services**

```bash
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
sudo systemctl status php-fpm
```

```bash
sudo systemctl enable nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

**What's Happening:**
- **enable** - Service starts automatically on boot
- **start/restart** - Starts the service now
- **status** - Verify it's running (look for "active (running)")

---

### **Step 8: Create Test PHP File**

```bash
sudo mkdir -p /var/www/html
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
echo "<?php echo 'Hello from PHP-FPM!'; ?>" | sudo tee /var/www/html/index.php
```

**Why?** These files test that PHP execution works.

---

### **Step 9: Test from Jump Host**

```bash
curl http://stapp03:8096/index.php
curl http://stapp03:8096/info.php
```

**Expected Results:**
- `index.php` → Shows "Hello from PHP-FPM!"
- `info.php` → Shows full PHP configuration page

---

## Troubleshooting Guide

### Problem 1: **502 Bad Gateway**

**Meaning:** Nginx can't connect to PHP-FPM

**Check:**
```bash
sudo systemctl status php-fpm
ls -l /var/run/php-fpm/default.sock
```

**Fix:**
- Ensure PHP-FPM is running
- Check socket file exists and has correct permissions (nginx:nginx)

---

### Problem 2: **File Not Found**

**Meaning:** Nginx can't locate PHP file

**Check:**
```bash
ls -l /var/www/html/
```

**Fix:**
- Verify files exist in document root
- Check file permissions (should be readable by nginx user)

---

### Problem 3: **Connection Refused**

**Meaning:** Port 8096 isn't accessible

**Check:**
```bash
sudo ss -tlnp | grep 8096
sudo firewall-cmd --list-all
```

**Fix:**
```bash
sudo firewall-cmd --permanent --add-port=8096/tcp
sudo firewall-cmd --reload
```

---

## Interview Questions & Answers

### Q1: **Why use Unix socket instead of TCP?**

**Answer:** Unix sockets are faster because they avoid network protocol overhead. When Nginx and PHP-FPM are on the same machine, Unix sockets provide:
- Lower latency
- Higher throughput
- Better security (file permissions control access)
- No network port exposure

---

### Q2: **What happens if permissions on the socket are wrong?**

**Answer:** Nginx won't be able to communicate with PHP-FPM, resulting in a 502 Bad Gateway error. The socket file must be:
- Owned by nginx user
- Readable/writable by nginx group
- Typically `0660` permissions

---

### Q3: **Can you run multiple PHP-FPM pools?**

**Answer:** Yes! You can create multiple pools in `/etc/php-fpm.d/` with different:
- Socket paths
- User/group settings
- PHP configurations
- Resource limits

This is useful for hosting multiple websites with different PHP requirements.

---

### Q4: **What's the difference between Apache and Nginx for PHP?**

**Answer:**
- **Apache:** Uses mod_php (PHP embedded in Apache process) - simpler but heavier
- **Nginx:** Requires PHP-FPM (separate process) - more complex but faster and more scalable

Nginx is preferred for high-traffic sites.

---

### Q5: **How would you scale this setup?**

**Answer:**
1. **Vertical:** Increase PHP-FPM worker processes
2. **Horizontal:** Use multiple app servers behind a load balancer
3. **Caching:** Add Redis/Memcached for session storage
4. **CDN:** Offload static assets

---

## Real-World Applications

### 1. **WordPress Hosting**
- Most WordPress hosts use Nginx + PHP-FPM
- Unix sockets handle millions of requests efficiently

### 2. **Laravel Applications**
- Modern PHP frameworks require PHP-FPM
- Nginx serves static assets, PHP-FPM handles application logic

### 3. **Multi-Tenant Systems**
- Each customer gets their own PHP-FPM pool
- Isolated resources and permissions

---

## 🎯 Key Takeaways for Your Friends

1. **Nginx is a web server, not a PHP interpreter** - it needs PHP-FPM
2. **Unix sockets are faster than TCP** when both services are local
3. **Permissions matter** - socket files must be accessible to Nginx
4. **This is production-grade** - major companies use this exact setup
5. **Separation of concerns** - Nginx handles HTTP, PHP-FPM handles code

---

## 📚 Further Learning

- **Next Challenge:** Add SSL/TLS certificate to Nginx
- **Advanced Topic:** PHP-FPM performance tuning (pm.max_children, pm.start_servers)
- **Related Skill:** Set up Nginx as a reverse proxy for Node.js apps

---
