# AutoRecon Cheatsheet

## Quick Reference

### Basic Usage

```bash
# Simple scan
autorecon 192.168.1.1

# With output directory
autorecon 192.168.1.1 -o results

# Multiple targets
autorecon 192.168.1.1 192.168.1.2 192.168.1.3

# CIDR range
autorecon 192.168.1.0/24

# From file
autorecon -iL targets.txt
```

### Scan Options

```bash
# Top 1000 ports (default)
autorecon 192.168.1.1 --ports tcp=top-1000

# All TCP ports
autorecon 192.168.1.1 --ports tcp=1-65535

# Specific ports
autorecon 192.168.1.1 --ports tcp=22,80,443

# UDP ports
autorecon 192.168.1.1 --ports udp=53,161,162

# Combined TCP/UDP
autorecon 192.168.1.1 --ports tcp=1-10000,udp=top-100

# Skip UDP scanning
autorecon 192.168.1.1 --no-udp

# Skip TCP scanning
autorecon 192.168.1.1 --no-tcp
```

### Threading & Performance

```bash
# Default threads: 10
autorecon 192.168.1.1

# Custom thread count
autorecon 192.168.1.1 --threads 50

# Use masscan (faster)
autorecon 192.168.1.1 --port-scanner masscan

# Verbose output
autorecon 192.168.1.1 -v

# Quiet mode
autorecon 192.168.1.1 -q
```

### Evasion Techniques

```bash
# Slow scan (T2 timing)
autorecon 192.168.1.1 --nmap-args "-T2 --max-rate 100"

# Fragment packets
autorecon 192.168.1.1 --nmap-args "-f --mtu 24"

# Spoof source port (DNS)
autorecon 192.168.1.1 --nmap-args "-g 53"

# Spoof MAC address
autorecon 192.168.1.1 --nmap-args "--spoof-mac 0"

# Combined evasion
autorecon 192.168.1.1 --nmap-args "-T2 -f -g 53 --max-rate 100" --threads 3
```

### Tool Selection

```bash
# Enable specific tools only
autorecon 192.168.1.1 --plugins nikto,gobuster,whatweb

# Disable specific tools
autorecon 192.168.1.1 --no-plugins smbclient,enum4linux

# List available plugins
autorecon --list-plugins
```

### Nmap Arguments

```bash
# Add custom nmap args
autorecon 192.168.1.1 --nmap-args "-sV -sC -O"

# Vulnerability scripts
autorecon 192.168.1.1 --nmap-args "--script=vuln"

# OS detection
autorecon 192.168.1.1 --nmap-args "-O"

# Aggressive scan
autorecon 192.168.1.1 --nmap-args "-A"
```

## Common Workflows

### Quick Web Scan

```bash
autorecon 192.168.1.1 \
    --ports tcp=80,443,8080,8443 \
    --no-udp \
    --threads 20 \
    -o web_scan
```

### Comprehensive Full Scan

```bash
autorecon 192.168.1.1 \
    --ports tcp=1-65535,udp=top-100 \
    --threads 10 \
    -o comprehensive_scan
```

### Network Range Scan

```bash
autorecon 192.168.1.0/24 \
    --ports tcp=top-100 \
    --threads 50 \
    -o network_scan
```

### Stealth Scan

```bash
autorecon 192.168.1.1 \
    --ports tcp=top-100 \
    --threads 5 \
    --nmap-args "-T2 --max-rate 50 -f" \
    --no-udp \
    -o stealth_scan
```

## Configuration

### Config File Location

```bash
~/.config/AutoRecon/config.yml
```

### Create Config Directory

```bash
mkdir -p ~/.config/AutoRecon
```

### Sample Config

```yaml
general:
  threads: 20
  timeout: 600
  
scanning:
  tcp_ports: "top-1000"
  udp_ports: "top-100"
  
tools:
  nikto:
    timeout: 600
    args: "-C all"
  gobuster:
    wordlist: "/usr/share/wordlists/dirb/common.txt"
    extensions: "php,html,js,txt"
    threads: 50
  smbclient:
    args: "-L -N"
    timeout: 120
```

