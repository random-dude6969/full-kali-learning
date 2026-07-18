---
title: "svreport - sipvicious Report Generator"
description: "Reporting and analysis tool for generating PDF/CSV exports of sipvicious scan results"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - reporting
  - documentation
tags:
  - sip
  - voip
  - reporting
  - documentation
  - sipvicious
install_command: "sudo apt install sipvicious"
version: "0.3.4"
github_repo: "https://github.com/EnableSecurity/sipvicious"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipvicious"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - sipvicious
  - svmap
  - svwar
  - svcrack
---

# svreport - sipvicious Report Generator

svreport is the reporting and analysis component of the sipvicious suite. It provides functionality to export, search, and manage scan results stored in SQLite databases, generating professional PDF and CSV reports for VoIP security assessments.

## 1. Introduction

After conducting SIP scans with svmap, enumeration with svwar, or credential testing with svcrack, svreport consolidates and presents these findings in actionable formats. The tool supports multiple export formats, advanced searching, and data management operations.

### Key Capabilities

| Feature | Description |
|---------|-------------|
| Export Formats | PDF, CSV |
| Search | Query-based result filtering |
| List | Browse stored results |
| Delete | Remove specific results |
| Purge | Clean old data |

### Report Types

| Type | Content | Use Case |
|------|---------|----------|
| Device Report | Discovered SIP devices | Network inventory |
| Extension Report | Valid extensions found | User enumeration |
| Credential Report | Cracked passwords | Security audit |
| Combined Report | All findings | Full assessment |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipvicious
```

### Verify Installation

```bash
svreport --version
which svreport
```

### Dependencies

| Package | Purpose |
|---------|---------|
| python3 | Runtime |
| sqlite3 | Database access |
| reportlab (optional) | PDF generation |

## 3. Core Concepts

### Data Storage

sipvicious stores results in SQLite databases:

```
~/.sipvicious/
├── svmap_*.db        # Scanner results
├── svwar_*.db        # Enumeration results
├── svcrack_*.db      # Credential results
└── scan_log.db       # Global scan log
```

### Database Schema

```sql
-- Devices table (svmap results)
CREATE TABLE devices (
    ip TEXT,
    port INTEGER,
    user_agent TEXT,
    response_code INTEGER,
    timestamp DATETIME
);

-- Extensions table (svwar results)
CREATE TABLE extensions (
    extension TEXT,
    status TEXT,
    method TEXT,
    timestamp DATETIME
);

-- Credentials table (svcrack results)
CREATE TABLE credentials (
    extension TEXT,
    password TEXT,
    timestamp DATETIME
);
```

### Report Generation Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  SQLite DB  │────▶│  svreport   │────▶│  PDF/CSV    │
│  (Results)  │     │  (Query)    │     │  (Report)   │
└─────────────┘     └─────────────┘     └─────────────┘
```

## 4. Command Reference

### Basic Syntax

```
svreport <command> [options]
```

### Commands

| Command | Description |
|---------|-------------|
| `list` | List all stored results |
| `export` | Export results to file |
| `search` | Search results by query |
| `delete` | Delete specific results |
| `purge` | Remove old results |

### Export Options

| Flag | Description | Example |
|------|-------------|---------|
| `-f` | Output format | `-f csv` |
| `-o` | Output filename | `-o report.pdf` |
| `--query` | Filter query | `--query "extension:1000"` |
| `--all` | Export all data | `--all` |

### Search Options

| Flag | Description | Example |
|------|-------------|---------|
| `--query` | Search query | `--query "user_agent:Asterisk"` |
| `--field` | Specific field | `--field extension` |
| `--regex` | Regex pattern | `--regex "1[0-9]{3}"` |

### Delete/Purge Options

| Flag | Description | Example |
|------|-------------|---------|
| `--query` | Target query | `--query "ip:192.168.1.100"` |
| `--older-than` | Age filter (days) | `--older-than 30` |
| `--all` | Delete all | `--all` |

## 5. Examples

### List Results

```bash
# List all stored results
svreport list

# List specific database
svreport list --db ~/.sipvicious/svmap_192.168.1.0_24.db

# List with details
svreport list --verbose
```

### Export to CSV

```bash
# Export all results
svreport export -f csv -o results.csv

# Export with filter
svreport export -f csv -o devices.csv --query "response_code:200"

# Export specific table
svreport export -f csv -o extensions.csv --table extensions
```

### Export to PDF

```bash
# Generate PDF report
svreport export -f pdf -o audit_report.pdf

# PDF with custom title
svreport export -f pdf -o report.pdf --title "VoIP Security Audit"

# PDF with executive summary
svreport export -f pdf -o report.pdf --summary
```

### Search Results

```bash
# Search by IP
svreport search --query "ip:192.168.1.100"

# Search by User-Agent
svreport search --query "user_agent:Asterisk"

# Search by extension
svreport search --query "extension:1000"

# Search with regex
svreport search --regex "1[0-9]{3}"
```

### Delete Results

```bash
# Delete specific results
svreport delete --query "ip:192.168.1.100"

# Delete old results
svreport delete --older-than 30

# Delete all results
svreport delete --all
```

