# Day 18: Configure LAMP Server |  Understanding the LAMP Stack & WordPress Deployment Challenge

## 🎯 What You're Actually Building

You're setting up a **LAMP stack** (Linux, Apache, MySQL/MariaDB, PHP) to host WordPress across multiple servers. This is a classic real-world scenario for high-availability web applications.

## 🏗️ Architecture Overview

Let me explain what each server does and WHY this architecture exists:

```
[Users] 
   ↓
[Load Balancer (stlb01)] ← Distributes traffic
   ↓
[App Servers (stapp01, stapp02, stapp03)] ← Run Apache + PHP
   ↓                    ↓
[Shared Storage (ststor01)] [Database (stdb01)]
   ↓                    ↓
[/var/www/html]      [MariaDB]
```

### Why This Design?

1. **Multiple App Servers**: If one fails, others keep serving users (High Availability)
2. **Shared Storage**: All app servers read the same WordPress files (Consistency)
3. **Separate Database**: Isolates data layer for security and performance
4. **Load Balancer**: Distributes incoming requests evenly across app servers

---

## 📚 Core Concepts You Need to Understand

### 1. **What is Apache (httpd)?**

Apache is a **web server** - software that:
- Listens on a network port (default 80, your case 6300)
- Receives HTTP requests from browsers
- Serves files (HTML, images) or forwards requests to PHP
- Returns responses to users

**Key Files:**
- `/etc/httpd/conf/httpd.conf` - Main configuration file
- `/var/www/html/` - Default document root (where website files live)

### 2. **What is PHP?**

PHP is a **server-side scripting language**:
- WordPress is written in PHP
- Apache needs PHP module (`mod_php`) to execute `.php` files
- When Apache sees `index.php`, it passes it to PHP interpreter
- PHP processes the code and returns HTML to Apache

**Key Packages:**
- `php` - Core interpreter
- `php-mysql` - Allows PHP to talk to MySQL/MariaDB
- `php-fpm` - FastCGI Process Manager (alternative to mod_php)

### 3. **What is MariaDB?**

MariaDB is a **relational database** (MySQL fork):
- Stores WordPress data: posts, users, settings
- Listens on port 3306
- Uses SQL language for queries
- Manages users and permissions

**Key Concepts:**
- **Database**: Container for tables (e.g., `kodekloud_db10`)
- **User**: Account with specific privileges (e.g., `kodekloud_sam`)
- **Grant**: Permission system (SELECT, INSERT, UPDATE, DELETE)

### 4. **What is a Mounted Directory?**

The storage server has `/var/www/html` **shared via NFS** (Network File System):
- Storage server exports the directory
- App servers mount it (like a network drive)
- All servers see the SAME files
- Changes on one server appear on all others

**Why?** Without this, you'd need to copy WordPress files to 3 separate servers and keep them in sync manually.

---

## 🛠️ Step-by-Step Solution with Deep Explanations

### **Phase 1: Install Apache and PHP on App Servers**

#### Connect to Each App Server

```bash
# From jump host, SSH to each app server
ssh tony@stapp01.stratos.xfusioncorp.com
# Password: Ir0nM@n
```

**What's happening:**
- `ssh` = Secure Shell protocol for remote terminal access
- You're logging into the app server as user `tony`
- Jump host acts as a gateway to the internal network

#### Switch to Root User

```bash
sudo su -
```

**Why?**
- Installing packages requires root (administrator) privileges
- `sudo su -` switches to root user
- `-` flag loads root's environment (proper PATH variables)

#### Install Required Packages

```bash
yum install httpd php php-mysqlnd -y
```

**Breaking it down:**
- `yum` = Package manager for RHEL/CentOS-based systems
- `httpd` = Apache HTTP Server package
- `php` = PHP interpreter
- `php-mysqlnd` = PHP MySQL Native Driver (connects PHP to MySQL/MariaDB)
- `-y` = Automatically answer "yes" to prompts

**What happens behind the scenes:**
1. yum checks configured repositories (package sources)
2. Downloads packages and dependencies
3. Installs binaries to `/usr/sbin/` (httpd) and `/usr/bin/` (php)
4. Creates configuration files in `/etc/`
5. Sets up systemd service units

