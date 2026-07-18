---
title: "samdump2 - SAM Database Hash Extractor"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Credential Dumping"
  - "Credential Access"
  - "Password Attacks"
tags:
  - "samdump2"
  - "credential-dumping"
  - "windows"
  - "sam"
  - "password-hashing"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Credential Dumping
mitre_technique_id: T1003
version: "latest"
tool_url: "https://sourceforge.net/projects/ophcrack/files/samdump2/"
kali_url: "https://www.kali.org/tools/samdump2/"
license: "GPL-2.0"
---

# samdump2 - SAM Database Hash Extractor

## Overview

**samdump2** is a tool to extract password hashes from Windows NT/2000/XP/2003 SAM database files. It reads the SAM and SYSTEM registry hives to extract LM and NTLM password hashes, making it an essential tool for password recovery and forensic analysis.

**Key Features**:
- **SAM extraction** - Extract password hashes from SAM database
- **SYSTEM hive support** - Uses SYSTEM hive for boot key
- **LM/NTLM hashes** - Extract both hash types
- **Offline extraction** - No need to boot Windows
- **Simple interface** - Easy to use command-line tool
- **Cross-platform** - Works on Linux, macOS, and Windows

**Supported Operations**:
- **SAM hash extraction** - Extract LM and NTLM hashes
- **Boot key extraction** - Extract SYSKEY from SYSTEM hive
- **User enumeration** - List all users in SAM database
- **Hash formatting** - Output in L0phtcrack format

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y samdump2
```

### From Source

```bash
# Install dependencies
sudo apt install -y build-essential

# Clone and build
git clone https://sourceforge.net/projects/ophcrack/files/samdump2/
cd samdump2
./configure
make
sudo make install
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `SYSTEM` | Path to SYSTEM registry hive |
| `SAM` | Path to SAM registry hive |
| `-o OUTPUT` | Output file (default: stdout) |
| `-b BOOTKEY` | Boot key (optional, extracted from SYSTEM if not provided) |
| `-h` | Show help |

### Usage Syntax

```bash
samdump2 <SYSTEM> <SAM>
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
```

### Extract Password Hashes

**Basic Extraction**:
```bash
samdump2 $SYSTEM $SAM
```

**Output to File**:
```bash
samdump2 $SYSTEM $SAM > hashes.txt
```

**Output Example**:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
John:1000:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::
Jane:1001:aad3b435b51404eeaad3b435b51404ee:2b836f3c2a5b6b2f0e2f2f5b2f5b2f5b:::
```

### With Custom Boot Key

```bash
samdump2 $SYSTEM $SAM -b 1234567890abcdef1234567890abcdef
```

### Offline Forensic Analysis

**Copy Hives for Analysis**:
```bash
# Copy hives
cp $SYSTEM /forensics/
cp $SAM /forensics/

# Unmount
umount /mnt/windows

# Extract hashes
samdump2 /forensics/SYSTEM /forensics/SAM > hashes.txt
```

**Crack Extracted Hashes**:
```bash
# With John the Ripper
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# With Hashcat
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Integration with Other Tools

**Impacket Integration**:
```bash
# Use impacket for remote extraction
secretsdump.py administrator:password123@192.168.1.100

# Or with hash
secretsdump.py -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0 administrator@192.168.1.100
```

**Metasploit Integration**:
```bash
# Use found credentials
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
    samdump2 $system $sam >> all_hashes.txt
done < hive_list.txt
```

**Parallel Extraction**:
```bash
# Extract from multiple systems in parallel
for system in /forensics/*/SYSTEM; do
    dir=$(dirname $system)
    samdump2 $system $dir/SAM > ${dir}_hashes.txt &
done
wait
```

### Complete Credential Extraction

**Extract All Credentials**:
```bash
# Create output directory
mkdir -p /forensics/credentials

# Extract SAM hashes
samdump2 $SYSTEM $SAM > /forensics/credentials/sam_hashes.txt

# Extract cached domain credentials
cachedump $SYSTEM $SECURITY > /forensics/credentials/cached_hashes.txt

# Extract LSA secrets
lsadump $SYSTEM $SECURITY > /forensics/credentials/lsa_secrets.txt
```

### Forensic Workflow

**Complete Forensic Extraction**:
```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Copy hives
cp /mnt/windows/Windows/System32/config/SAM /forensics/
cp /mnt/windows/Windows/System32/config/SYSTEM /forensics/
cp /mnt/windows/Windows/System32/config/SECURITY /forensics/

# 3. Unmount
umount /mnt/windows

# 4. Extract all credentials
samdump2 /forensics/SYSTEM /forensics/SAM > sam_hashes.txt
cachedump /forensics/SYSTEM /forensics/SECURITY > cached_hashes.txt
lsadump /forensics/SYSTEM /forensics/SECURITY > lsa_secrets.txt

# 5. Crack hashes
john --format=nt sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

## Output Interpretation

### Hash Format

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

### Hash Types

**LM Hash**:
- Older hash type (LAN Manager)
- Weak and easily cracked
- Often empty on modern Windows

**NT Hash**:
- Newer hash type (NTLM)
- Stronger than LM but still crackable
- Used for authentication

## Integration with Other Tools

### SAM + SYSTEM → Hash Cracking

```bash
# Extract hashes
samdump2 SYSTEM SAM > hashes.txt

# Crack with John
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with Hashcat
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Cached Credentials → Hash Cracking

```bash
# Extract cached credentials
cachedump SYSTEM SECURITY > cached_hashes.txt

# Crack with John
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### LSA Secrets → Password Recovery

```bash
# Extract LSA secrets
lsadump SYSTEM SECURITY > lsa_secrets.txt

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
sudo samdump2 SYSTEM SAM
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
samdump2 $SYSTEM $SAM > sam_hashes.txt
cachedump $SYSTEM $SECURITY > cached_hashes.txt
lsadump $SYSTEM $SECURITY > lsa_secrets.txt

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
samdump2 $SYSTEM $SAM > sam_hashes.txt
cachedump $SYSTEM $SECURITY > cached_hashes.txt
lsadump $SYSTEM $SECURITY > lsa_secrets.txt

# 4. Unmount
umount /mnt/windows
```

## Resources

- **SourceForge**: https://sourceforge.net/projects/ophcrack/files/samdump2/
- **Kali Linux Tools**: https://www.kali.org/tools/samdump2/
- **License**: GPL-2.0

## Related Tools

- **creddump7**: Python tool for extracting credentials from Windows registry hives
- **chntpw**: Windows password hash editor
- **mimikatz**: Windows credential dumping tool (runs on Windows)
- **Responder**: LLMNR/NBT-NS/MDNS poisoner for credential capture