## Output Structure

```
results/
└── [timestamp]/
    └── [target]/
        ├── TCP_[port]/
        │   ├── nmap.txt
        │   ├── nmap.xml
        │   └── [tool].txt
        └── summary.txt
```

## Result Analysis

```bash
# Find all open ports
grep -r "open" results/*/*/nmap.txt | grep -v "closed"

# Find web servers
grep -rl "http" results/*/*/nmap.txt

# Find SMB services
grep -rl "smb" results/*/*/nmap.txt

# Extract IPs
grep -rh "Nmap scan report for" results/*/*/nmap.txt | awk '{print $NF}' | sort -u

# View summary
cat results/*/[target]/summary.txt
```

## Installation

```bash
# pip (recommended)
pip install autorecon

# Kali Linux
sudo apt install autorecon

# From source
git clone https://github.com/Tib3rius/AutoRecon.git
cd AutoRecon
pip install -r requirements.txt
```

## Quick Commands

```bash
# Check version
autorecon --version

# Show help
autorecon --help

# Test against localhost
autorecon 127.0.0.1 -o /tmp/test

# Check required tools
which nmap nikto whatweb smbclient gobuster sslscan
```

## Integration Examples

### Feed to Metasploit

```bash
# Extract open ports
OPEN_PORTS=$(grep -rh "/tcp" results/*/*/nmap.txt | grep "open" | awk -F'/' '{print $1}' | sort -n | uniq)

# Create MSF resource script
echo "use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.1.1
set PORTS $OPEN_PORTS
run" > msf_commands.rc

# Run with MSF
msfconsole -r msf_commands.rc
```

### Feed to Nuclei

```bash
# Extract web URLs
grep -rh "open" results/*/*/TCP_*/nmap.txt | grep "http" | \
    awk '{print "http://TARGET:" $1}' > nuclei_targets.txt

# Run Nuclei
nuclei -l nuclei_targets.txt -severity critical,high,medium
```

## Troubleshooting

```bash
# Command not found
pip install autorecon

# Permission denied
sudo autorecon 192.168.1.1

# Target unreachable
ping -c 3 192.168.1.1

# Port in use
pkill -f autorecon

# Slow scanning
autorecon 192.168.1.1 --threads 50 --ports tcp=top-1000

# High memory
autorecon 192.168.1.1 --threads 5 --no-plugins nikto gobuster
```

## Quick Reference Table

| Command | Description |
|---------|-------------|
| `autorecon <target>` | Basic scan |
| `-o <dir>` | Output directory |
| `--ports tcp=<range>` | TCP port range |
| `--ports udp=<range>` | UDP port range |
| `--threads <N>` | Thread count |
| `--no-udp` | Skip UDP |
| `--no-tcp` | Skip TCP |
| `-v` | Verbose |
| `-q` | Quiet |
| `--plugins <list>` | Enable tools |
| `--no-plugins <list>` | Disable tools |
| `--nmap-args "<args>"` | Custom nmap args |
| `-iL <file>` | Targets from file |

## Service-Specific Scans

```bash
# Web servers
autorecon 192.168.1.1 --ports tcp=80,443,8080,8443 --no-udp

# SSH/FTP
autorecon 192.168.1.1 --ports tcp=21,22 --no-udp

# SMB
autorecon 192.168.1.1 --ports tcp=139,445 --no-udp

# Database
autorecon 192.168.1.1 --ports tcp=3306,5432,1433 --no-udp

# Mail
autorecon 192.168.1.1 --ports tcp=25,110,143,993,995 --no-udp
```

---

**Quick Tip**: Always start with `--ports tcp=top-1000` for initial recon, then expand to full port range if needed.

**Remember**: AutoRecon requires root for SYN scanning. Use `sudo` or add `-sT` for connect scan.