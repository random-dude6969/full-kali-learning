---
tool_name: "nmap"
display_name: "Nmap"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "network_information"
install_command: "sudo apt install nmap"
version: "7.99"
github_repo: "https://github.com/nmap/nmap"
kali_tools_page: "https://www.kali.org/tools/nmap/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: true
tags: ["network", "scanning", "ports", "os-detection", "service-detection", "nse"]
related_tools: ["zenmap", "ncat", "masscan", "unicornscan"]
---

# Nmap (Network Mapper)

## Chapter 1: Introduction & Overview

### What is Nmap?
Nmap (Network Mapper) is the industry-standard open-source tool for network discovery and security auditing. It uses raw IP packets in creative ways to determine what hosts are available on the network, what services (application name and version) those hosts are offering, what operating systems (and OS versions) they are running, what type of packet filters/firewalls are in use, and dozens of other characteristics.

### History & Background
- **Created:** 1997 by Gordon Lyon (Fyodor)
- **Maintained:** Nmap Development Team
- **License:** Nmap Public Source License (based on GPLv2)
- **Purpose:** Network exploration, security auditing, port scanning, service detection

Nmap is one of the most important tools in any cybersecurity professional's toolkit. It's used by network administrators, security professionals, and penetration testers worldwide.

### When to Use This Tool
- **Network inventory:** Discover all devices on a network
- **Port scanning:** Find open ports and services
- **Service enumeration:** Identify running services and versions
- **OS fingerprinting:** Determine operating systems
- **Vulnerability assessment:** Find potential security weaknesses
- **Firewall testing:** Analyze firewall rules and configurations

### When NOT to Use This Tool
- Without explicit written authorization from the network owner
- On production networks without proper planning
- For malicious purposes or unauthorized access

### Legal and Ethical Considerations
- **Always obtain written permission** before scanning networks you don't own
- **Understand your scope** - know exactly what you're authorized to scan
- **Be careful with timing** - aggressive scans can cause disruptions
- **Document everything** - keep records of your authorization and activities
- **Follow responsible disclosure** - report vulnerabilities properly

