# Metagoofil Cheatsheet

## Quick Start

```bash
# Basic scan - PDFs from target domain
metagoofil -d example.com -f pdf -o results/

# All document types
metagoofil -d example.com -f pdf,doc,docx,ppt,pptx,odt,xls,xlsx -o results/

# Limited results, CSV output
metagoofil -d target.com -f pdf,doc,docx -l 50 -o results/ -F csv
```

## Command Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain | `-d example.com` |
| `-f` / `-t` | File types to search | `-f pdf,doc,docx` |
| `-l` | Max results | `-l 100` |
| `-n` | Results per page | `-n 10` |
| `-s` | Start result number | `-s 20` |
| `-o` | Output directory | `-o /tmp/results` |
| `-F` | Output format | `-F csv` / `-F html` / `-F json` |
| `-e` | Export metadata file | `-e metadata.txt` |
| `-d` | Delay between requests (sec) | `-d 30` |
| `-u` | Custom user agent | `-u "MyBot/1.0"` |
| `--timeout` | Download timeout | `--timeout 60` |
| `--threads` | Number of threads | `--threads 5` |
| `--verbose` | Verbose output | `--verbose` |

## Supported File Types

`pdf`, `doc`, `docx`, `ppt`, `pptx`, `odt`, `xls`, `xlsx`

## Common Workflows

### Pre-Engagement Recon
```bash
# 1. Enumerate domains
amass enum -passive -d target.com > domains.txt

# 2. Scan all domains
while read domain; do
    metagoofil -d "$domain" -f pdf,doc,docx -l 100 -o "results_$domain/" -F csv
done < domains.txt

# 3. Extract unique authors
find results_* -name "*.csv" -exec cat {} \; | cut -d',' -f4 | sort | uniq -c | sort -rn
```

### Bug Bounty Recon
```bash
metagoofil -d target.com -f pdf,doc,docx -l 200 -o bounty_results/
grep -ri "internal\|confidential\|draft" bounty_results/metadata/ 2>/dev/null
grep -ri "password\|api.key\|token" bounty_results/downloads/ 2>/dev/null
```

### Email Discovery
```bash
# Combine with TheHarvester
theharvester -d example.com -b google -f harvester.html
metagoofil -d example.com -f pdf -o meta_results/
grep -rhoE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' meta_results/ | sort | uniq
```

## Quick Analysis

```bash
# Extract all authors
grep -h "Author:" results/metadata/*.txt | sort | uniq -c | sort -rn

# Extract all emails
grep -rhoE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' results/ | sort | uniq

# Extract software versions
grep -h "Creator:" results/metadata/*.txt | sort | uniq -c | sort -rn

# Extract internal paths
grep -h "Path: C:\\" results/metadata/*.txt | sort | uniq
```

## Anti-Detection

```bash
# Rotate user agents
metagoofil -d target.com -f pdf -u "Mozilla/5.0 (Windows NT 10.0; Win64; x64)" -o results/

# Route through proxy
export http_proxy="http://proxy:8080"
metagoofil -d target.com -f pdf -o results/

# Slow down requests
metagoofil -d target.com -f pdf -d 30 -o results/
```

## Installation

```bash
sudo apt install metagoofil poppler-utils libimage-exiftool-perl pst-utils
pip3 install olefile
```

## Dependencies

| Tool | Purpose | Install |
|------|---------|---------|
| `pdfinfo` | PDF metadata | `sudo apt install poppler-utils` |
| `exiftool` | Universal metadata | `sudo apt install libimage-exiftool-perl` |
| `readpst` | PST files | `sudo apt install pst-utils` |
| `olefile` | Office docs | `pip3 install olefile` |