#### Configure Apache to Listen on Port 6300

```bash
sed -i 's/^Listen 80$/Listen 6300/' /etc/httpd/conf/httpd.conf
```

**Deep dive:**
- `sed` = Stream editor for text manipulation
- `-i` = Edit file in-place (modify the actual file)
- `s/PATTERN/REPLACEMENT/` = Substitution command
- `^Listen 80$` = Regex: line starting with "Listen 80" and nothing after
- `Listen 6300` = New value

**Why change the port?**
- Default port 80 might conflict with other services
- Custom ports add a layer of security through obscurity
- Load balancer will know to forward to port 6300

**Alternative manual method:**
```bash
vi /etc/httpd/conf/httpd.conf
# Find: Listen 80
# Change to: Listen 6300
# Save: :wq
```

#### Enable and Start Apache

```bash
systemctl enable httpd
systemctl start httpd
```

**What's systemctl?**
- Modern Linux service manager (systemd)
- `enable` = Start service automatically at boot
- `start` = Start service now

**How it works:**
1. systemd reads `/usr/lib/systemd/system/httpd.service`
2. Creates symlink in `/etc/systemd/system/multi-user.target.wants/`
3. Launches Apache daemon process

#### Verify Apache is Running

```bash
systemctl status httpd
netstat -tuln | grep 6300
```