### Key Concepts
- **Port:** A logical endpoint for network communication (0-65535)
- **Service:** An application listening on a port (HTTP, SSH, FTP, etc.)
- **Scan Type:** Different methods of probing ports (SYN, Connect, ACK, etc.)
- **NSE:** Nmap Scripting Engine for extended functionality
- **OS Fingerprinting:** Identifying the operating system through network behavior
- **Host Discovery:** Determining if a host is online

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 512 MB (2 GB recommended)
- **Disk Space:** 50 MB for base install
- **Network:** Any network interface
- **Privileges:** Root required for SYN scans, OS detection, and some scripts

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install nmap
```

#### Method 2: From Source
```bash
git clone https://github.com/nmap/nmap.git
cd nmap
./configure
make
sudo make install
```

#### Method 3: Docker
```bash
docker run -it uzyonen/nmap
```

### Packages Included
When you install `nmap`, you get these tools:
- **nmap** - The main scanner
- **ncat** - Netcat reimplementation with SSL support
- **ndiff** - Scan comparison utility
- **nping** - Packet generation/response analysis
- **zenmap** - GUI frontend (install separately)

### Configuration
Nmap uses several configuration files:
- `/usr/share/nmap/` - Nmap data files
- `~/.nmap/` - User configuration
- Script database: `/usr/share/nmap/scripts/script.db`

### Verification
```bash
nmap --version
# Output: Nmap 7.99 ( https://nmap.org )
```

---

## Chapter 3: Basic Usage

### First Scan
```bash
nmap scanme.nmap.org
```
**Output:** Shows the 1000 most common ports on the target.

### Essential Commands

#### Simple Host Discovery
```bash
nmap -sn 192.168.1.0/24
```
**Explanation:** Pings all hosts in the subnet to find which are online (no port scan).

#### Quick Port Scan
```bash
nmap 192.168.1.1
```
**Explanation:** Scans the 1000 most common TCP ports.

#### Service Version Detection
```bash
nmap -sV 192.168.1.1
```
**Explanation:** Probes open ports to determine service versions.

#### Aggressive Scan
```bash
nmap -A 192.168.1.1
```
**Explanation:** Enables OS detection, version detection, script scanning, and traceroute.

#### Full Port Scan
```bash
nmap -p- 192.168.1.1
```
**Explanation:** Scans all 65535 ports (slow but thorough).

### Complete Flags Reference

#### Scan Types
| Flag | Description | Example |
|------|-------------|---------|
| `-sS` | SYN stealth scan (default with root) | `nmap -sS target` |
| `-sT` | TCP connect scan | `nmap -sT target` |
| `-sA` | ACK scan | `nmap -sA target` |
| `-sW` | Window scan | `nmap -sW target` |
| `-sM` | Maimon scan | `nmap -sM target` |
| `-sN` | TCP NULL scan | `nmap -sN target` |
| `-sF` | FIN scan | `nmap -sF target` |
| `-sX` | Xmas scan | `nmap -sX target` |
| `-sU` | UDP scan | `nmap -sU target` |
| `-sI` | Idle scan (zombie) | `nmap -sI zombie:port target` |
| `-sY` | SCTP INIT scan | `nmap -sY target` |
| `-sZ` | SCTP COOKIE-ECHO scan | `nmap -sZ target` |
| `-sO` | IP protocol scan | `nmap -sO target` |

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `-iL` | Input from file | `nmap -iL targets.txt` |
| `-iR` | Random targets | `nmap -iR 100` |
| `--exclude` | Exclude hosts | `nmap --exclude 192.168.1.1 target` |
| `--excludefile` | Exclude from file | `nmap --excludefile exclude.txt target` |

#### Port Specification
| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Specific ports | `nmap -p 80,443 target` |
| `-p-` | All ports | `nmap -p- target` |
| `-p1-1000` | Port range | `nmap -p1-1000 target` |
| `-p U:53,T:80` | Mixed TCP/UDP | `nmap -p U:53,T:80 target` |
| `-F` | Fast mode (fewer ports) | `nmap -F target` |
| `--top-ports` | Top N ports | `nmap --top-ports 100 target` |
| `-r` | Sequential (not random) | `nmap -r target` |

#### Host Discovery
| Flag | Description | Example |
|------|-------------|---------|
| `-sn` | Ping scan only | `nmap -sn 192.168.1.0/24` |
| `-Pn` | Skip host discovery | `nmap -Pn target` |
| `-PS` | TCP SYN ping | `nmap -PS80 target` |
| `-PA` | TCP ACK ping | `nmap -PA80 target` |
| `-PU` | UDP ping | `nmap -PU40125 target` |
| `-PY` | SCTP INIT ping | `nmap -PY target` |
| `-PE` | ICMP echo | `nmap -PE target` |
| `-PP` | ICMP timestamp | `nmap -PP target` |
| `-PM` | ICMP mask | `nmap -PM target` |
| `-PO` | IP protocol ping | `nmap -PO1 target` |

#### Timing & Performance
| Flag | Description | Example |
|------|-------------|---------|
| `-T0` | Paranoid (very slow) | `nmap -T0 target` |
| `-T1` | Sneaky | `nmap -T1 target` |
| `-T2` | Polite | `nmap -T2 target` |
| `-T3` | Normal (default) | `nmap -T3 target` |
| `-T4` | Aggressive | `nmap -T4 target` |
| `-T5` | Insane (very fast) | `nmap -T5 target` |
| `--min-rate` | Min packets/sec | `nmap --min-rate 100 target` |
| `--max-rate` | Max packets/sec | `nmap --max-rate 1000 target` |
| `--host-timeout` | Per-host timeout | `nmap --host-timeout 30s target` |
| `--min-rtt-timeout` | Min RTT timeout | `nmap --min-rtt-timeout 100ms target` |
| `--max-retries` | Max retransmissions | `nmap --max-retries 3 target` |
| `--scan-delay` | Delay between probes | `nmap --scan-delay 1s target` |
| `--max-scan-delay` | Max delay | `nmap --max-scan-delay 10s target` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-oN` | Normal output | `nmap -oN output.txt target` |
| `-oX` | XML output | `nmap -oX output.xml target` |
| `-oG` | Grepable output | `nmap -oG output.txt target` |
| `-oA` | All formats | `nmap -oA output target` |
| `-v` | Verbose | `nmap -v target` |
| `-vv` | Very verbose | `nmap -vv target` |
| `-d` | Debug | `nmap -d target` |
| `-dd` | More debug | `nmap -dd target` |
| `--reason` | Show port state reason | `nmap --reason target` |
| `--open` | Show only open ports | `nmap --open target` |
| `--packet-trace` | Show all packets | `nmap --packet-trace target` |
| `--append-output` | Append to file | `nmap --append-output output.txt target` |
| `--resume` | Resume scan | `nmap --resume aborted.scan` |