### Purge Data

```bash
# Remove old data
svreport purge --older-than 30

# Remove all data
svreport purge --all

# Preview before purge
svreport purge --older-than 30 --dry-run
```

### Advanced Reports

```bash
# Executive summary
svreport export -f pdf -o executive.pdf --summary --title "Executive Summary"

# Technical report
svreport export -f pdf -o technical.pdf --details --appendix

# Compliance report
svreport export -f pdf -o compliance.pdf --framework "PCI-DSS"
```

## 6. Advanced Usage

### Custom Report Templates

```bash
# Generate CSV with custom columns
svreport export -f csv -o custom.csv \
    --columns "ip,port,user_agent,extension"

# Filter by multiple criteria
svreport export -f csv -o filtered.csv \
    --query "ip:192.168.1.0/24 AND response_code:200"
```

### Batch Processing

```bash
#!/bin/bash
# Generate reports for multiple scans
for db in ~/.sipvicious/svmap_*.db; do
    name=$(basename $db .db)
    svreport export -f pdf -o "${name}.pdf" --db "$db"
    svreport export -f csv -o "${name}.csv" --db "$db"
done
```

### Integration with Other Tools

```bash
# Import CSV to spreadsheet
svreport export -f csv -o results.csv
libreoffice --calc results.csv

# Send via email
svreport export -f pdf -o report.pdf
mail -s "VoIP Audit Report" admin@company.com < report.pdf

# Archive results
svreport export -f csv -o results.csv
tar -czf sip_audit_$(date +%Y%m%d).tar.gz results.csv ~/.sipvicious/
```

### Database Analysis

```bash
# Direct SQLite queries
sqlite3 ~/.sipvicious/svmap_*.db "SELECT COUNT(*) FROM devices;"

sqlite3 ~/.sipvicious/svwar_*.db "SELECT extension, status FROM extensions;"

sqlite3 ~/.sipvicious/svcrack_*.db "SELECT extension, password FROM credentials;"
```

## 7. Detection and Defense

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| Database Access | Reading SQLite files |
| File Access | Accessing ~/.sipvicious/ |
| Export Activity | Generating PDF/CSV files |

### Defensive Measures

**File Protection:**

```bash
# Restrict database access
chmod 600 ~/.sipvicious/*.db

# Monitor access
auditctl -w ~/.sipvicious/ -p r -k sipvicious_report

# Encrypt databases
openssl enc -aes-256-cbc -salt -in db.db -out db.db.enc
```

**Detection Rules:**

```bash
# Audit rule
auditctl -w ~/.sipvicious/ -p rwa -k sipvicious_report

# inotifywait monitoring
inotifywait -m ~/.sipvicious/ -e access,modify
```

### Countermeasures for Operators

- Encrypt exported reports
- Secure PDF/CSV files
- Limit database access
- Archive old results
- Clean up after assessments

## 8. Troubleshooting

### Common Issues

#### Database Not Found

```bash
# Check database location
ls -la ~/.sipvicious/

# Verify scan results exist
sqlite3 ~/.sipvicious/svmap_*.db ".tables"

# Check permissions
ls -la ~/.sipvicious/*.db
```

#### Export Errors

```bash
# Verify reportlab installed
pip3 show reportlab

# Install if missing
pip3 install reportlab

# Try CSV export instead
svreport export -f csv -o results.csv
```

#### Search Returns Nothing

```bash
# List all results
svreport list

# Check query syntax
svreport search --query "ip:192.168.1.100"

# Try broader search
svreport search --query "192.168.1"
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| Slow export | Limit query results |
| Large files | Filter by date range |
| Memory issues | Export in chunks |

### Debug Commands

```bash
# Verbose output
svreport list --verbose

# Check database size
ls -lh ~/.sipvicious/*.db

# Direct SQLite access
sqlite3 ~/.sipvicious/svmap_*.db

# View schema
sqlite3 ~/.sipvicious/svmap_*.db ".schema"
```

### Data Recovery

```bash
# Check for backups
ls -la ~/.sipvicious/*.bak

# Recover from journal
sqlite3 ~/.sipvicious/svmap_*.db ".recover"

# Export raw data
sqlite3 -header -csv ~/.sipvicious/svmap_*.db \
    "SELECT * FROM devices;" > recovered.csv
```

## Resources

### Official Documentation
- [svreport GitHub](https://github.com/EnableSecurity/sipvicious)
- [sipvicious Wiki](https://github.com/EnableSecurity/sipvicious/wiki)

### Related Tools
- **svmap**: Network scanner
- **svwar**: Extension war dialer
- **svcrack**: Credential cracker
- **svcrash**: Scan controller

### Report Libraries
- [ReportLab - Python PDF](https://www.reportlab.com/)
- [SQLite Python](https://docs.python.org/3/library/sqlite3.html)

### Related MITRE ATT&CK Techniques
- **T1005 - Data from Local System**: Collecting scan results
- **T1039 - Data from Network Shared Drive**: Network data collection
- **T1560 - Archive Collected Data**: Compressing reports
