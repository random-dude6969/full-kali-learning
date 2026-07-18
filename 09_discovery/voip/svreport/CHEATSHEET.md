---
title: "svreport Cheatsheet"
description: "Quick reference for svreport sipvicious reporting tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - reporting
tags:
  - svreport
  - sipvicious
  - reporting
  - quick-reference
---

# svreport Cheatsheet

## Quick Start

```bash
# List results
svreport list

# Export to CSV
svreport export -f csv -o results.csv

# Export to PDF
svreport export -f pdf -o report.pdf

# Search results
svreport search --query "ip:192.168.1.100"
```

## Commands

| Command | Description |
|---------|-------------|
| `list` | List stored results |
| `export` | Export to file |
| `search` | Search results |
| `delete` | Delete results |
| `purge` | Remove old data |

## Common Flags

### Export

| Flag | Description | Example |
|------|-------------|---------|
| `-f` | Format (csv/pdf) | `-f csv` |
| `-o` | Output file | `-o report.pdf` |
| `--query` | Filter query | `--query "ip:192.168.1.100"` |
| `--table` | Specific table | `--table devices` |
| `--title` | PDF title | `--title "Audit Report"` |
| `--summary` | Executive summary | `--summary` |

### Search

| Flag | Description | Example |
|------|-------------|---------|
| `--query` | Search query | `--query "extension:1000"` |
| `--regex` | Regex pattern | `--regex "1[0-9]{3}"` |

### Delete/Purge

| Flag | Description | Example |
|------|-------------|---------|
| `--query` | Target query | `--query "ip:192.168.1.100"` |
| `--older-than` | Age (days) | `--older-than 30` |
| `--all` | All results | `--all` |
| `--dry-run` | Preview | `--dry-run` |

## Examples

### List Results

```bash
# All results
svreport list

# Specific database
svreport list --db ~/.sipvicious/svmap_*.db

# Verbose
svreport list --verbose
```

### Export CSV

```bash
# All results
svreport export -f csv -o results.csv

# Filtered
svreport export -f csv -o devices.csv --query "response_code:200"

# Specific table
svreport export -f csv -o extensions.csv --table extensions
```

### Export PDF

```bash
# Basic PDF
svreport export -f pdf -o report.pdf

# With title
svreport export -f pdf -o report.pdf --title "VoIP Audit"

# Executive summary
svreport export -f pdf -o report.pdf --summary
```

### Search

```bash
# By IP
svreport search --query "ip:192.168.1.100"

# By User-Agent
svreport search --query "user_agent:Asterisk"

# By extension
svreport search --query "extension:1000"

# Regex
svreport search --regex "1[0-9]{3}"
```

### Delete/Purge

```bash
# Delete by query
svreport delete --query "ip:192.168.1.100"

# Delete old
svreport delete --older-than 30

# Purge old data
svreport purge --older-than 30

# Preview purge
svreport purge --older-than 30 --dry-run
```

## Chaining

### With Scan Tools

```bash
# Scan → Report
svmap 192.168.1.0/24 -o scan.txt
svreport export -f csv -o devices.csv

# Enum → Report
svwar -e 1000-9999 192.168.1.100
svreport export -f csv -o extensions.csv --table extensions

# Crack → Report
svcrack -u 1000 -d passwords.txt 192.168.1.100
svreport export -f pdf -o credentials.pdf
```

### With Other Tools

```bash
# Export → Email
svreport export -f pdf -o report.pdf
mail -s "Audit Report" admin@company.com < report.pdf

# Export → Archive
svreport export -f csv -o results.csv
tar -czf audit_$(date +%Y%m%d).tar.gz results.csv

# Export → Spreadsheet
svreport export -f csv -o results.csv
libreoffice --calc results.csv
```

### Database Queries

```bash
# Direct SQLite
sqlite3 ~/.sipvicious/svmap_*.db "SELECT * FROM devices;"

sqlite3 ~/.sipvicious/svwar_*.db "SELECT extension, status;"

sqlite3 ~/.sipvicious/svcrack_*.db "SELECT extension, password;"

# Count results
sqlite3 ~/.sipvicious/svmap_*.db "SELECT COUNT(*) FROM devices;"
```

## Troubleshooting

### Database Not Found

```bash
# Check location
ls -la ~/.sipvicious/

# Verify tables
sqlite3 ~/.sipvicious/svmap_*.db ".tables"

# Check permissions
ls -la ~/.sipvicious/*.db
```

### Export Errors

```bash
# Check reportlab
pip3 show reportlab

# Install if missing
pip3 install reportlab

# Try CSV instead
svreport export -f csv -o results.csv
```

### Search Returns Nothing

```bash
# List all
svreport list

# Check query
svreport search --query "ip:192.168.1.100"

# Broader search
svreport search --query "192.168.1"
```

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | No results |

### Debug Commands

```bash
# Verbose
svreport list --verbose

# Database size
ls -lh ~/.sipvicious/*.db

# Schema
sqlite3 ~/.sipvicious/svmap_*.db ".schema"

# Raw export
sqlite3 -header -csv ~/.sipvicious/svmap_*.db \
    "SELECT * FROM devices;" > raw.csv
```

---

**Last Updated**: 2026
**Tool Version**: sipvicious 0.3.4
**Category**: SIP Reporting