#### Service/Version Detection
| Flag | Description | Example |
|------|-------------|---------|
| `-sV` | Version detection | `nmap -sV target` |
| `--version-intensity` | Probe intensity (0-9) | `nmap --version-intensity 9 target` |
| `--version-light` | Light intensity (2) | `nmap --version-light target` |
| `--version-all` | Try all probes (9) | `nmap --version-all target` |
| `--version-trace` | Debug version scan | `nmap --version-trace target` |

#### OS Detection
| Flag | Description | Example |
|------|-------------|---------|
| `-O` | Enable OS detection | `nmap -O target` |
| `--osscan-limit` | Limit OS detection | `nmap --osscan-limit target` |
| `--osscan-guess` | Aggressive OS guess | `nmap --osscan-guess target` |

#### Scripting Engine
| Flag | Description | Example |
|------|-------------|---------|
| `-sC` | Default scripts | `nmap -sC target` |
| `--script` | Specific scripts | `nmap --script http-title target` |
| `--script-args` | Script arguments | `nmap --script http-brute --script-args http-brute.path=/admin target` |
| `--script-updatedb` | Update script DB | `nmap --script-updatedb` |
| `--script-help` | Script help | `nmap --script-help "http-*"` |
| `--script-trace` | Show script I/O | `nmap --script-trace target` |

#### Firewall/IDS Evasion
| Flag | Description | Example |
|------|-------------|---------|
| `-f` | Fragment packets | `nmap -f target` |
| `--mtu` | Custom MTU | `nmap --mtu 24 target` |
| `-D` | Decoy scan | `nmap -D RND:10 target` |
| `-S` | Spoof source IP | `nmap -S 10.0.0.1 target` |
| `-e` | Specify interface | `nmap -e eth0 target` |
| `-g` | Spoof source port | `nmap -g 53 target` |
| `--proxies` | Use proxies | `nmap --proxies http://proxy:8080 target` |
| `--data` | Custom hex payload | `nmap --data DEADBEEF target` |
| `--data-string` | Custom ASCII payload | `nmap --data-string "Hello" target` |
| `--data-length` | Random data length | `nmap --data-length 50 target` |
| `--ip-options` | IP options | `nmap --ip-options "R" target` |
| `--ttl` | Set TTL | `nmap --ttl 64 target` |
| `--spoof-mac` | Spoof MAC | `nmap --spoof-mac 0 target` |
| `--badsum` | Invalid checksum | `nmap --badsum target` |

#### Miscellaneous
| Flag | Description | Example |
|------|-------------|---------|
| `-6` | IPv6 scanning | `nmap -6 target` |
| `-A` | Aggressive scan | `nmap -A target` |
| `--send-eth` | Raw Ethernet | `nmap --send-eth target` |
| `--send-ip` | Raw IP | `nmap --send-ip target` |
| `--privileged` | Assume root | `nmap --privileged target` |
| `--unprivileged` | Assume user | `nmap --unprivileged target` |
| `-V` | Version | `nmap -V` |
| `-h` | Help | `nmap -h` |

