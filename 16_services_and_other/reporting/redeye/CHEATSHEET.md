# RedEye - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Install
pip3 install redeye

# Launch server
python3 -m redeye server

# Access at http://localhost:8080
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Port | 8080 |
| Database | SQLite |
| Interface | Web-based |
| Language | Python |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Graph Visualization | Interactive network graphs |
| Multi-tool Import | Nmap, Nessus, Burp, etc. |
| Relationship Mapping | Host connections |
| Attack Path Analysis | Identify attack vectors |
| Filtering | Filter by hosts/services/ports |
| Export | Generate reports and images |

---

## Import Commands

```bash
# Generate Nmap XML
nmap -sV -sC -p- -oX scan.xml target.com

# Generate Masscan XML
masscan -p1-65535 -oX masscan.xml 192.168.1.0/24

# Import via UI
# Import > Select tool type > Upload file
```

---

## Supported Import Formats

| Tool | Format |
|------|--------|
| Nmap | XML |
| Nessus | XML |
| Burp Suite | XML |
| Masscan | XML |
| OpenVAS | XML |
| Custom | JSON/CSV |

---

## Graph Operations

| Action | Method |
|--------|--------|
| Zoom | Mouse wheel |
| Pan | Click and drag |
| Select | Click node |
| Multi-select | Ctrl+click |
| Drag | Move nodes |
| Context menu | Right-click |

---

## Node Types

| Type | Description |
|------|-------------|
| Host | Network host |
| Service | Running service |
| Port | Open port |
| Vulnerability | Security issue |

---

## Filtering

```bash
# Filter by host
Sidebar > Hosts > Select host

# Filter by service
Sidebar > Services > Select service

# Filter by port
Sidebar > Ports > Enter port

# Search
Search bar > Enter query
```

---

## Export Options

| Format | Use Case |
|--------|----------|
| PNG | Graph image |
| SVG | Vector image |
| PDF | Document |
| JSON | Data export |
| CSV | Spreadsheet |

---

## Complete Recon Workflow

```bash
# 1. Scan with Nmap
nmap -sV -sC -p- -oX nmap.xml target.com

# 2. Run Nikto
nikto -h target.com -o nikto.html

# 3. Import into RedEye
# UI > Import > Upload files

# 4. Analyze graph
# View relationships, identify patterns

# 5. Export report
# Export > Report > Select format

# 6. Document in CherryTree/Dradis
```

---

## Attack Path Analysis

### Common Paths
```
Internet → Web Server → Database
Internet → Mail Server → Internal Network
Internal → Domain Controller → All Hosts
```

### Analysis Steps
1. Identify entry points
2. Trace connections
3. Identify dependencies
4. Map attack vectors
5. Document findings

---

## Troubleshooting

```bash
# Check Python version
python3 --version

# Install dependencies
pip3 install -r requirements.txt

# Check port availability
netstat -tlnp | grep 8080

# Clear browser cache
# Try different browser
```
