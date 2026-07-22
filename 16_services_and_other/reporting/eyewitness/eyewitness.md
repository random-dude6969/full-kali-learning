# EyeWitness - Web Screenshot Tool

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

EyeWitness is a powerful web screenshot automation tool designed for penetration testers and security professionals. It captures screenshots of websites, provides server header information, and identifies default credentials. EyeWitness is invaluable for reconnaissance, documentation, and creating visual evidence during security assessments.

### 1.1 What is EyeWitness?

EyeWitness is a Python-based tool that uses Selenium and Firefox/Chrome in headless mode to capture screenshots of web applications. It can process URLs from text files, Nmap XML output, and Nessus XML files.

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Author | Chris Truncer (Red Siege) |
| License | BSD |
| Language | Python 3 |
| Browser | Firefox (Geckodriver) |
| Version | 20230525.1 |
| Dependencies | Selenium, PyVirtualDisplay, PyFuzzyWuzzy |

### 1.3 Why Use EyeWitness?

- **Rapid Triage:** Quickly visualize hundreds of web servers
- **Documentation:** Capture evidence for reports
- **Default Credentials:** Identify default login pages
- **Service Discovery:** Visual confirmation of services
- **Automation:** Process large lists automatically
- **Reporting:** Generate HTML reports with screenshots

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install eyewitness

# Verify installation
eyewitness --help
```

### 2.2 From Source

```bash
# Clone repository
git clone https://github.com/FortyNorthSecurity/EyeWitness.git
cd EyeWitness

# Install dependencies
./setup.sh

# Or manually
pip3 install -r requirements.txt

# Install Firefox and Geckodriver
sudo apt install firefox xvfb
```

### 2.3 Dependencies

```bash
# Required packages
sudo apt install firefox geckodriver xvfb python3-selenium \
    python3-pyvirtualdisplay python3-fuzzywuzzy python3-netaddr
```

---

## 3. Basic Usage

### 3.1 Command Syntax

```bash
eyewitness [OPTIONS] -f <URL_FILE> | -x <XML_FILE> | --single <URL>
```

### 3.2 Simple Screenshot

```bash
# Single URL
eyewitness --single http://example.com -d screenshots

# From file
eyewitness -f urls.txt -d screenshots --headless

# From Nmap XML
eyewitness -x nmap_results.xml -d screenshots --headless
```

---

## 4. Command-Line Options

### 4.1 Input Options

| Option | Description |
|--------|-------------|
| `-f Filename` | Line-separated file with URLs |
| `-x Filename.xml` | Nmap XML or Nessus file |
| `--single URL` | Single URL to capture |

### 4.2 Web Options

| Option | Description |
|--------|-------------|
| `--web` | HTTP screenshot using Selenium |
| `--no-dns` | Skip DNS resolution |
| `--prepend-https` | Add http:// to URLs without protocol |

### 4.3 Timing Options

| Option | Description |
|--------|-------------|
| `--timeout N` | Max seconds to wait (default: 7) |
| `--jitter N` | Random delay between requests |
| `--delay N` | Delay before screenshot |
| `--max-retries N` | Max retries on timeout |

### 4.4 Output Options

| Option | Description |
|--------|-------------|
| `-d DIR` | Output directory name |
| `--results N` | Hosts per report page |
| `--no-prompt` | Don't prompt to open report |

### 4.5 Report Options

| Option | Description |
|--------|-------------|
| `--user-agent AGENT` | Custom user agent |
| `--proxy-ip IP` | Proxy IP address |
| `--proxy-port PORT` | Proxy port |
| `--proxy-type TYPE` | Proxy type (socks5/http) |

### 4.6 Advanced Options

| Option | Description |
|--------|-------------|
| `--show-selenium` | Show browser window |
| `--resolve` | Resolve IP/Hostname |
| `--add-http-ports PORTS` | Additional HTTP ports |
| `--add-https-ports PORTS` | Additional HTTPS ports |
| `--only-ports PORTS` | Exclusive ports |
| `--cookies KEY=VAL` | Additional cookies |

### 4.7 Resume Options

| Option | Description |
|--------|-------------|
| `--resume ew.db` | Resume from database file |

---

## 5. Scanning Modes

### 5.1 Text File Input

```bash
# Create urls.txt
echo "http://example.com" > urls.txt
echo "http://target.com" >> urls.txt
echo "https://192.168.1.1" >> urls.txt