---

## Chapter 4: Intermediate Usage

### Output Formats

#### XML Output (for tool integration)
```bash
nmap -oX output.xml 192.168.1.1
```

#### Grepable Output (for text processing)
```bash
nmap -oG output.txt 192.168.1.1
# Example: 192.168.1.1Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
```

#### All Formats
```bash
nmap -oA scan_results 192.168.1.1
# Creates: scan_results.nmap, scan_results.xml, scan_results.gnmap
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Batch scan multiple targets
TARGETS_FILE="targets.txt"
OUTPUT_DIR="scan_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r target; do
    echo "[*] Scanning $target"
    nmap -sV -sC -oA "$OUTPUT_DIR/$(echo $target | tr '.' '_')" "$target"
done < "$TARGETS_FILE"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess
import xml.etree.ElementTree as ET
import json

def scan_target(target):
    """Run nmap scan and return parsed results"""
    result = subprocess.run(
        ['nmap', '-sV', '-oX', '-', target],
        capture_output=True, text=True
    )
    return ET.fromstring(result.stdout)

def parse_results(xml_root):
    """Parse nmap XML output"""
    hosts = []
    for host in xml_root.findall('.//host'):
        ip = host.find('address').get('addr')
        hostnames = [h.get('name') for h in host.findall('.//hostname')]
        
        ports = []
        for port in host.findall('.//port'):
            state = port.find('state').get('state')
            service = port.find('service')
            ports.append({
                'port': port.get('portid'),
                'protocol': port.get('protocol'),
                'state': state,
                'service': service.get('name') if service is not None else 'unknown',
                'version': service.get('version', '') if service is not None else ''
            })
        
        hosts.append({
            'ip': ip,
            'hostnames': hostnames,
            'ports': ports
        })
    return hosts

# Usage
xml = scan_target('192.168.1.1')
results = parse_results(xml)
print(json.dumps(results, indent=2))
```

#### Advanced Scripting Examples
```bash
# Parallel scanning with GNU parallel
parallel nmap -sV -oA results_{#} ::: 192.168.1.{1..254}

# Extract open ports
nmap -oG - 192.168.1.1 | awk '/Open/{for(i=3;i<=NF;i++) if($i ~ /open/) print $2, $i}'

# Conditional scanning
if nmap -p 22 --open -q target | grep -q "22/open"; then
    echo "SSH is open, running ssh-audit"
    ssh-audit target
fi

# Scan and output to CSV
nmap -sV -oG - target | awk -F'[/:]' '/open/{print $2","$5","$6}' > results.csv
```

### Combining with Other Tools

#### Metasploit Integration
```bash
# Scan and import to Metasploit
nmap -sV -sC -oX scan.xml target
msfconsole -x "db_import scan.xml; hosts; services; exit"
```

#### With Gobuster
```bash
# Find web servers, then enumerate directories
nmap -p 80,443,8080,8443 -oG - target | awk '/open/{print $2}' | while read ip; do
    gobuster dir -u http://$ip -w /usr/share/wordlists/dirb/common.txt
done
```

#### With Searchsploit
```bash
# Find services, then search for exploits
nmap -sV target | grep "Open" | awk '{print $3}' | while read service; do
    searchsploit $service
done
```

### Configuration Tuning

#### Parallel Host Scanning
```bash
# Scan multiple hosts simultaneously
nmap --min-hostgroup 256 --min-parallelism 128 192.168.1.0/24
```

#### Rate Limiting
```bash
# Slow scan to avoid detection
nmap -T1 --scan-delay 5s --max-parallelism 1 target

# Fast scan (be careful!)
nmap -T5 --min-rate 10000 target
```

### Advanced Output Processing

#### Generate HTML Report
```bash
nmap -oX output.xml target
xsltproc /usr/share/nmap/nmap.xsl output.xml > report.html
```