**Understanding the commands:**
- `status` = Shows service state (active/inactive)
- `netstat -tuln` = Shows network connections
  - `-t` = TCP connections
  - `-u` = UDP connections
  - `-l` = Listening sockets
  - `-n` = Numeric addresses (don't resolve names)
- `grep 6300` = Filter for your port

**Expected output:**
```
tcp6       0      0 :::6300                 :::*                    LISTEN
```

#### Configure Firewall (if firewalld is running)

```bash
firewall-cmd --permanent --add-port=6300/tcp
firewall-cmd --reload
```

**Firewall concepts:**
- Linux firewall blocks incoming connections by default
- `--permanent` = Survives reboot
- `--add-port=6300/tcp` = Allow traffic on port 6300
- `--reload` = Apply changes without restarting firewall

---

### **Phase 2: Configure MariaDB on Database Server**

#### Connect to Database Server

```bash
ssh peter@stdb01.stratos.xfusioncorp.com
# Password: Sp!dy
sudo su -
```

#### Install MariaDB

```bash
yum install mariadb-server -y
```

**What gets installed:**
- `mariadb-server` = Database server daemon (mysqld)
- Client tools: `mysql`, `mysqladmin`
- System databases: `mysql`, `information_schema`, `performance_schema`

#### Enable and Start MariaDB

```bash
systemctl enable mariadb
systemctl start mariadb
systemctl status mariadb
```

#### Secure MariaDB Installation (Optional but Recommended)

```bash
mysql_secure_installation
```

**What this script does:**
- Sets root password for database (different from Linux root!)
- Removes anonymous users
- Disables remote root login
- Removes test database
- Reloads privilege tables

**For lab environment, you can skip this or use simple password.**

#### Create Database and User

```bash
mysql -u root -p
# If no password set yet, just press Enter
```

**Inside MySQL prompt:**

```sql
CREATE DATABASE kodekloud_db10;
```

**What this does:**
- Creates a new database (logical container for tables)
- Like creating a new folder for organizing data
- WordPress will create tables inside this database

```sql
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'ksH85UJjhb';
```

**Breaking down the syntax:**
- `'kodekloud_sam'` = Username
- `'%'` = Host wildcard (user can connect from ANY IP)
  - Alternative: `'kodekloud_sam'@'172.16.238.%'` (only from app subnet)
  - Alternative: `'kodekloud_sam'@'localhost'` (only local connections)
- `IDENTIFIED BY 'ksH85UJjhb'` = Sets password

**Security note:** In production, never use `'%'` - specify exact IPs!

```sql
GRANT ALL PRIVILEGES ON kodekloud_db10.* TO 'kodekloud_sam'@'%';
```

**Understanding GRANT:**
- `ALL PRIVILEGES` = Full control (SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, etc.)
- `ON kodekloud_db10.*` = On database `kodekloud_db10`, all tables (`*`)
- `TO 'kodekloud_sam'@'%'` = For this user from any host

**Alternative granular permissions (production best practice):**
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON kodekloud_db10.* TO 'kodekloud_sam'@'%';
```

```sql
FLUSH PRIVILEGES;
```

**Why FLUSH?**
- MySQL caches privilege tables in memory
- FLUSH forces reload of grant tables
- Ensures changes take effect immediately

```sql
EXIT;
```

#### Allow Remote Connections to MariaDB

By default, MariaDB only listens on localhost (`127.0.0.1`). You need to allow connections from app servers.

```bash
vi /etc/my.cnf
```

**Add or modify:**
```ini
[mysqld]
bind-address = 0.0.0.0
```

**What this means:**
- `bind-address` = IP address MySQL listens on
- `0.0.0.0` = Listen on all network interfaces
- Default `127.0.0.1` = Only local connections

**Restart MariaDB:**
```bash
systemctl restart mariadb
```

#### Configure Firewall for MySQL

```bash
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload
```

#### Test Database Connection from App Server

```bash
# From app server (stapp01)
mysql -h 172.16.239.10 -u kodekloud_sam -p'ksH85UJjhb' kodekloud_db10
```

**Breaking down the command:**
- `mysql` = MySQL/MariaDB client
- `-h 172.16.239.10` = Host (database server IP)
- `-u kodekloud_sam` = Username
- `-p'ksH85UJjhb'` = Password (no space after -p!)
- `kodekloud_db10` = Database to connect to

**If successful, you'll see:**
```
MariaDB [kodekloud_db10]>
```

---

### **Phase 3: Install WordPress**

#### Download WordPress on Storage Server

```bash
ssh natasha@ststor01.stratos.xfusioncorp.com
# Password: Bl@kW
sudo su -
cd /var/www/html
```

#### Download and Extract WordPress

```bash
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
```

**Understanding the commands:**
- `wget` = Download files from web
- `tar -xzf` = Extract compressed archive
  - `-x` = Extract
  - `-z` = Decompress gzip
  - `-f` = File to extract

**Result:** Creates `/var/www/html/wordpress/` directory

#### Move WordPress Files to Document Root

```bash
mv wordpress/* .
rmdir wordpress
rm latest.tar.gz
```

**Why move files?**
- URL should be `http://server/` not `http://server/wordpress/`
- Moving contents to `/var/www/html/` makes WordPress the root application

#### Set Correct Permissions

```bash
chown -R apache:apache /var/www/html
chmod -R 755 /var/www/html
```

**Understanding Linux permissions:**

```
User  Group  Others
rwx   rwx    rwx
421   421    421
```

- `755` = Owner: read+write+execute (7), Group: read+execute (5), Others: read+execute (5)
- `chown apache:apache` = Makes Apache web server the owner
- `-R` = Recursive (apply to all files/subdirectories)

**Why Apache needs ownership:**
- WordPress uploads files (themes, plugins, media)
- Apache process must write to directories like `wp-content/uploads/`
- Without proper permissions, you'll get "Cannot write to directory" errors

#### Configure WordPress Database Connection

```bash
cp wp-config-sample.php wp-config.php
vi wp-config.php
```

**Edit these lines:**
```php
define( 'DB_NAME', 'kodekloud_db10' );
define( 'DB_USER', 'kodekloud_sam' );
define( 'DB_PASSWORD', 'ksH85UJjhb' );
define( 'DB_HOST', '172.16.239.10' );
```

**What's happening:**
- WordPress uses these constants to connect to database
- `DB_HOST` = Database server IP (not localhost!)
- When user visits site, WordPress:
  1. Reads `wp-config.php`
  2. Connects to MariaDB using credentials
  3. Queries database for content
  4. Generates HTML pages dynamically

---

### **Phase 4: Testing and Troubleshooting**

#### Test from App Server

```bash
# From stapp01
curl http://localhost:6300
```

**Expected output:**
HTML content with WordPress installation page or error messages.

#### Test Database Connectivity

Create a test PHP file:

```bash
vi /var/www/html/test-db.php
```

```php
<?php
$host = '172.16.239.10';
$user = 'kodekloud_sam';
$pass = 'ksH85UJjhb';
$db = 'kodekloud_db10';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "App is able to connect to the database using user kodekloud_sam";
$conn->close();
?>
```

**Access via browser:**
```
http://<load-balancer-url>:6300/test-db.php
```

**What this script does:**
1. Creates MySQL connection using `mysqli` extension
2. Tests connection with credentials
3. Displays success/failure message
4. This simulates what WordPress does internally

#### Check Apache Error Logs

```bash
tail -f /var/log/httpd/error_log
```

**Common errors you might see:**
- `Permission denied` = File ownership/permission issue
- `Can't connect to MySQL server` = Database not accessible
- `Access denied for user` = Wrong credentials or GRANT missing

---

## 🧠 Key Concepts to Internalize

### 1. **The Request Flow**

```
User Browser
    ↓ [HTTP Request to LB:6300]
Load Balancer (stlb01)
    ↓ [Forwards to one of the app servers]
App Server (stapp01) - Apache on port 6300
    ↓ [Sees .php file, passes to PHP interpreter]
PHP Interpreter
    ↓ [Executes WordPress code]
WordPress Code
    ↓ [Connects to database]
MariaDB (stdb01:3306)
    ↓ [Returns query results]
PHP Interpreter
    ↓ [Generates HTML]
Apache
    ↓ [Sends response]
User Browser
```

### 2. **Why Multiple App Servers Share Storage**

**Problem without shared storage:**
- User uploads image to stapp01
- Load balancer sends next request to stapp02
- stapp02 doesn't have the image → broken link

**Solution:**
- NFS mount ensures all servers see same files
- Upload goes to shared `/var/www/html`
- All servers can serve the same content

### 3. **Database vs File Storage**

**Database (MariaDB):**
- Structured data: posts, users, comments, settings
- Queried using SQL
- Optimized for search and relationships

**File System:**
- Unstructured data: images, videos, themes, plugins
- Accessed directly by Apache
- Better for large binary files

### 4. **Security Layers**

1. **Network:** Firewall rules, private subnets
2. **Authentication:** Database users, Linux users, WordPress users
3. **Authorization:** MySQL GRANTS, file permissions (chmod/chown)
4. **Application:** WordPress security plugins, input validation

---

## 📝 Common Mistakes & How to Avoid Them

### Mistake 1: Wrong File Permissions
**Symptom:** WordPress can't upload files
**Solution:** `chown -R apache:apache /var/www/html && chmod -R 755 /var/www/html`

### Mistake 2: Database Not Accessible Remotely
**Symptom:** `Can't connect to MySQL server on 172.16.239.10`
**Fix:** Check `bind-address` in `/etc/my.cnf` and firewall rules

### Mistake 3: SELinux Blocking Connections
**Symptom:** Permission denied errors even with correct chmod
**Solution:** 
```bash
setenforce 0  # Temporary (testing only!)
# OR
setsebool -P httpd_can_network_connect_db on
```

### Mistake 4: Forgetting to Restart Services
**Rule:** Always restart after config changes!
```bash
systemctl restart httpd
systemctl restart mariadb
```

---

## 🚀 What You've Actually Learned

By completing this challenge, you now understand:

1. ✅ **Web Server Basics:** How Apache serves content and executes PHP
2. ✅ **Database Fundamentals:** Users, privileges, remote connections
3. ✅ **Linux Administration:** Package management, services, permissions
4. ✅ **Networking:** Ports, firewalls, SSH, HTTP flow
5. ✅ **High Availability:** Load balancing, shared storage, redundancy
6. ✅ **Application Architecture:** Separation of concerns (web/app/data layers)

---

## 🔍 How to Approach Future Projects

### The DevOps Debugging Framework

When something doesn't work:

1. **Identify the layer:**
   - Network? (Can you ping? Telnet to port?)
   - Service? (Is process running? `systemctl status`)
   - Config? (Check config files, look for typos)
   - Permissions? (Can user access files? `ls -la`)
   - Application? (Check logs: `/var/log/`)

2. **Test incrementally:**
   - Don't configure everything at once
   - Test after each step
   - Use `curl`, `telnet`, `mysql` CLI to verify

3. **Read error messages carefully:**
   - Logs tell you EXACTLY what's wrong
   - Google the exact error (in quotes)
   - Check official documentation

4. **Understand before executing:**
   - Ask "What does this command change?"
   - Ask "How can I verify it worked?"
   - Ask "What's the rollback procedure?"

---

## 📚 Resources for Deep Learning

### Essential Documentation
- [Apache HTTP Server Docs](https://httpd.apache.org/docs/)
- [MariaDB Knowledge Base](https://mariadb.com/kb/en/)
- [WordPress Codex](https://codex.wordpress.org/)
- [PHP Manual](https://www.php.net/manual/en/)

### Hands-On Practice
- [KodeKloud Engineer Tasks](https://kodekloud.com/kodekloud-engineer/)
- [Linux Journey](https://linuxjourney.com/)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)

### Debugging Skills
- [The Art of Command Line](https://github.com/jlevy/the-art-of-command-line)
- `man` pages: `man httpd`, `man mysql`, `man systemctl`

---

## ✅ Next Challenge: Install and Configure Web Application

Now let me explain the next level...

### What This Typically Involves

"Install and Configure Web Application" challenges build on what you learned:

**Common scenarios:**
1. **Deploy from Git:**
   ```bash
   git clone <repo>
   npm install  # or pip install, composer install
   Configure environment variables
   Start application service
   ```

2. **Reverse Proxy Configuration:**
   - App runs on `localhost:3000`
   - Apache/Nginx proxies requests to it
   - Allows multiple apps on same server

3. **Containerized Deployments:**
   - Docker containers instead of direct installs
   - Docker Compose for multi-container apps
   - Volume mounts for persistent data

4. **CI/CD Integration:**
   - Jenkins jobs for automated deployment
   - Git hooks trigger builds
   - Blue-green deployments for zero downtime

### Example: Node.js App Behind Apache Reverse Proxy

#### Install Node.js
```bash
yum install nodejs npm -y
```

#### Clone Application
```bash
cd /opt
git clone https://github.com/example/app.git
cd app
npm install
```

#### Create Systemd Service
```bash
vi /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Node.js App
After=network.target

[Service]
Type=simple
User=nodejs
WorkingDirectory=/opt/app
ExecStart=/usr/bin/node server.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Why systemd service?**
- Starts app automatically on boot
- Restarts on crashes
- Managed like other services (`systemctl start myapp`)

#### Configure Apache as Reverse Proxy
```bash
vi /etc/httpd/conf.d/myapp.conf
```

```apache
<VirtualHost *:80>
    ServerName myapp.example.com
    
    ProxyPreserveHost On
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
    
    ErrorLog /var/log/httpd/myapp_error.log
    CustomLog /var/log/httpd/myapp_access.log combined
</VirtualHost>
```

**What's a reverse proxy?**
- Client connects to Apache (port 80)
- Apache forwards request to Node.js (port 3000)
- Node.js sends response back through Apache
- Client never knows about port 3000

**Why use this?**
- SSL termination at Apache level
- Single entry point for multiple apps
- Apache handles static files, Node.js handles dynamic content
- Better security (app not directly exposed)

---

## 🎓 Final Thoughts

You succeeded in the challenge - that's huge! The fact that you want to understand WHY shows you're on the right path to becoming a strong engineer.

**Remember:**
- **Memorization** = Fragile knowledge that breaks in new scenarios
- **Understanding** = Transferable skills that adapt to any problem

**Your learning approach going forward:**
1. Complete the task (even if copying commands)
2. Go back and ask "why" for each step
3. Break something intentionally, then fix it
4. Teach someone else (or write it down)
5. Apply the concept in a different project

The challenge you completed is a microcosm of production DevOps:
- Multi-server architecture
- Service configuration
- Database management
- Troubleshooting
- Documentation

Master these fundamentals, and frameworks/tools become easy to learn.
