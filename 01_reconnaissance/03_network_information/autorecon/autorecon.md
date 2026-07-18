---
tool_name: autorecon
display_name: AutoRecon
mitre_tactic: Reconnaissance
mitre_tactic_id: TA0043
mitre_subcategory: Active Scanning
install_command: pip install autorecon
version: 2.0.2
github_repo: https://github.com/Tib3rius/AutoRecon
kali_tools_page: https://www.kali.org/tools/autorecon/
difficulty_level: Beginner
requires_root: true
gui_available: false
tags:
  - reconnaissance
  - network-scanning
  - automation
  - pentest
  - enumeration
  - multi-threaded
  - service-detection
related_tools:
  - nmap
  - masscan
  - nikto
  - gobuster
  - smbclient
  - enum4linux
  - sslscan
  - dnsrecon
  - whatweb
  - wfuzz
---

# AutoRecon

## Table of Contents

1. [Chapter 1: Introduction & Overview](#chapter-1-introduction--overview)
2. [Chapter 2: Installation & Setup](#chapter-2-installation--setup)
3. [Chapter 3: Basic Usage](#chapter-3-basic-usage)
4. [Chapter 4: Intermediate Usage](#chapter-4-intermediate-usage)
5. [Chapter 5: Advanced Usage](#chapter-5-advanced-usage)
6. [Chapter 6: Real-World Scenarios](#chapter-6-real-world-scenarios)
7. [Chapter 7: Detection & Defense](#chapter-7-detection--defense)
8. [Chapter 8: Troubleshooting](#chapter-8-troubleshooting)
9. [Resources](#resources)

---

## Chapter 1: Introduction & Overview

### What is AutoRecon?

AutoRecon is a multi-threaded network reconnaissance tool developed by Tib3rius that automates port scanning, service detection, and vulnerability identification. It orchestrates multiple security tools into a cohesive workflow, providing comprehensive reconnaissance with minimal manual effort.

#### Core Capabilities

- **Automated Port Scanning**: TCP and UDP port discovery using nmap and masscan
- **Service Detection**: Automatic identification of running services and versions
- **Tool Orchestration**: Runs appropriate enumeration tools based on discovered services
- **Multi-threading**: Parallel scanning for speed
- **Organized Output**: Structured results organized by host and service
- **Plugin System**: Extensible architecture for custom tools

#### Architecture Overview

AutoRecon operates in distinct phases:

```
Phase 1: TCP Port Discovery (nmap/masscan)
    ↓
Phase 2: UDP Port Discovery (nmap)
    ↓
Phase 3: Service Detection & Version Scanning
    ↓
Phase 4: Automatic Tool Selection & Execution
    ↓
Phase 5: Result Organization & Reporting
```

### History and Development

AutoRecon was created to address the repetitive nature of manual network reconnaissance. Tib3rius, a well-known figure in the penetration testing community (creator of Privilege Escalation Awesome Scripts Suite - PEASS), developed AutoRecon to:

- Standardize reconnaissance methodology
- Reduce time spent on repetitive tasks
- Ensure comprehensive service coverage
- Maintain consistent documentation

The tool has evolved from a simple port scanner wrapper to a full-featured reconnaissance framework supporting dozens of enumeration tools.

### When to Use AutoRecon

#### Ideal Use Cases

1. **Penetration Testing** - Initial reconnaissance phase
2. **Security Assessments** - Network auditing and compliance
3. **Bug Bounty Hunting** - Scope discovery and attack surface mapping
4. **CTF Competitions** - Rapid service identification
5. **Red Team Operations** - Infrastructure mapping
6. **Incident Response** - Post-breach reconnaissance
7. **Network Monitoring** - Periodic security audits

#### When NOT to Use AutoRecon

- **Stealth-Critical Operations** - AutoRecon generates noticeable traffic
- **Rate-Limited Networks** - May trigger IDS/IPS
- **Extremely Large Networks** - May require distributed approach
- **Non-Standard Services** - Custom protocols need manual tools

### Legal Considerations

#### Authorization Requirements

**WARNING**: Always obtain explicit written authorization before scanning any system you do not own.

```bash
$ autorecon 192.168.1.1
[ERROR] You must have authorization to scan the target!
[ERROR] Unauthorized scanning is illegal and unethical!
[ERROR] Use: autorecon --help for usage information
```

#### Legal Frameworks

- **Computer Fraud and Abuse Act (CFAA)** - US federal law
- **General Data Protection Regulation (GDPR)** - EU data protection
- **Computer Misuse Act** - UK computer crime law
- **Local Laws** - Varies by jurisdiction

#### Best Practices

1. **Written Authorization**: Obtain signed agreement before testing
2. **Scope Definition**: Clearly define IP ranges and ports
3. **Time Windows**: Scan during agreed-upon hours
4. **Communication**: Maintain contact with client during testing
5. **Documentation**: Record all activities for audit trail

### Key Concepts

#### Threading Model

AutoRecon uses Python's threading capabilities to parallelize operations:

- **Port Scanner Threads**: Multiple nmap/masscan instances
- **Enumeration Threads**: Parallel tool execution
- **Configurable Concurrency**: Control via `--threads` flag

#### Plugin Architecture

AutoRecon's plugin system allows:

- Service-specific tool selection
- Custom tool arguments
- Timeout configuration
- Output parsing

#### Output Organization

Results follow a consistent structure:

```
results/
└── [timestamp]/
    └── [target]/
        ├── TCP_[port]/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   └── [tool_output].txt
        └── summary.txt
```

---

## Chapter 2: Installation & Setup

### Installation Methods

#### Method 1: pip Install (Recommended)

```bash
$ pip install autorecon
Collecting autorecon
  Downloading autorecon-2.0.2-py3-none-any.whl (24 kB)
Installing collected packages: autorecon
Successfully installed autorecon-2.0.2

$ autorecon --version
autorecon 2.0.2
```

#### Method 2: GitHub Clone

```bash
$ git clone https://github.com/Tib3rius/AutoRecon.git
Cloning into 'AutoRecon'...
remote: Enumerating objects: 1234, done.
remote: Total 1234 (delta 0), reused 0 (delta 0), pack-reused 1234
Receiving objects: 100% (1234/1234), 256.78 KiB | 5.12 MiB/s, done.
Resolving deltas: 100% (567/567), done.

$ cd AutoRecon
$ pip install -r requirements.txt
$ pip install -e .
```

#### Method 3: Docker

```bash
$ docker pull tib3rius/autorecon
Using default tag: latest
latest: Pulling from tib3rius/autorecon
Digest: sha256:abc123...
Status: Downloaded newer image for tib3rius/autorecon:latest

$ docker run -v $(pwd)/output:/output tib3rius/autorecon 192.168.1.1
```

#### Method 4: Kali Linux

```bash
$ sudo apt update && sudo apt install autorecon
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  autorecon
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 24.5 kB of archives.
After this operation, 102 kB of additional disk space will be used.
Get:1 http://kali mirrors/kali kali-rolling/main amd64 autorecon all 2.0.2-0kali1 [24.5 kB]
Fetched 24.5 kB in 0s (102 kB/s)
Selecting previously unselected package autorecon.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../autorecon_2.0.2-0kali1_all.deb ...
Unpacking autorecon (2.0.2-0kali1) ...
Setting up autorecon (2.0.2-0kali1) ...
```

### Dependency Installation

#### Core Dependencies

```bash
# Debian/Ubuntu/Kali
$ sudo apt update
$ sudo apt install -y \
    nmap \
    masscan \
    whatweb \
    nikto \
    gobuster \
    smbclient \
    enum4linux \
    sslscan \
    dnsrecon \
    python3-pip

# Verify installation
$ which nmap nikto whatweb smbclient
/usr/bin/nmap
/usr/bin/nikto
/usr/bin/whatweb
/usr/bin/smbclient
```

#### Complete Tool Installation Script

```bash
#!/bin/bash
# install_autorecon_deps.sh

echo "=== Installing AutoRecon Dependencies ==="

# Update system
sudo apt update && sudo apt upgrade -y

# Core tools
echo "[+] Installing core tools..."
sudo apt install -y nmap masscan python3-pip git

# Web tools
echo "[+] Installing web tools..."
sudo apt install -y whatweb nikto dirb gobuster wfuzz ffuf

# Network tools
echo "[+] Installing network tools..."
sudo apt install -y smbclient enum4linux enum4linux-ng sslscan testssl.sh dnsrecon fierce dnsenum

# SSL/TLS tools
echo "[+] Installing SSL tools..."
sudo apt install -y sslscan testssl.sh sslyze

# SNMP tools
echo "[+] Installing SNMP tools..."
sudo apt install -y snmp snmpcheck onesixtyone snmpwalk

# SMTP tools
echo "[+] Installing SMTP tools..."
sudo apt install -y smtp-user-enum

# Install AutoRecon
echo "[+] Installing AutoRecon..."
pip3 install autorecon

echo "=== Installation Complete ==="
autorecon --version
```

### Configuration

#### Configuration File Location

```bash
# Default config location
~/.config/AutoRecon/config.yml

# Create config directory
$ mkdir -p ~/.config/AutoRecon
```

#### Configuration File Format

```yaml
# ~/.config/AutoRecon/config.yml

# General settings
general:
  threads: 10
  timeout: 300
  max_retries: 3
  
# Scan settings
scanning:
  tcp_ports: "1-65535"
  udp_ports: "top-100"
  scan_rate: 10000
  
# Tool-specific settings
tools:
  nmap:
    args: "-sV -sC"
    timeout: 600
    
  nikto:
    args: "-Tuning x6"
    timeout: 300
    
  gobuster:
    wordlist: "/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt"
    extensions: "php,html,js,txt"
    threads: 50
    
  smbclient:
    args: "-L -N"
    timeout: 120
    
  whatweb:
    args: "--color=never"
    timeout: 180
    
  sslscan:
    args: "--no-colour"
    timeout: 300

# Output settings
output:
  base_dir: "./results"
  verbose: false
  quiet: false
```

#### Custom Wordlists

```bash
# Create custom wordlist directory
$ mkdir -p ~/.config/AutoRecon/wordlists

# Download common wordlists
$ wget -O ~/.config/AutoRecon/wordlists/common.txt \
    https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt
--2024-01-15 14:30:00--  https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt
Resolving raw.githubusercontent.com... 151.101.2.133
Connecting to raw.githubusercontent.com|151.101.2.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 220585 (215K) [text/plain]
Saving to: '/home/user/.config/AutoRecon/wordlists/common.txt'

common.txt         100%[===================>] 215.42K  1.23MB/s    in 0.2s

2024-01-15 14:30:01 (1.23 MB/s) - 'common.txt' saved [220585/220585]
```

#### Environment Variables

```bash
# Add to ~/.bashrc or ~/.zshrc
export AUTORECON_THREADS=20
export AUTORECON_TIMEOUT=600
export AUTORECON_WORDLIST="/usr/share/wordlists/dirb/common.txt"

# Reload shell
$ source ~/.bashrc
```

### Verification

#### Test Installation

```bash
$ autorecon --version
autorecon 2.0.2

$ autorecon --help
usage: autorecon [-h] [--version] [-t TARGETS [TARGETS ...]] [-o OUTPUT_DIR]
                 [--threads THREADS] [--timeout TIMEOUT] [--ports PORTS]
                 [--no-udp] [--no-tcp] [--nmap-args NMAP_ARGS]
                 [--plugins PLUGINS] [--no-plugins NO_PLUGINS]
                 [-v] [-q]

AutoRecon - Multi-threaded network reconnaissance tool

options:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -t, --targets         target IP address(es) or CIDR range
  -o, --output-dir      output directory for results
  --threads             number of threads (default: 10)
  --timeout             scan timeout in seconds (default: 300)
  --ports               port specification (default: tcp=top-1000)
  --no-udp              disable UDP scanning
  --no-tcp              disable TCP scanning
  --nmap-args           additional nmap arguments
  --plugins             enable specific plugins
  --no-plugins          disable specific plugins
  -v, --verbose         verbose output
  -q, --quiet           suppress output

$ which nmap nikto whatweb smbclient
/usr/bin/nmap
/usr/bin/nikto
/usr/bin/whatweb
/usr/bin/smbclient
```

#### Health Check Script

```bash
#!/bin/bash
# autorecon-health.sh

echo "=== AutoRecon Health Check ==="

# Check AutoRecon
if command -v autorecon &> /dev/null; then
    echo "[✓] AutoRecon installed: $(autorecon --version)"
else
    echo "[✗] AutoRecon not found"
    exit 1
fi

# Check required tools
TOOLS=("nmap" "nikto" "whatweb" "smbclient" "gobuster" "sslscan")
for tool in "${TOOLS[@]}"; do
    if command -v "$tool" &> /dev/null; then
        echo "[✓] $tool: Found"
    else
        echo "[✗] $tool: Missing"
    fi
done

# Check config directory
if [ -d "$HOME/.config/AutoRecon" ]; then
    echo "[✓] Config directory exists"
else
    echo "[!] Config directory missing"
fi

echo ""
echo "=== Health Check Complete ==="
```

### Updating

```bash
# Update via pip
$ pip3 install --upgrade autorecon
Requirement already satisfied: autorecon in /usr/lib/python3/dist-packages (2.0.2)
Collecting autorecon
  Downloading autorecon-2.0.3-py3-none-any.whl (24 kB)
Installing collected packages: autorecon
  Attempting uninstall: autorecon
    Found existing installation: autorecon 2.0.2
    Uninstalling autorecon-2.0.2:
      Successfully uninstalled autorecon-2.0.2
Successfully installed autorecon-2.0.3

# Update from GitHub
$ cd AutoRecon
$ git pull
$ pip3 install -e .
```

---

## Chapter 3: Basic Usage

### First Run

#### Quick Start

```bash
# Basic scan against a target
$ autorecon 192.168.1.1
[INFO] Starting AutoRecon against 192.168.1.1
[INFO] TCP port scanning with nmap...
[INFO] Found 5 open TCP ports
[INFO] Starting service enumeration...
[INFO] Scan complete. Results saved to ./results/2024-01-15_14-30-00/192.168.1.1/

# Scan with output directory
$ autorecon 192.168.1.1 -o results
[INFO] Starting AutoRecon against 192.168.1.1
[INFO] Output directory: results/2024-01-15_14-30-00/192.168.1.1
[INFO] TCP port scanning with nmap...
[INFO] Found 3 open TCP ports
[INFO] Scan complete.

# Scan multiple targets
$ autorecon 192.168.1.1 192.168.1.2 192.168.1.3
[INFO] Starting AutoRecon against 3 targets
[INFO] Target 1: 192.168.1.1
[INFO] Target 2: 192.168.1.2
[INFO] Target 3: 192.168.1.3
[INFO] All scans complete.
```

### Command Structure

```
autorecon [options] target1 [target2 ...]
```

### Core Options

#### Target Specification

```bash
# Single IP
$ autorecon 192.168.1.1

# Multiple IPs
$ autorecon 192.168.1.1 192.168.1.2

# IP range
$ autorecon 192.168.1.1-100

# CIDR notation
$ autorecon 192.168.1.0/24

# Hostname
$ autorecon example.com

# From file
$ autorecon -iL targets.txt
```

#### Output Options

```bash
# Specify output directory
$ autorecon 192.168.1.1 -o /path/to/results

# Default output location
# ./results/[timestamp]/[target]/

# Verbose output
$ autorecon 192.168.1.1 -v
[INFO] Starting AutoRecon against 192.168.1.1
[DEBUG] Loading configuration from ~/.config/AutoRecon/config.yml
[DEBUG] Starting nmap scan with args: -sV -sC -O
[INFO] TCP port scanning with nmap...
[DEBUG] nmap command: nmap -sV -sC -O -oN nmap.txt -oX nmap.xml 192.168.1.1
[INFO] Found 4 open TCP ports
[DEBUG] Port 22: open (ssh OpenSSH 8.2)
[DEBUG] Port 80: open (http Apache 2.4.41)
[DEBUG] Port 443: open (https nginx 1.18.0)
[DEBUG] Port 8080: open (http Apache Tomcat 9.0.40)

# Quiet mode
$ autorecon 192.168.1.1 -q
[Silent scan with minimal output]
```

### Flag Reference Tables

#### General Flags

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `-h` | `--help` | Show help message | `autorecon -h` |
| `-v` | `--verbose` | Verbose output | `autorecon -v 192.168.1.1` |
| `-q` | `--quiet` | Suppress output | `autorecon -q 192.168.1.1` |
| | `--version` | Show version | `autorecon --version` |

#### Target Flags

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `-t` | `--targets` | Target specification | `autorecon -t 192.168.1.1` |
| `-iL` | `--target-list` | Targets from file | `autorecon -iL targets.txt` |

#### Output Flags

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `-o` | `--output-dir` | Output directory | `autorecon -o ./results` |
| `-v` | `--verbose` | Verbose output | `autorecon -v` |
| `-q` | `--quiet` | Quiet mode | `autorecon -q` |

#### Scan Flags

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `--ports` | | Port specification | `autorecon --ports tcp=1-1000` |
| `--no-udp` | | Disable UDP scanning | `autorecon --no-udp` |
| `--no-tcp` | | Disable TCP scanning | `autorecon --no-tcp` |
| `--no-port-scan` | | Skip port scan phase | `autorecon --no-port-scan` |
| `--thread` | `--threads` | Thread count | `autorecon --threads 20` |
| `--timeout` | | Scan timeout | `autorecon --timeout 600` |

#### Tool Flags

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `--nmap-args` | | Additional nmap args | `autorecon --nmap-args "-T2"` |
| `--plugins` | | Enable plugins | `autorecon --plugins nikto,gobuster` |
| `--no-plugins` | | Disable plugins | `autorecon --no-plugins smbclient` |

### Scan Options

#### Port Specification

```bash
# Specify TCP ports
$ autorecon 192.168.1.1 --ports tcp=1-1000
[INFO] Scanning TCP ports 1-1000

# Specify UDP ports
$ autorecon 192.168.1.1 --ports udp=53,161,162
[INFO] Scanning UDP ports 53,161,162

# All ports
$ autorecon 192.168.1.1 --ports tcp=1-65535
[INFO] Scanning all TCP ports (1-65535)
[INFO] This may take a while...

# Top ports only
$ autorecon 192.168.1.1 --ports tcp=top-1000
[INFO] Scanning top 1000 TCP ports

# Combined TCP and UDP
$ autorecon 192.168.1.1 --ports tcp=1-10000,udp=top-100
[INFO] Scanning TCP ports 1-10000 and UDP top-100 ports
```

#### Threading Options

```bash
# Set thread count
$ autorecon 192.168.1.1 --threads 20
[INFO] Using 20 threads for scanning

# Default threads: 10
$ autorecon 192.168.1.1
[INFO] Using 10 threads (default)
```

### Common Workflows

#### Example 1: Quick Scan

```bash
# Fast scan of common ports
$ autorecon 192.168.1.1 \
    --ports tcp=top-100 \
    --threads 20 \
    -o quick_scan
[INFO] Starting quick scan against 192.168.1.1
[INFO] Scanning top 100 TCP ports with 20 threads
[INFO] Found 3 open ports in 45 seconds
[INFO] Results saved to quick_scan/2024-01-15_14-30-00/192.168.1.1/
```

#### Example 2: Comprehensive Scan

```bash
# Full port scan with all tools
$ autorecon 192.168.1.1 \
    --ports tcp=1-65535,udp=top-100 \
    --threads 10 \
    -o comprehensive_scan
[INFO] Starting comprehensive scan against 192.168.1.1
[INFO] Scanning all TCP ports and top 100 UDP ports
[INFO] This may take 30-60 minutes depending on network
[INFO] Found 12 open TCP ports and 3 open UDP ports
[INFO] Results saved to comprehensive_scan/2024-01-15_14-30-00/192.168.1.1/
```

#### Example 3: Web Server Focus

```bash
# Focus on HTTP/HTTPS services
$ autorecon 192.168.1.1 \
    --ports tcp=80,443,8080,8443 \
    --no-udp \
    -o web_scan
[INFO] Starting web-focused scan against 192.168.1.1
[INFO] Scanning HTTP/HTTPS ports only
[INFO] Found 2 web servers
[INFO] Running whatweb, nikto, and gobuster
[INFO] Results saved to web_scan/2024-01-15_14-30-00/192.168.1.1/
```

#### Example 4: Network Range

```bash
# Scan entire subnet
$ autorecon 192.168.1.0/24 \
    --ports tcp=top-100 \
    --threads 50 \
    -o network_scan
[INFO] Starting network scan against 192.168.1.0/24
[INFO] Scanning 254 hosts with top 100 TCP ports
[INFO] Using 50 threads for parallel scanning
[INFO] Found 47 hosts with open ports
[INFO] Results saved to network_scan/2024-01-15_14-30-00/
```

#### Example 5: Multiple Targets

```bash
# Scan multiple hosts
$ autorecon 192.168.1.1 192.168.1.2 192.168.1.3 \
    -o multi_target
[INFO] Starting scan against 3 targets
[INFO] Target 1: 192.168.1.1 - Found 5 open ports
[INFO] Target 2: 192.168.1.2 - Found 3 open ports
[INFO] Target 3: 192.168.1.3 - Found 8 open ports
[INFO] All scans complete
[INFO] Results saved to multi_target/2024-01-15_14-30-00/
```

### Output Structure

#### Default Directory Layout

```
results/
└── 2024-01-15_14-30-00/
    └── 192.168.1.1/
        ├── TCP_22/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   └── ssh_audit.txt
        ├── TCP_80/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   ├── whatweb.txt
        │   ├── nikto.txt
        │   └── gobuster.txt
        ├── TCP_443/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   ├── sslscan.txt
        │   └── nikto.txt
        ├── TCP_445/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   ├── smbclient.txt
        │   └── enum4linux.txt
        ├── UDP_161/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   └── snmpwalk.txt
        └── summary.txt
```

#### Understanding Output Files

```bash
# View scan summary
$ cat results/*/192.168.1.1/summary.txt
AutoRecon Scan Summary
======================
Target: 192.168.1.1
Scan Time: 2024-01-15 14:30:00
Duration: 342 seconds

Open TCP Ports:
  22/tcp - ssh (OpenSSH 8.2p1)
  80/tcp - http (Apache httpd 2.4.41)
  443/tcp - https (nginx 1.18.0)
  445/tcp - microsoft-ds (Samba smbd 4.11.6)
  8080/tcp - http (Apache Tomcat 9.0.40)

Open UDP Ports:
  161/snmp (net-snmp 5.8)

Tools Executed:
  nmap, whatweb, nikto, gobuster, smbclient, enum4linux, sslscan

# View Nmap results
$ cat results/*/192.168.1.1/TCP_80/nmap.txt
Nmap scan report for 192.168.1.1
Host is up (0.0023s latency).
Scanned at 2024-01-15 14:30:00 UTC for 42s

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
Service detection performed. Please report any incorrect results.
Nmap done: 1 IP address (1 host up) scanned in 42.34 seconds

# View web enumeration
$ cat results/*/192.168.1.1/TCP_80/nikto.txt
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:     192.168.1.1
+ Target Hostname: 192.168.1.1
+ Target Port:   80
+ Start Time:    2024-01-15 14:30:15
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ /admin/: Admin directory found.
+ /backup/: Backup directory found.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7915 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:       2024-01-15 14:32:45 (150 seconds)
---------------------------------------------------------------------------
```

### Tool-Specific Output

#### Nmap Output

```xml
<!-- nmap.xml - Machine-readable format -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<nmaprun args="nmap -sV -sC -O -oN nmap.txt -oX nmap.xml 192.168.1.1"
         start="1705337400" startstr="2024-01-15 14:30:00"
         version="7.92" xmloutputversion="1.05">
  <scaninfo type="syn" protocol="tcp" numservices="1" services="80"/>
  <host starttime="1705337400" endtime="1705337442">
    <status state="up" reason="echo-reply" reason_ttl="64"/>
    <address addr="192.168.1.1" addrtype="ipv4"/>
    <hostnames>
      <hostname name="192.168.1.1" type="ptr"/>
    </hostnames>
    <ports>
      <port protocol="tcp" portid="80">
        <state state="open" reason="syn-ack" reason_ttl="64"/>
        <service name="http" product="Apache httpd" version="2.4.41"
                 extrainfo="(Ubuntu)" method="probed" conf="10">
          <cpe>cpe:/a:apache:http_server:2.4.41</cpe>
        </service>
        <script id="http-methods" output="Supported Methods: GET HEAD POST OPTIONS">
          <table key="http-methods">
            <elem key="GET">true</elem>
            <elem key="HEAD">true</elem>
            <elem key="POST">true</elem>
            <elem key="OPTIONS">true</elem>
          </table>
        </script>
      </port>
    </ports>
    <os>
      <osmatch name="Linux 4.15 - 5.6" accuracy="96" line="1234">
        <osclass type="general purpose" vendor="Linux" osfamily="Linux"
                 osgen="4.X" accuracy="96">
          <cpe>cpe:/o:linux:linux_kernel:4</cpe>
        </osclass>
      </osmatch>
    </os>
  </host>
  <runstats>
    <finished time="1705337442" timestr="2024-01-15 14:30:42" elapsed="42.34"
              summary="Nmap done at 2024-01-15 14:30:42; 1 IP address (1 host up) scanned in 42.34 seconds"
              exit="success"/>
    <hosts up="1" down="0" total="1"/>
  </runstats>
</nmaprun>
```

#### Web Scan Output

```bash
# whatweb output
$ whatweb -v 192.168.1.1
http://192.168.1.1 [200 OK] Apache[2.4.41], Country[US], HTTPServer[Apache Linux], IP[192.168.1.1], Title[Apache2 Ubuntu Default Page]

# nikto output
$ nikto -h 192.168.1.1
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:     192.168.1.1
+ Target Hostname: 192.168.1.1
+ Target Port:   80
+ Start Time:    2024-01-15 14:30:15
---------------------------------------------------------------------------
+ Server: Apache/2.4.41
+ /admin/: Admin directory found
+ /backup/: Backup directory found
+ /config.php.bak: Configuration backup file found
+ /wp-admin/: WordPress admin found
+ OSVDB-3268: /icons/: Directory indexing found
---------------------------------------------------------------------------

# gobuster output
$ gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.1.0
===============================================================
[+] Url:                     http://192.168.1.1
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Status codes:            200,204,301,302,307,401,403
[+] Content Length:          10918
===============================================================
2024/01/15 14:31:22 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 310] [--> http://192.168.1.1/admin/]
/backup               (Status: 301) [Size: 309] [--> http://192.168.1.1/backup/]
/icons                (Status: 301) [Size: 309] [--> http://192.168.1.1/icons/]
/index.html           (Status: 200) [Size: 10918]
/server-status        (Status: 403) [Size: 277]
===============================================================
2024/01/15 14:31:45 Finished
===============================================================
```

### Processing Results

#### Quick Analysis

```bash
# Find all open ports
$ grep -r "open" results/*/TCP_*/nmap.txt | grep -v "closed\|filtered"
results/2024-01-15_14-30-00/192.168.1.1/TCP_22/nmap.txt:22/tcp open  ssh OpenSSH 8.2p1
results/2024-01-15_14-30-00/192.168.1.1/TCP_80/nmap.txt:80/tcp open  http Apache 2.4.41
results/2024-01-15_14-30-00/192.168.1.1/TCP_443/nmap.txt:443/tcp open  https nginx 1.18.0

# Find web servers
$ grep -rl "http" results/*/*/nmap.txt
results/2024-01-15_14-30-00/192.168.1.1/TCP_80/nmap.txt
results/2024-01-15_14-30-00/192.168.1.1/TCP_443/nmap.txt

# Find SMB services
$ grep -rl "smb" results/*/*/nmap.txt
results/2024-01-15_14-30-00/192.168.1.1/TCP_445/nmap.txt

# Find all services
$ grep -r "service:" results/*/*/nmap.txt | sort -u
results/2024-01-15_14-30-00/192.168.1.1/TCP_22/nmap.txt:Service: ssh
results/2024-01-15_14-30-00/192.168.1.1/TCP_80/nmap.txt:Service: http
results/2024-01-15_14-30-00/192.168.1.1/TCP_443/nmap.txt:Service: https
results/2024-01-15_14-30-00/192.168.1.1/TCP_445/nmap.txt:Service: microsoft-ds
```

#### Pipeline Integration

```bash
# Extract IPs from results
$ grep -rh "Nmap scan report for" results/*/*/nmap.txt | \
    awk '{print $NF}' | sort -u > discovered_ips.txt
$ cat discovered_ips.txt
192.168.1.1
192.168.1.2
192.168.1.3

# Extract hostnames
$ grep -rh "Nmap scan report for" results/*/*/nmap.txt | \
    awk '{print $5}' | sort -u > discovered_hostnames.txt
$ cat discovered_hostnames.txt
192.168.1.1
webserver.local
dc01.local

# Feed to other tools
$ cat discovered_ips.txt | nmap -iL - -sV -oA further_scan
Nmap scan report for 192.168.1.1
Nmap scan report for 192.168.1.2
Nmap scan report for 192.168.1.3
```

---

## Chapter 4: Intermediate Usage

### Automation with Bash

#### Batch Scanning Script

```bash
#!/bin/bash
# batch_autorecon.sh - Scan multiple targets from file

TARGETS_FILE=$1
OUTPUT_BASE="batch_scan_$(date +%Y%m%d)"

if [ -z "$TARGETS_FILE" ]; then
    echo "Usage: $0 <targets.txt>"
    exit 1
fi

mkdir -p "$OUTPUT_BASE"

echo "[*] Starting batch AutoRecon scan"
echo "[*] Targets: $TARGETS_FILE"
echo "[*] Output: $OUTPUT_BASE"

# Read targets and scan
while IFS= read -r target; do
    echo ""
    echo "========================================="
    echo "Scanning: $target"
    echo "========================================="
    
    # Create target-specific directory
    TARGET_DIR="$OUTPUT_BASE/$(echo $target | tr '.' '_')"
    mkdir -p "$TARGET_DIR"
    
    # Run AutoRecon
    autorecon "$target" \
        -o "$TARGET_DIR" \
        --threads 20 \
        2>&1 | tee "$TARGET_DIR/log.txt"
    
    echo "[+] Completed: $target"
    
done < "$TARGETS_FILE"

echo ""
echo "[+] Batch scan complete!"
echo "[*] Results in: $OUTPUT_BASE"
```

**Example Output:**
```bash
$ ./batch_autorecon.sh targets.txt
[*] Starting batch AutoRecon scan
[*] Targets: targets.txt
[*] Output: batch_scan_20240115

=========================================
Scanning: 192.168.1.1
=========================================
[INFO] Starting AutoRecon against 192.168.1.1
[INFO] TCP port scanning with nmap...
[INFO] Found 5 open TCP ports
[INFO] Scan complete.
[+] Completed: 192.168.1.1

=========================================
Scanning: 192.168.1.2
=========================================
[INFO] Starting AutoRecon against 192.168.1.2
[INFO] TCP port scanning with nmap...
[INFO] Found 3 open TCP ports
[INFO] Scan complete.
[+] Completed: 192.168.1.2

[+] Batch scan complete!
[*] Results in: batch_scan_20240115
```

#### Parallel Scanning

```bash
#!/bin/bash
# parallel_autorecon.sh - Run multiple scans in parallel

TARGETS=("192.168.1.1" "192.168.1.2" "192.168.1.3")
PARALLEL=3

echo "[*] Starting parallel AutoRecon scans"
echo "[*] Targets: ${TARGETS[*]}"
echo "[*] Parallel jobs: $PARALLEL"

# Function to run scan
scan_target() {
    local target=$1
    local output="parallel_$(echo $target | tr '.' '_')"
    
    echo "[+] Starting scan: $target"
    autorecon "$target" -o "$output" --threads 10
    
    echo "[+] Completed: $target -> $output"
}

# Export function
export -f scan_target

# Run in parallel
printf '%s\n' "${TARGETS[@]}" | xargs -P $PARALLEL -I {} bash -c 'scan_target "$@"' _

echo "[+] All scans complete!"
```

**Example Output:**
```bash
$ ./parallel_autorecon.sh
[*] Starting parallel AutoRecon scans
[*] Targets: 192.168.1.1 192.168.1.2 192.168.1.3
[*] Parallel jobs: 3
[+] Starting scan: 192.168.1.1
[+] Starting scan: 192.168.1.2
[+] Starting scan: 192.168.1.3
[+] Completed: 192.168.1.1 -> parallel_192_168_1_1
[+] Completed: 192.168.1.2 -> parallel_192_168_1_2
[+] Completed: 192.168.1.3 -> parallel_192_168_1_3
[+] All scans complete!
```

#### Automated Report Generation

```bash
#!/bin/bash
# generate_report.sh - Create HTML report from AutoRecon results

RESULTS_DIR=$1

if [ -z "$RESULTS_DIR" ]; then
    echo "Usage: $0 <results_directory>"
    exit 1
fi

REPORT="report_$(date +%Y%m%d).html"

cat > "$REPORT" << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>AutoRecon Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        h1 { color: #333; }
        .host { border: 1px solid #ddd; padding: 10px; margin: 10px 0; }
        .service { background: #f5f5f5; padding: 5px; margin: 5px 0; }
        pre { background: #000; color: #0f0; padding: 10px; overflow-x: auto; }
    </style>
</head>
<body>
    <h1>AutoRecon Reconnaissance Report</h1>
EOF

# Find all targets
for target_dir in "$RESULTS_DIR"/*/; do
    target=$(basename "$target_dir")
    
    cat >> "$REPORT" << EOF
    <div class="host">
        <h2>Target: $target</h2>
EOF
    
    # Find all services
    for service_dir in "$target_dir"*/; do
        service=$(basename "$service_dir")
        
        cat >> "$REPORT" << EOF
        <div class="service">
            <h3>$service</h3>
EOF
        
        # Include key findings
        if [ -f "$service_dir/nmap.txt" ]; then
            echo "<pre>" >> "$REPORT"
            head -50 "$service_dir/nmap.txt" >> "$REPORT"
            echo "</pre>" >> "$REPORT"
        fi
        
        echo "        </div>" >> "$REPORT"
    done
    
    echo "    </div>" >> "$REPORT"
done

echo "</body></html>" >> "$REPORT"

echo "[+] Report generated: $REPORT"
```

**Example Output:**
```bash
$ ./generate_report.sh results/2024-01-15_14-30-00/
[+] Report generated: report_20240115.html
```

### Automation with Python

#### Python AutoRecon Wrapper

```python
#!/usr/bin/env python3
"""
autorecon_wrapper.py - Python wrapper for AutoRecon automation
"""

import subprocess
import json
import os
import time
from pathlib import Path
from typing import List, Dict, Optional
import argparse
from datetime import datetime


class AutoReconWrapper:
    """Wrapper class for AutoRecon automation"""
    
    def __init__(self, output_dir: str = "autorecon_results"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        
    def run_scan(self, target: str, **kwargs) -> Dict:
        """Run AutoRecon scan against target"""
        print(f"[*] Starting AutoRecon scan against {target}")
        
        output_dir = self.output_dir / target.replace('.', '_')
        output_dir.mkdir(exist_ok=True)
        
        cmd = ["autorecon", target, "-o", str(output_dir)]
        
        # Add options
        if 'ports' in kwargs:
            cmd.extend(["--ports", kwargs['ports']])
        if 'threads' in kwargs:
            cmd.extend(["--threads", str(kwargs['threads'])])
        if 'quiet' in kwargs and kwargs['quiet']:
            cmd.append("-q")
        
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=kwargs.get('timeout', 3600)
            )
            
            return {
                "target": target,
                "status": "success",
                "output_dir": str(output_dir),
                "returncode": result.returncode,
                "stdout": result.stdout,
                "stderr": result.stderr
            }
            
        except subprocess.TimeoutExpired:
            return {
                "target": target,
                "status": "timeout",
                "output_dir": str(output_dir)
            }
        except Exception as e:
            return {
                "target": target,
                "status": "error",
                "error": str(e)
            }
    
    def batch_scan(self, targets: List[str], **kwargs) -> List[Dict]:
        """Run scans against multiple targets"""
        results = []
        
        for target in targets:
            result = self.run_scan(target, **kwargs)
            results.append(result)
            
            # Brief pause between scans
            time.sleep(2)
        
        return results
    
    def parse_results(self, target: str) -> Dict:
        """Parse AutoRecon results for a target"""
        target_dir = self.output_dir / target.replace('.', '_')
        
        if not target_dir.exists():
            return {"error": "Results directory not found"}
        
        parsed = {
            "target": target,
            "services": [],
            "open_ports": [],
            "findings": []
        }
        
        # Parse Nmap results
        for nmap_file in target_dir.rglob("nmap.txt"):
            service_dir = nmap_file.parent.name
            parsed["services"].append(service_dir)
            
            with open(nmap_file) as f:
                content = f.read()
                
                # Extract open ports
                for line in content.split('\n'):
                    if '/tcp' in line and 'open' in line:
                        parsed["open_ports"].append(line.strip())
        
        return parsed
    
    def generate_summary(self, target: str) -> str:
        """Generate human-readable summary"""
        results = self.parse_results(target)
        
        summary = [
            "=" * 60,
            f"AutoRecon Summary: {target}",
            "=" * 60,
            "",
            f"Open Ports: {len(results.get('open_ports', []))}",
            f"Services Scanned: {len(results.get('services', []))}",
            "",
            "Open Ports:",
            "-" * 40
        ]
        
        for port in results.get('open_ports', []):
            summary.append(f"  {port}")
        
        return "\n".join(summary)


def main():
    parser = argparse.ArgumentParser(description="AutoRecon Wrapper")
    parser.add_argument("-t", "--target", help="Single target")
    parser.add_argument("-f", "--file", help="Targets file")
    parser.add_argument("-o", "--output", default="autorecon_results")
    parser.add_argument("-p", "--ports", help="Port specification")
    parser.add_argument("--threads", type=int, default=10)
    
    args = parser.parse_args()
    
    wrapper = AutoReconWrapper(args.output)
    
    if args.target:
        result = wrapper.run_scan(
            args.target,
            ports=args.ports,
            threads=args.threads
        )
        print(wrapper.generate_summary(args.target))
    elif args.file:
        with open(args.file) as f:
            targets = [line.strip() for line in f if line.strip()]
        
        results = wrapper.batch_scan(
            targets,
            ports=args.ports,
            threads=args.threads
        )
        
        print(f"\n[+] Completed {len(results)} scans")
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
```

**Example Usage:**
```bash
$ python3 autorecon_wrapper.py -t 192.168.1.1
[*] Starting AutoRecon scan against 192.168.1.1
============================================================
AutoRecon Summary: 192.168.1.1
============================================================

Open Ports: 5
Services Scanned: 5

Open Ports:
----------------------------------------
  22/tcp open  ssh OpenSSH 8.2p1
  80/tcp open  http Apache 2.4.41
  443/tcp open  https nginx 1.18.0
  445/tcp open  microsoft-ds Samba smbd 4.11.6
  8080/tcp open  http Apache Tomcat 9.0.40
```

#### Results Analyzer

```python
#!/usr/bin/env python3
"""
analyze_autorecon.py - Analyze and correlate AutoRecon results
"""

import json
import os
import re
from pathlib import Path
from typing import Dict, List, Set
from collections import defaultdict, Counter
import argparse


class AutoReconAnalyzer:
    """Analyze AutoRecon scan results"""
    
    def __init__(self, results_dir: str):
        self.results_dir = Path(results_dir)
        self.all_services = defaultdict(list)
        self.all_ports = defaultdict(list)
        self.vulnerabilities = []
        
    def scan_all_results(self):
        """Scan all results in directory"""
        for target_dir in self.results_dir.iterdir():
            if target_dir.is_dir():
                self.analyze_target(target_dir.name)
    
    def analyze_target(self, target: str):
        """Analyze results for single target"""
        target_dir = self.results_dir / target
        
        if not target_dir.exists():
            return
        
        # Find all service directories
        for service_dir in target_dir.iterdir():
            if service_dir.is_dir():
                service_name = service_dir.name
                self.all_services[target].append(service_name)
                
                # Parse Nmap results
                nmap_file = service_dir / "nmap.txt"
                if nmap_file.exists():
                    self._parse_nmap(target, nmap_file)
                
                # Parse web results
                for tool in ['nikto', 'whatweb', 'gobuster']:
                    tool_file = service_dir / f"{tool}.txt"
                    if tool_file.exists():
                        self._parse_tool_output(target, tool, tool_file)
    
    def _parse_nmap(self, target: str, nmap_file: Path):
        """Parse Nmap output"""
        with open(nmap_file) as f:
            content = f.read()
        
        # Extract open ports
        port_pattern = r'(\d+)/tcp\s+open\s+(\S+)\s*(.*)'
        for match in re.finditer(port_pattern, content):
            port, service, version = match.groups()
            self.all_ports[target].append({
                'port': int(port),
                'service': service,
                'version': version.strip()
            })
        
        # Extract vulnerabilities from scripts
        vuln_pattern = r'VULNERABLE\s*\n\s*State:\s*VULNERABLE'
        if re.search(vuln_pattern, content):
            self.vulnerabilities.append({
                'target': target,
                'file': str(nmap_file),
                'type': 'nmap_vulnerability'
            })
    
    def _parse_tool_output(self, target: str, tool: str, tool_file: Path):
        """Parse tool-specific output"""
        with open(tool_file) as f:
            content = f.read()
        
        # Check for common vulnerability indicators
        if tool == 'nikto':
            vuln_patterns = [
                r'OSVDB-\d+',
                r'Vulnerability Found',
                r'SQL Injection',
                r'XSS'
            ]
            for pattern in vuln_patterns:
                if re.search(pattern, content, re.IGNORECASE):
                    self.vulnerabilities.append({
                        'target': target,
                        'tool': tool,
                        'file': str(tool_file),
                        'type': 'web_vulnerability'
                    })
                    break
    
    def get_statistics(self) -> Dict:
        """Get overall statistics"""
        stats = {
            'total_targets': len(self.all_services),
            'total_services': sum(len(s) for s in self.all_services.values()),
            'total_ports': sum(len(p) for p in self.all_ports.values()),
            'total_vulnerabilities': len(self.vulnerabilities),
            'common_services': Counter(),
            'common_ports': Counter()
        }
        
        for target, ports in self.all_ports.items():
            for port_info in ports:
                stats['common_services'][port_info['service']] += 1
                stats['common_ports'][port_info['port']] += 1
        
        return stats
    
    def generate_report(self) -> str:
        """Generate comprehensive report"""
        stats = self.get_statistics()
        
        report = [
            "=" * 60,
            "AUTORECON ANALYSIS REPORT",
            "=" * 60,
            "",
            "SUMMARY",
            "-" * 40,
            f"Targets scanned: {stats['total_targets']}",
            f"Total services: {stats['total_services']}",
            f"Total open ports: {stats['total_ports']}",
            f"Vulnerabilities found: {stats['total_vulnerabilities']}",
            "",
            "TOP SERVICES",
            "-" * 40
        ]
        
        for service, count in stats['common_services'].most_common(10):
            report.append(f"  {service}: {count} occurrences")
        
        report.extend([
            "",
            "TOP PORTS",
            "-" * 40
        ])
        
        for port, count in stats['common_ports'].most_common(10):
            report.append(f"  {port}: {count} occurrences")
        
        if self.vulnerabilities:
            report.extend([
                "",
                "VULNERABILITIES",
                "-" * 40
            ])
            
            for vuln in self.vulnerabilities[:10]:
                report.append(f"  [{vuln['type']}] {vuln['target']}")
        
        return "\n".join(report)


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("results_dir", help="AutoRecon results directory")
    parser.add_argument("--json", action="store_true", help="Output JSON")
    
    args = parser.parse_args()
    
    analyzer = AutoReconAnalyzer(args.results_dir)
    analyzer.scan_all_results()
    
    if args.json:
        stats = analyzer.get_statistics()
        print(json.dumps(stats, indent=2, default=str))
    else:
        print(analyzer.generate_report())


if __name__ == "__main__":
    main()
```

**Example Output:**
```bash
$ python3 analyze_autorecon.py results/2024-01-15_14-30-00/
============================================================
AUTORECON ANALYSIS REPORT
============================================================

SUMMARY
----------------------------------------
Targets scanned: 3
Total services: 16
Total open ports: 24
Vulnerabilities found: 5

TOP SERVICES
----------------------------------------
  http: 6 occurrences
  https: 3 occurrences
  ssh: 3 occurrences
  microsoft-ds: 3 occurrences
  snmp: 2 occurrences

TOP PORTS
----------------------------------------
  80: 3 occurrences
  443: 3 occurrences
  22: 3 occurrences
  445: 3 occurrences
  161: 2 occurrences

VULNERABILITIES
----------------------------------------
  [nmap_vulnerability] 192.168.1.1
  [web_vulnerability] 192.168.1.1
  [nmap_vulnerability] 192.168.1.2
```

### Integration with Other Tools

#### AutoRecon + Metasploit Pipeline

```bash
#!/bin/bash
# autorecon_msf.sh - Feed AutoRecon results to Metasploit

TARGET=$1
RESULTS_DIR="autorecon_$(date +%Y%m%d)"

echo "[*] Starting reconnaissance pipeline"

# Run AutoRecon
autorecon "$TARGET" -o "$RESULTS_DIR"

# Extract open ports
OPEN_PORTS=$(grep -rh "/tcp" "$RESULTS_DIR"/*/TCP_*/nmap.txt | \
    grep "open" | awk -F'/' '{print $1}' | sort -n | uniq)

echo "[+] Open ports: $OPEN_PORTS"

# Generate Metasploit resource script
cat > "$RESULTS_DIR/msf_commands.rc" << EOF
use auxiliary/scanner/portscan/tcp
set RHOSTS $TARGET
set PORTS $OPEN_PORTS
run

use auxiliary/scanner/discovery/udp_sweep
set RHOSTS $TARGET
run
EOF

echo "[+] Metasploit resource script: $RESULTS_DIR/msf_commands.rc"
echo "[*] Run with: msfconsole -r $RESULTS_DIR/msf_commands.rc"
```

**Example Output:**
```bash
$ ./autorecon_msf.sh 192.168.1.1
[*] Starting reconnaissance pipeline
[INFO] Starting AutoRecon against 192.168.1.1
[INFO] TCP port scanning with nmap...
[INFO] Found 5 open TCP ports
[INFO] Scan complete.
[+] Open ports: 22 80 443 445 8080
[+] Metasploit resource script: autorecon_20240115/msf_commands.rc
[*] Run with: msfconsole -r autorecon_20240115/msf_commands.rc
```

#### AutoRecon + Nuclei Integration

```python
#!/usr/bin/env python3
"""
autorecon_nuclei.py - Feed AutoRecon results to Nuclei
"""

import subprocess
from pathlib import Path
import json


class AutoReconNuclei:
    def __init__(self, results_dir: str):
        self.results_dir = Path(results_dir)
    
    def extract_urls(self) -> list:
        """Extract URLs from AutoRecon results"""
        urls = []
        
        for nmap_file in self.results_dir.rglob("nmap.txt"):
            with open(nmap_file) as f:
                for line in f:
                    if '/tcp' in line and 'open' in line:
                        if 'http' in line.lower():
                            # Extract port
                            port = line.split('/')[0]
                            target = nmap_file.parts[-2]
                            urls.append(f"http://{target}:{port}")
                            urls.append(f"https://{target}:{port}")
        
        return list(set(urls))
    
    def run_nuclei(self, urls: list):
        """Run Nuclei against discovered URLs"""
        output_file = self.results_dir / "nuclei_results.txt"
        
        # Write URLs to file
        urls_file = self.results_dir / "nuclei_targets.txt"
        with open(urls_file, 'w') as f:
            f.write('\n'.join(urls))
        
        # Run Nuclei
        cmd = [
            "nuclei",
            "-l", str(urls_file),
            "-o", str(output_file),
            "-severity critical,high,medium"
        ]
        
        subprocess.run(cmd)
        
        return output_file
    
    def run(self):
        """Execute full pipeline"""
        print("[*] Extracting URLs from AutoRecon results...")
        urls = self.extract_urls()
        print(f"[+] Found {len(urls)} URLs")
        
        if urls:
            print("[*] Running Nuclei scans...")
            self.run_nuclei(urls)
            print("[+] Nuclei scan complete")


def main():
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python autorecon_nuclei.py <results_dir>")
        sys.exit(1)
    
    pipeline = AutoReconNuclei(sys.argv[1])
    pipeline.run()


if __name__ == "__main__":
    main()
```

**Example Output:**
```bash
$ python3 autorecon_nuclei.py results/2024-01-15_14-30-00/
[*] Extracting URLs from AutoRecon results...
[+] Found 6 URLs
[*] Running Nuclei scans...
[+] Nuclei scan complete
```

### Tips for Intermediate Users

#### Performance Optimization

```bash
# Use masscan for faster port scanning
$ autorecon 192.168.1.1 --port-scanner masscan
[INFO] Using masscan for port scanning (faster)
[INFO] Scanning TCP ports 1-65535
[INFO] Completed in 12 seconds

# Increase thread count for faster enumeration
$ autorecon 192.168.1.1 --threads 50
[INFO] Using 50 threads for scanning
[INFO] Enumeration completed 3x faster

# Limit to specific services
$ autorecon 192.168.1.1 --no-udp --ports tcp=top-1000
[INFO] Scanning top 1000 TCP ports only
[INFO] UDP scanning disabled
```

#### Custom Tool Configuration

```bash
# Add custom wordlist for directory busting
$ autorecon 192.168.1.1 --gobuster-wordlist /path/to/wordlist.txt
[INFO] Using custom wordlist for gobuster
[INFO] Found 23 directories

# Skip specific tools
$ autorecon 192.168.1.1 --no-plugins nikto smbclient
[INFO] Skipping nikto and smbclient
[INFO] Running: whatweb, gobuster, sslscan

# Add extra Nmap scripts
$ autorecon 192.168.1.1 --nmap-args "--script=vuln"
[INFO] Running nmap with vulnerability scripts
[INFO] Found 2 potential vulnerabilities
```

---

## Chapter 5: Advanced Usage

### Evasion Techniques

#### Scan Rate Manipulation

```bash
# Slow scan to avoid detection
$ autorecon 192.168.1.1 \
    --nmap-args "-T2 --max-rate 100" \
    --threads 5
[INFO] Starting slow scan with rate limiting
[INFO] Max rate: 100 packets/second
[INFO] Scan may take 30+ minutes

# Random delays between scans
$ autorecon 192.168.1.1 \
    --nmap-args "-T3 --randomize-hosts" \
    --threads 3
[INFO] Randomizing host order
[INFO] Using 3 threads to reduce detection

# Fragment packets
$ autorecon 192.168.1.1 \
    --nmap-args "-f --mtu 24" \
    --no-udp
[INFO] Using packet fragmentation
[INFO] MTU: 24 bytes
[INFO] UDP scanning disabled
```

#### Source Port Manipulation

```bash
# Use source port 53 (DNS) to bypass some firewalls
$ autorecon 192.168.1.1 \
    --nmap-args "-g 53 -S 192.168.1.100"
[INFO] Spoofing source port 53 (DNS)
[INFO] Spoofing source IP: 192.168.1.100

# Spoof MAC address
$ autorecon 192.168.1.1 \
    --nmap-args "--spoof-mac 0"
[INFO] Randomizing MAC address
[INFO] Using randomized MAC for scanning
```

#### Timing and Evasion Profiles

```yaml
# ~/.config/AutoRecon/config.yml
evasion:
  paranoid:
    threads: 2
    timeout: 600
    nmap_args: "-T1 --max-rate 50 -f"
    scan_rate: 100
    
  sneaky:
    threads: 5
    timeout: 300
    nmap_args: "-T2 --max-rate 200"
    scan_rate: 500
    
  normal:
    threads: 10
    timeout: 180
    nmap_args: "-T3"
    scan_rate: 1000
    
  aggressive:
    threads: 50
    timeout: 60
    nmap_args: "-T5 --min-rate 5000"
    scan_rate: 10000
```

```bash
# Use evasion profile
$ autorecon 192.168.1.1 --profile sneaky
[INFO] Loading evasion profile: sneaky
[INFO] Threads: 5, Timeout: 300s
[INFO] Nmap args: -T2 --max-rate 200
[INFO] Starting stealth scan...
```

### Scaling for Large Engagements

#### Enterprise Network Scanning

```python
#!/usr/bin/env python3
"""
enterprise_autorecon.py - Scale AutoRecon for enterprise networks
"""

import subprocess
import multiprocessing
import json
from pathlib import Path
from typing import List, Dict
import ipaddress
import time


class EnterpriseAutoRecon:
    """Scale AutoRecon for large networks"""
    
    def __init__(self, output_base: str = "enterprise_scan"):
        self.output_base = Path(output_base)
        self.output_base.mkdir(exist_ok=True)
        
    def parse_network_range(self, cidr: str) -> List[str]:
        """Parse CIDR to list of IPs"""
        network = ipaddress.ip_network(cidr, strict=False)
        return [str(ip) for ip in network.hosts()]
    
    def scan_chunk(self, chunk: List[str], chunk_id: int) -> Dict:
        """Scan a chunk of IPs"""
        chunk_dir = self.output_base / f"chunk_{chunk_id}"
        chunk_dir.mkdir(exist_ok=True)
        
        # Create target list file
        targets_file = chunk_dir / "targets.txt"
        with open(targets_file, 'w') as f:
            f.write('\n'.join(chunk))
        
        # Run AutoRecon
        cmd = [
            "autorecon",
            "-iL", str(targets_file),
            "-o", str(chunk_dir),
            "--threads", "20",
            "--ports", "tcp=top-1000"
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        return {
            "chunk_id": chunk_id,
            "targets": len(chunk),
            "status": "success" if result.returncode == 0 else "failed",
            "output_dir": str(chunk_dir)
        }
    
    def parallel_scan(self, cidr: str, chunk_size: int = 25, workers: int = 4):
        """Scan network in parallel chunks"""
        print(f"[*] Scanning network: {cidr}")
        
        # Parse IPs
        ips = self.parse_network_range(cidr)
        print(f"[+] Total IPs: {len(ips)}")
        
        # Split into chunks
        chunks = [ips[i:i+chunk_size] for i in range(0, len(ips), chunk_size)]
        print(f"[+] Chunks: {len(chunks)}")
        
        # Run in parallel
        with multiprocessing.Pool(workers) as pool:
            results = pool.starmap(
                self.scan_chunk,
                [(chunk, i) for i, chunk in enumerate(chunks)]
            )
        
        # Aggregate results
        successful = sum(1 for r in results if r['status'] == 'success')
        print(f"\n[+] Completed: {successful}/{len(chunks)} chunks")
        
        return results
    
    def merge_results(self) -> Dict:
        """Merge all chunk results"""
        merged = {
            "targets": [],
            "services": [],
            "open_ports": []
        }
        
        for chunk_dir in self.output_base.glob("chunk_*"):
            for target_dir in chunk_dir.iterdir():
                if target_dir.is_dir():
                    merged["targets"].append(target_dir.name)
                    
                    # Parse Nmap results
                    for nmap_file in target_dir.rglob("nmap.txt"):
                        with open(nmap_file) as f:
                            for line in f:
                                if '/tcp' in line and 'open' in line:
                                    merged["open_ports"].append(line.strip())
        
        return merged


def main():
    import argparse
    
    parser = argparse.ArgumentParser()
    parser.add_argument("-c", "--cidr", required=True, help="Network CIDR")
    parser.add_argument("-o", "--output", default="enterprise_scan")
    parser.add_argument("--chunk-size", type=int, default=25)
    parser.add_argument("--workers", type=int, default=4)
    
    args = parser.parse_args()
    
    scanner = EnterpriseAutoRecon(args.output)
    scanner.parallel_scan(
        args.cidr,
        chunk_size=args.chunk_size,
        workers=args.workers
    )
    
    # Merge results
    results = scanner.merge_results()
    print(f"\n[+] Total targets: {len(results['targets'])}")
    print(f"[+] Total open ports: {len(results['open_ports'])}")


if __name__ == "__main__":
    main()
```

**Example Output:**
```bash
$ python3 enterprise_autorecon.py -c 10.0.0.0/24
[*] Scanning network: 10.0.0.0/24
[+] Total IPs: 254
[+] Chunks: 11
[+] Completed: 10/11 chunks

[+] Total targets: 47
[+] Total open ports: 312
```

#### Distributed Scanning

```bash
#!/bin/bash
# distributed_autorecon.sh - Distribute scans across multiple machines

NETWORKS=("10.0.0.0/24" "10.0.1.0/24" "10.0.2.0/24" "10.0.3.0/24")
SERVERS=("server1" "server2" "server3" "server4")

echo "[*] Starting distributed AutoRecon scan"

# Distribute networks to servers
for i in "${!NETWORKS[@]}"; do
    network="${NETWORKS[$i]}"
    server="${SERVERS[$((i % ${#SERVERS[@]}))]}"
    
    echo "[+] Assigning $network to $server"
    
    # SSH and run AutoRecon
    ssh "$server" "autorecon $network -o /data/scans/$(echo $network | tr '/' '_')" &
done

# Wait for all scans to complete
wait

echo "[+] All distributed scans complete!"

# Merge results
echo "[*] Merging results..."
for server in "${SERVERS[@]}"; do
    rsync -avz "$server:/data/scans/" ./merged_results/
done

echo "[+] Results merged to ./merged_results/"
```

**Example Output:**
```bash
$ ./distributed_autorecon.sh
[*] Starting distributed AutoRecon scan
[+] Assigning 10.0.0.0/24 to server1
[+] Assigning 10.0.1.0/24 to server2
[+] Assigning 10.0.2.0/24 to server3
[+] Assigning 10.0.3.0/24 to server4
[+] All distributed scans complete!
[*] Merging results...
sending incremental file list
10.0.0.0_24/
10.0.0.0_24/10.0.0.1/
...
[+] Results merged to ./merged_results/
```

### Custom Plugin Development

#### Creating Custom Plugins

```python
#!/usr/bin/env python3
"""
custom_plugin.py - Example AutoRecon custom plugin
"""

import subprocess
from pathlib import Path
from typing import Dict, List, Optional


class CustomPlugin:
    """Custom AutoRecon plugin"""
    
    name = "custom_scanner"
    description = "Custom security scanner plugin"
    
    # Service types this plugin handles
    service_types = ["http", "https"]
    
    def __init__(self, config: Dict):
        self.config = config
        self.wordlist = config.get('wordlist', '/usr/share/wordlists/dirb/common.txt')
    
    def run(self, target: str, port: int, service: str) -> Dict:
        """Run plugin against target"""
        results = {
            "target": target,
            "port": port,
            "service": service,
            "findings": []
        }
        
        # Run custom commands
        findings = self._run_custom_scan(target, port)
        results["findings"].extend(findings)
        
        return results
    
    def _run_custom_scan(self, target: str, port: int) -> List[Dict]:
        """Execute custom scan logic"""
        findings = []
        
        # Example: Check for common vulnerabilities
        url = f"http://{target}:{port}"
        
        # Check for sensitive files
        sensitive_paths = [
            "/.env",
            "/.git/config",
            "/backup.zip",
            "/config.php.bak"
        ]
        
        for path in sensitive_paths:
            cmd = ["curl", "-s", "-o", "/dev/null", "-w", "%{http_code}", f"{url}{path}"]
            
            try:
                result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
                
                if result.stdout.strip() == "200":
                    findings.append({
                        "type": "sensitive_file",
                        "path": path,
                        "severity": "high"
                    })
            except:
                continue
        
        return findings
    
    def parse_results(self, output: str) -> Dict:
        """Parse plugin output"""
        return {"raw_output": output}
```

#### Plugin Registration

```python
#!/usr/bin/env python3
"""
register_plugin.py - Register custom plugin with AutoRecon
"""

import yaml
from pathlib import Path


def register_plugin(plugin_name: str, plugin_path: str):
    """Register custom plugin in AutoRecon config"""
    config_path = Path.home() / ".config" / "AutoRecon" / "config.yml"
    
    # Load existing config
    if config_path.exists():
        with open(config_path) as f:
            config = yaml.safe_load(f)
    else:
        config = {}
    
    # Add plugin
    if 'plugins' not in config:
        config['plugins'] = {}
    
    config['plugins'][plugin_name] = {
        'path': plugin_path,
        'enabled': True
    }
    
    # Save config
    with open(config_path, 'w') as f:
        yaml.dump(config, f)
    
    print(f"[+] Plugin registered: {plugin_name}")


def main():
    import argparse
    
    parser = argparse.ArgumentParser()
    parser.add_argument("name", help="Plugin name")
    parser.add_argument("path", help="Plugin path")
    
    args = parser.parse_args()
    register_plugin(args.name, args.path)


if __name__ == "__main__":
    main()
```

**Example Usage:**
```bash
$ python3 register_plugin.py custom_scanner /path/to/custom_plugin.py
[+] Plugin registered: custom_scanner

$ autorecon --plugins custom_scanner 192.168.1.1
[INFO] Loading plugin: custom_scanner
[INFO] Running custom scan against 192.168.1.1
[INFO] Found 2 sensitive files
```

### Advanced Configuration

#### Custom Nmap Templates

```xml
<!-- ~/.config/AutoRecon/nmap_templates/stealth.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<nmaprun args="nmap -sS -sV -O -T2 --max-rate 100">
  <scaninfo type="syn" protocol="tcp"/>
</nmaprun>
```

#### Advanced YAML Configuration

```yaml
# ~/.config/AutoRecon/advanced_config.yml

# Advanced scanning profiles
profiles:
  stealth:
    description: "Stealthy reconnaissance"
    tcp_ports: "top-100"
    udp_ports: "none"
    threads: 5
    timeout: 600
    nmap_args: "-sS -T2 --max-rate 100 -f"
    tools:
      - whatweb
      - nikto
      
  comprehensive:
    description: "Full comprehensive scan"
    tcp_ports: "1-65535"
    udp_ports: "top-1000"
    threads: 20
    timeout: 300
    nmap_args: "-sV -sC -O"
    tools:
      - all
      
  web_focused:
    description: "Focus on web services"
    tcp_ports: "80,443,8080,8443,8000,8888"
    udp_ports: "none"
    threads: 10
    tools:
      - whatweb
      - nikto
      - gobuster
      - wfuzz

# Tool-specific configurations
tool_configs:
  gobuster:
    wordlist: "/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt"
    extensions: "php,html,js,txt,bak,old"
    threads: 50
    timeout: 300
    
  nikto:
    tuning: "1234567890abcde"
    timeout: 600
    args: "-C all"
    
  smbclient:
    args: "-L -N -d 1"
    timeout: 120
```

### Advanced Output Processing

#### Real-time Monitoring

```python
#!/usr/bin/env python3
"""
monitor_autorecon.py - Real-time monitoring of AutoRecon scans
"""

import os
import time
import json
from pathlib import Path
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler


class ScanMonitor(FileSystemEventHandler):
    """Monitor AutoRecon scan progress"""
    
    def __init__(self, results_dir: str):
        self.results_dir = Path(results_dir)
        self.stats = {
            "files_created": 0,
            "services_found": 0,
            "vulnerabilities": 0
        }
    
    def on_created(self, event):
        """Handle file creation events"""
        if not event.is_directory:
            self.stats["files_created"] += 1
            
            # Check for new service discovery
            file_path = Path(event.src_path)
            if file_path.name == "nmap.txt":
                self._analyze_nmap(file_path)
    
    def _analyze_nmap(self, nmap_file: Path):
        """Analyze new Nmap output"""
        try:
            with open(nmap_file) as f:
                content = f.read()
            
            # Count open ports
            open_ports = content.count("/tcp") - content.count("/tcp) closed")
            self.stats["services_found"] += open_ports
            
            # Check for vulnerabilities
            if "VULNERABLE" in content:
                self.stats["vulnerabilities"] += 1
                print(f"[!] VULNERABILITY FOUND: {nmap_file}")
            
        except Exception:
            pass
    
    def get_status(self) -> str:
        """Get current monitoring status"""
        return f"""
Scan Progress:
  Files created: {self.stats['files_created']}
  Services found: {self.stats['services_found']}
  Vulnerabilities: {self.stats['vulnerabilities']}
"""


def monitor_directory(results_dir: str):
    """Start monitoring directory"""
    event_handler = ScanMonitor(results_dir)
    observer = Observer()
    observer.schedule(event_handler, results_dir, recursive=True)
    observer.start()
    
    try:
        while True:
            time.sleep(5)
            print(event_handler.get_status())
    except KeyboardInterrupt:
        observer.stop()
    
    observer.join()


if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python monitor_autorecon.py <results_dir>")
        sys.exit(1)
    
    monitor_directory(sys.argv[1])
```

**Example Output:**
```bash
$ python3 monitor_autorecon.py results/2024-01-15_14-30-00/

Scan Progress:
  Files created: 12
  Services found: 8
  Vulnerabilities: 0

Scan Progress:
  Files created: 24
  Services found: 15
  Vulnerabilities: 1
[!] VULNERABILITY FOUND: results/2024-01-15_14-30-00/192.168.1.1/TCP_80/nmap.txt
```

---

## Chapter 6: Real-World Scenarios

### Penetration Testing Engagement

#### Scenario: Corporate Network Assessment

**Client**: Manufacturing company
**Scope**: Internal network assessment
**Duration**: 1 week

```bash
#!/bin/bash
# pentest_autorecon.sh - Penetration testing reconnaissance workflow

CLIENT_IP="10.0.0.0/16"
ENGAGEMENT="mfg_pentest_$(date +%Y%m%d)"
OUTPUT_DIR="/pentest/$ENGAGEMENT"

echo "=== Internal Network Assessment ==="
echo "Client: Manufacturing Corp"
echo "Scope: $CLIENT_IP"
echo "Engagement: $ENGAGEMENT"

mkdir -p "$OUTPUT_DIR"

# Phase 1: Network discovery
echo ""
echo "[Phase 1] Network Discovery"
echo "---------------------------"
nmap -sn "$CLIENT_IP" -oG "$OUTPUT_DIR/alive_hosts.txt"
ALIVE=$(grep "Up" "$OUTPUT_DIR/alive_hosts.txt" | wc -l)
echo "Alive hosts: $ALIVE"

# Phase 2: Port scanning
echo ""
echo "[Phase 2] Port Scanning"
echo "-----------------------"
autorecon "$CLIENT_IP" \
    -o "$OUTPUT_DIR" \
    --ports tcp=top-1000 \
    --threads 50 \
    --no-udp

# Phase 3: Service enumeration
echo ""
echo "[Phase 3] Service Enumeration"
echo "-----------------------------"
# Extract live hosts with open ports
grep -rh "Nmap scan report for" "$OUTPUT_DIR"/*/TCP_*/nmap.txt | \
    awk '{print $NF}' | sort -u > "$OUTPUT_DIR/services_hosts.txt"

# Phase 4: Web application scanning
echo ""
echo "[Phase 4] Web Application Scanning"
echo "-----------------------------------"
while read -r host; do
    # Check for HTTP services
    if grep -q "$host" "$OUTPUT_DIR"/*/TCP_80*/nmap.txt 2>/dev/null; then
        echo "  Web server found: $host"
        nikto -h "$host" -o "$OUTPUT_DIR/nikto_$host.txt"
    fi
done < "$OUTPUT_DIR/services_hosts.txt"

echo ""
echo "=== Assessment Complete ==="
echo "Results: $OUTPUT_DIR"
```

**Example Output:**
```bash
$ ./pentest_autorecon.sh
=== Internal Network Assessment ===
Client: Manufacturing Corp
Scope: 10.0.0.0/16
Engagement: mfg_pentest_20240115

[Phase 1] Network Discovery
---------------------------
Nmap scan report for 10.0.0.1
Nmap scan report for 10.0.0.2
...
Alive hosts: 247

[Phase 2] Port Scanning
-----------------------
[INFO] Starting AutoRecon against 10.0.0.0/16
[INFO] Scanning 247 hosts
[INFO] Found 1842 open TCP ports
[INFO] Scan complete.

[Phase 3] Service Enumeration
-----------------------------
[INFO] Extracting live hosts with services

[Phase 4] Web Application Scanning
-----------------------------------
  Web server found: 10.0.0.10
  Web server found: 10.0.0.25
  Web server found: 10.0.0.50

=== Assessment Complete ===
Results: /pentest/mfg_pentest_20240115
```

#### Deliverable Report

```markdown
## Internal Network Assessment Report

### Executive Summary
- Network range: 10.0.0.0/16
- Hosts discovered: 247
- Open ports found: 1,842
- Critical vulnerabilities: 12
- High vulnerabilities: 34

### Key Findings
1. **Unpatched Systems**: 23 systems running outdated software
2. **Weak Credentials**: Default credentials on 8 network devices
3. **Exposed Services**: SMB, RDP, and SSH accessible from multiple subnets
4. **Missing Updates**: Critical Windows patches missing on 45 hosts

### Recommendations
- Implement network segmentation
- Deploy patch management system
- Disable unnecessary services
- Implement strong password policies
```

### Bug Bounty Hunting

#### Scenario: Public Bug Bounty Program

**Target**: Technology company
**Focus**: Network services and web applications

```python
#!/usr/bin/env python3
"""
bugbounty_autorecon.py - Bug bounty reconnaissance automation
"""

import subprocess
import json
from pathlib import Path
from typing import List, Dict
import requests
import time


class BugBountyRecon:
    def __init__(self, program: str, scope: List[str]):
        self.program = program
        self.scope = scope
        self.output_dir = Path(f"bugbounty_{program}")
        self.output_dir.mkdir(exist_ok=True)
        
    def run_autorecon(self, target: str) -> Path:
        """Run AutoRecon against target"""
        output_dir = self.output_dir / target.replace('.', '_')
        
        cmd = [
            "autorecon",
            target,
            "-o", str(output_dir),
            "--ports", "tcp=top-1000",
            "--threads", "20"
        ]
        
        subprocess.run(cmd, capture_output=True)
        return output_dir
    
    def find_vulnerabilities(self, results_dir: Path) -> List[Dict]:
        """Extract vulnerabilities from results"""
        vulns = []
        
        for nmap_file in results_dir.rglob("nmap.txt"):
            with open(nmap_file) as f:
                content = f.read()
            
            # Check for vulnerable services
            vulnerable_services = [
                "vsftpd 2.3.4",
                "ProFTPD 1.3.5",
                "Apache/2.4.49",
                "Apache/2.4.50",
                "OpenSSH 7.2"
            ]
            
            for service in vulnerable_services:
                if service in content:
                    vulns.append({
                        "file": str(nmap_file),
                        "vulnerable_service": service,
                        "severity": "critical"
                    })
            
            # Check for anonymous access
            if "Anonymous FTP login allowed" in content:
                vulns.append({
                    "file": str(nmap_file),
                    "issue": "Anonymous FTP",
                    "severity": "high"
                })
        
        return vulns
    
    def check_web_vulnerabilities(self, results_dir: Path) -> List[Dict]:
        """Check web application vulnerabilities"""
        vulns = []
        
        # Check nikto results
        for nikto_file in results_dir.rglob("nikto.txt"):
            with open(nikto_file) as f:
                content = f.read()
            
            # Look for high/critical findings
            if "OSVDB-" in content:
                vulns.append({
                    "file": str(nikto_file),
                    "type": "web_vulnerability",
                    "severity": "high"
                })
        
        return vulns
    
    def generate_report(self) -> str:
        """Generate bug bounty report"""
        all_vulns = []
        
        for target in self.scope:
            target_dir = self.output_dir / target.replace('.', '_')
            if target_dir.exists():
                vulns = self.find_vulnerabilities(target_dir)
                all_vulns.extend(vulns)
        
        report = f"""
=== Bug Bounty Reconnaissance Report ===

Program: {self.program}
Scope: {', '.join(self.scope)}
Total Vulnerabilities: {len(all_vulns)}

VULNERABILITIES:
"""
        for vuln in all_vulns:
            report += f"\n  [{vuln.get('severity', 'unknown')}] {vuln.get('vulnerable_service', vuln.get('issue', 'Unknown'))}"
        
        return report


def main():
    recon = BugBountyRecon(
        program="tech_startup",
        scope=["app.target.com", "api.target.com", "admin.target.com"]
    )
    
    for target in recon.scope:
        print(f"[*] Scanning {target}...")
        recon.run_autorecon(target)
    
    print(recon.generate_report())


if __name__ == "__main__":
    main()
```

**Example Output:**
```bash
$ python3 bugbounty_autorecon.py
[*] Scanning app.target.com...
[*] Scanning api.target.com...
[*] Scanning admin.target.com...

=== Bug Bounty Reconnaissance Report ===

Program: tech_startup
Scope: app.target.com, api.target.com, admin.target.com
Total Vulnerabilities: 3

VULNERABILITIES:
  [critical] vsftpd 2.3.4
  [high] Anonymous FTP
  [high] web_vulnerability
```

### CTF Competition

#### Scenario: Network CTF Challenge

**Challenge Type**: Network reconnaissance and exploitation
**Goal**: Find flags across multiple services

```bash
#!/bin/bash
# ctf_autorecon.sh - CTF network reconnaissance

CHALLENGE_IP=$1
CHALLENGE_DIR="ctf_$(date +%Y%m%d_%H%M%S)"

echo "=== CTF Network Reconnaissance ==="
echo "Target: $CHALLENGE_IP"

mkdir -p "$CHALLENGE_DIR"

# Run AutoRecon with all tools
autorecon "$CHALLENGE_IP" \
    -o "$CHALLENGE_DIR" \
    --ports tcp=1-65535,udp=top-100 \
    --threads 50

# Analyze results
echo ""
echo "[*] Analyzing results..."

# Find open services
echo ""
echo "=== Open Services ==="
grep -rh "open" "$CHALLENGE_DIR"/*/TCP_*/nmap.txt | sort -u

# Find potential flags
echo ""
echo "=== Potential Flags ==="
for result_file in $(find "$CHALLENGE_DIR" -name "*.txt"); do
    grep -i "flag\|ctf\|key\|secret" "$result_file" 2>/dev/null
done

# Check for common CTF vulnerabilities
echo ""
echo "=== Vulnerability Check ==="
grep -rl "Anonymous" "$CHALLENGE_DIR" 2>/dev/null
grep -rl "default password" "$CHALLENGE_DIR" 2>/dev/null
grep -rl "weak encryption" "$CHALLENGE_DIR" 2>/dev/null

echo ""
echo "=== Scan Complete ==="
echo "Results: $CHALLENGE_DIR"
```

**Example Output:**
```bash
$ ./ctf_autorecon.sh 10.10.10.100
=== CTF Network Reconnaissance ===
Target: 10.10.10.100
[INFO] Starting AutoRecon against 10.10.10.100
[INFO] Scanning all TCP ports and top 100 UDP ports
[INFO] Found 12 open TCP ports and 2 open UDP ports
[INFO] Scan complete.

[*] Analyzing results...

=== Open Services ===
22/tcp open  ssh OpenSSH 7.6p1
80/tcp open  http Apache httpd 2.4.29
445/tcp open  microsoft-ds Samba smbd 4.7.6
3306/tcp open  mysql MySQL 5.7.25
8080/tcp open  http-proxy Squid http proxy 4.6

=== Potential Flags ===
flag{r3con_m4st3r}
CTF{smb_4n0nym0us}

=== Vulnerability Check ===
./ctf_20240115_143000/10.10.10.100/TCP_445/nmap.txt
./ctf_20240115_143000/10.10.10.100/TCP_22/nmap.txt

=== Scan Complete ===
Results: ctf_20240115_143000
```

### Red Team Operations

#### Scenario: Adversary Simulation

**Objective**: Map target infrastructure for initial access
**Method**: Stealthy reconnaissance with minimal footprint

```python
#!/usr/bin/env python3
"""
redteam_autorecon.py - Red team reconnaissance with stealth
"""

import subprocess
import json
import random
import time
from typing import List, Dict


class RedTeamRecon:
    def __init__(self, target: str):
        self.target = target
        self.results = {
            "target": target,
            "scan_time": None,
            "services": [],
            "vulnerabilities": []
        }
    
    def stealth_scan(self) -> Dict:
        """Perform stealthy reconnaissance"""
        print(f"[*] Starting stealth scan against {self.target}")
        
        # Random delay to avoid patterns
        delay = random.uniform(30, 120)
        print(f"[*] Waiting {delay:.0f} seconds to avoid detection...")
        time.sleep(delay)
        
        # Run AutoRecon with stealth settings
        cmd = [
            "autorecon",
            self.target,
            "-o", f"redteam_{self.target}",
            "--ports", "tcp=top-100",
            "--threads", "5",
            "--nmap-args", "-T2 --max-rate 100 -f"
        ]
        
        start_time = time.time()
        subprocess.run(cmd, capture_output=True)
        self.results["scan_time"] = time.time() - start_time
        
        return self.results
    
    def analyze_results(self, results_dir: str) -> List[Dict]:
        """Analyze scan results for attack vectors"""
        findings = []
        
        # Parse results
        from pathlib import Path
        
        for nmap_file in Path(results_dir).rglob("nmap.txt"):
            with open(nmap_file) as f:
                content = f.read()
            
            # Find attack vectors
            attack_vectors = [
                {"service": "SMB", "port": 445, "tool": "smbclient"},
                {"service": "SSH", "port": 22, "tool": "hydra"},
                {"service": "RDP", "port": 3389, "tool": "rdp_scan"},
                {"service": "WinRM", "port": 5985, "tool": "evil-winrm"}
            ]
            
            for vector in attack_vectors:
                if f"{vector['port']}/tcp" in content and "open" in content:
                    findings.append({
                        "target": self.target,
                        "service": vector["service"],
                        "port": vector["port"],
                        "exploit_tool": vector["tool"],
                        "risk": "high"
                    })
        
        return findings
    
    def generate_report(self) -> str:
        """Generate red team report"""
        report = f"""
=== Red Team Reconnaissance Report ===

Target: {self.target}
Scan Duration: {self.results['scan_time']:.0f} seconds

Attack Vectors Identified:
"""
        for finding in self.results.get("services", []):
            report += f"\n  {finding['service']} (Port {finding['port']}) -> {finding['exploit_tool']}"
        
        return report


def main():
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python redteam_autorecon.py <target>")
        sys.exit(1)
    
    recon = RedTeamRecon(sys.argv[1])
    recon.stealth_scan()
    
    print(recon.generate_report())


if __name__ == "__main__":
    main()
```

**Example Output:**
```bash
$ python3 redteam_autorecon.py 192.168.1.100
[*] Starting stealth scan against 192.168.1.100
[*] Waiting 74 seconds to avoid detection...
[INFO] Starting AutoRecon against 192.168.1.100
[INFO] Using stealth settings
[INFO] Scan complete.

=== Red Team Reconnaissance Report ===

Target: 192.168.1.100
Scan Duration: 2847 seconds

Attack Vectors Identified:
  SMB (Port 445) -> smbclient
  SSH (Port 22) -> hydra
```

### Incident Response

#### Scenario: Security Incident Investigation

**Objective**: Map attacker infrastructure after breach
**Method**: Rapid reconnaissance of affected systems

```bash
#!/bin/bash
# ir_autorecon.sh - Incident response reconnaissance

INCIDENT_ID=$1
AFFECTED_HOSTS="affected_hosts.txt"

echo "=== Incident Response Reconnaissance ==="
echo "Incident ID: $INCIDENT_ID"

# Create incident directory
IR_DIR="/forensics/$INCIDENT_ID"
mkdir -p "$IR_DIR"

# Read affected hosts
if [ ! -f "$AFFECTED_HOSTS" ]; then
    echo "[!] No affected hosts file found"
    exit 1
fi

# Run AutoRecon on affected hosts
echo "[*] Scanning affected hosts..."
while read -r host; do
    echo "  Scanning: $host"
    autorecon "$host" \
        -o "$IR_DIR/$host" \
        --ports tcp=top-1000 \
        --threads 10
done < "$AFFECTED_HOSTS"

# Analyze for indicators of compromise
echo ""
echo "[*] Analyzing for IOCs..."
for result_dir in "$IR_DIR"/*/; do
    # Check for suspicious services
    grep -l "4444\|5555\|6666\|31337" "$result_dir"/*/nmap.txt 2>/dev/null
done

echo ""
echo "=== IR Reconnaissance Complete ==="
echo "Results: $IR_DIR"
```

**Example Output:**
```bash
$ ./ir_autorecon.sh INC-2024-001
=== Incident Response Reconnaissance ===
Incident ID: INC-2024-001
[*] Scanning affected hosts...
  Scanning: 10.0.0.50
  Scanning: 10.0.0.51
  Scanning: 10.0.0.52
[INFO] Scan complete for all hosts.

[*] Analyzing for IOCs...
/forensics/INC-2024-001/10.0.0.50/TCP_4444/nmap.txt
/forensics/INC-2024-001/10.0.0.51/TCP_5555/nmap.txt

=== IR Reconnaissance Complete ===
Results: /forensics/INC-2024-001
```

---

## Chapter 7: Detection & Defense

### How AutoRecon is Detected

#### Network-Based Detection

AutoRecon generates distinctive network traffic patterns that can be detected:

1. **Port Scanning Patterns**
   - SYN scan signatures (nmap default)
   - Sequential port scanning behavior
   - High volume of connection attempts

2. **Service Enumeration Signatures**
   - HTTP user-agent strings from whatweb, nikto
   - SMB null session attempts
   - SNMP community string guessing

3. **Traffic Volume Indicators**
   - Burst of packets in short time frame
   - Multiple connections to different ports
   - Unusual protocol combinations

#### Host-Based Detection

```bash
# Example IDS/IPS signatures
# Snort rules for AutoRecon detection

# Detect nmap SYN scan
alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"AutoRecon - Nmap SYN Scan"; \
  flags:S; threshold:type both, track by_src, count 30, seconds 60; sid:1000001;)

# Detect nikto scanning
alert tcp $HOME_NET any -> $EXTERNAL_NET 80 (msg:"AutoRecon - Nikto Scan Detected"; \
  content:"Nikto"; http_uri; sid:1000002;)

# Detect gobuster directory brute force
alert tcp $HOME_NET any -> $EXTERNAL_NET 80 (msg:"AutoRecon - Gobuster Detected"; \
  flow:to_server,established; content:"GET /"; http_uri; \
  threshold:type both, track by_src, count 100, seconds 60; sid:1000003;)
```

#### Log Analysis Indicators

```bash
# Apache access.log patterns
# AutoRecon/nikto generates distinctive requests

# WhatWeb user-agent
192.168.1.100 - - [15/Jan/2024:14:30:00 +0000] "GET / HTTP/1.1" 200 10918 "-" "WhatWeb/0.5.5"

# Nikto scanning pattern
192.168.1.100 - - [15/Jan/2024:14:30:01 +0000] "GET /admin/ HTTP/1.1" 301 310 "-" "Nikto/2.1.6"
192.168.1.100 - - [15/Jan/2024:14:30:01 +0000] "GET /backup/ HTTP/1.1" 301 309 "-" "Nikto/2.1.6"
192.168.1.100 - - [15/Jan/2024:14:30:01 +0000] "GET /icons/ HTTP/1.1" 301 309 "-" "Nikto/2.1.6"

# Gobuster directory enumeration
192.168.1.100 - - [15/Jan/2024:14:31:22 +0000] "GET /admin HTTP/1.1" 301 310 "-" "Go-http-client/1.1"
192.168.1.100 - - [15/Jan/2024:14:31:22 +0000] "GET /backup HTTP/1.1" 301 309 "-" "Go-http-client/1.1"
```

### Defense Strategies

#### Network-Level Defenses

```bash
# Firewall rules to detect/block AutoRecon

# iptables rules for rate limiting
iptables -A INPUT -p tcp --syn -m limit --limit 10/second --limit-burst 30 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# Block suspicious user agents
iptables -A INPUT -p tcp --dport 80 -m string --string "Nikto" --algo bm -j DROP
iptables -A INPUT -p tcp --dport 80 -m string --string "WhatWeb" --algo bm -j DROP
iptables -A INPUT -p tcp --dport 80 -m string --string "Go-http-client" --algo bm -j DROP

# Rate limit connections per source IP
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 10 -j DROP
```

#### Host-Level Defenses

```bash
# fail2ban configuration for AutoRecon detection
# /etc/fail2ban/jail.local

[autorecon]
enabled = true
port = http,https
filter = autorecon
logpath = /var/log/apache2/access.log
maxretry = 10
findtime = 60
bantime = 3600

# fail2ban filter
# /etc/fail2ban/filter.d/autorecon.conf
[Definition]
failregex = ^<HOST>.*"(GET|POST).*" (200|301|302|403|404).*"(Nikto|WhatWeb|Go-http-client)"
ignoreregex =
```

#### Application-Level Defenses

```python
# Web application WAF rules
# ModSecurity rules for AutoRecon detection

# Detect nikto scanning
SecRule REQUEST_HEADERS:User-Agent "@contains Nikto" \
    "id:100001,phase:1,deny,status:403,msg:'Nikto scan detected'"

# Detect whatweb scanning
SecRule REQUEST_HEADERS:User-Agent "@contains WhatWeb" \
    "id:100002,phase:1,deny,status:403,msg:'WhatWeb scan detected'"

# Detect gobuster brute force
SecRule REQUEST_URI "@rx ^/(admin|backup|config|wp-admin)" \
    "id:100003,phase:1,deny,status:403,msg:'Directory brute force detected'"
```

### Blue Team Response

#### Incident Response Playbook

```bash
#!/bin/bash
# respond_to_autorecon.sh - Respond to AutoRecon detection

ALERT_FILE=$1
INCIDENT_ID="IR-$(date +%Y%m%d-%H%M%S)"

echo "=== AutoRecon Detection Response ==="
echo "Incident ID: $INCIDENT_ID"

# Parse alert
ATTACKER_IP=$(grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' "$ALERT_FILE" | head -1)

echo "Attacker IP: $ATTACKER_IP"

# Block IP immediately
echo "[1] Blocking attacker IP..."
iptables -A INPUT -s "$ATTACKER_IP" -j DROP
echo "    IP blocked: $ATTACKER_IP"

# Capture forensic data
echo "[2] Capturing forensic data..."
mkdir -p "/forensics/$INCIDENT_ID"
tcpdump -i eth0 host "$ATTACKER_IP" -w "/forensics/$INCIDENT_ID/capture.pcap" &
TCPDUMP_PID=$!

# Collect logs
echo "[3] Collecting logs..."
cp /var/log/apache2/access.log "/forensics/$INCIDENT_ID/"
cp /var/log/auth.log "/forensics/$INCIDENT_ID/"
cp /var/log/syslog "/forensics/$INCIDENT_ID/"

# Analyze attacker activity
echo "[4] Analyzing attacker activity..."
grep "$ATTACKER_IP" /var/log/apache2/access.log | \
    awk '{print $7}' | sort | uniq -c | sort -rn > "/forensics/$INCIDENT_ID/uri_analysis.txt"

# Generate report
echo "[5] Generating incident report..."
cat > "/forensics/$INCIDENT_ID/report.md" << EOF
# Incident Report: $INCIDENT_ID

## Summary
- Detection Time: $(date)
- Attacker IP: $ATTACKER_IP
- Activity: AutoRecon network reconnaissance

## Actions Taken
1. IP blocked via iptables
2. Forensic capture initiated
3. Logs collected for analysis

## Indicators of Compromise
- Source IP: $ATTACKER_IP
- User-Agent strings: Nikto, WhatWeb, Go-http-client
- Request patterns: Sequential directory enumeration

## Recommendations
1. Investigate attacker IP ownership
2. Check for compromise of scanned services
3. Implement rate limiting on web servers
4. Deploy WAF rules for detection
EOF

echo ""
echo "=== Response Complete ==="
echo "Incident ID: $INCIDENT_ID"
echo "Forensics: /forensics/$INCIDENT_ID/"
```

**Example Output:**
```bash
$ ./respond_to_autorecon.sh alert.log
=== AutoRecon Detection Response ===
Incident ID: IR-20240115-143000
Attacker IP: 192.168.1.100
[1] Blocking attacker IP...
    IP blocked: 192.168.1.100
[2] Capturing forensic data...
[3] Collecting logs...
[4] Analyzing attacker activity...
[5] Generating incident report...

=== Response Complete ===
Incident ID: IR-20240115-143000
Forensics: /forensics/IR-20240115-143000/
```

### Evasion Countermeasures

#### Detecting Evasion Attempts

```bash
# Monitor for packet fragmentation
tcpdump -i eth0 'ip[6:2] & 0x2000 != 0' -w fragment_capture.pcap

# Monitor for source port manipulation
tcpdump -i eth0 'src port 53 and not port 53' -w spoofed_port.pcap

# Monitor for timing manipulation
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0' -w timing_analysis.pcap
```

#### Advanced Detection Techniques

```python
#!/usr/bin/env python3
"""
detect_evasion.py - Detect AutoRecon evasion techniques
"""

import re
from pathlib import Path
from typing import Dict, List


class EvasionDetector:
    """Detect AutoRecon evasion techniques"""
    
    def __init__(self, logs_dir: str):
        self.logs_dir = Path(logs_dir)
        self.suspicious_patterns = {
            'fragmentation': r'frag',
            'source_port_manipulation': r'source port 53',
            'timing_evasion': r'max-rate|T[12]',
            'mac_spoofing': r'spoof-mac'
        }
    
    def analyze_traffic(self, pcap_file: str) -> Dict:
        """Analyze network traffic for evasion"""
        findings = {
            'fragmentation': False,
            'source_port_manipulation': False,
            'timing_evasion': False,
            'mac_spoofing': False
        }
        
        # Read pcap with tcpdump
        import subprocess
        result = subprocess.run(
            ['tcpdump', '-r', pcap_file, '-vv'],
            capture_output=True,
            text=True
        )
        
        for line in result.stdout.split('\n'):
            for pattern_name, pattern in self.suspicious_patterns.items():
                if re.search(pattern, line, re.IGNORECASE):
                    findings[pattern_name] = True
        
        return findings
    
    def generate_report(self, findings: Dict) -> str:
        """Generate evasion detection report"""
        report = ["=== AutoRecon Evasion Detection Report ===\n"]
        
        for technique, detected in findings.items():
            status = "DETECTED" if detected else "Not detected"
            report.append(f"{technique.replace('_', ' ').title()}: {status}")
        
        return "\n".join(report)


def main():
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python detect_evasion.py <pcap_file>")
        sys.exit(1)
    
    detector = EvasionDetector(".")
    findings = detector.analyze_traffic(sys.argv[1])
    print(detector.generate_report(findings))


if __name__ == "__main__":
    main()
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Command not found"

```bash
$ autorecon 192.168.1.1
bash: autorecon: command not found

# Solution: Install AutoRecon
$ pip install autorecon

# Or if installed locally
$ python3 -m pip install --user autorecon

# Verify installation
$ which autorecon
/usr/local/bin/autorecon
```

#### Error: "Permission denied"

```bash
$ autorecon 192.168.1.1
[ERROR] Permission denied: /var/log/autorecon

# Solution: Run with sudo (not recommended) or fix permissions
$ sudo autorecon 192.168.1.1

# Better solution: Use custom output directory
$ autorecon 192.168.1.1 -o ~/autorecon_results
```

#### Error: "Target unreachable"

```bash
$ autorecon 192.168.1.1
[INFO] Starting AutoRecon against 192.168.1.1
[ERROR] Target is unreachable

# Solution: Verify network connectivity
$ ping -c 3 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=1.45 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=1.12 ms

# Check firewall rules
$ sudo iptables -L -n
```

#### Error: "Port already in use"

```bash
$ autorecon 192.168.1.1
[ERROR] Address already in use

# Solution: Kill existing processes
$ pkill -f autorecon
$ pkill -f nmap

# Or wait and retry
$ sleep 10 && autorecon 192.168.1.1
```

### Performance Issues

#### Slow Scanning

```bash
# Problem: Scanning takes too long
$ time autorecon 192.168.1.1 --ports tcp=1-65535
real    45m23s
user    12m45s
sys     2m30s

# Solution 1: Reduce port range
$ autorecon 192.168.1.1 --ports tcp=top-1000
real    5m12s

# Solution 2: Increase threads
$ autorecon 192.168.1.1 --threads 50
real    8m45s

# Solution 3: Use masscan
$ autorecon 192.168.1.1 --port-scanner masscan
real    2m30s
```

#### High Memory Usage

```bash
# Problem: AutoRecon consuming too much memory
$ ps aux | grep autorecon
user    12345  45.2  12.5  1234567 1234567 pts/0  Sl+  14:30  10:00 autorecon

# Solution 1: Reduce threads
$ autorecon 192.168.1.1 --threads 5

# Solution 2: Scan fewer ports
$ autorecon 192.168.1.1 --ports tcp=top-100

# Solution 3: Use lightweight tools
$ autorecon 192.168.1.1 --no-plugins nikto gobuster
```

#### Network Congestion

```bash
# Problem: Network saturated during scan
$ iftop -i eth0
# Shows high bandwidth usage

# Solution: Rate limit scanning
$ autorecon 192.168.1.1 --nmap-args "-T2 --max-rate 100"

# Monitor network during scan
$ nload eth0
```

### Tool-Specific Issues

#### Nmap Issues

```bash
# Problem: Nmap version detection failing
$ autorecon 192.168.1.1
[INFO] Nmap version detection failed

# Solution: Update nmap scripts
$ sudo nmap --script-updatedb

# Verify nmap installation
$ nmap --version
Nmap version 7.92 ( https://nmap.org )
```

#### Nikto Issues

```bash
# Problem: Nikto timing out
$ autorecon 192.168.1.1
[INFO] Nikto scan timed out after 300 seconds

# Solution: Increase timeout in config
# ~/.config/AutoRecon/config.yml
tools:
  nikto:
    timeout: 600

# Or skip nikto
$ autorecon 192.168.1.1 --no-plugins nikto
```

#### Gobuster Issues

```bash
# Problem: Gobuster wordlist not found
$ autorecon 192.168.1.1
[ERROR] Gobuster wordlist not found: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Solution: Install wordlists
$ sudo apt install wordlists

# Or use custom wordlist
$ autorecon 192.168.1.1 --gobuster-wordlist /usr/share/wordlists/dirb/common.txt
```

### Configuration Issues

#### Config File Errors

```bash
# Problem: Invalid YAML configuration
$ autorecon 192.168.1.1
[ERROR] Invalid configuration file: ~/.config/AutoRecon/config.yml

# Solution: Validate YAML
$ python3 -c "import yaml; yaml.safe_load(open('$HOME/.config/AutoRecon/config.yml'))"

# Reset to default configuration
$ rm ~/.config/AutoRecon/config.yml
$ autorecon --help
```

#### Plugin Issues

```bash
# Problem: Plugin not loading
$ autorecon 192.168.1.1 --plugins custom_scanner
[ERROR] Plugin 'custom_scanner' not found

# Solution: Verify plugin location
$ ls -la ~/.config/AutoRecon/plugins/
$ cat ~/.config/AutoRecon/config.yml | grep plugins
```

### Frequently Asked Questions

#### General Questions

**Q: Does AutoRecon require root privileges?**
```bash
# A: Yes, for SYN scanning (default)
$ autorecon 192.168.1.1
[ERROR] Root privileges required for SYN scanning

# Solution: Run with sudo
$ sudo autorecon 192.168.1.1

# Or use connect scan (slower but no root needed)
$ autorecon 192.168.1.1 --nmap-args "-sT"
```

**Q: Can AutoRecon scan through firewalls?**
```bash
# A: Limited capability - it uses standard scanning techniques
# For firewall evasion, consider:
$ autorecon 192.168.1.1 --nmap-args "-f -T2 --max-rate 50"

# Or use specialized evasion tools
```

**Q: How do I resume an interrupted scan?**
```bash
# A: AutoRecon doesn't have built-in resume
# Workaround: Scan specific ports that weren't completed
$ autorecon 192.168.1.1 --ports tcp=1-1000
```

#### Technical Questions

**Q: What's the difference between AutoRecon and manual nmap?**
```bash
# A: AutoRecon automates tool selection and orchestration

# Manual workflow (multiple commands)
$ nmap -sV -sC 192.168.1.1
$ whatweb 192.168.1.1
$ nikto -h 192.168.1.1
$ gobuster dir -u http://192.168.1.1 -w wordlist.txt

# AutoRecon (single command)
$ autorecon 192.168.1.1
```

**Q: Can I customize which tools AutoRecon runs?**
```bash
# A: Yes, via configuration or command-line flags

# Enable specific tools
$ autorecon 192.168.1.1 --plugins nikto,gobuster

# Disable specific tools
$ autorecon 192.168.1.1 --no-plugins smbclient enum4linux

# Full configuration in YAML
$ nano ~/.config/AutoRecon/config.yml
```

**Q: How do I interpret AutoRecon output?**
```bash
# A: Results are organized by service and tool

$ ls results/2024-01-15_14-30-00/192.168.1.1/
TCP_22/     # SSH service results
TCP_80/     # HTTP service results
TCP_443/    # HTTPS service results
summary.txt # Overall scan summary

# View summary first
$ cat results/2024-01-15_14-30-00/192.168.1.1/summary.txt
```

---

## Resources

### Official Documentation

- **AutoRecon GitHub**: https://github.com/Tib3rius/AutoRecon
- **Tib3rius GitHub**: https://github.com/Tib3rius
- **AutoRecon Wiki**: https://github.com/Tib3rius/AutoRecon/wiki

### Related Tools

- **Nmap**: Network scanner - https://nmap.org
- **Masscan**: Fast port scanner - https://github.com/robertdavidgraham/masscan
- **Nikto**: Web server scanner - https://cirt.net/Nikto2
- **Gobuster**: Directory/File brute-force - https://github.com/OJ/gobuster
- **WhatWeb**: Web technology detector - https://www.morningstarsecurity.com/research/whatweb
- **SMBclient**: SMB enumeration - https://www.samba.org/
- **SSLScan**: SSL/TLS scanner - https://github.com/rbsec/sslscan

### Learning Resources

- **Pentest Monkey**: https://pentestmonkey.net
- **HackTheBox**: https://www.hackthebox.com
- **TryHackMe**: https://tryhackme.com
- **Offensive Security**: https://www.offensive-security.com

### Community

- **Reddit r/netsec**: https://reddit.com/r/netsec
- **Reddit r/hacking**: https://reddit.com/r/hacking
- **Discord**: Various cybersecurity communities

### Books

- "The Web Application Hacker's Handbook"
- "Penetration Testing" by Georgia Weidman
- "Black Hat Python" by Justin Seitz

---

**Document Version**: 2.0
**Last Updated**: 2024-01-15
**Author**: AutoRecon Documentation Team
**License**: GPL-3.0