# Scan
eyewitness -f urls.txt -d screenshots --headless
```

### 5.2 Nmap XML Input

```bash
# Generate Nmap XML
nmap -sV -sC -p- -oX scan.xml target.com

# Process with EyeWitness
eyewitness -x scan.xml -d screenshots --headless
```

### 5.3 Nessus XML Input

```bash
# Export from Nessus
# My Scans > Select > Export > Nessus

# Process with EyeWitness
eyewitness -x nessus_export.nessus -d screenshots --headless
```

### 5.4 Single URL

```bash
# Capture single URL
eyewitness --single http://example.com -d screenshots --headless
```

---

## 6. Screenshot Capture Options

### 6.1 Headless Mode

```bash
# Default headless mode (no GUI)
eyewitness -f urls.txt -d screenshots --headless
```

### 6.2 Visible Browser

```bash
# Show browser window (for debugging)
eyewitness -f urls.txt -d screenshots --show-selenium
```

### 6.3 Custom User Agent

```bash
# Mimic specific browser
eyewitness -f urls.txt -d screenshots \
    --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

### 6.4 With Cookies

```bash
# Add authentication cookies
eyewitness -f urls.txt -d screenshots \
    --cookies "session=abc123; logged_in=true"
```

---

## 7. Output Formats and Directory Structure

### 7.1 Output Directory

```
screenshots/
├──Eyewitness.html          # Main report
├──style.css                # Report styling
├──screenshots/
│   ├── http___example_com.png
│   ├── http___target_com.png
│   └── ...
└── Evidence.db             # SQLite database
```

### 7.2 Report Structure

The HTML report contains:
- Host information
- Screenshots
- Server headers
- Default credential checks
- Page titles

---

## 8. Threading and Performance

### 8.1 Thread Count

```bash
# Default (1 thread)
eyewitness -f urls.txt -d screenshots

# Multiple threads
eyewitness -f urls.txt -d screenshots --threads 10

# High concurrency
eyewitness -f urls.txt -d screenshots --threads 20
```

### 8.2 Rate Limiting

```bash
# Random delay between requests
eyewitness -f urls.txt -d screenshots --jitter 5

# Fixed delay
eyewitness -f urls.txt -d screenshots --delay 3

# Timeout settings
eyewitness -f urls.txt -d screenshots --timeout 15
```

---

## 9. Integration with Nmap Workflows

### 9.1 Complete Recon Workflow

```bash
# 1. Port scan
nmap -sV -sC -p- -oX scan.xml target.com

# 2. Process with EyeWitness
eyewitness -x scan.xml -d screenshots --headless

# 3. Review report
xdg-open screenshots/Eyewitness.html
```

### 9.2 Mass Scanning

```bash
# Scan multiple targets
nmap -iL targets.txt -sV -oX all_scans.xml

# Process all results
eyewitness -x all_scans.xml -d screenshots --headless --threads 10
```

### 9.3 Custom Ports

```bash
# Include non-standard ports
eyewitness -x scan.xml -d screenshots \
    --add-http-ports 8080,8443 \
    --add-https-ports 8443,443
```

---

## 10. Reporting Capabilities

### 10.1 HTML Report

The default output is an HTML report that can be opened in any browser:

```bash
# Open report
xdg-open screenshots/Eyewitness.html
```

### 10.2 Report Contents

- **Host Summary:** All scanned hosts
- **Screenshots:** Visual evidence
- **Server Headers:** Technology identification
- **Default Credentials:** Known default logins
- **Page Titles:** Application identification

### 10.3 Report Customization

```bash
# Custom results per page
eyewitness -f urls.txt -d screenshots --results 50

# Auto-open report
eyewitness -f urls.txt -d screenshots --no-prompt
```

---

## 11. Batch Processing

### 11.1 Large-Scale Processing

```bash
#!/bin/bash
# process_targets.sh

INPUT_FILE="targets.txt"
OUTPUT_DIR="batch_$(date +%Y%m%d)"
BATCH_SIZE=1000

# Split input file
split -l $BATCH_SIZE "$INPUT_FILE" batch_

# Process each batch
for batch in batch_*; do
    eyewitness -f "$batch" -d "${OUTPUT_DIR}_${batch}" \
        --headless --threads 10 --no-prompt
    rm "$batch"
done

# Combine results
# Manual merge of HTML reports
```

### 11.2 Automated Workflow