#### Parse XML with xmllint
```bash
# Extract all open ports
xmllint --xpath "//port[state[@state='open']]/@portid" output.xml

# Extract host addresses
xmllint --xpath "//address/@addr" output.xml
```

#### Convert to JSON
```python
import xml.etree.ElementTree as ET
import json

tree = ET.parse('output.xml')
root = tree.getroot()

data = []
for host in root.findall('.//host'):
    host_data = {
        'ip': host.find('address').get('addr'),
        'ports': []
    }
    for port in host.findall('.//port'):
        host_data['ports'].append({
            'port': port.get('portid'),
            'state': port.find('state').get('state'),
            'service': port.find('service').get('name', '')
        })
    data.append(host_data)

with open('output.json', 'w') as f:
    json.dump(data, f, indent=2)
```

---

## Chapter 5: Advanced Usage

### Evasion Techniques

#### Firewall/IDS Evasion
```bash
# Fragment packets to evade basic firewalls
nmap -f target

# Use decoys to hide real IP
nmap -D 192.168.1.100,ME,192.168.1.101 target

# Spoof source IP (requires -e)
nmap -S 10.0.0.1 -e eth0 target

# Use bad checksums
nmap --badsum target

# Custom data length to confuse IDS
nmap --data-length 50 target

# MAC address spoofing
macchanger -r eth0
nmap --spoof-mac 0 target

# Custom MTU
nmap --mtu 24 target
```

#### Timing Evasion
```bash
# Slow scan to avoid detection
nmap -T1 --scan-delay 5s target

# Random delays
nmap --scan-delay 2s --max-parallelism 1 target

# Idle scan (zombie host) - very stealthy
nmap -sI zombie-host:zombie-port target

# Slow rate
nmap --min-rate 1 --max-rate 10 target
```

#### Protocol Evasion
```bash
# Use SCTP instead of TCP
nmap -sY target

# IPv6 scanning
nmap -6 target

# Custom packet generation
nmap --packet-trace target

# Send via raw Ethernet
nmap --send-eth target
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# OSINT gathering
theharvester -d target.com -b google
amass enum -passive -d target.com
subfinder -d target.com

# DNS enumeration
dnsenum target.com
dnsrecon -d target.com
```

#### Phase 2: Scanning
```bash
# Host discovery
nmap -sn 192.168.1.0/24 -oG hosts.txt

# Quick scan of discovered hosts
nmap -T4 -F 192.168.1.1 -oA initial

# Detailed scan
nmap -sV -sC -p- -T4 192.168.1.1 -oA detailed

# Vulnerability scan
nmap --script vuln -p 80,443 target

# UDP scan
nmap -sU --top-ports 100 target
```

#### Phase 3: Exploitation
```bash
# Import to Metasploit
msfconsole -x "db_import detailed.xml; hosts; services"

# Use discovered services
msfconsole -x "use exploit/unix/webapp/php_include; set RHOSTS target; exploit"
```

#### Phase 4: Post-Exploitation
```bash
# Privilege escalation
linpeas.sh
winpeas.exe

# Lateral movement
crackmapexec smb 192.168.1.0/24 -u user -p pass
```

### Nmap Scripting Engine (NSE)

#### Built-in Script Categories
| Category | Description | Example Scripts |
|----------|-------------|-----------------|
| `auth` | Authentication testing | `ssh-brute`, `ftp-brute` |
| `broadcast` | Broadcast discovery | `broadcast-dhcp`, `broadcast-avahi` |
| `brute` | Brute force attacks | `http-brute`, `smb-brute` |
| `default` | Default scripts | `default` (equivalent to `-sC`) |
| `discovery` | Information gathering | `http-enum`, `smb-enum-shares` |
| `dos` | Denial of service | `http-slowloris` (use carefully!) |
| `exploit` | Active exploitation | `http-shellshock`, `smb-vuln-ms08-067` |
| `external` | External scripts | `whois-ip`, `ip-geolocation-*` |
| `fuzzer` | Fuzzing | `http-fuzzer`, `smb-fuzz` |
| `intrusive` | Intrusive tests | `http-sql-injection`, `smb-brute` |
| `malware` | Malware detection | `ssl-heartbleed`, `http-shellshock` |
| `safe` | Safe scripts | `http-headers`, `ssl-cert` |
| `version` | Version detection | `ftp-syst`, `http-server-header` |
| `vuln` | Vulnerability detection | `ssl-poodle`, `smb-vuln-*` |

