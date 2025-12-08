# Complete Networking Guide for DevOps Engineers

A comprehensive, practical guide to networking fundamentals essential for DevOps roles.
---
## Table of Contents
1. [What is Networking & Why It Matters](#1-what-is-networking--why-it-matters)
2. [Network Types & Topologies](#2-network-types--topologies)
3. [Network Hardware Components](#3-network-hardware-components)
4. [IP Addressing Deep Dive](#4-ip-addressing-deep-dive)
5. [Subnetting Explained Simply](#5-subnetting-explained-simply)
6. [DNS - The Internet's Phone Book](#6-dns---the-internets-phone-book)
7. [Network Protocols](#7-network-protocols)
8. [OSI & TCP/IP Models](#8-osi--tcpip-models)
9. [Ports & Services](#9-ports--services)
10. [Routing Explained](#10-routing-explained)
11. [Network Troubleshooting Tools](#11-network-troubleshooting-tools)
12. [Security Concepts](#12-security-concepts)
13. [DevOps-Specific Scenarios](#13-devops-specific-scenarios)

---

## 1. What is Networking & Why It Matters

### The Simple Truth
Networking is **how computers talk to each other**. That's it. Everything else is just different ways of making that conversation happen.

### Why DevOps Engineers MUST Know This
```
Your Job → Deploy applications → Applications need to talk
                                 ↓
                    Servers, databases, users
                                 ↓
                         ALL use networking
```

**Real-world examples:**
- User can't access website → Networking problem
- Database connection fails → Networking problem  
- Container can't talk to another container → Networking problem
- CI/CD pipeline stuck → Could be networking

---

## 2. Network Types & Topologies

### LAN (Local Area Network)
**Think:** Your home/office WiFi

```
[Your Laptop] ←→ [Router] ←→ [Printer]
                    ↕
              [Smart TV]
```

**Key Points:**
- Small geographical area
- Fast speeds (100 Mbps - 10 Gbps)
- You own/control it
- Example: Office network, home network

### WAN (Wide Area Network)
**Think:** The Internet

```
[Your Office LAN] ←→ Internet ←→ [AWS Data Center]
                        ↕
                [Google Servers]
```

**Key Points:**
- Large geographical area (cities, countries, world)
- Slower than LAN
- Owned by telecom companies
- Example: Internet, corporate networks connecting multiple offices

### MAN (Metropolitan Area Network)
**Think:** City-wide network
- Bigger than LAN, smaller than WAN
- Example: University campus network

---

## 3. Network Hardware Components

### Router
**What it does:** Connects different networks together

```
Internet ←→ [ROUTER] ←→ Your Home Network
                ↓
         Decides where data goes
```

**Real example:**
```
Package delivery analogy:
Your home address: 192.168.1.5
Amazon warehouse: 54.239.28.85

Router = Post office that knows how to get packages between these addresses
```

### Switch
**What it does:** Connects devices within the SAME network

```
[Computer1] ─┐
[Computer2] ─┼─ [SWITCH] ─ [Router] → Internet
[Computer3] ─┘
```

**Key difference from Router:**
- Switch = Connects devices in same network (like a power strip)
- Router = Connects different networks (like a bridge between islands)

### Modem
**What it does:** Converts signals between your ISP and your network

```
ISP (cable/fiber) → [MODEM] → Digital signal → [Router] → Your devices
```

### Hub (Mostly obsolete)
- Dumb device that broadcasts to everyone
- Replaced by switches
- Don't use these anymore

### NIC (Network Interface Card)
**What it does:** Allows computer to connect to network

```
Every device has one:
- Your laptop: WiFi card
- Server: Ethernet port
- Phone: WiFi/Cellular chip
```

**Important:** Each NIC has a unique **MAC address**

---

## 4. IP Addressing Deep Dive

### What is an IP Address?

**Simple analogy:** Like your home address, but for computers

```
Street Address          IP Address
123 Main Street    →    192.168.1.100
City, State            Network, Host
```

### IPv4 (What you'll use 99% of the time)

**Structure:**
```
192.168.1.100
 ↓   ↓   ↓  ↓
 |   |   |  └── Numbers 0-255
 |   |   └───── 4 groups (octets)
 |   └────────── Separated by dots
 └────────────── Each octet is 8 bits
```

**Why 0-255?**
```
8 bits = 2^8 = 256 possible values (0-255)

Binary to Decimal:
11000000 = 192
10101000 = 168
00000001 = 1
01100100 = 100
```

### IP Address Classes

**Think of these as neighborhood types:**

#### Class A: Huge Organizations
```
Range: 1.0.0.0 to 126.0.0.0
Example: 10.0.0.1

Format: Network.Host.Host.Host
        [10].    [0].  [0].  [1]
         ↑        ↑     ↑     ↑
      Network    └─────┴─────┘
                  Your devices
```

**Can have:** 16 million+ devices
**Used by:** Google, Facebook, huge companies

#### Class B: Medium Organizations
```
Range: 128.0.0.0 to 191.255.0.0
Example: 172.16.0.1

Format: Network.Network.Host.Host
        [172].   [16].   [0].  [1]
         ↑        ↑       ↑     ↑
      Network    └───────┴────┘
                  Your devices
```

**Can have:** 65,000+ devices
**Used by:** Universities, medium companies

#### Class C: Small Organizations
```
Range: 192.0.0.0 to 223.255.255.0
Example: 192.168.1.1

Format: Network.Network.Network.Host
        [192].  [168].   [1].    [1]
         ↑       ↑        ↑       ↑
            Network            Host
```

**Can have:** 254 devices
**Used by:** Homes, small offices, your WiFi

### Special IP Addresses

#### 127.0.0.1 (Localhost/Loopback)
```
Your computer talking to itself

[Your Computer]
     ↓
127.0.0.1
     ↑
[Your Computer]

Uses:
- Testing web servers locally
- Development
- "Is my network stack working?"
```

#### Private IP Ranges (Non-routable on Internet)
```
Class A: 10.0.0.0    to 10.255.255.255
Class B: 172.16.0.0  to 172.31.255.255
Class C: 192.168.0.0 to 192.168.255.255

These ONLY work inside your local network
```

**Why private IPs exist:**
```
Problem: Not enough IPv4 addresses for everyone

Solution: Use private IPs inside networks,
          NAT translates to public IP when going to Internet

Your home:
Device1: 192.168.1.10 ─┐
Device2: 192.168.1.11 ─┼→ [Router/NAT] → Public IP: 203.0.113.1 → Internet
Device3: 192.168.1.12 ─┘
```

#### 0.0.0.0 (Special "Any" Address)
```
Means: "All IP addresses on this machine"

Used when:
- Server listens on all network interfaces
- Default route in routing tables
```

### Public vs Private IPs

```
PUBLIC IP (Example: 8.8.8.8)
- Unique across entire Internet
- Can be reached from anywhere
- Costs money (usually)
- Assigned by ISP or cloud provider

PRIVATE IP (Example: 192.168.1.5)
- Only unique in YOUR network
- Cannot be reached from Internet
- Free
- You assign them yourself
```

### IPv6 (The Future)
```
Format: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

Why it exists: We ran out of IPv4 addresses
How many addresses: 340 undecillion (basically infinite)

Current reality: Most systems use both IPv4 and IPv6
```

---

## 5. Subnetting Explained Simply

### What Problem Does Subnetting Solve?

**Scenario without subnetting:**
```
Company has 192.168.1.0 network (254 possible devices)

[All 254 devices in one network]
- Slow (broadcast storm)
- Security nightmare
- Hard to manage
```

**With subnetting:**
```
192.168.1.0/24 split into:
- 192.168.1.0/26   → HR Department (62 devices)
- 192.168.1.64/26  → Engineering (62 devices)
- 192.168.1.128/26 → Marketing (62 devices)
- 192.168.1.192/26 → Guest WiFi (62 devices)

Benefits:
✓ Better performance
✓ Easier security rules
✓ Organized
```

### Understanding Subnet Masks

**What it is:** Tells computer which part is network, which is host

```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0
                ↑   ↑   ↑   ↑
              These are NETWORK portion
                          └─ This is HOST portion
```

**In binary (how computer sees it):**
```
IP:          11000000.10101000.00000001.01100100
Mask:        11111111.11111111.11111111.00000000
             └──────── Network ─────────┘└─ Host ─┘
```

### CIDR Notation (The /24 Thing)

**Format:** IP/Number (e.g., 192.168.1.0/24)

**The number means:** How many bits are for the network

```
/24 = 255.255.255.0
      └── 24 bits (3 octets) for network

/16 = 255.255.0.0
      └── 16 bits (2 octets) for network

/8 = 255.0.0.0
     └── 8 bits (1 octet) for network
```

### Common Subnet Masks Table

| CIDR | Subnet Mask     | # of Hosts | Use Case               |
|------|-----------------|------------|------------------------|
| /32  | 255.255.255.255 | 1          | Single device          |
| /30  | 255.255.255.252 | 2          | Point-to-point links   |
| /29  | 255.255.255.248 | 6          | Very small network     |
| /28  | 255.255.255.240 | 14         | Small team             |
| /27  | 255.255.255.224 | 30         | Small office           |
| /26  | 255.255.255.192 | 62         | Medium office          |
| /25  | 255.255.255.128 | 126        | Larger office          |
| /24  | 255.255.255.0   | 254        | Default home/small biz |
| /16  | 255.255.0.0     | 65,534     | Large organization     |
| /8   | 255.0.0.0       | 16,777,214 | Massive network        |

### Practical Subnetting Example

**Scenario:** You have 192.168.1.0/24, need 4 departments

**Step 1: Calculate how many subnets needed**
```
4 departments = need 4 subnets
2^2 = 4 (need 2 bits for subnets)
```

**Step 2: New subnet mask**
```
Original: /24 (255.255.255.0)
Add 2 bits: /26 (255.255.255.192)
```

**Step 3: List the subnets**
```
Subnet 1: 192.168.1.0/26   (Range: .0-.63)
Subnet 2: 192.168.1.64/26  (Range: .64-.127)
Subnet 3: 192.168.1.128/26 (Range: .128-.191)
Subnet 4: 192.168.1.192/26 (Range: .192-.255)
```

**Step 4: Assign to departments**
```
HR:          192.168.1.0/26
Engineering: 192.168.1.64/26
Marketing:   192.168.1.128/26
Guest WiFi:  192.168.1.192/26
```

### Special Addresses in Each Subnet

**Every subnet has:**

```
Network Address (First IP):
192.168.1.0 - Cannot assign to device (identifies the network)

Broadcast Address (Last IP):
192.168.1.63 - Cannot assign to device (sends to all devices)

Usable IPs (Middle IPs):
192.168.1.1 to 192.168.1.62 - Assign to devices

Gateway (Usually First Usable IP):
192.168.1.1 - Usually the router
```

---

## 6. DNS - The Internet's Phone Book

### What DNS Does (Simple Explanation)

**You type:** `www.google.com`  
**Computer needs:** `142.250.185.46`  
**DNS translates:** Name → Number

```
Human: "Call John Smith"
DNS: "John Smith's number is 555-1234"
Human: Actually calls 555-1234
```

### How DNS Works (Step by Step)

```
1. You type: www.example.com

2. Your computer checks:
   - Browser cache (did I visit this before?)
   - Operating system cache
   
3. If not found, asks DNS Resolver (usually your ISP)

4. DNS Resolver checks its cache

5. If not found, starts the DNS Query Process:

   Root Server (.) → "Try .com servers"
            ↓
   TLD Server (.com) → "Try example.com nameservers"
            ↓
   Authoritative Server → "www.example.com = 93.184.216.34"
            ↓
   Back to your computer

6. Your computer connects to 93.184.216.34
```

### DNS Hierarchy

```
                    . (Root)
                    ↓
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       .com        .net        .org     ← TLD (Top Level Domain)
        ↓
    google.com                          ← Domain
        ↓
    www.google.com                      ← Subdomain
```

### DNS Record Types

#### A Record (Address Record)
```
Maps domain name to IPv4 address

example.com → 93.184.216.34

Used for: Websites, services
```

#### AAAA Record (Quad-A)
```
Maps domain name to IPv6 address

example.com → 2606:2800:220:1:248:1893:25c8:1946

Used for: IPv6 websites
```

#### CNAME Record (Canonical Name)
```
Alias from one name to another

www.example.com → example.com → 93.184.216.34

Used for: Subdomain pointing to main domain
```

#### MX Record (Mail Exchange)
```
Directs email to mail server

example.com → mail.example.com (priority: 10)

Used for: Email routing
```

#### TXT Record (Text Record)
```
Stores text information

example.com → "v=spf1 include:_spf.example.com ~all"

Used for: SPF, DKIM (email security), verification
```

#### NS Record (Name Server)
```
Delegates subdomain to nameservers

example.com → ns1.example.com, ns2.example.com

Used for: DNS delegation
```

### DNS in Action (Real Example)

```bash
# Check DNS records
nslookup google.com

# Output:
Server:  192.168.1.1
Address: 192.168.1.1#53

Name:    google.com
Address: 142.250.185.46
```

```bash
# Detailed DNS query
dig google.com

# Shows:
- Query time
- Which DNS server answered
- TTL (Time To Live)
- IP address
```

### DNS Caching & TTL

**TTL (Time To Live):** How long to cache the DNS result

```
Short TTL (300 seconds = 5 minutes):
✓ Changes propagate quickly
✗ More DNS queries (slower)

Used for: Services that change IPs frequently

Long TTL (86400 seconds = 24 hours):
✓ Fewer DNS queries (faster)
✗ Changes take longer to propagate

Used for: Stable services
```

### Common DNS Issues & Solutions

```
Problem: "Cannot resolve hostname"
Meaning: DNS can't find the IP address

Checks:
1. Is domain spelled correctly?
2. Does domain exist?
3. Is DNS server reachable?

Fix:
# Test with different DNS server
nslookup example.com 8.8.8.8
```

```
Problem: "DNS propagation delay"
Meaning: DNS changes haven't spread everywhere yet

Takes: Up to 48 hours (usually faster)

Fix: Wait, or use a different DNS server to test
```

### Important DNS Servers

```
Google Public DNS:
8.8.8.8
8.8.4.4

Cloudflare DNS:
1.1.1.1
1.0.0.1

Why use these?
- Faster than ISP DNS
- More reliable
- Privacy-focused (Cloudflare)
```

---

## 7. Network Protocols

### What is a Protocol?

**Simple answer:** Rules for how computers communicate

**Analogy:** 
```
Human conversation protocol:
1. Say "hello"
2. Take turns speaking
3. Say "goodbye"

Network protocol:
1. Establish connection
2. Exchange data
3. Close connection
```

### HTTP/HTTPS (Web Protocol)

#### HTTP (HyperText Transfer Protocol)
```
What: How web browsers talk to web servers
Port: 80

Example:
Browser: "GET /index.html HTTP/1.1"
Server: "HTTP/1.1 200 OK
         Content-Type: text/html
         
         <html>...</html>"
```

#### HTTPS (HTTP Secure)
```
What: HTTP + Encryption (SSL/TLS)
Port: 443

Why it matters:
HTTP:  "password123" → Anyone can see
HTTPS: "j83ks#$%..."  → Encrypted, safe

When to use:
✓ Always (especially for login, payments)
✗ Never use plain HTTP for sensitive data
```

#### HTTP Status Codes (What They Mean)

```
2xx = Success
200 OK               → Everything worked
201 Created          → New resource created
204 No Content       → Worked, but no data to return

3xx = Redirection
301 Moved Permanently → Resource moved forever
302 Found            → Resource moved temporarily
304 Not Modified     → Use cached version

4xx = Client Error (Your fault)
400 Bad Request      → You sent invalid data
401 Unauthorized     → Not logged in
403 Forbidden        → Logged in, but not allowed
404 Not Found        → Resource doesn't exist
429 Too Many Requests → You're being rate-limited

5xx = Server Error (Their fault)
500 Internal Server Error → Server crashed
502 Bad Gateway          → Proxy/Gateway problem
503 Service Unavailable  → Server overloaded/down
504 Gateway Timeout      → Proxy/Gateway timeout
```

### TCP (Transmission Control Protocol)

**What it is:** Reliable, ordered data delivery

```
Like certified mail:
✓ Guaranteed delivery
✓ In correct order
✓ Error checking
✗ Slower (because of all the checks)

Used for:
- Web browsing (HTTP/HTTPS)
- Email
- File transfer
- Anything where accuracy matters
```

#### Three-Way Handshake
```
Client → Server: "SYN" (Let's talk)
Server → Client: "SYN-ACK" (OK, let's talk)
Client → Server: "ACK" (Great, here's my data)

Only THEN does data transfer begin

Why: Ensures both sides are ready
```

### UDP (User Datagram Protocol)

**What it is:** Fast, unreliable data delivery

```
Like shouting across a room:
✓ Very fast
✗ Not guaranteed delivery
✗ Might arrive out of order
✗ No error checking

Used for:
- Video streaming (Netflix, YouTube)
- Online gaming
- Voice calls (VoIP)
- DNS queries
- Anything where speed > accuracy
```

**Why use UDP for video?**
```
TCP: "Packet 5 missing! Stop! Resend!"
     → Video freezes

UDP: "Packet 5 missing? Whatever, keep going"
     → Video might glitch for 1 frame, but keeps playing
```

### FTP (File Transfer Protocol)

```
What: Transfer files between computers
Ports: 20 (data), 21 (control)

Problems:
✗ Not encrypted
✗ Old and clunky

Better alternatives:
✓ SFTP (SSH File Transfer Protocol) - encrypted
✓ SCP (Secure Copy Protocol) - encrypted
```

### SSH (Secure Shell)

```
What: Securely connect to remote computers
Port: 22

Uses:
✓ Remote server management
✓ File transfer (SCP, SFTP)
✓ Tunneling other protocols

Example:
ssh user@192.168.1.100
```

### SMTP/IMAP/POP3 (Email Protocols)

```
SMTP (Simple Mail Transfer Protocol)
- Port: 25, 587, 465
- Used for: SENDING email
- Like: Dropping letter in mailbox

IMAP (Internet Message Access Protocol)
- Port: 143, 993
- Used for: READING email (keeps on server)
- Like: Going to post office to read mail (leaves mail there)

POP3 (Post Office Protocol v3)
- Port: 110, 995
- Used for: READING email (downloads and deletes from server)
- Like: Picking up mail from post office (takes mail home)
```

### DHCP (Dynamic Host Configuration Protocol)

```
What: Automatically assigns IP addresses

Without DHCP:
You: Manually set IP, subnet, gateway, DNS (error-prone)

With DHCP:
Computer: "I need an IP address"
DHCP Server: "Here's 192.168.1.50, subnet 255.255.255.0,
              gateway 192.168.1.1, DNS 8.8.8.8"
Computer: "Thanks!" (Automatically configured)

Why it matters:
✓ No manual configuration
✓ Prevents IP conflicts
✓ Easy to manage lots of devices
```

### ICMP (Internet Control Message Protocol)

```
What: Network diagnostic messages

Used by:
- ping (is host alive?)
- traceroute (what path does traffic take?)

Example:
ping google.com
- Sends ICMP Echo Request
- Receives ICMP Echo Reply
- Shows latency (round-trip time)
```

---

## 8. OSI & TCP/IP Models

### Why These Models Matter

**Think of them as:** Layers of a wedding cake

```
Each layer:
- Has a specific job
- Uses the layer below it
- Provides service to layer above
- Can be swapped out without breaking others

Example: Change WiFi to Ethernet
- Only Physical layer changes
- Everything else works the same
```

### OSI Model (7 Layers)

```
7. Application  ← What you see (browser, email)
6. Presentation ← Encryption, formatting
5. Session      ← Keeping connections open
4. Transport    ← TCP/UDP (reliable delivery)
3. Network      ← IP addresses, routing
2. Data Link    ← MAC addresses, switches
1. Physical     ← Cables, WiFi signals
```

#### Layer 1: Physical
```
Job: Move bits (1s and 0s) over physical medium

Examples:
- Ethernet cables
- WiFi radio waves
- Fiber optic light

Devices:
- Cables
- Hubs
- Repeaters

Problems:
- Cable unplugged
- Signal interference
```

#### Layer 2: Data Link
```
Job: Move data between devices on SAME network

Addressing: MAC addresses
Format: Frames

Examples:
- Ethernet
- WiFi (802.11)
- PPP

Devices:
- Switches
- Bridges
- Network cards

Problems:
- Duplicate MAC addresses
- Switch port errors
```

**Frame Example:**
```
┌──────────┬──────────┬──────┬─────┬──────────┐
│  Dest    │  Source  │ Type │Data │ Checksum │
│   MAC    │   MAC    │      │     │          │
└──────────┴──────────┴──────┴─────┴──────────┘
```

#### Layer 3: Network
```
Job: Move data between DIFFERENT networks

Addressing: IP addresses
Format: Packets

Examples:
- IPv4
- IPv6
- ICMP

Devices:
- Routers

Problems:
- Routing loops
- TTL expired
- No route to host
```

**Packet Example:**
```
┌───────────┬────────────┬─────┬────────┐
│  Source   │    Dest    │ TTL │  Data  │
│     IP    │     IP     │     │        │
└───────────┴────────────┴─────┴────────┘
```

#### Layer 4: Transport
```
Job: Reliable delivery between applications

Protocols:
- TCP (reliable)
- UDP (fast)

Port numbers: Identify applications

Examples:
- Web server: Port 80/443
- SSH: Port 22
- DNS: Port 53

Problems:
- Port already in use
- Connection refused
- Timeout
```

**Segment Example:**
```
┌────────────┬───────────┬───────┬────────┐
│  Source    │   Dest    │ Flags │  Data  │
│   Port     │   Port    │(SYN,  │        │
│            │           │ ACK)  │        │
└────────────┴───────────┴───────┴────────┘
```

#### Layer 5: Session
```
Job: Maintain connections between applications

Examples:
- SQL database session
- SSH session
- Remote desktop

What it handles:
- Session setup
- Maintaining session
- Session teardown

Problems:
- Session timeout
- Connection dropped
```

#### Layer 6: Presentation
```
Job: Format data for application

Functions:
- Encryption (SSL/TLS)
- Compression (gzip)
- Encoding (ASCII, UTF-8)

Examples:
- JPEG images
- MP3 audio
- SSL/TLS encryption

Problems:
- Encoding mismatch
- Decryption failure
```

#### Layer 7: Application
```
Job: Network services to end users

Examples:
- HTTP/HTTPS (web)
- SMTP (email)
- FTP (file transfer)
- DNS (name resolution)
- SSH (remote shell)

What users see:
- Web browsers
- Email clients
- File transfer programs

Problems:
- Application crashes
- Protocol errors
- Authentication failure
```

### TCP/IP Model (4 Layers)

**Simpler, practical version used in real world:**

```
OSI Model          TCP/IP Model
─────────────────  ───────────────
7. Application  ┐
6. Presentation ├→  4. Application
5. Session      ┘

4. Transport    →   3. Transport

3. Network      →   2. Internet

2. Data Link    ┐
1. Physical     ┘→  1. Network Access
```

#### 1. Network Access (Physical + Data Link)
```
What: How bits move on physical network

Includes:
- Ethernet
- WiFi
- MAC addresses
```

#### 2. Internet (Network)
```
What: Routing between networks

Protocol: IP (Internet Protocol)

Handles:
- IP addressing
- Routing
- Fragmentation
```

#### 3. Transport
```
What: End-to-end communication

Protocols:
- TCP (connection-oriented)
- UDP (connectionless)

Handles:
- Port numbers
- Error checking
- Flow control
```

#### 4. Application
```
What: User-facing protocols

Includes:
- HTTP, HTTPS
- SMTP, IMAP
- FTP, SSH
- DNS
```

### How Data Flows Through Layers

**Sending data:**
```
Application: "Send this email"
    ↓ (add email headers)
Transport: "Here's TCP segment to port 25"
    ↓ (add TCP header with ports)
Internet: "Here's IP packet to 192.168.1.5"
    ↓ (add IP header with addresses)
Network Access: "Here's frame to MAC aa:bb:cc:dd:ee:ff"
    ↓ (add Ethernet header)
Physical: "Here are the bits" (electrical signals)
```

**Receiving data:**
```
Physical: "Got some bits"
    ↓
Network Access: "Frame for MAC aa:bb:cc:dd:ee:ff, remove Ethernet header"
    ↓
Internet: "Packet for IP 192.168.1.5, remove IP header"
    ↓
Transport: "Segment for port 25, remove TCP header"
    ↓
Application: "Email received!"
```

### Practical DevOps Example

**User loads www.example.com:**

```
Layer 7: Browser says "GET /index.html HTTP/1.1"
    ↓
Layer 4: TCP wraps it, adds source/dest ports (random/80)
    ↓
Layer 3: IP wraps it, adds source/dest IPs (your IP/server IP)
    ↓
Layer 2: Ethernet wraps it, adds source/dest MACs
    ↓
Layer 1: Sends electrical signals over cable

──────────────────────────────────────────────────

Server receives, unwraps in reverse:
    ↑
Layer 1: Receives bits
    ↑
Layer 2: Removes Ethernet header
    ↑
Layer 3: Removes IP header
    ↑
Layer 4: Removes TCP header
    ↑
Layer 7: Web server sees "GET /index.html"
         Sends back HTML
```

---

## 9. Ports & Services

### What is a Port?

**Analogy:** Apartment building

```
Building = IP Address (192.168.1.100)
Apartments = Ports

Apartment 80  = Web server
Apartment 22  = SSH server
Apartment 3306 = MySQL database

Delivery address:
192.168.1.100:80 = Web page at 192.168.1.100
```

**Technical definition:**
```
- 16-bit number (0-65535)
- Identifies specific application/service
- Allows multiple services on one IP
```

### Port Ranges

```
0-1023:      Well-known ports (System/privileged)
             - Require admin access to bind
             - Standard services
             
1024-49151:  Registered ports (User/semi-privileged)
             - Assigned to specific services
             - Don't require admin access
             
49152-65535: Dynamic/Private ports (Ephemeral)
             - Used for client connections
             - Temporary
```
### Essential Ports for DevOps

#### Web Services
```
Port 80  (HTTP)
- Unencrypted web traffic
- Status: Deprecated (use 443 instead)
Example: http://example.com

Port 443 (HTTPS)
- Encrypted web traffic
- Always use this for websites
Example: https://example.com

Port 8080 (HTTP Alternate)
- Development/testing web servers
- Non-privileged port (no root needed)
- Often used for proxies
Example: http://localhost:8080

Port 8443 (HTTPS Alternate)
- Development/testing HTTPS
- Alternative to 443
```

#### Remote Access
```
Port 22 (SSH)
- Secure shell access
- Remote server management
- File transfer (SCP, SFTP)
Example: ssh user@192.168.1.100

Port 3389 (RDP)
- Remote Desktop Protocol
- Windows remote access
Example: Remote Desktop to Windows servers

Port 5900 (VNC)
- Virtual Network Computing
- Cross-platform remote desktop
```

#### Databases
```
Port 3306 (MySQL/MariaDB)
- MySQL database server
Example: mysql -h 192.168.1.100 -P 3306

Port 5432 (PostgreSQL)
- PostgreSQL database
Example: psql -h 192.168.1.100 -p 5432

Port 27017 (MongoDB)
- MongoDB NoSQL database
Default connection string

Port 6379 (Redis)
- Redis cache/database
Example: redis-cli -h 192.168.1.100 -p 6379

Port 1433 (MS SQL Server)
- Microsoft SQL Server
Windows database server
```

#### Email
```
Port 25 (SMTP)
- Sending email (unencrypted)
- Server-to-server communication

Port 587 (SMTP Submission)
- Sending email (encrypted with STARTTLS)
- Client-to-server (preferred)

Port 465 (SMTPS)
- SMTP over SSL/TLS
- Legacy, but still used

Port 143 (IMAP)
- Reading email (unencrypted)

Port 993 (IMAPS)
- IMAP over SSL/TLS (encrypted)
- Preferred for email reading

Port 110 (POP3)
- Reading email (unencrypted, downloads)

Port 995 (POP3S)
- POP3 over SSL/TLS
```

#### File Transfer
```
Port 20 (FTP Data)
- FTP data transfer

Port 21 (FTP Control)
- FTP commands
⚠️ Unencrypted, avoid if possible

Port 69 (TFTP)
- Trivial FTP
- Simple, no authentication
- Used for network boot, configs

Port 22 (SFTP)
- Secure FTP over SSH
- Preferred method
```

#### DNS & Directory Services
```
Port 53 (DNS)
- Domain Name System
- Both TCP and UDP
Example: nslookup, dig

Port 389 (LDAP)
- Lightweight Directory Access Protocol
- User/group management

Port 636 (LDAPS)
- LDAP over SSL/TLS
- Secure directory access
```

#### Container & Orchestration
```
Port 2375 (Docker)
- Docker daemon (unencrypted)
⚠️ Don't expose to internet

Port 2376 (Docker TLS)
- Docker daemon (encrypted)
- Secure Docker API

Port 6443 (Kubernetes API)
- Kubernetes API server
- Cluster management

Port 10250 (Kubelet)
- Kubernetes node agent
- Pod management

Port 2379-2380 (etcd)
- Kubernetes key-value store
- Cluster state
```

#### Monitoring & Logging
```
Port 9090 (Prometheus)
- Prometheus metrics server
- Time-series monitoring

Port 3000 (Grafana)
- Grafana dashboards
- Visualization

Port 9100 (Node Exporter)
- Prometheus node metrics
- Server monitoring

Port 5601 (Kibana)
- Elasticsearch visualization
- Log analysis UI

Port 9200 (Elasticsearch)
- Elasticsearch API
- Search and analytics

Port 5044 (Logstash)
- Log data ingestion
- ELK stack component
```

#### Message Queues
```
Port 5672 (RabbitMQ)
- AMQP protocol
- Message broker

Port 15672 (RabbitMQ Management)
- RabbitMQ web UI
- Admin interface

Port 9092 (Kafka)
- Apache Kafka broker
- Event streaming

Port 4222 (NATS)
- NATS messaging
- Lightweight message broker
```

#### CI/CD Tools
```
Port 8080 (Jenkins)
- Jenkins web UI
- Build automation

Port 9000 (SonarQube)
- Code quality analysis
- Static analysis

Port 8081 (Nexus)
- Artifact repository
- Package management

Port 5000 (Docker Registry)
- Private Docker registry
- Container images
```

#### Version Control
```
Port 22 (Git SSH)
- Git over SSH
Example: git@github.com:user/repo.git

Port 443 (Git HTTPS)
- Git over HTTPS
Example: https://github.com/user/repo.git

Port 9418 (Git Protocol)
- Git native protocol
- Fast, read-only
```

### Port States

```
OPEN
- Port is accepting connections
- Service is listening
- Accessible

CLOSED
- Port is accessible but nothing listening
- Firewall allows it, but no service

FILTERED
- Firewall/filter is blocking
- Can't tell if open or closed
- Security measure

LISTENING
- Server side: waiting for connections
- Service is ready
```

### Checking Ports

#### From Linux/Mac
```bash
# Check if port is listening locally
netstat -tuln | grep :80
ss -tuln | grep :80

# Check specific port on remote host
telnet example.com 80
nc -zv example.com 80

# Scan ports (if nmap installed)
nmap -p 80,443,22 example.com

# Check what's using a port
lsof -i :8080
sudo netstat -tulnp | grep :8080
```

#### From Windows
```cmd
# Show all listening ports
netstat -an | findstr LISTENING

# Check specific port
netstat -an | findstr :80

# Check what's using a port
netstat -ano | findstr :8080

# Test connection to port
Test-NetConnection -ComputerName example.com -Port 80
```

#### Using curl
```bash
# Check if HTTP port is open
curl -v http://example.com:80

# Check HTTPS
curl -v https://example.com:443

# Check with timeout
curl --connect-timeout 5 http://example.com:8080
```

### Security Best Practices

```
✓ DO:
- Close unused ports
- Use firewalls to restrict access
- Use encrypted ports (443 vs 80, 993 vs 143)
- Change default ports for sensitive services
- Document which ports are used
- Monitor port usage

✗ DON'T:
- Expose database ports to internet
- Use unencrypted protocols in production
- Leave development ports open in production
- Forget to update firewall rules
- Use well-known ports for custom services
```

---

## 10. Routing Explained

### What is Routing?

**Simple definition:** Finding the path from source to destination

**Analogy:**
```
Like GPS navigation:
- You're at point A
- Want to get to point B
- Router figures out the best path
- Multiple routes possible
- Chooses fastest/shortest
```

### Basic Routing Concepts

#### Routing Table
**What it is:** Map that tells router where to send packets

```
Destination     Gateway         Interface   Metric
0.0.0.0/0       192.168.1.1    eth0        0        ← Default route
192.168.1.0/24  0.0.0.0        eth0        0        ← Local network
10.0.0.0/8      192.168.1.254  eth0        10       ← Remote network
```

**Reading the table:**
- **Destination:** Where packet is going
- **Gateway:** Next hop router (0.0.0.0 = direct connection)
- **Interface:** Which network card to use
- **Metric:** Cost (lower = better)

#### View Your Routing Table

```bash
# Linux/Mac
route -n
ip route show
netstat -rn

# Windows
route print
netstat -r
```

### Default Route (Default Gateway)

**What it is:** Where to send traffic when no specific route exists

```
Think of it as:
"If I don't know where something is, send it here"

Format: 0.0.0.0/0 or default

Example:
Destination     Gateway
0.0.0.0/0       192.168.1.1

Meaning: Send everything I don't know about to 192.168.1.1
```

**Real-world scenario:**
```
Your computer: 192.168.1.100
Default gateway: 192.168.1.1 (your router)

You visit google.com (142.250.185.46):
1. Computer checks: "Is 142.250.185.46 in my network?" No
2. Computer checks routing table: "Any specific route?" No
3. Computer uses default route: Send to 192.168.1.1
4. Router figures out rest of path to Google
```

### Static vs Dynamic Routing

#### Static Routing
```
What: Manually configured routes
Don't change unless admin changes them

Pros:
✓ Simple
✓ Predictable
✓ No routing protocol overhead
✓ More secure

Cons:
✗ Manual work
✗ Doesn't adapt to failures
✗ Hard to manage large networks

Use cases:
- Small networks
- Point-to-point links
- Security-critical routes
```

**Example:**
```bash
# Add static route (Linux)
sudo ip route add 10.0.0.0/8 via 192.168.1.254

# Add static route (Windows)
route add 10.0.0.0 mask 255.0.0.0 192.168.1.254
```

#### Dynamic Routing
```
What: Routers automatically share route information
Adapts to network changes

Protocols:
- RIP (Routing Information Protocol)
- OSPF (Open Shortest Path First)
- BGP (Border Gateway Protocol)
- EIGRP (Enhanced Interior Gateway Routing Protocol)

Pros:
✓ Automatic updates
✓ Adapts to failures
✓ Scales well

Cons:
✗ Complex
✗ Uses bandwidth/CPU
✗ Potential security issues

Use cases:
- Large networks
- Internet backbone
- Networks that change frequently
```

### How Routers Make Decisions

**Longest Prefix Match:** Router chooses most specific route

```
Routing table:
10.0.0.0/8        → Gateway A
10.1.0.0/16       → Gateway B
10.1.2.0/24       → Gateway C

Destination: 10.1.2.50

Which route matches?
✓ 10.0.0.0/8      matches (8 bits)
✓ 10.1.0.0/16     matches (16 bits)
✓ 10.1.2.0/24     matches (24 bits) ← MOST SPECIFIC, USE THIS

Sends to Gateway C
```

### Routing Example Step-by-Step

**Scenario:** Send packet from 192.168.1.100 to 8.8.8.8

```
┌─────────────────────────────────────────────────────────┐
│ Step 1: Your Computer (192.168.1.100)                  │
│ - Check: Is 8.8.8.8 in my subnet? No                   │
│ - Action: Send to default gateway 192.168.1.1          │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Step 2: Home Router (192.168.1.1)                      │
│ - Check routing table                                   │
│ - No specific route to 8.8.8.8                         │
│ - Action: Send to ISP gateway                          │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Step 3: ISP Router                                      │
│ - Checks routing table                                  │
│ - Knows path to 8.8.8.0/24 network                     │
│ - Action: Forward to next ISP router                   │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Steps 4-N: Internet Backbone Routers                    │
│ - Each router checks routing table                      │
│ - Forwards packet closer to destination                │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Final Step: Google's Router                             │
│ - Recognizes 8.8.8.8 is local                          │
│ - Delivers to Google DNS server                         │
└─────────────────────────────────────────────────────────┘
```

### TTL (Time To Live)

**What it is:** Prevents packets from looping forever

```
How it works:
1. Start with TTL = 64 (typical)
2. Each router decrements: TTL = 63, 62, 61...
3. If TTL reaches 0: Drop packet, send error

Purpose:
- Prevents routing loops
- Limits packet lifetime
- Used by traceroute to map paths
```

**Example routing loop (why TTL matters):**
```
Router A → Router B → Router C → Router A → (infinite loop)

Without TTL: Packet loops forever, wastes bandwidth
With TTL: After 64 hops, packet dies, error sent back
```

### Traceroute - See The Path

**What it does:** Shows every router hop to destination

```bash
# Linux/Mac
traceroute google.com

# Windows
tracert google.com

# Output:
1  192.168.1.1      1.2ms    (home router)
2  10.0.0.1         5.3ms    (ISP router)
3  72.14.215.85     12.8ms   (ISP backbone)
4  142.250.185.46   15.2ms   (Google)
```

**How it works:**
```
Send packets with increasing TTL:
- TTL=1: Dies at first router, router sends error back
- TTL=2: Dies at second router, router sends error back
- TTL=3: Dies at third router, router sends error back
- Continue until reaching destination
```

### Route Metrics

**What it is:** Cost of using a route (lower = better)

**Factors considered:**
```
- Hop count (# of routers)
- Bandwidth (faster = better)
- Delay (lower = better)
- Reliability (more reliable = better)
- Load (less congested = better)

Example:
Route A: 3 hops, 100 Mbps, 5ms delay → Metric: 10
Route B: 2 hops, 10 Mbps, 50ms delay → Metric: 15
Choose Route A (lower metric)
```

### Common Routing Issues

#### Problem: Routing Loop
```
Symptom: Packet keeps going in circles
Cause: Misconfigured routing tables

Example:
Router A → Router B → Router C → Router A

Fix:
- Correct routing tables
- Enable routing protocols
- TTL prevents infinite loops
```

#### Problem: Asymmetric Routing
```
Symptom: Traffic goes different path each direction
Cause: Different routing costs

Example:
Request:  A → B → C → D
Response: D → E → F → A

Usually OK, but can cause firewall issues
```

#### Problem: No Route to Host
```
Symptom: "No route to host" error
Cause: Missing route or incorrect gateway

Check:
# View routing table
ip route show

# Test connectivity
ping gateway_ip
traceroute destination_ip

Fix:
# Add missing route
sudo ip route add <network> via <gateway>
```

### Cloud Networking Routing

#### AWS Route Tables
```
Every VPC has route table

Default routes:
10.0.0.0/16  → local    (VPC internal)
0.0.0.0/0    → igw-xxx  (Internet Gateway)

Custom routes:
192.168.0.0/16 → vpn-xxx  (VPN connection)
10.1.0.0/16    → pcx-xxx  (VPC peering)
```

#### Azure Routing
```
System routes (automatic):
- Virtual network address space
- Internet
- Azure services

Custom routes (UDR - User Defined Routes):
- Force traffic through firewall
- Direct to on-premises
- Route between subnets
```

#### GCP Routes
```
Auto-created routes:
- Subnet routes
- Default internet gateway

Custom routes:
- Static routes
- Dynamic routes (Cloud Router)
```

---

## 11. Network Troubleshooting Tools

### ping - Basic Connectivity Test

**What it does:** Tests if host is reachable

```bash
# Basic ping
ping google.com

# Ping with count
ping -c 4 google.com     # Linux/Mac
ping -n 4 google.com     # Windows

# Continuous ping (Ctrl+C to stop)
ping -t google.com       # Windows
ping google.com          # Linux/Mac (default continuous)

# Change packet size
ping -s 1024 google.com  # Linux/Mac
ping -l 1024 google.com  # Windows

# Output example:
PING google.com (142.250.185.46): 56 data bytes
64 bytes from 142.250.185.46: icmp_seq=0 ttl=116 time=15.2 ms
64 bytes from 142.250.185.46: icmp_seq=1 ttl=116 time=14.8 ms

--- google.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 14.8/15.0/15.2/0.2 ms
```

**What the output means:**
```
ttl=116: Time to live (started at ~120, went through ~4 routers)
time=15.2 ms: Round-trip time (lower = better)
0% packet loss: All packets received (good!)

Good results:
✓ 0% packet loss
✓ Consistent times
✓ Low latency (<50ms for nearby, <200ms for far)

Bad results:
✗ Packet loss (>1%)
✗ Wildly varying times
✗ Very high latency
```

**Common issues:**
```
"Request timeout" or "Destination host unreachable"
Causes:
- Host is down
- Firewall blocking ICMP
- Network problem
- Wrong IP address

"100% packet loss"
Causes:
- Cable unplugged
- WiFi disconnected
- Firewall blocking everything
- Network interface down
```

### traceroute/tracert - Path Discovery

**What it does:** Shows every hop to destination

```bash
# Linux/Mac
traceroute google.com
traceroute -I google.com  # Use ICMP instead of UDP

# Windows
tracert google.com

# Output example:
traceroute to google.com (142.250.185.46), 30 hops max
 1  192.168.1.1       1.234 ms   (home router)
 2  10.0.0.1          5.678 ms   (ISP router)
 3  72.14.215.85     12.345 ms   (ISP backbone)
 4  * * *                        (firewall blocking)
 5  142.250.185.46   15.234 ms   (destination)
```

**Reading the output:**
```
Column 1: Hop number
Column 2: IP or hostname
Column 3-5: Round-trip times (3 probes sent)

* * * means:
- Router not responding (firewall)
- Packet dropped
- ICMP disabled

Use cases:
✓ Find where connection fails
✓ Identify slow hops
✓ See routing path
✓ Troubleshoot network issues
```

### nslookup - DNS Lookup

**What it does:** Query DNS servers

```bash
# Basic lookup
nslookup google.com

# Output:
Server:  192.168.1.1
Address: 192.168.1.1#53

Non-authoritative answer:
Name:    google.com
Address: 142.250.185.46

# Query specific DNS server
nslookup google.com 8.8.8.8

# Reverse lookup (IP to hostname)
nslookup 142.250.185.46

# Query specific record type
nslookup -type=MX google.com  # Mail servers
nslookup -type=NS google.com  # Name servers
nslookup -type=TXT google.com # TXT records
```

**Common issues:**
```
"Server failed" or "No response from server"
Fix: Check DNS server IP, try 8.8.8.8 or 1.1.1.1

"Non-existent domain"
Fix: Check spelling, verify domain exists

"Timeout"
Fix: Firewall blocking DNS (port 53)
```

### dig - Advanced DNS Tool

**What it does:** Detailed DNS queries (Linux/Mac)

```bash
# Basic query
dig google.com

# Short answer only
dig google.com +short

# Query specific record type
dig google.com MX     # Mail servers
dig google.com NS     # Name servers
dig google.com TXT    # Text records
dig google.com AAAA   # IPv6 address

# Trace full DNS resolution path
dig google.com +trace

# Query specific DNS server
dig @8.8.8.8 google.com

# Output explanation:
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       142.250.185.46
   ↑                     ↑       ↑      ↑            ↑
Domain                 TTL     Class  Type      IP Address
```

### netstat - Network Connections

**What it does:** Shows network connections and listening ports

```bash
# Linux/Mac
netstat -tuln          # All listening ports (TCP/UDP)
netstat -tulnp         # Include program names (needs sudo)
netstat -an            # All connections

# Windows
netstat -an            # All connections and ports
netstat -ano           # Include process ID
netstat -b             # Show executable name (admin required)

# Common options:
-t  TCP connections
-u  UDP connections
-l  Listening ports
-n  Show numbers (don't resolve names)
-p  Show program/PID
-a  All connections

# Output example:
Proto Recv-Q Send-Q Local Address      Foreign Address    State
tcp        0      0 0.0.0.0:22         0.0.0.0:*          LISTEN
tcp        0      0 192.168.1.100:443  93.184.216.34:8080 ESTABLISHED

Reading output:
- Local Address: Your machine (IP:Port)
- Foreign Address: Remote machine (IP:Port)
- State: Connection status
  - LISTEN: Waiting for connections
  - ESTABLISHED: Active connection
  - TIME_WAIT: Connection closing
  - CLOSE_WAIT: Waiting for close
```

**Common uses:**
```bash
# Find what's using port 8080
netstat -tulnp | grep :8080
lsof -i :8080           # Mac/Linux alternative

# See all connections to a specific IP
netstat -an | grep 192.168.1.5

# Check if port is listening
netstat -tuln | grep :80
```

### ss - Modern Socket Statistics

**What it does:** Faster, more detailed alternative to netstat

```bash
# Show all TCP listening ports
ss -tln

# Show all connections
ss -tan

# Show processes
ss -tlnp

# Filter by port
ss -tln sport = :80

# Filter by state
ss -tan state established

# Common options:
-t  TCP
-u  UDP
-l  Listening
-n  Numeric (don't resolve)
-p  Process info
-a  All sockets

# Output similar to netstat but faster
```

### tcpdump - Packet Capture

**What it does:** Capture and analyze network traffic

```bash
# Capture on interface
sudo tcpdump -i eth0

# Capture specific port
sudo tcpdump -i eth0 port 80

# Capture specific host
sudo tcpdump -i eth0 host 192.168.1.100

# Save to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Capture HTTP traffic
sudo tcpdump -i eth0 'tcp port 80'

# Capture with details
sudo tcpdump -i eth0 -v -XX

# Common filters:
port 80                 # Port 80
host 192.168.1.100      # Specific host
src 192.168.1.100       # Source IP
dst 192.168.1.100       # Destination IP
tcp                     # TCP only
udp                     # UDP only
icmp                    # ICMP (ping)

# Output example:
12:34:56.789 IP 192.168.1.100.54321 > 93.184.216.34.80: Flags [S]
     ↑            ↑                                  ↑        ↑
  Timestamp    Source IP:Port              Dest IP:Port   TCP Flags
```

### curl - HTTP/HTTPS Testing

**What it does:** Make HTTP requests, test APIs

```bash
# Basic GET request
curl http://example.com

# See HTTP headers
curl -I http://example.com

# Verbose output (detailed)
curl -v https://example.com

# Follow redirects
curl -L http://example.com

# POST request
curl -X POST -d "key=value" http://api.example.com

# POST JSON
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"value"}' http://api.example.com

# Test with timeout
curl --connect-timeout 5 http://example.com

# Save to file
curl -o output.html http://example.com

# Check response time
curl -w "@-" -o /dev/null -s http://example.com << 'EOF'
    time_namelookup:  %{time_namelookup}\n
       time_connect:  %{time_connect}\n
    time_appconnect:  %{time_appconnect}\n
      time_redirect:  %{time_redirect}\n
   time_pretransfer:  %{time_pretransfer}\n
 time_starttransfer:  %{time_starttransfer}\n
                    ----------\n
         time_total:  %{time_total}\n
EOF

# Test HTTPS certificate
curl -vI https://example.com 2>&1 | grep -i "ssl\|tls\|cert"
```

### telnet - Port Testing

**What it does:** Test if port is open

```bash
# Test port
telnet example.com 80

# If connection succeeds:
Trying 93.184.216.34...
Connected to example.com.
Escape character is '^]'.

# If fails:
telnet: Unable to connect to remote host: Connection refused

# Common uses:
telnet mail.example.com 25    # Test SMTP
telnet example.com 3306        # Test MySQL
telnet example.com 22          # Test SSH

# Modern alternative (more reliable):
nc -zv example.com 80          # Linux/Mac
Test-NetConnection example.com -Port 80  # PowerShell
```

### nmap - Network Scanner

**What it does:** Scan ports, discover services

```bash
# Scan single host
nmap 192.168.1.100

# Scan range
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 80,443,22 192.168.1.100

# Scan all ports
nmap -p- 192.168.1.100

# Service version detection
nmap -sV 192.168.1.100

# OS detection
sudo nmap -O 192.168.1.100

# Fast scan (most common ports)
nmap -F 192.168.1.100

# Output example:
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

# ⚠️ Warning:
# Scanning networks you don't own may be illegal
# Always get permission first
```

### ip - Modern Network Configuration

**What it does:** Configure and display network settings

```bash
# Show all interfaces
ip addr show
ip a  # Shorthand

# Show specific interface
ip addr show eth0

# Show routing table
ip route show
ip r  # Shorthand

# Add IP address
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Show interface statistics
ip -s link

# Show neighbors (ARP table)
ip neigh show
```

### arp - Address Resolution Protocol

**What it does:** Maps IP to MAC addresses

```bash
# Show ARP cache
arp -a        # All systems
arp -n        # Numeric (Linux)

# Output example:
? (192.168.1.1) at aa:bb:cc:dd:ee:ff [ether] on eth0
  ↑              ↑
  IP Address     MAC Address

# Add static ARP entry
sudo arp -s 192.168.1.100 aa:bb:cc:dd:ee:ff

# Delete ARP entry
sudo arp -d 192.168.1.100

# Modern alternative:
ip neigh show
```

### Troubleshooting Workflow

**Step-by-step approach:**

```
1. Can you reach localhost?
   ping 127.0.0.1
   → Tests network stack

2. Can you reach default gateway?
   ping 192.168.1.1
   → Tests local network

3. Can you reach external IP?
   ping 8.8.8.8
   → Tests internet connectivity

4. Can you resolve DNS?
   nslookup google.com
   → Tests DNS

5. Can you reach destination?
   ping google.com
   → Tests end-to-end

6. Which hop fails?
   traceroute google.com
   → Finds problem location

7. Is service listening?
   netstat -tuln | grep :80
   → Tests if service is up

8. Can you connect to service?
   telnet example.com 80
   → Tests port connectivity

9. Capture traffic
   sudo tcpdump -i eth0
   → Deep dive analysis
```

---

## 12. Security Concepts

### Firewalls

**What it is:** Network security system that controls traffic

```
Think of it as: Security guard at building entrance

Checks:
- Source IP
- Destination IP
- Port numbers
- Protocol
- Connection state
```

#### Firewall Types

**Host-based Firewall:**
```
Runs on individual computer

Examples:
- iptables/firewalld (Linux)
- Windows Firewall
- macOS firewall

Protects: Single machine

Use case: Protect individual servers/workstations
```

**
