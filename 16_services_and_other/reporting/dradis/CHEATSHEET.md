# Dradis - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Kali Linux
sudo apt install dradis
sudo dradis-start
# Access at http://127.0.0.1:3000

# Stop service
sudo dradis-stop
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| URL | `http://127.0.0.1:3000` |
| Port | 3000 |
| Version | 5.0.0 (CE) |
| Database | SQLite/PostgreSQL |
| Framework | Ruby on Rails |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Centralized Evidence | All tool outputs in one place |
| Real-time Collaboration | Multiple users, simultaneous editing |
| Tool Import | Nmap, Nessus, Burp, Metasploit |
| Report Generation | HTML, PDF, Word export |
| Methodology Tracking | OWASP, PTES, custom |
| QA Checks | Validate findings completeness |

---

## Import Commands

```bash
# Nmap XML output
nmap -sV -sC -oX nmap_results.xml target.com

# Nessus export (from Nessus UI)
# My Scans > Select > Export > Nessus

# Burp Suite export (from Burp UI)
# Project options > Export > XML

# Metasploit XML export
msfconsole -q
db_export -f xml /path/to/export.xml

# OpenVAS export (from OpenVAS UI)
# Scans > Reports > Export > XML
```

---

## API Usage

```bash
# Generate API token in UI: Team > API Tokens

# List nodes
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:3000/api/nodes

# Create node
curl -X POST -H "Authorization: Token YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"node":{"label":"New Host","type_id":1}}' \
    http://localhost:3000/api/nodes

# Update node
curl -X PUT -H "Authorization: Token YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"node":{"label":"Updated"}}' \
    http://localhost:3000/api/nodes/1

# Delete node
curl -X DELETE -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:3000/api/nodes/1
```

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl + N | New node |
| Ctrl + Shift + N | New child node |
| Ctrl + Enter | Save |
| Ctrl + / | Toggle sidebar |
| Esc | Close dialog |

---

## Export Formats

| Format | Extension | Use Case |
|--------|-----------|----------|
| HTML | .html | Web report |
| PDF | .pdf | Formal document |
| Word | .docx | Client delivery |
| Text | .txt | Plain text |
| CSV | .csv | Data analysis |

---

## Troubleshooting

```bash
# Check service status
sudo systemctl status dradis

# Start service
sudo systemctl start dradis

# View logs
tail -f /var/log/dradis/production.log

# Clear cache
cd /usr/share/dradis-ce
bundle exec rake tmp:clear

# Restart service
sudo systemctl restart dradis
```

---

## Methodology Tracking

### OWASP Testing Guide Phases
1. Information Gathering
2. Configuration Management
3. Identity Management
4. Authentication
5. Authorization
6. Session Management
7. Input Validation
8. Error Handling
9. Cryptography
10. Business Logic
11. Client-Side Testing

### PTES Phases
1. Pre-engagement
2. Intelligence Gathering
3. Threat Modeling
4. Vulnerability Analysis
5. Exploitation
6. Post Exploitation
7. Reporting

---

## QA Check Rules

| Rule | Description |
|------|-------------|
| Missing Evidence | Finding without evidence |
| Missing Severity | Finding without CVSS/severity |
| Empty Description | Finding without description |
| Missing Remediation | Finding without fix |
| No References | Finding without references |
| Orphaned Evidence | Evidence not linked to finding |

---

## Integration Tips

```bash
# Automate Nmap + Dradis import
nmap -sV -sC -oX scan.xml target.com
# Then import via Dradis UI

# Batch import multiple files
# Use Dradis API or CLI tools

# Export and post-process
# HTML export can be styled with CSS
```