#### Popular NSE Scripts
```bash
# HTTP scripts
nmap --script http-title target
nmap --script http-enum target
nmap --script http-headers target
nmap --script http-methods target
nmap --script http-auth target

# SSL scripts
nmap --script ssl-cert target
nmap --script ssl-enum-ciphers target
nmap --script ssl-heartbleed target
nmap --script ssl-poodle target

# SMB scripts
nmap --script smb-enum-shares target
nmap --script smb-enum-users target
nmap --script smb-vuln-* target

# Vulnerability scripts
nmap --script vuln target
nmap --script "vuln and not (dos or brute)" target

# Combined scripts
nmap --script "http-* and not http-slowloris" target
```

#### Writing Custom NSE Scripts
```lua
-- my-scanner.nse - Custom HTTP title scanner
local nmap = require "nmap"
local http = require "http"
local shortport = require "shortport"
local stdnse = require "stdnse"

description = [[
  Custom script to extract HTTP title and server header
]]

categories = {"safe", "discovery"}

-- Define port rule
portrule = shortport.http

-- Main action function
action = function(host, port)
  -- Make HTTP request
  local response = http.get(host, port, "/")
  
  if not response or not response.status then
    return nil
  end
  
  -- Extract title
  local title = response.body:match("<title>(.-)</title>")
  
  -- Extract server header
  local server = response.header["server"]
  
  -- Build result
  local result = {}
  if title then
    table.insert(result, "Title: " .. title)
  end
  if server then
    table.insert(result, "Server: " .. server)
  end
  table.insert(result, "Status: " .. response.status)
  
  return table.concat(result, "\n")
end
```

#### Script Usage Examples
```bash
# Run specific script
nmap --script http-title target

# Run script category
nmap --script "safe and discovery" target

# Run multiple scripts
nmap --script "http-* and ssl-*" target

# Script with arguments
nmap --script http-brute --script-args http-brute.path=/admin target

# Script with output
nmap --script http-enum -oX output.xml target

# Update script database
nmap --script-updatedb

# Show script help
nmap --script-help http-*
```

### Advanced Configurations

#### Custom Configuration File
```bash
# Create a scan profile
cat > ~/.nmap/profiles/aggressive.conf << EOF
-sV -sC -p- --min-rate 1000 --max-retries 3 -T4
EOF

# Use profile
nmap --profile aggressive target
```

#### Environment Variables
```bash
# Set custom NSE path
export NMAP_DIR=/opt/custom-nmap
export NSE_DIR=$NMAP_DIR/scripts
```

### Performance Optimization

#### Large Scale Scanning
```bash
# Split targets across multiple instances
split -l 1000 targets.txt target_chunk_
for f in target_chunk_*; do
    nmap -iL $f -oA results_$f --min-rate 1000 &
done
wait

# Merge results
for f in results_*.xml; do
    echo "Processing $f"
done
```

#### Resource Management
```bash
# CPU affinity for single-core
taskset -c 0 nmap target

# Limit memory
ulimit -v 2000000

# Use ramdisk for output
mount -t tmpfs -o size=1G tmpfs /tmp/nmap_output
nmap -oA /tmp/nmap_output/scan target
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Network Audit
**Objective:** Discover all devices and services on a corporate network

**Steps:**
```bash
# 1. Find all live hosts
nmap -sn 10.0.0.0/24 -oG hosts.txt