```bash
#!/bin/bash
# auto_recon.sh

TARGET=$1
DATE=$(date +%Y%m%d)

# Create output directory
mkdir -p recon_$DATE

# Nmap scan
nmap -sV -sC -p- -oX recon_$DATE/scan.xml $TARGET

# EyeWitness screenshots
eyewitness -x recon_$DATE/scan.xml -d recon_$DATE/screenshots \
    --headless --threads 10 --no-prompt

echo "Recon complete: recon_$DATE/"
```

---

## 12. Comparison with Alternatives

### 12.1 EyeWitness vs GoWitness

| Feature | EyeWitness | GoWitness |
|---------|------------|-----------|
| Language | Python | Go |
| Browser | Firefox | Chrome |
| Speed | Medium | Faster |
| Memory | Higher | Lower |
| Features | More | Fewer |

### 12.2 EyeWitness vs Aquatone

| Feature | EyeWitness | Aquatone |
|---------|------------|----------|
| Language | Python | Go |
| Browser | Firefox | Chrome |
| Cluster Support | No | Yes |
| Cloud Support | No | Yes |

### 12.3 EyeWitness vs CutyCapt

| Feature | EyeWitness | CutyCapt |
|---------|------------|----------|
| Input | Files, XML | URLs |
| Output | HTML report | Images |
| Batch | Yes | Manual |
| Headers | Yes | No |

---

## 13. Common Reconnaissance Scenarios

### 13.1 External Penetration Test

```bash
# 1. DNS enumeration
dig target.com ANY

# 2. Port scan
nmap -sV -sC -p- -oX ext_scan.xml target.com

# 3. Screenshots
eyewitness -x ext_scan.xml -d external_recon --headless

# 4. Web technology fingerprint
whatweb -i urls.txt
```

### 13.2 Internal Network Assessment

```bash
# 1. Internal scan
nmap -sV -sC -p- -oX int_scan.xml 192.168.1.0/24

# 2. Process results
eyewitness -x int_scan.xml -d internal_recon --headless

# 3. Review and categorize
# Open HTML report and review screenshots
```

### 13.3 Bug Bounty Recon

```bash
# 1. Subdomain enumeration
subfinder -d target.com -o subdomains.txt

# 2. Probe for live hosts
cat subdomains.txt | httpx -o live_hosts.txt

# 3. Screenshots
eyewitness -f live_hosts.txt -d bugbounty_recon --headless
```

---

## 14. Best Practices

### 14.1 Naming Conventions

```bash
# Use descriptive output directories
eyewitness -f urls.txt -d target_external_20240115

# Or use dates
eyewitness -f urls.txt -d recon_$(date +%Y%m%d)
```

### 14.2 Rate Limiting

```bash
# Be polite - add delays
eyewitness -f urls.txt -d screenshots --jitter 5 --timeout 15

# For large scans, use threading carefully
eyewitness -f urls.txt -d screenshots --threads 5
```

### 14.3 Documentation Workflow

1. Capture screenshots during recon
2. Review HTML report
3. Identify interesting targets
4. Document findings in CherryTree/Dradis
5. Export for final report

---

## 15. Troubleshooting

### Common Issues

**Problem:** EyeWitness won't start

```bash
# Check dependencies
pip3 install -r requirements.txt

# Check Firefox/Geckodriver
which firefox
which geckodriver

# Install if missing
sudo apt install firefox geckodriver
```

**Problem:** Blank screenshots

```bash
# Increase timeout
eyewitness -f urls.txt -d screenshots --timeout 15

# Add delay
eyewitness -f urls.txt -d screenshots --delay 3

# Show browser for debugging
eyewitness -f urls.txt -d screenshots --show-selenium
```

**Problem:** Connection errors

```bash
# Check network connectivity
curl http://example.com

# Use proxy
eyewitness -f urls.txt -d screenshots \
    --proxy-ip 127.0.0.1 --proxy-port 8080 --proxy-type http
```

**Problem:** Resume interrupted scan

```bash
# Use resume option
eyewitness -f urls.txt -d screenshots --resume ew.db
```

---

## References

- [EyeWitness GitHub](https://github.com/FortyNorthSecurity/EyeWitness)
- [EyeWitness Documentation](https://github.com/FortyNorthSecurity/EyeWitness/wiki)
- [Kali Linux EyeWitness Package](https://www.kali.org/tools/eyewitness/)
- [Red Siege Blog](https://www.redsiege.com/blog/)
