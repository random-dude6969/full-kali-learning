# Faraday - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Kali Linux
sudo apt install faraday
sudo faraday-start
# Access at http://127.0.0.1:24004

# Docker
docker run -p 24004:24004 -p 9000:9000 faradaysec/faraday

# Stop service
sudo faraday-stop
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Web UI | `http://127.0.0.1:24004` |
| API | `http://127.0.0.1:9000` |
| Default Workspace | `default` |
| Database | PostgreSQL/SQLite |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Multi-tool Integration | Run Nmap, Nikto, Metasploit, etc. |
| Real-time Collaboration | Multiple users, simultaneous editing |
| Plugin Architecture | Extend with custom tools |
| Built-in Reporting | PDF, HTML, CSV export |
| Vulnerability Management | Track findings through lifecycle |
| REST API | Automation and integration |

---

## Running Tools

```bash
# Nmap scan
nmap -sV -sC -oX scan.xml target.com

# Nikto scan
nikto -h target.com -o nikto.html

# SQLMap
sqlmap -u "http://target.com/?id=1" --batch

# Masscan (fast port scan)
masscan -p1-65535 --rate=1000 -oX masscan.xml target.com
```

---

## API Usage

```bash
# Get API token
curl -X POST http://localhost:9000/v3/session/token \
    -d '{"email":"admin@faraday.io","password":"secret"}'

# List hosts
curl -H "Authorization: Bearer TOKEN" \
    http://localhost:9000/v3/ws/default/host

# Create host
curl -X POST http://localhost:9000/v3/ws/default/host \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer TOKEN" \
    -d '{"name":"192.168.1.1","os":"Linux"}'

# List vulnerabilities
curl -H "Authorization: Bearer TOKEN" \
    http://localhost:9000/v3/ws/default/vuln

# Create vulnerability
curl -X POST http://localhost:9000/v3/ws/default/vuln \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer TOKEN" \
    -d '{"name":"SQL Injection","severity":"critical","host_id":1}'
```

---

## Import Formats

| Tool | Format |
|------|--------|
| Nmap | XML |
| Nessus | XML |
| OpenVAS | XML |
| Burp Suite | XML |
| Metasploit | XML |
| Nikto | HTML/XML |
| SQLMap | HTML |
| Masscan | XML |

---

## Vulnerability Lifecycle

```
New → Confirmed → In Progress → Remediated → Verified → Closed
```

| Status | Description |
|--------|-------------|
| New | Initial finding |
| Confirmed | Verified as real |
| In Progress | Being remediated |
| Remediated | Fix applied |
| Verified | Fix verified |
| Closed | Finding resolved |

---

## Workspace Commands

```bash
# Create workspace via API
curl -X POST http://localhost:9000/v3/ws/ \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer TOKEN" \
    -d '{"name":"new_workspace"}'

# List workspaces
curl -H "Authorization: Bearer TOKEN" \
    http://localhost:9000/v3/ws
```

---

## Troubleshooting

```bash
# Check service status
sudo systemctl status faraday

# Start service
sudo systemctl start faraday

# View logs
tail -f /var/log/faraday/production.log

# Check database
faraday-manage db status

# Restart service
sudo systemctl restart faraday
```

---

## Integration Examples

```bash
# Full recon workflow
nmap -sV -sC -p- -oX scan.xml target.com
nikto -h target.com -o nikto.html
# Import both into Faraday via UI

# API automation
#!/bin/bash
TOKEN=$(curl -s -X POST http://localhost:9000/v3/session/token \
    -d '{"email":"admin@faraday.io","password":"secret"}' | jq -r '.token')

curl -H "Authorization: Bearer $TOKEN" \
    http://localhost:9000/v3/ws/default/host
```
