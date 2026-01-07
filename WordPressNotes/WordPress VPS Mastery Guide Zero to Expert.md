# WordPress VPS Mastery Guide: Zero to Expert

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Essential Terminology](#essential-terminology)
3. [Pre-Setup Checklist](#pre-setup-checklist)
4. [Step-by-Step VPS Setup](#step-by-step-vps-setup)
5. [WordPress Installation](#wordpress-installation)
6. [Security Hardening](#security-hardening)
7. [Daily Practice Exercises](#daily-practice-exercises)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Resources for Deep Learning](#resources-for-deep-learning)

---

## Foundation Concepts

### What is a VPS? (Virtual Private Server)

**Simple Analogy**: Think of hosting like living spaces:
- **Shared Hosting** = Apartment building (you share resources with neighbors, if they're noisy, you're affected)
- **VPS Hosting** = Condo (you have your own space within a building, but it's isolated)
- **Dedicated Server** = Standalone house (everything is yours, but expensive)

**Technical Explanation**:
A VPS is a virtualized server created by dividing a powerful physical server into multiple isolated virtual machines. Each VPS has:
- Dedicated CPU cores
- Guaranteed RAM
- Dedicated storage space
- Its own operating system (usually Linux)
- Root/administrator access

**Why VPS for WordPress?**
1. **Performance**: Your site loads faster (2-3x faster than shared hosting)
2. **Control**: You can install custom software, configure server settings
3. **Scalability**: Easy to upgrade RAM, CPU, or storage as your site grows
4. **Security**: Isolated from other users
5. **Learning**: You learn real server administration skills

---

## Essential Terminology

### DNS (Domain Name System)
**What it is**: The internet's phonebook

**Simple Explanation**: 
- Humans remember: `www.mywebsite.com`
- Computers use: `192.168.1.1` (IP address)
- DNS translates the human-friendly name to computer-friendly numbers

**Real-world analogy**: Like calling someone by their name (John) instead of their phone number (555-1234)

**Key DNS Records**:
```
A Record     → Points domain to IPv4 address (192.168.1.1)
AAAA Record  → Points domain to IPv6 address
CNAME Record → Points subdomain to another domain (www → example.com)
MX Record    → Directs email to mail server
TXT Record   → Stores text information (used for verification)
```

**Example in Practice**:
```
Domain: mysite.com
A Record: mysite.com → 45.76.123.45 (your VPS IP)
A Record: www.mysite.com → 45.76.123.45
```

### SSL/TLS (Secure Sockets Layer/Transport Layer Security)

**What it is**: Encryption for web traffic

**Visual Representation**:
```
Without SSL (HTTP):
You → "My password is 12345" → Server
      ↑ Anyone can read this!

With SSL (HTTPS):
You → "Kj#mL90p!Xq" → Server
      ↑ Encrypted, unreadable
```

**Why it matters**:
- Protects login credentials
- Encrypts payment information
- Google ranks HTTPS sites higher
- Browsers show "Not Secure" warning without SSL

**Types of SSL**:
1. **Let's Encrypt** (Free) - Perfect for blogs, small businesses
2. **Paid SSL** ($50-200/year) - Comes with warranty, better for e-commerce

### IP Address (Internet Protocol Address)

**What it is**: Unique identifier for devices on a network

**Types**:
```
IPv4: 192.168.1.1 (older, running out)
IPv6: 2001:0db8:85a3:0000:0000:8a2e:0370:7334 (newer, unlimited)
```

**Public vs Private**:
- **Public IP**: Your VPS address on the internet (like your home street address)
- **Private IP**: Internal network addresses (like apartment numbers inside a building)

### Proxy Server

**What it is**: An intermediary server

**Simple Analogy**: Like a personal assistant who handles your calls
```
You → Proxy → Website
     ↑ Proxy makes requests on your behalf
```

**Common Types**:
1. **Reverse Proxy** (Nginx, Apache) - Sits in front of your web server
2. **CDN** (Cloudflare) - Caches your site globally for faster access
3. **VPN** - Encrypts all your internet traffic

**Why use it?**
- Load balancing (distribute traffic across multiple servers)
- Caching (store frequently accessed content)
- Security (hide your real server IP)

### Port Numbers

**What it is**: Doors to your server

**Common Ports**:
```
Port 80  → HTTP (web traffic, unencrypted)
Port 443 → HTTPS (web traffic, encrypted)
Port 22  → SSH (secure remote access)
Port 21  → FTP (file transfer)
Port 3306 → MySQL (database)
```

**Analogy**: Your server is a building. Ports are numbered doors. Each service uses a specific door.

### SSH (Secure Shell)

**What it is**: Secure way to remotely control your server

**Think of it as**: 
- Remote desktop, but text-based
- You type commands on your local computer that execute on the remote server

**Basic SSH Usage**:
```bash
# Connect to your server
ssh username@your-server-ip

# Example
ssh root@45.76.123.45
```

### Root Access

**What it is**: Superuser/administrator privileges on Linux

**Windows equivalent**: Administrator account

**Power & Responsibility**:
```bash
# Regular user
Cannot delete system files

# Root user (dangerous!)
rm -rf / # This deletes EVERYTHING!
```

**Best Practice**: Create a non-root user with sudo privileges for daily tasks

### Database (MySQL/MariaDB)

**What it is**: Organized storage for your website data

**Structure**:
```
Database: wordpress_db
  ↓
Tables: wp_posts, wp_users, wp_comments
  ↓
Rows: Individual posts, users, comments
  ↓
Columns: post_title, post_content, post_date
```

**WordPress uses database for**:
- Blog posts and pages
- User accounts
- Settings and configurations
- Comments
- Plugin/theme settings

### PHP

**What it is**: Programming language that powers WordPress

**How it works**:
```
1. User visits: mysite.com/blog
2. Server runs PHP code
3. PHP queries database for blog posts
4. PHP generates HTML page
5. Browser displays the page
```

**PHP Version Matters**: WordPress requires PHP 7.4+ (recommended: 8.1+)

---

## Pre-Setup Checklist

### Things You Need Before Starting

#### 1. Choose Your VPS Provider
**Hostinger** (as in your video):
- Pros: Affordable ($6/month), beginner-friendly panel, good support
- Cons: Smaller than competitors

**Alternatives to Research**:
- **DigitalOcean**: Developer-friendly, excellent documentation
- **Vultr**: Similar to DO, good performance
- **Linode**: Reliable, great support
- **AWS Lightsail**: Amazon's simplified VPS

#### 2. Pick a Domain Name
**Best Practices**:
- Short (under 15 characters)
- Easy to spell
- No numbers or hyphens
- .com extension preferred
- Relevant to your content

**Domain Registrars**:
- Namecheap
- Google Domains
- Cloudflare (cheapest, at-cost pricing)

#### 3. Decide on Your WordPress Setup
**Two Main Approaches**:

**A. Control Panel Method** (Easier - Recommended for Beginners)
- Cloud Panel (Hostinger's option)
- cPanel
- Plesk

**B. Manual Installation** (Learn more, more control)
- Direct SSH commands
- Understanding each component

#### 4. Tools You'll Need

**On Windows**:
```
1. PuTTY - SSH client (download from putty.org)
2. FileZilla - FTP client (for file uploads)
3. Notepad++ - Text editor for code
4. Browser (Chrome/Firefox with developer tools)
```

**On Ubuntu**:
```bash
# Already built-in!
1. Terminal (SSH client)
2. Nano/Vim (text editors)
3. FileZilla (install: sudo apt install filezilla)
```

---

## Step-by-Step VPS Setup

### Phase 1: Purchase and Initial Configuration

#### Step 1.1: Buy VPS Hosting

**On Hostinger**:
1. Visit hostinger.com
2. Navigate: Hosting → VPS Hosting
3. Choose plan based on your needs:

```
Traffic/Size Guide:
- KVM 1 (1GB RAM): Small blog (up to 5,000 monthly visitors)
- KVM 2 (2GB RAM): Medium site (5,000-20,000 visitors) ← Recommended
- KVM 4 (4GB RAM): High traffic (20,000-50,000 visitors)
- KVM 8 (8GB RAM): Very high traffic or e-commerce
```

4. Select 12-month plan (best value)
5. Apply coupon code: `vps1`
6. Complete payment

#### Step 1.2: Configure Your VPS

**Location Selection**:
```
If your audience is in:
- USA → New York or Los Angeles server
- Europe → Netherlands or UK server
- Asia → Singapore or India server
- Global → Use a CDN later (Cloudflare)
```

**Why location matters**: Each 1,000 miles adds ~15ms latency

**Operating System Selection**:

**Recommended: Ubuntu 22.04 LTS with Cloud Panel**

Why Ubuntu?
- Most popular Linux distribution
- Largest community (more tutorials)
- Long-term support (LTS)
- Best compatibility with WordPress

**Alternative OS Options**:
```
Ubuntu    → Best for beginners, most documentation
CentOS    → Enterprise-focused (discontinued, use Rocky)
Debian    → Stable, less frequent updates
Rocky     → CentOS replacement
AlmaLinux → Another CentOS alternative
```

#### Step 1.3: Control Panel Setup

**Cloud Panel Configuration**:

1. Set control panel password:
```
Requirements:
- Minimum 12 characters
- Mix of uppercase, lowercase
- Numbers and symbols
- Store in password manager (Bitwarden, LastPass, 1Password)
```

2. Set server hostname:
```
Good examples:
- server1.yourdomain.com
- vps-prod-01
- wordpress-main

Avoid:
- Generic names (server, vps)
- Personal info
```

3. Set root/admin password (make it different from panel password!)

#### Step 1.4: Understanding Your Dashboard

**H Panel Overview**:
```
Dashboard Shows:
├── Server Status
│   ├── CPU Usage
│   ├── RAM Usage
│   ├── Disk Space
│   └── Bandwidth
├── Server Details
│   ├── IP Address (SAVE THIS!)
│   ├── Operating System
│   └── Control Panel URL
└── Actions
    ├── Restart Server
    ├── Change Password
    └── Upgrade Plan
```

**What to note down immediately**:
```
VPS IP Address: _______________
Control Panel URL: _______________
Control Panel Username: admin
Control Panel Password: (in password manager)
Root Password: (in password manager)
```

---

### Phase 2: Domain Configuration

#### Step 2.1: Purchase Domain

**On Hostinger**:
1. Domains menu → Search domain
2. Check availability
3. Choose .com (most trusted)
4. 1-year registration minimum
5. Privacy protection (WHOIS guard) - Enable this!

**What is WHOIS Privacy?**
Without it, your personal info (name, address, email) is public in domain registry.

#### Step 2.2: Understanding DNS Configuration

**What we're doing**: Telling the internet "when someone types mysite.com, send them to IP address X.X.X.X"

**Visual Representation**:
```
User types: mysite.com
            ↓
        DNS Server
            ↓
    "mysite.com = 45.76.123.45"
            ↓
    User's browser connects to 45.76.123.45
            ↓
        Your VPS serves the website
```

#### Step 2.3: Point Domain to VPS

**In Hostinger H Panel**:

1. Go to: Domains → Select your domain → DNS/Name Servers
2. Click "DNS Records" tab
3. Delete existing A records (if any)
4. Add two new A records:

```
Record 1:
Type: A
Name: @ (represents root domain)
Points to: [Your VPS IP - from H Panel]
TTL: 14400 (4 hours)

Record 2:
Type: A
Name: www
Points to: [Your VPS IP]
TTL: 14400
```

**What this means**:
- `@` → Makes `mysite.com` work
- `www` → Makes `www.mysite.com` work
- Both point to same VPS IP

#### Step 2.4: DNS Propagation

**What is it?**: Time for DNS changes to spread globally

**Timeline**:
```
Immediate: 0-5 minutes (rare)
Typical: 1-4 hours
Maximum: 24-48 hours
```

**Check propagation status**:
1. Visit: whatsmydns.net
2. Enter your domain
3. Select "A" record type
4. Click Search

**What you should see**: Your VPS IP showing in most locations worldwide

**During propagation**:
- ✅ You can continue VPS setup
- ✅ Install WordPress
- ❌ Don't install SSL yet (domain must propagate first)

---

### Phase 3: WordPress Installation via Cloud Panel

#### Step 3.1: Access Cloud Panel

**Login Process**:

1. Get URL from H Panel: "Panel Access" section
2. URL format: `https://your-vps-ip:8443`
3. Browser security warning - Why?

```
Your browser sees:
"This site uses HTTPS but I don't recognize the certificate"

Why it happens:
VPS has a self-signed SSL (not verified by authority)

Is it safe?
YES - This is normal for control panels
Click "Advanced" → "Proceed anyway"
```

4. Username: `admin`
5. Password: (your control panel password)

#### Step 3.2: Create WordPress Site

**In Cloud Panel Dashboard**:

1. Click "Add Site"
2. Select "Create a WordPress Site"

**Configuration Fields**:

```
Domain Name: yourdomain.com
├── Enter without www
└── Panel adds both @ and www automatically

Site User: 
├── This is SSH/SFTP username
├── Choose: wordpress_user or wp_admin
└── Used for file access via FTP

Site Password:
├── For SSH/SFTP access
├── Different from WordPress admin password!
└── Generate strong password

WordPress Admin User:
├── Your WordPress dashboard username
└── Avoid "admin" (too common, security risk)

WordPress Admin Email:
├── Your email
└── Used for notifications and password resets

WordPress Admin Password:
├── Your WordPress login password
└── Store in password manager
```

3. Click "Create" - Wait 2-3 minutes

**What's happening behind the scenes**:
```
1. Panel creates database (wordpress_db)
2. Downloads WordPress files
3. Creates wp-config.php (WordPress configuration)
4. Sets up web server (Nginx)
5. Configures PHP
6. Sets proper file permissions
```

#### Step 3.3: Save Critical Information

**Create a secure note with**:
```
=== WORDPRESS SITE CREDENTIALS ===

Domain: yourdomain.com

--- WordPress Dashboard ---
URL: yourdomain.com/wp-admin
Username: [your admin username]
Password: [in password manager]
Email: [your email]

--- FTP/SSH Access ---
Host: yourdomain.com (or VPS IP)
Port: 22 (SSH) or 21 (FTP)
Username: [site user from panel]
Password: [site password from panel]

--- Database ---
Database Name: [shown in panel]
Database User: [shown in panel]
Database Password: [shown in panel]
Database Host: localhost

--- VPS Access ---
SSH: ssh root@[VPS-IP]
Root Password: [in password manager]
```

---

### Phase 4: SSL Certificate Installation

#### Step 4.1: Why SSL is Critical

**Before SSL (HTTP)**:
```
Browser → "username: john, password: 12345" → Server
          ↑ VISIBLE TO ANYONE MONITORING NETWORK
```

**After SSL (HTTPS)**:
```
Browser → "8fj#Kl@9mP0!qR..." → Server
          ↑ ENCRYPTED, UNREADABLE
```

**Requirements for SSL**:
- ✅ Domain must be propagated
- ✅ Website must be accessible via domain
- ✅ Ports 80 and 443 must be open

#### Step 4.2: Install Let's Encrypt SSL

**In Cloud Panel**:

1. Go to: Sites → Select your domain
2. Click "SSL/TLS" tab
3. You'll see self-signed certificate (ignore it)
4. Click "Actions" → "New Let's Encrypt Certificate"

**Configuration**:
```
Domain: yourdomain.com
Auto-renew: Enabled (SSL expires every 90 days)
Include www: Yes (covers www.yourdomain.com)
```

5. Click "Create and Install"

**What happens**:
```
1. Let's Encrypt verifies you own the domain
2. Issues free SSL certificate
3. Installs it on your server
4. Configures auto-renewal (runs every 60 days)
```

#### Step 4.3: Verify SSL Installation

**Tests to perform**:

1. **Browser test**:
```
Visit: https://yourdomain.com
Look for: 🔒 padlock icon in address bar
Click padlock → "Connection is secure"
```

2. **Online test**:
```
Visit: ssllabs.com/ssltest
Enter your domain
Wait for scan (2-3 minutes)
Target grade: A or A+
```

3. **Force HTTPS redirect**:
```
Try visiting: http://yourdomain.com (without s)
Should redirect to: https://yourdomain.com
```

---

### Phase 5: WordPress Configuration

#### Step 5.1: First Login

**Access WordPress Dashboard**:
```
URL: yourdomain.com/wp-admin
Enter credentials saved earlier
```

**First-time Setup Wizard** (if appears):
1. Select language
2. Site title: "Your Blog Name"
3. Tagline: Short description
4. Skip search engine visibility (for now)

#### Step 5.2: Essential Settings

**Settings → General**:
```
Site Title: Your website name
Tagline: Descriptive subtitle
WordPress Address (URL): https://yourdomain.com
Site Address (URL): https://yourdomain.com
Email Address: Your admin email
Timezone: Your timezone
Date Format: Choose preference
Time Format: Choose preference
```

**Settings → Permalinks**:
```
Recommended: Post name
URL structure: https://yourdomain.com/sample-post/

Why?
- SEO friendly
- Readable URLs
- Avoid: Plain (ugly ?p=123 URLs)
```

**Settings → Reading**:
```
Your homepage displays: Choose:
- Latest posts (for blog)
- Static page (for business site)

Posts per page: 10 (default, adjust based on content)
Search engine visibility: UNCHECK (allow indexing)
```

#### Step 5.3: Delete Default Content

**Clean slate**:
```
Posts → All Posts → Delete "Hello World"
Pages → All Pages → Delete "Sample Page"
Comments → Delete default comment
Themes → Delete unused default themes (keep one as backup)
Plugins → Delete "Hello Dolly" and "Akismet" (reinstall if needed)
```

#### Step 5.4: Install Essential Plugins

**Security**:
1. **Wordfence Security** (or Sucuri Security)
   - Firewall
   - Malware scanner
   - Login security

2. **UpdraftPlus** (backup)
   - Automatic backups
   - One-click restore
   - Cloud storage support

**Performance**:
3. **WP Super Cache** (or W3 Total Cache)
   - Page caching
   - Faster load times

4. **Smush** (image optimization)
   - Compress images automatically
   - Lazy loading

**SEO**:
5. **Yoast SEO** (or Rank Math)
   - SEO optimization
   - XML sitemaps
   - Meta descriptions

**Utility**:
6. **Classic Editor** (if you prefer old editor)
7. **Contact Form 7** (forms)

**Installation**:
```
Plugins → Add New → Search plugin name → Install Now → Activate
```

---

## Security Hardening

### Level 1: Basic Security (Do This First)

#### 1. Change WordPress Admin Username

**Problem**: Default "admin" username = 50% of hack success

**Solution**:
```
1. Create new admin user (strong username like "wp_john_482")
2. Assign Administrator role
3. Log out
4. Log in as new user
5. Delete old "admin" user
6. Assign old posts to new user
```

#### 2. Install Wordfence

**Configuration**:
```
Wordfence → All Options:

Basic Options:
✓ Enable Firewall
✓ Enable Malware Scan
✓ Enable Login Security

Login Security:
✓ Limit login attempts (3 tries, 20-minute lockout)
✓ Enable 2FA (two-factor authentication)
✓ Block admin username login
```

#### 3. Disable File Editing

**What**: Prevent hackers from editing theme/plugin files via dashboard

**How**: Edit `wp-config.php`:
```bash
# SSH into server
ssh root@your-vps-ip

# Navigate to WordPress root
cd /home/[site-user]/public_html

# Edit wp-config.php
nano wp-config.php

# Add this line before "That's all, stop editing!":
define('DISALLOW_FILE_EDIT', true);

# Save: CTRL+O, Enter, CTRL+X
```

#### 4. Change Database Prefix

**Default**: All tables start with `wp_`

**Why change**: Automated hacks target `wp_` prefix

**How** (careful!):
```bash
# Backup database first!

# In phpMyAdmin or database tool:
1. Backup database
2. Rename all tables from wp_ to [random]_
   Example: wp_posts → xyz123_posts
3. Update wp-config.php:
   $table_prefix = 'xyz123_';
4. Update options table:
   UPDATE xyz123_options SET option_name = replace(option_name, 'wp_', 'xyz123_')
5. Update usermeta table:
   UPDATE xyz123_usermeta SET meta_key = replace(meta_key, 'wp_', 'xyz123_')
```

### Level 2: Intermediate Security

#### 5. Two-Factor Authentication (2FA)

**Install**: Wordfence or Google Authenticator plugin

**Setup**:
```
1. Install Google Authenticator app on phone
2. WordPress → Users → Your Profile
3. Enable 2FA
4. Scan QR code with phone app
5. Save backup codes (in password manager)
```

#### 6. SFTP Instead of FTP

**Difference**:
```
FTP  → Unencrypted (passwords sent in plain text)
SFTP → Encrypted (secure file transfer)
```

**FileZilla Setup**:
```
Protocol: SFTP - SSH File Transfer Protocol
Host: yourdomain.com
Port: 22
Username: [site user from panel]
Password: [site password]
```

#### 7. Change SSH Port (Advanced)

**Why**: Default port 22 is constantly attacked

**How**:
```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Find line: #Port 22
# Change to: Port 2222 (or any number 1024-65535)

# Save and restart SSH
sudo systemctl restart sshd

# Update firewall
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp

# Future SSH connections:
ssh -p 2222 root@your-vps-ip
```

### Level 3: Advanced Security

#### 8. Fail2Ban (Automatic IP Blocking)

**What**: Automatically bans IPs after failed login attempts

**Install on Ubuntu**:
```bash
sudo apt update
sudo apt install fail2ban

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local

# Under [DEFAULT] section, set:
bantime = 1h
maxretry = 3
findtime = 10m

# Enable and start
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
```

#### 9. Cloudflare (Free CDN + DDoS Protection)

**What it does**:
```
User → Cloudflare → Your Server
       ↑ Blocks attacks, caches content, hides real IP
```

**Setup**:
```
1. Create account at cloudflare.com
2. Add site (enter your domain)
3. Copy provided nameservers
4. Change nameservers at Hostinger:
   - Domains → DNS/Nameservers
   - Change to Cloudflare's nameservers
5. Wait 24 hours for propagation
6. Enable "Proxy" (orange cloud) for A records
```

**Benefits**:
- Free SSL
- DDoS protection
- Global CDN (faster site worldwide)
- Hides your real server IP

#### 10. Regular Backups

**Strategy**:
```
Daily: Database backup (small, critical)
Weekly: Full site backup (files + database)
Monthly: Download backup to local computer
```

**UpdraftPlus Configuration**:
```
Settings → Existing Backups:

Files backup schedule: Weekly
Database backup schedule: Daily

Remote Storage:
✓ Google Drive (free 15GB)
or ✓ Dropbox
or ✓ Amazon S3

Backup retention: Keep 4 backups
```

**Manual Backup via SSH**:
```bash
# Backup database
mysqldump -u [db_user] -p [db_name] > backup_$(date +%Y%m%d).sql

# Backup files (entire WordPress directory)
tar -czf wordpress_backup_$(date +%Y%m%d).tar.gz /home/[site-user]/public_html

# Download to local machine (from your computer):
scp root@your-vps-ip:/path/to/backup_20240107.sql ~/Downloads/
```

---

## Daily Practice Exercises

### Week 1: Foundation Building

**Day 1-2: Linux Command Line Basics**
```bash
# Navigate directories
pwd              # Show current directory
ls               # List files
ls -la           # List all files with details
cd /path         # Change directory
cd ..            # Go up one level
cd ~             # Go to home directory

# File operations
touch file.txt   # Create empty file
nano file.txt    # Edit file
cat file.txt     # Display file contents
cp file1 file2   # Copy file
mv file1 file2   # Move/rename file
rm file.txt      # Delete file (careful!)

# Directory operations
mkdir folder     # Create directory
rmdir folder     # Remove empty directory
rm -rf folder    # Remove directory and contents (DANGEROUS!)

# Practice task:
# Create a test directory, create files, move them around, delete them
```

**Day 3-4: Understanding WordPress File Structure**
```bash
# SSH into your server
ssh root@your-vps-ip

# Navigate to WordPress directory
cd /home/[site-user]/public_html

# Explore structure
ls -la

# You'll see:
wp-admin/        # WordPress dashboard files
wp-content/      # Themes, plugins, uploads
  ├── themes/
  ├── plugins/
  └── uploads/
wp-includes/     # WordPress core files
wp-config.php    # Main configuration file
index.php        # Entry point

# Practice task:
# Find your active theme folder
# List all installed plugins
# Check uploads folder size
```

**Day 5-6: WordPress Dashboard Mastery**
```
Task list:
□ Create 5 test posts
□ Create 3 pages (About, Contact, Privacy)
□ Upload 10 images to Media Library
□ Install and activate 3 different themes (test each)
□ Install 5 plugins and understand what each does
□ Create categories and tags
□ Create menu and assign to location
□ Customize theme colors/fonts
□ Add widgets to sidebar
□ Create a user with Editor role
```

**Day 7: Review and Document**
```
Create a personal wiki/notes with:
□ Every Linux command you learned
□ What each WordPress file/folder does
□ Common problems you encountered
□ How you solved them
□ Questions for further research
```

### Week 2: Intermediate Skills

**Day 8-9: Database Operations**
```bash
# SSH into server
ssh root@your-vps-ip

# Access MySQL
mysql -u root -p

# Basic SQL commands
SHOW DATABASES;
USE wordpress_db;
SHOW TABLES;
DESCRIBE wp_posts;
SELECT * FROM wp_posts LIMIT 5;
SELECT * FROM wp_users;

# Practice task:
# Count total posts: SELECT COUNT(*) FROM wp_posts WHERE post_status='publish';
# Find all admin users: SELECT * FROM wp_users;
# Backup database (from bash): mysqldump -u [user] -p [db_name] > backup.sql
```

**Day 10-11: Theme Customization**
```php
# Understand theme structure
themes/your-theme/
├── style.css        # Main stylesheet
├── functions.php    # Theme functions
├── index.php        # Main template
├── header.php       # Header template
├── footer.php       # Footer template
├── sidebar.php      # Sidebar template
└── single.php       # Single post template

# Practice task:
# Create child theme (safe way to customize)
# Modify header.php to add custom text
# Edit footer.php to change copyright
# Add custom CSS to style.css
```

**Day 12-13: Plugin Development Basics**
```php
# Create simple plugin
# File: wp-content/plugins/my-first-plugin/my-first-plugin.php

<?php
/*
Plugin Name: My First Plugin
Description: Learning plugin development
Version: 1.0
Author: Your Name
*/

// Add action hook
add_action('wp_footer', 'my_custom_footer_text');

function my_custom_footer_text() {
    echo '<p>This text added by my plugin!</p>';
}
?>

# Practice task:
# Create plugin that adds text to footer
# Create plugin that registers custom post type
# Create plugin that adds dashboard widget
```

**Day 14: Performance Optimization**
```
Tasks:
□ Install Query Monitor plugin (see slow queries)
□ Install P3 Plugin Profiler (see slow plugins)
□ Optimize all images with Smush
□ Enable caching plugin
□ Test site speed on GTmetrix.com and PageSpeed Insights
□ Remove unused plugins/themes
□ Update all plugins/themes/core
□ Enable Gzip compression
```

### Week 3: Advanced Topics

**Day 15-16: Security Hardening**
```bash
# Practice tasks:
□ Set up Fail2Ban and test it
□ Configure Wordfence firewall rules
□ Set up Cloudflare
□ Create database backup script
□ Set up automatic backups with cron
□ Test restore from backup
□ Scan site for malware


# WordPress VPS Resources & Troubleshooting Guide

## Part 2: Advanced Learning & Problem Solving

---

## Week 3-4: Advanced Topics (Continued)

### Day 17-18: SSH & Server Management

**Advanced SSH Techniques**:
```bash
# Key-based authentication (more secure than passwords)

# On your local machine (Ubuntu or Windows with WSL):
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Press Enter to save in default location
# Set a strong passphrase

# Copy public key to server:
ssh-copy-id root@your-vps-ip
# Or manually:
cat ~/.ssh/id_rsa.pub | ssh root@your-vps-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Now you can login without password:
ssh root@your-vps-ip

# Disable password authentication for security:
sudo nano /etc/ssh/sshd_config
# Find and change:
PasswordAuthentication no
# Save and restart SSH:
sudo systemctl restart sshd
```

**Server Monitoring**:
```bash
# Check system resources
htop                    # Interactive process viewer
free -h                 # Memory usage
df -h                   # Disk usage
du -sh /path/to/folder  # Folder size
netstat -tuln           # Check open ports

# Check logs
tail -f /var/log/nginx/access.log   # Watch web traffic live
tail -f /var/log/nginx/error.log    # Watch errors
grep "error" /var/log/syslog        # Search for errors

# Process management
ps aux | grep nginx     # Find nginx processes
ps aux | grep php       # Find PHP processes
kill [PID]              # Kill process by ID
systemctl restart nginx # Restart web server
systemctl restart php8.1-fpm  # Restart PHP
```

### Day 19-20: Git Version Control

**Why use Git with WordPress**:
- Track all changes to theme/plugin code
- Collaborate with others
- Easy rollback if something breaks
- Deploy changes systematically

**Setup Git for WordPress**:
```bash
# SSH into server
ssh root@your-vps-ip

# Navigate to WordPress directory
cd /home/[site-user]/public_html

# Initialize git (only for themes/plugins, not entire WordPress)
cd wp-content/themes/your-theme
git init
git add .
git commit -m "Initial theme commit"

# Create .gitignore
nano .gitignore
# Add:
*.log
wp-config.php
.htaccess
wp-content/uploads/

# Connect to GitHub
git remote add origin https://github.com/yourusername/your-repo.git
git push -u origin main
```

**Daily Git Workflow**:
```bash
# Before making changes
git status              # Check current state
git pull                # Get latest changes

# After making changes
git status              # See what changed
git add filename.php    # Stage specific file
git add .               # Stage all changes
git commit -m "Description of changes"
git push                # Push to remote repository

# If you mess up
git log                 # See history
git checkout -- filename.php  # Discard changes to file
git reset HEAD~1        # Undo last commit (keep changes)
git reset --hard HEAD~1 # Undo last commit (lose changes!)
```

### Day 21: WP-CLI (WordPress Command Line)

**What is WP-CLI**: Manage WordPress from command line (super powerful!)

**Installation**:
```bash
# Download WP-CLI
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

# Make it executable
chmod +x wp-cli.phar

# Move to system path
sudo mv wp-cli.phar /usr/local/bin/wp

# Test
wp --info
```

**Common WP-CLI Commands**:
```bash
# Navigate to WordPress directory first
cd /home/[site-user]/public_html

# Update WordPress core
wp core update

# Update all plugins
wp plugin update --all

# List all plugins
wp plugin list

# Install and activate plugin
wp plugin install wordfence --activate

# Create new post
wp post create --post_title="Test Post" --post_content="Content here" --post_status=publish

# Search and replace in database (careful!)
wp search-replace 'http://oldsite.com' 'https://newsite.com'

# Reset admin password
wp user update admin_username --user_pass=new_password

# Database operations
wp db export backup.sql       # Export database
wp db import backup.sql       # Import database
wp db optimize                # Optimize database

# Clear cache
wp cache flush

# Generate fake content for testing
wp post generate --count=50
```

---

## Troubleshooting Guide

### Common Problems & Solutions

#### Problem 1: "Error Establishing Database Connection"

**Symptoms**: White page with this error message

**Causes & Solutions**:

```bash
1. Check database credentials
   # Edit wp-config.php
   nano wp-config.php
   
   # Verify these are correct:
   define('DB_NAME', 'your_database');
   define('DB_USER', 'your_username');
   define('DB_PASSWORD', 'your_password');
   define('DB_HOST', 'localhost');  # Sometimes needs server IP

2. Check if MySQL is running
   systemctl status mysql
   # If not running:
   sudo systemctl start mysql

3. Check database exists
   mysql -u root -p
   SHOW DATABASES;
   # If your database is missing, restore from backup

4. Check user permissions
   mysql -u root -p
   GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';
   FLUSH PRIVILEGES;
```

#### Problem 2: White Screen of Death (WSOD)

**Symptoms**: Blank white page, no error message

**Solutions**:

```bash
1. Enable debugging
   # Edit wp-config.php
   nano wp-config.php
   
   # Add before "That's all, stop editing":
   define('WP_DEBUG', true);
   define('WP_DEBUG_LOG', true);
   define('WP_DEBUG_DISPLAY', false);
   
   # Check debug log:
   tail -f wp-content/debug.log

2. Disable all plugins via database
   mysql -u root -p
   USE your_database;
   UPDATE wp_options SET option_value = '' WHERE option_name = 'active_plugins';

3. Switch to default theme
   # Rename your theme folder
   mv wp-content/themes/your-theme wp-content/themes/your-theme-backup
   # WordPress will automatically use default theme

4. Increase PHP memory limit
   nano wp-config.php
   # Add:
   define('WP_MEMORY_LIMIT', '256M');

5. Check error logs
   tail -100 /var/log/nginx/error.log
   tail -100 /var/log/php8.1-fpm.log
```

#### Problem 3: 500 Internal Server Error

**Symptoms**: Generic server error message

**Solutions**:

```bash
1. Check .htaccess file
   # Rename temporarily
   mv .htaccess .htaccess-backup
   # Try accessing site
   # If works, regenerate permalinks in WordPress admin

2. Increase PHP limits
   # Find php.ini
   php --ini
   # Edit php.ini
   sudo nano /etc/php/8.1/fpm/php.ini
   
   # Increase these values:
   memory_limit = 256M
   max_execution_time = 300
   upload_max_filesize = 64M
   post_max_size = 64M
   
   # Restart PHP
   sudo systemctl restart php8.1-fpm

3. Check file permissions
   # WordPress files should be:
   find /home/[site-user]/public_html -type d -exec chmod 755 {} \;
   find /home/[site-user]/public_html -type f -exec chmod 644 {} \;
   
   # wp-config.php should be more restrictive:
   chmod 600 wp-config.php

4. Check server logs
   tail -50 /var/log/nginx/error.log
```

#### Problem 4: Site Hacked/Malware

**Symptoms**: Redirects, spam links, defaced pages

**Immediate Actions**:

```bash
1. Take site offline
   # Create maintenance mode
   nano .maintenance
   # Add:
   <?php $upgrading = time(); ?>

2. Change ALL passwords
   - WordPress admin
   - Database user
   - FTP/SSH
   - Hosting control panel

3. Scan for malware
   # Install Wordfence
   wp plugin install wordfence --activate
   
   # Or use command line scanner
   sudo apt install clamav
   sudo freshclam  # Update virus definitions
   sudo clamscan -r /home/[site-user]/public_html

4. Check for backdoors
   # Look for suspicious recently modified files
   find /home/[site-user]/public_html -type f -mtime -7 -ls
   
   # Search for common backdoor functions
   grep -r "eval(base64_decode" /home/[site-user]/public_html
   grep -r "base64_decode" /home/[site-user]/public_html
   grep -r "system(" /home/[site-user]/public_html
   grep -r "exec(" /home/[site-user]/public_html

5. Clean and restore
   # Delete WordPress core and reinstall (keeps content)
   wp core download --force
   
   # Or restore from clean backup
   # Delete all files except wp-content/uploads
   # Restore from backup
   # Reinstall WordPress

6. Harden security
   - Update everything (WordPress, themes, plugins)
   - Remove unused themes/plugins
   - Install security plugin
   - Enable 2FA
   - Change database prefix
   - Disable file editing
```

#### Problem 5: Slow Website

**Diagnostic Process**:

```bash
1. Test speed
   # Visit these sites:
   - GTmetrix.com
   - PageSpeed Insights
   - Pingdom Tools
   
   # Note:
   - Load time (should be < 3 seconds)
   - Page size (should be < 3MB)
   - Number of requests (should be < 50)

2. Identify bottleneck
   # Install Query Monitor plugin
   wp plugin install query-monitor --activate
   
   # Check in WP admin:
   - Slow database queries
   - Memory usage
   - PHP errors

3. Common fixes:

   A. Optimize images
      wp plugin install smush --activate
      # Or use command line:
      sudo apt install jpegoptim optipng
      find wp-content/uploads -name "*.jpg" -exec jpegoptim --strip-all {} \;
      find wp-content/uploads -name "*.png" -exec optipng -o5 {} \;

   B. Enable caching
      wp plugin install wp-super-cache --activate
      # Configure in Settings → WP Super Cache

   C. Use CDN (Cloudflare)
      # Already covered in security section

   D. Optimize database
      wp db optimize
      
      # Or manually:
      mysql -u root -p
      USE your_database;
      OPTIMIZE TABLE wp_posts, wp_postmeta, wp_options;

   E. Reduce plugins
      wp plugin list
      # Deactivate and delete unused plugins

   F. Enable Gzip compression
      # Add to nginx config:
      sudo nano /etc/nginx/nginx.conf
      
      # Add in http block:
      gzip on;
      gzip_vary on;
      gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
      
      # Restart nginx:
      sudo systemctl restart nginx

4. Server-level optimization
   # Enable PHP OPcache
   sudo nano /etc/php/8.1/fpm/php.ini
   
   # Find and set:
   opcache.enable=1
   opcache.memory_consumption=128
   opcache.max_accelerated_files=10000
   opcache.revalidate_freq=60
   
   # Restart PHP:
   sudo systemctl restart php8.1-fpm
```

#### Problem 6: Can't Upload Files/Images

**Symptoms**: Upload fails, "HTTP error", file size errors

**Solutions**:

```bash
1. Check PHP upload limits
   sudo nano /etc/php/8.1/fpm/php.ini
   
   # Increase these:
   upload_max_filesize = 64M
   post_max_size = 64M
   max_execution_time = 300
   
   # Restart PHP:
   sudo systemctl restart php8.1-fpm

2. Check nginx upload limits
   sudo nano /etc/nginx/nginx.conf
   
   # Add in http block:
   client_max_body_size 64M;
   
   # Restart nginx:
   sudo systemctl restart nginx

3. Check disk space
   df -h
   # If disk is full, delete old backups, optimize database

4. Check folder permissions
   # uploads folder must be writable
   chmod 755 wp-content/uploads
   chown -R www-data:www-data wp-content/uploads
```

#### Problem 7: Forgot WordPress Admin Password

**Solutions**:

```bash
1. Via WP-CLI (easiest)
   cd /home/[site-user]/public_html
   wp user list  # Get username
   wp user update USERNAME --user_pass=new_password

2. Via MySQL
   mysql -u root -p
   USE your_database;
   UPDATE wp_users SET user_pass=MD5('new_password') WHERE user_login='admin';
   # Better: use wp_hash_password()
   
3. Via phpMyAdmin (if installed)
   - Login to phpMyAdmin
   - Select database
   - Browse wp_users table
   - Edit admin user row
   - Change user_pass to MD5 hash of new password

4. Via PHP file (last resort)
   # Create reset.php in WordPress root:
   nano reset.php
   
   # Add:
   <?php
   require_once('wp-load.php');
   wp_set_password('new_password', 1);
   echo 'Password Reset!';
   ?>
   
   # Visit: yourdomain.com/reset.php
   # DELETE reset.php immediately after!
```

---

## Learning Resources

### Official Documentation

**WordPress Codex & Developer Docs**:
```
https://developer.wordpress.org/
├── Themes
│   └── https://developer.wordpress.org/themes/
├── Plugins
│   └── https://developer.wordpress.org/plugins/
├── REST API
│   └── https://developer.wordpress.org/rest-api/
└── Block Editor
    └── https://developer.wordpress.org/block-editor/
```

**WordPress Support Forums**:
- https://wordpress.org/support/
- Search before asking
- Be specific with error messages
- Include WordPress version, theme, plugins

### Linux & Server Administration

**Free Courses**:
1. **Linux Journey** (linuxjourney.com)
   - Interactive, beginner-friendly
   - Covers everything from basics to advanced

2. **OverTheWire: Bandit** (overthewire.org/wargames/bandit/)
   - Learn Linux command line through games
   - Engaging and practical

3. **DigitalOcean Tutorials**
   - https://www.digitalocean.com/community/tutorials
   - Excellent step-by-step guides
   - Ubuntu-focused

**Books** (free PDFs available):
- "The Linux Command Line" by William Shotts
- "Linux Basics for Hackers" by OccupyTheWeb
- "Ubuntu Server Guide" (official)

### WordPress Development

**Free Courses**:
1. **WordPress.tv**
   - https://wordpress.tv/
   - Free video tutorials from WordCamps

2. **WPBeginner**
   - https://www.wpbeginner.com/
   - Beginner-friendly tutorials
   - Regular updates

3. **Smashing Magazine**
   - https://www.smashingmagazine.com/category/wordpress
   - Advanced techniques

**YouTube Channels**:
- **WPCrafter** - General WordPress
- **Elegant Themes** - Divi builder focused
- **WPEngine** - Enterprise WordPress
- **Traversy Media** - Web development including WP

### Security

**Resources**:
1. **OWASP Top 10**
   - https://owasp.org/www-project-top-ten/
   - Learn most common security vulnerabilities

2. **WordPress Security Whitepaper**
   - https://wordpress.org/about/security/
   - Official security best practices

3. **Sucuri Blog**
   - https://blog.sucuri.net/
   - Security news and guides

4. **Wordfence Blog**
   - https://www.wordfence.com/blog/
   - Threat intelligence, tutorials

### PHP Programming

**If you want to develop themes/plugins**:

1. **PHP Manual**
   - https://www.php.net/manual/en/
   - Official documentation

2. **PHP The Right Way**
   - https://phptherightway.com/
   - Best practices

3. **Laracasts** (paid but excellent)
   - https://laracasts.com/
   - Video tutorials

**Practice Projects**:
- Create custom WordPress shortcodes
- Build a simple plugin (todo list)
- Create child theme with custom features
- Build custom post types

### Networking & DNS

**Understanding DNS**:
1. **How DNS Works (Comic)**
   - https://howdns.works/
   - Visual, easy to understand

2. **Cloudflare Learning Center**
   - https://www.cloudflare.com/learning/
   - Comprehensive networking guides

**Tools to Practice**:
- `dig` command (DNS lookup)
- `nslookup` (DNS queries)
- `traceroute` (network path)
- `ping` (connectivity test)

### Performance Optimization

**Testing Tools**:
- GTmetrix.com (detailed analysis)
- PageSpeed Insights (Google's tool)
- WebPageTest.org (advanced testing)
- Pingdom Tools

**Learning Resources**:
1. **WP Rocket Blog**
   - https://wp-rocket.me/blog/
   - Caching and optimization

2. **KeyCDN Blog**
   - https://www.keycdn.com/blog/
   - CDN and performance

### Community & Support

**Forums & Groups**:
1. **WordPress.org Forums**
   - Official support

2. **Reddit**
   - r/WordPress
   - r/webdev
   - r/webhosting

3. **Stack Exchange**
   - wordpress.stackexchange.com
   - serverfault.com

4. **Discord/Slack**
   - WordPress Slack: make.wordpress.org/chat
   - WebDev Discord servers

---

## Your Learning Roadmap

### Month 1: Foundation
```
Week 1: VPS Setup & Basic Linux
□ Set up VPS
□ Learn basic Linux commands
□ Install WordPress manually (not via panel)
□ Understand file structure

Week 2: WordPress Basics
□ Master WordPress dashboard
□ Install themes/plugins
□ Create content
□ Understand permalinks, media library

Week 3: Security Basics
□ Install Wordfence
□ Set up backups
□ Change default settings
□ Enable 2FA

Week 4: Basic Customization
□ Install and customize theme
□ Basic CSS changes
□ Widget areas
□ Menu creation
```

### Month 2: Intermediate
```
Week 1: Database & PHP
□ Learn MySQL basics
□ Understand wp-config.php
□ Basic PHP syntax
□ WordPress template hierarchy

Week 2: Theme Development
□ Create child theme
□ Understand theme files
□ Custom templates
□ Template tags

Week 3: Plugin Development
□ First simple plugin
□ WordPress hooks (actions/filters)
□ Plugin best practices

Week 4: Performance
□ Caching setup
□ Image optimization
□ Database optimization
□ Speed testing
```

### Month 3: Advanced
```
Week 1: Advanced Security
□ Fail2Ban setup
□ Hardened SSH
□ Cloudflare configuration
□ Security audit

Week 2: Git & Deployment
□ Version control
□ GitHub integration
□ Deployment workflows

Week 3: WP-CLI Mastery
□ All common commands
□ Custom scripts
□ Automation

Week 4: Scaling & Optimization
□ Load balancing concepts
□ Caching strategies
□ CDN implementation
□ Database replication basics
```

### Month 4-6: Specialization
**Choose your path**:

**Path A: Theme Developer**
- Advanced CSS/Sass
- JavaScript/jQuery
- React (for Gutenberg blocks)
- Theme frameworks (Genesis, Underscores)

**Path B: Plugin Developer**
- Advanced PHP
- WordPress APIs
- Database optimization
- Creating commercial plugins

**Path C: Systems Administrator**
- Advanced Linux
- Server clustering
- Load balancing
- DevOps tools (Docker, Kubernetes)

**Path D: Security Specialist**
- Penetration testing
- Security auditing
- Malware analysis
- Incident response

---

## Quick Reference Cheat Sheet

### Essential Linux Commands
```bash
# Navigation
cd [directory]    # Change directory
pwd               # Show current path
ls -la            # List all files

# File operations
cp [source] [dest]    # Copy
mv [source] [dest]    # Move/rename
rm [file]             # Delete
nano [file]           # Edit file

# Permissions
chmod 755 [file]      # Set permissions
chown user:group [file]  # Change owner

# System
systemctl restart nginx    # Restart service
systemctl status mysql     # Check status
tail -f /var/log/file.log  # Watch log

# Search
grep -r "text" .      # Search in files
find . -name "*.php"  # Find files
```

### Essential WP-CLI Commands
```bash
wp core update              # Update WordPress
wp plugin list              # List plugins
wp plugin update --all      # Update all plugins
wp db export backup.sql     # Backup database
wp cache flush              # Clear cache
wp search-replace old new   # Find and replace
```

### Essential MySQL Commands
```sql
SHOW DATABASES;                    -- List databases
USE database_name;                 -- Select database
SHOW TABLES;                       -- List tables
SELECT * FROM wp_users;            -- View users
DESCRIBE table_name;               -- Show table structure
```

### Common File Locations
```
WordPress Root: /home/[user]/public_html/
Config File: /home/[user]/public_html/wp-config.php
Nginx Config: /etc/nginx/sites-available/
PHP Config: /etc/php/8.1/fpm/php.ini
MySQL Config: /etc/mysql/mysql.conf.d/mysqld.cnf
Error Logs: /var/log/nginx/error.log
```

---

## Final Tips for Success

### 1. Document Everything
Create a personal wiki/notes with:
- Every command you use
- Problems you encounter and solutions
- Configuration changes
- Passwords and credentials (encrypted!)

### 2. Practice Regularly
- Spend 30-60 minutes daily
- Hands-on practice > reading
- Break things in test environment
- Learn by fixing mistakes

### 3. Join Community
- Ask questions on forums
- Help others (best way to learn)
- Attend WordPress meetups (meetup.com)
- Follow WordPress developers on Twitter

### 4. Build Real Projects
Theory is important, but build:
- Personal blog
- Portfolio site
- Client project (even if free)
- Contribute to open source

### 5. Stay Updated
WordPress releases updates every 3-4 months:
- Follow WordPress.org blog
- Subscribe to security newsletters
- Test updates on staging first
- Read changelogs

### 6. Never Stop Learning
Technology evolves constantly:
- New PHP versions
- New WordPress features
- New security threats
- New best practices

---

## Conclusion

This guide covers a lot, but remember:
- **Don't rush** - Master basics before advancing
- **Make mistakes** - They're the best teacher
- **Ask questions** - Community is helpful
- **Practice daily** - Consistency beats intensity
- **Have fun** - Enjoy the learning journey!

You now have everything needed to go from absolute beginner to confident WordPress VPS administrator. Take it step by step, refer back to this guide, and most importantly - start doing!