# 2. Quick scan of all hosts
nmap -T4 -F -iL <(grep "Up" hosts.txt | awk '{print $2}') -oA quick_scan

# 3. Detailed scan of interesting hosts
nmap -sV -sC -p- -T4 <interesting_ips> -oA detailed_scan

# 4. Vulnerability assessment
nmap --script vuln -p 80,443,22,445 <interesting_ips> -oA vuln_scan
```

**Results:** Complete network inventory with services and vulnerabilities

### Scenario 2: CTF Challenge
**Objective:** Find hidden services on a target machine

**Steps:**
```bash
# 1. Initial scan
nmap -sC -sV target -oA initial

# 2. All ports scan
nmap -p- -T4 target -oA all_ports

# 3. Service enumeration
nmap -sV -sC -p <open_ports> target -oA enum

# 4. Vulnerability scan
nmap --script vuln -p <open_ports> target

# 5. Banner grabbing
nmap -sV --version-trace -p <open_ports> target
```

### Scenario 3: Bug Bounty
**Objective:** Find vulnerabilities in a web application

**Steps:**
```bash
# 1. Subdomain discovery
subfinder -d target.com -o subdomains.txt
amass enum -passive -d target.com >> subdomains.txt

# 2. Port scan web servers
nmap -p 80,443,8080,8443 -iL subdomains.txt -oG web_hosts.txt

# 3. Service detection
nmap -sV -p 80,443,8080,8443 -iL <(awk '/Up/{print $2}' web_hosts.txt) -oA web_services

# 4. Web vulnerability scripts
nmap --script http-enum,http-methods,http-title -p 80,443 -iL <hosts> -oA web_enum

# 5. SSL testing
nmap --script ssl-enum-ciphers,ssl-heartbleed -p 443 -iL <hosts> -oA ssl_test
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Nmap

#### Network Signatures
```
# SYN scan pattern
TCP SYN to multiple ports in rapid succession
SYN → SYN-ACK → RST (typical nmap pattern)

# Nmap User-Agent
User-Agent: Mozilla/5.0 (compatible; Nmap Scripting Engine)

# NSE Script signatures
GET / HTTP/1.1
User-Agent: Nmap Scripting Engine
```

#### IDS/IPS Rules
```
# Snort/Suricata rules
alert tcp any any -> $HOME_NET any (msg:"Nmap SYN Scan"; 
  flags:S; threshold:type both,track by_src,count 20,seconds 1; sid:1000001;)

alert http any any -> $HOME_NET any (msg:"Nmap NSE Script"; 
  content:"User-Agent|3a| Nmap"; sid:1000002;)
```

#### Log Analysis
```bash
# Check for nmap patterns in logs
grep -i "nmap" /var/log/auth.log
grep "SYN" /var/log/syslog | awk '{print $5}' | sort | uniq -c | sort -rn
```

### Mitigation Strategies

#### Rate Limiting
```bash
# iptables rate limiting
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

#### Connection Tracking
```bash
# Monitor connections
conntrack -L | grep ESTABLISHED
ss -tn state established | awk '{print $5}' | cut -d: -f1 | sort | uniq -c
```

#### Service Hardening
```bash
# Disable unnecessary services
systemctl stop service_name
systemctl disable service_name

# Bind services to specific IPs
# Configure services to only listen on internal interfaces
```

### Blue Team Response
1. **Monitor for reconnaissance** - Watch for port sweeps
2. **Analyze scan patterns** - Identify scanner IP and methodology
3. **Check for follow-up attacks** - Look for exploitation attempts
4. **Block scanner IPs** - Update firewall rules
5. **Document the incident** - Record all findings
6. **Report to management** - Notify stakeholders

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "You requested a scan type which requires root privileges"
**Cause:** SYN scan (-sS) and OS detection (-O) require root
**Solution:**
```bash
# Run with sudo
sudo nmap -sS target

