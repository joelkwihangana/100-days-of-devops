# **Complete Guide: Deploying Websites on Apache & Nginx (From Zero to Production)**

## **Table of Contents**
1. [Apache Deep Dive](#apache-deep-dive)
2. [Nginx Deep Dive](#nginx-deep-dive)
3. [Apache vs Nginx Comparison](#apache-vs-nginx)
4. [Real Production Scenarios](#real-production-scenarios)
5. [Hostinger VPS Deployment Guide](#hostinger-vps-deployment)
6. [Resources & Next Steps](#resources)

---

# **Apache Deep Dive**

## **What is Apache?**

Apache HTTP Server (httpd) is a **web server** software that:
- Listens for HTTP requests on specific ports (default: 80 for HTTP, 443 for HTTPS)
- Serves files (HTML, CSS, JS, images) to browsers
- Can run server-side code (PHP, Python, etc.)
- Acts as a **middleman** between users and your website files

**Think of it like a restaurant waiter:**
- Customer (browser) asks for food (website)
- Waiter (Apache) brings the food from the kitchen (filesystem)
- Customer receives the meal (sees the website)

---

## **Breaking Down Every Command You Used**

### **1. Installing Apache**

```bash
sudo yum install httpd -y
```

**What happens behind the scenes:**
- `yum` is the package manager for RHEL/CentOS (like apt for Ubuntu)
- It downloads Apache from official repositories
- `-y` automatically says "yes" to all prompts
- Installs these key components:
  - **httpd binary**: The main Apache program
  - **Config files**: Rules for how Apache behaves
  - **systemd service**: Allows starting/stopping Apache with systemctl
  - **Modules**: Extra functionality (SSL, PHP support, etc.)

**Files created:**
- `/usr/sbin/httpd` - The Apache program itself
- `/etc/httpd/conf/httpd.conf` - Main configuration file
- `/var/www/html/` - Default web root (where you put website files)
- `/var/log/httpd/` - Logs (access and error logs)

---

### **2. Changing the Port**

```bash
sudo vi /etc/httpd/conf/httpd.conf
# Change: Listen 80 → Listen 5004
```

**Why ports matter:**
- A server can have many services (web, database, SSH)
- Each service needs a unique "door" (port number)
- Port 80 = HTTP (web traffic)
- Port 443 = HTTPS (secure web traffic)
- Port 5004 = Custom port (avoids conflicts, useful for testing)

**When would you use custom ports in real life?**
- Running multiple web servers on one machine
- Security (hiding from automated port scanners)
- Development/testing environments
- Company firewall rules

---

### **3. The Alias Directive**

```apache
Alias /ecommerce "/var/www/html/ecommerce"
<Directory "/var/www/html/ecommerce">
    AllowOverride None
    Require all granted
</Directory>
```

**Let's decode this line by line:**

#### **`Alias /ecommerce "/var/www/html/ecommerce"`**
- Maps a URL path to a filesystem path
- When someone visits `http://yoursite.com/ecommerce/`
- Apache looks in `/var/www/html/ecommerce/` on the server

**Real-world analogy:**
- Like a shortcut on your desktop
- The shortcut points to a folder somewhere else
- Clicking it opens the real folder

#### **`<Directory>` block**
- Sets rules for that specific folder
- **AllowOverride None**: Ignores `.htaccess` files (security/performance)
- **Require all granted**: Anyone can access (no password required)

**Other common Directory options:**
```apache
# Password protect a directory
<Directory "/var/www/html/admin">
    AuthType Basic
    AuthName "Admin Area"
    AuthUserFile /etc/httpd/.htpasswd
    Require valid-user
</Directory>

# Block access to a directory
<Directory "/var/www/html/private">
    Require all denied
</Directory>
```

---

### **4. File Permissions**

```bash
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
```

**Why this matters:**
Linux is **paranoid about security**. Files have ownership and permissions.

#### **Understanding `chown`:**
- `apache:apache` means **user:group**
- Apache runs as the `apache` user (not root, for security)
- If Apache can't read files, it returns **403 Forbidden**

#### **Understanding `chmod 755`:**
This is a **3-digit permission code**:

| Digit | Who | Permission | Binary | Meaning |
|-------|-----|------------|--------|---------|
| 7 | Owner (apache) | rwx | 111 | Read, Write, Execute |
| 5 | Group (apache) | r-x | 101 | Read, Execute (no write) |
| 5 | Others (everyone) | r-x | 101 | Read, Execute (no write) |

**Why 755 for websites?**
- Apache needs to READ files (serve them)
- Apache needs to EXECUTE directories (list contents)
- Others can view files (needed for public websites)

**What if permissions are wrong?**
```bash
# Too restrictive (chmod 700) → 403 Forbidden
# Too open (chmod 777) → Security risk (anyone can modify files)
```

---

### **5. Starting Apache**

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

**What's the difference?**
- `start`: Starts Apache NOW (temporary, stops on reboot)
- `enable`: Starts Apache automatically after server reboot
- Always use BOTH in production!

**Other useful systemctl commands:**
```bash
sudo systemctl stop httpd       # Stop Apache
sudo systemctl restart httpd    # Restart (use after config changes)
sudo systemctl reload httpd     # Reload config without dropping connections
sudo systemctl status httpd     # Check if running, view recent logs
```

---

### **6. Testing with curl**

```bash
curl http://localhost:5004/ecommerce/
```

**Why use curl instead of a browser?**
- Fast testing from command line
- Works on servers without GUI
- Shows raw HTML (helps debug)
- Can test from the server itself (no network issues)

**Other useful curl commands:**
```bash
# See HTTP headers
curl -I http://localhost:5004/ecommerce/

# Follow redirects
curl -L http://yoursite.com

# Save output to file
curl http://yoursite.com > output.html

# Check response time
curl -w "Time: %{time_total}s\n" http://yoursite.com
```

---

# **Nginx Deep Dive**

## **What is Nginx?**

Nginx (pronounced "engine-x") is a **modern web server** that works differently than Apache:

**Key differences:**
- **Apache**: One thread per connection (can handle ~10,000 connections)
- **Nginx**: Event-driven architecture (can handle ~100,000 connections)
- **Apache**: Better for dynamic content (PHP)
- **Nginx**: Better for static files and reverse proxy

**Real-world usage:**
- Netflix, Airbnb, Dropbox use Nginx
- Often used as a **reverse proxy** in front of Node.js/Python apps
- Great for serving static files (images, CSS, JS)

---

## **Same Challenge, but with Nginx**

### **Step 1: Install Nginx**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx -y

# CentOS/RHEL
sudo yum install epel-release -y
sudo yum install nginx -y
```

**Files created:**
- `/etc/nginx/nginx.conf` - Main config
- `/etc/nginx/sites-available/` - Site configs (Ubuntu)
- `/etc/nginx/conf.d/` - Site configs (CentOS)
- `/usr/share/nginx/html/` - Default web root
- `/var/log/nginx/` - Logs

---

### **Step 2: Change Port to 5004**

Edit the main config:
```bash
sudo vi /etc/nginx/nginx.conf
```

Find the `server` block and change:
```nginx
server {
    listen 5004;
    listen [::]:5004;  # IPv6 support
    server_name localhost;
    
    # ... rest of config
}
```

---

### **Step 3: Configure Multiple Sites**

Create a new config file:
```bash
sudo vi /etc/nginx/conf.d/websites.conf
```

Add:
```nginx
server {
    listen 5004;
    server_name localhost;
    
    # Ecommerce site
    location /ecommerce/ {
        alias /var/www/html/ecommerce/;
        index index.html index.htm;
    }
    
    # Apps site
    location /apps/ {
        alias /var/www/html/apps/;
        index index.html index.htm;
    }
}
```

**Key Nginx concepts:**

#### **`location` blocks = URL patterns**
```nginx
# Exact match
location = /exact {
    # Matches http://site.com/exact ONLY
}

# Prefix match
location /images/ {
    # Matches http://site.com/images/anything
}

# Regex match
location ~ \.php$ {
    # Matches any URL ending in .php
}
```

#### **`alias` vs `root`**
```nginx
# With alias:
location /ecommerce/ {
    alias /var/www/html/ecommerce/;
    # /ecommerce/product.html → /var/www/html/ecommerce/product.html
}

# With root:
location /ecommerce/ {
    root /var/www/html;
    # /ecommerce/product.html → /var/www/html/ecommerce/product.html
}
```
Use `alias` when the URL path doesn't match the directory structure.

---

### **Step 4: Start Nginx**

```bash
# Test config before starting (important!)
sudo nginx -t

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Reload after config changes
sudo systemctl reload nginx
```

---

### **Step 5: Copy Files and Set Permissions**

Same as Apache:
```bash
# Copy files
scp -r /home/thor/ecommerce steve@stapp02:/tmp/
sudo mv /tmp/ecommerce /var/www/html/

# Set permissions (Nginx runs as 'nginx' user)
sudo chown -R nginx:nginx /var/www/html/
sudo chmod -R 755 /var/www/html/
```

---

# **Apache vs Nginx: When to Use Which?**

| Feature | Apache | Nginx |
|---------|--------|-------|
| **Performance (static files)** | Good | Excellent |
| **Performance (high traffic)** | Moderate | Excellent |
| **PHP support** | Built-in (mod_php) | Needs PHP-FPM |
| **Configuration** | .htaccess files | Central config only |
| **Learning curve** | Easier | Steeper |
| **Memory usage** | Higher | Lower |
| **Use case** | Shared hosting, WordPress | High-traffic sites, reverse proxy |

**My recommendation:**
- **Learning?** Start with Apache (simpler)
- **Production?** Nginx for modern apps
- **WordPress/PHP?** Apache or Nginx+PHP-FPM
- **Node.js/React?** Nginx as reverse proxy

---

# **Real Production Scenarios**

## **Scenario 1: React + Node.js + PostgreSQL**

### **Architecture:**
```
User → Nginx (Port 80/443) → React (static files)
                            ↓
                      Node.js API (Port 3000)
                            ↓
                      PostgreSQL (Port 5432)
```

### **Nginx Configuration:**

```nginx
server {
    listen 80;
    server_name myapp.com;
    
    # Serve React build files
    location / {
        root /var/www/myapp/build;
        try_files $uri /index.html;  # SPA routing
    }
    
    # Proxy API requests to Node.js
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### **What this does:**
1. **React frontend** served as static files from `/var/www/myapp/build`
2. **API calls** (`/api/*`) proxied to Node.js on port 3000
3. **try_files** handles React Router (client-side routing)

---

## **Scenario 2: Separate Frontend and Backend Servers**

### **Architecture:**
```
User → Nginx (frontend.com) → React
     → Nginx (api.backend.com) → Node.js → Database
```

### **Frontend Nginx Config:**
```nginx
server {
    listen 80;
    server_name frontend.com;
    
    location / {
        root /var/www/react-app/build;
        try_files $uri /index.html;
    }
}
```

### **Backend Nginx Config:**
```nginx
server {
    listen 80;
    server_name api.backend.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## **Scenario 3: Multiple Apps on One Server**

```nginx
server {
    listen 80;
    server_name myserver.com;
    
    # App 1: E-commerce (React)
    location /shop/ {
        alias /var/www/ecommerce/build/;
        try_files $uri /shop/index.html;
    }
    
    # App 2: Admin Panel (Vue.js)
    location /admin/ {
        alias /var/www/admin/dist/;
        try_files $uri /admin/index.html;
    }
    
    # App 3: API (Node.js)
    location /api/ {
        proxy_pass http://localhost:3000/;
    }
    
    # App 4: Blog (WordPress on Apache)
    location /blog/ {
        proxy_pass http://localhost:8080/blog/;
    }
}
```

---

# **Hostinger VPS Deployment Guide**

## **Pre-Deployment Checklist**

Before touching Hostinger, prepare locally:

### **1. Build Your Frontend**
```bash
# React
npm run build
# Creates: build/ folder

# Vue
npm run build
# Creates: dist/ folder

# Next.js
npm run build
npm run export  # For static export
```

### **2. Prepare Your Backend**
```bash
# Node.js - use PM2 (process manager)
npm install -g pm2

# Create ecosystem file
pm2 ecosystem
```

Example `ecosystem.config.js`:
```javascript
module.exports = {
  apps: [{
    name: 'api',
    script: './server.js',
    instances: 2,
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
};
```

---

## **Hostinger VPS Setup (Step-by-Step)**

### **Step 1: Access Your VPS**

```bash
# SSH into your VPS (Hostinger provides IP and password)
ssh root@your-vps-ip
```

### **Step 2: Initial Server Setup**

```bash
# Update system
apt update && apt upgrade -y

# Create non-root user
adduser deploy
usermod -aG sudo deploy

# Setup SSH key (from your local machine)
ssh-copy-id deploy@your-vps-ip

# Disable root login (security)
sudo vi /etc/ssh/sshd_config
# Change: PermitRootLogin no
sudo systemctl restart sshd
```

### **Step 3: Install Required Software**

```bash
# Install Nginx
sudo apt install nginx -y

# Install Node.js (use NodeSource for latest version)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs -y

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib -y

# Install PM2
sudo npm install -g pm2
```

### **Step 4: Configure Firewall**

```bash
# Allow SSH, HTTP, HTTPS
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### **Step 5: Setup Database**

```bash
# Switch to postgres user
sudo -i -u postgres

# Create database and user
createdb myapp_db
createuser myapp_user
psql

# In psql:
ALTER USER myapp_user WITH PASSWORD 'strong_password';
GRANT ALL PRIVILEGES ON DATABASE myapp_db TO myapp_user;
\q
exit
```

### **Step 6: Deploy Backend**

```bash
# Create app directory
sudo mkdir -p /var/www/myapp-api
sudo chown deploy:deploy /var/www/myapp-api

# Upload your code (from local machine)
scp -r ./backend/* deploy@your-vps-ip:/var/www/myapp-api/

# On VPS: Install dependencies
cd /var/www/myapp-api
npm install --production

# Create .env file
vi .env
```

Example `.env`:
```
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://myapp_user:strong_password@localhost:5432/myapp_db
```

```bash
# Start with PM2
pm2 start ecosystem.config.js
pm2 startup systemd
pm2 save
```

### **Step 7: Deploy Frontend**

```bash
# Upload build files
scp -r ./frontend/build/* deploy@your-vps-ip:/var/www/myapp-frontend/
```

### **Step 8: Configure Nginx**

```bash
sudo vi /etc/nginx/sites-available/myapp
```

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    # Frontend
    location / {
        root /var/www/myapp-frontend;
        try_files $uri /index.html;
    }
    
    # Backend API
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Test and reload
sudo nginx -t
sudo systemctl reload nginx
```

### **Step 9: Setup SSL (Let's Encrypt)**

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal is set up automatically
# Test renewal:
sudo certbot renew --dry-run
```

---

## **Common Issues and Solutions**

### **Issue 1: 502 Bad Gateway**
**Cause:** Backend not running or wrong port
```bash
# Check if Node.js is running
pm2 status

# Check logs
pm2 logs

# Restart
pm2 restart all
```

### **Issue 2: 403 Forbidden**
**Cause:** Wrong permissions
```bash
sudo chown -R www-data:www-data /var/www/myapp-frontend
sudo chmod -R 755 /var/www/myapp-frontend
```

### **Issue 3: Database Connection Failed**
```bash
# Check PostgreSQL is running
sudo systemctl status postgresql

# Check connection
psql -U myapp_user -d myapp_db -h localhost
```

### **Issue 4: Changes Not Showing**
```bash
# Clear browser cache
# Or add cache-busting to Nginx:
location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

---

# **Resources & Next Steps**

## **Essential Learning Resources**

### **Apache**
- [Official Apache Docs](https://httpd.apache.org/docs/2.4/)
- [DigitalOcean Apache Tutorials](https://www.digitalocean.com/community/tags/apache)
- [Apache Configuration Cheat Sheet](https://www.keycdn.com/support/apache-configuration)

### **Nginx**
- [Official Nginx Docs](https://nginx.org/en/docs/)
- [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- [DigitalOcean Nginx Tutorials](https://www.digitalocean.com/community/tags/nginx)

### **Deployment**
- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [Let's Encrypt (SSL)](https://letsencrypt.org/getting-started/)
- [Linux Server Security](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04)

### **DevOps Practice**
- [KodeKloud](https://kodekloud.com/) - Hands-on labs
- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/) - Linux practice
- [Nginx Playground](https://nginx-playground.wizardzines.com/) - Interactive learning

---

## **Practice Projects (In Order of Difficulty)**

### **Beginner**
1. Deploy a static HTML/CSS/JS site on Apache
2. Same site, but on Nginx
3. Host 3 different static sites on one server using URL paths

### **Intermediate**
4. Deploy a React app with client-side routing
5. Deploy Node.js API with PM2
6. Connect React frontend to Node.js backend on same server

### **Advanced**
7. Deploy full-stack app with database (React + Node + PostgreSQL)
8. Setup SSL with Let's Encrypt
9. Configure Nginx caching and compression
10. Setup CI/CD with GitHub Actions

---

## **Quick Reference Cheat Sheet**

```bash
# Apache
sudo systemctl start httpd
sudo systemctl reload httpd
sudo apachectl configtest
tail -f /var/log/httpd/error_log

# Nginx
sudo systemctl start nginx
sudo nginx -t
sudo systemctl reload nginx
tail -f /var/log/nginx/error.log

# PM2
pm2 start app.js
pm2 status
pm2 logs
pm2 restart all
pm2 monit

# Permissions
sudo chown -R www-data:www-data /var/www/
sudo chmod -R 755 /var/www/

# Firewall
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## **Your Next Steps**

1. **Today**: Re-read this guide, take notes on unclear parts
2. **Tomorrow**: Practice on Hostinger VPS - start with static site
3. **Day 3**: Deploy a React app
4. **Day 4**: Add a Node.js backend
5. **Day 5**: Connect to database
6. **Day 6**: Setup SSL and domain
7. **Week 2**: Build your own project and deploy it

**Remember:** Everyone feels confused at first. The confidence comes from doing it 5-10 times, not from reading once. Break things, fix them, Google errors. That's how you truly learn!