# Or use TCP connect scan (doesn't require root)
nmap -sT target
```

#### Error: "No routes to host"
**Cause:** Network connectivity issue or firewall blocking
**Solution:**
```bash
# Check network connection
ping target
traceroute target

# Check routing
ip route show

# Try different scan type
nmap -Pn -sT target
```

#### Error: "Host seems down"
**Cause:** Host discovery is being blocked
**Solution:**
```bash
# Skip host discovery
nmap -Pn target

# Use different discovery method
nmap -PS22,80,443 target
```

#### Error: "Connection refused"
**Cause:** Target is not listening on that port
**Solution:**
```bash
# Verify with different scan
nmap -sT -p 80 target

# Check if service is running
nc -zv target 80
```

### Performance Optimization

#### Slow Scans
```bash
# Increase timing
nmap -T4 --min-rate 1000 target

# Limit to specific ports
nmap -p 22,80,443 target

# Use fast mode
nmap -F target
```

#### High Resource Usage
```bash
# Limit parallelism
nmap --max-parallelism 10 target

# Reduce retries
nmap --max-retries 2 target

# Use timeout
nmap --host-timeout 30s target
```

### FAQ

**Q: Is nmap legal to use?**
A: Yes, nmap is a legitimate security tool. However, scanning networks without permission is illegal in many jurisdictions. Always get written authorization before scanning.

**Q: How do I update nmap?**
A: `sudo apt update && sudo apt upgrade nmap`

**Q: What's the difference between -sS and -sT?**
A: `-sS` (SYN scan) is faster and stealthier but requires root. `-sT` (TCP connect) completes the full TCP handshake and doesn't require root.

**Q: Why are some ports shown as "filtered"?**
A: A firewall or packet filter is blocking the probe. The port may be open but protected.

**Q: How accurate is OS detection?**
A: OS detection is generally accurate but can be fooled by firewalls, NAT, or intentional OS spoofing. Use multiple detection methods for better accuracy.

**Q: What is NSE?**
A: Nmap Scripting Engine (NSE) allows you to write Lua scripts to extend nmap's functionality. There are hundreds of built-in scripts for various tasks.

---

## Resources

### Official Documentation
- [Nmap Homepage](https://nmap.org/)
- [Nmap Book (Official Guide)](https://nmap.org/book/)
- [NSE Documentation](https://nmap.org/nsedoc/)
- [GitHub Repository](https://github.com/nmap/nmap)

### Tutorials
- [Nmap Getting Started Guide](https://nmap.org/book/getting-started.html)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [NSE Script Guide](https://nmap.org/nsedoc/guide.html)

### Books
- "Nmap Network Scanning" by Gordon Lyon (Fyodor)
- "Nmap in the Enterprise" by Angela Orebaugh
- "Network Scanning Handbook" by Paul Dotson

### Videos
- [Nmap Official YouTube](https://www.youtube.com/naborez)
- [HackerSploit Nmap Tutorial](https://www.youtube.com/watch?v=3PfXOMfKjhM)
- [NetworkChuck Nmap](https://www.youtube.com/watch?v=qBqJtVwOMRk)

### Related Tools
- **zenmap:** GUI frontend for nmap
- **ncat:** Netcat reimplementation with SSL support
- **ndiff:** Scan comparison utility
- **nping:** Packet generation/response analysis
- **masscan:** Ultra-fast port scanner (alternative)
- **unicornscan:** Asynchronous TCP/UDP scanner

### Practice Resources
- [ScanMe.Nmap.org](https://nmap.org.scanme.html) - Official practice target
- [HackTheBox](https://www.hackthebox.com/) - CTF platform
- [TryHackMe](https://tryhackme.com/) - Learning platform
- [VulnHub](https://www.vulnhub.com/) - Vulnerable VMs

### Certifications That Use Nmap
- OSCP (Offensive Security Certified Professional)
- CEH (Certified Ethical Hacker)
- GPEN (GIAC Penetration Tester)
- PNPT (Practical Network Penetration Tester)
- CompTIA Security+
