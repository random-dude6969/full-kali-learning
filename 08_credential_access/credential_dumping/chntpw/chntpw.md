---
title: "chntpw - Windows Password Hash Editor"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Credential Dumping"
  - "Credential Access"
  - "Password Attacks"
tags:
  - "chntpw"
  - "password-hashing"
  - "windows"
  - "sam"
  - "registry"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Credential Dumping
mitre_technique_id: T1003
version: "latest"
tool_url: "https://pogostick.net/~pnh/ntpasswd/"
kali_url: "https://www.kali.org/tools/chntpw/"
license: "GPL-2.0"
---

# chntpw - Windows Password Hash Editor

## Overview

**chntpw** (Change NT Password) is a utility to view, edit, and reset Windows NT/2000/XP/2003/Vista/7/8/10 password hashes. It can directly edit the Windows Security Account Manager (SAM) database to change or reset user passwords, making it an essential tool for password recovery and forensic analysis.

**Key Features**:
- **Direct SAM editing** - Modify Windows password hashes directly
- **Password reset** - Clear or change user passwords
- **Password editing** - Edit existing password hashes
- **Registry support** - Works with Windows registry hives
- **Offline editing** - Edit passwords without booting Windows
- **Cross-platform** - Works on Linux, macOS, and Windows

**Supported Operations**:
- **Password reset** - Clear user passwords
- **Password change** - Set new password hashes
- **User enumeration** - List all users in SAM database
- **Hash extraction** - Extract password hashes for offline cracking
- **Registry editing** - Edit Windows registry hives

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y chntpw
```

### From Source

```bash
# Install dependencies
sudo apt install -y build-essential

# Clone and build
git clone https://github.com/ptoomey3/chntpw
cd chntpw
make
sudo make install
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-l` | List all users in SAM database |
| `-u USERNAME` | Specify username to edit |
| `-r` | Reset password (set to empty) |
| `-R` | Reset password (interactive) |
| `-e` | Edit password hash directly |
| `-h` | Show help |
| `-v` | Verbose output |

### Usage Modes

**Interactive Mode**:
```bash
chntpw /path/to/SAM
```

**List Users**:
```bash
chntpw -l /path/to/SAM
```

**Reset Password**:
```bash
chntpw -u username -r /path/to/SAM
```

**Edit Password Hash**:
```bash
chntpw -u username -e /path/to/SAM
```

## Usage Examples

### Mount Windows Partition

**Linux**:
```bash
# Mount NTFS partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# Or with explicit options
mount -t ntfs-3g -o rw /dev/sda1 /mnt/windows
```

**macOS**:
```bash
# Install macFUSE (osxfuse)
brew install --cask macfuse

# Mount NTFS partition
mkdir /Volumes/Windows
mount -t ntfs /dev/disk2s1 /Volumes/Windows
```

### List Users

```bash
# List all users in SAM database
chntpw -l /mnt/windows/Windows/System32/config/SAM
```

**Output Example**:
```
chntpw 1.0 - (c) 2005-2015,2017 Bernie Hargrove
SAM registry peaks: rid 0xf00, last modified: 2024-01-15 14:30:22

RID     User name              Admin?  Lock?
0x01f4  Administrator          *ADMIN  locked
0x01f5  Guest                  *ADMIN  
0x03e8  John                   *       locked
0x03e9  Jane                   *       locked
```

### Reset Password

**Interactive Reset**:
```bash
chntpw -R /mnt/windows/Windows/System32/config/SAM
```

**Reset Specific User**:
```bash
chntpw -u John -r /mnt/windows/Windows/System32/config/SAM
```

**Clear Password (Set to Empty)**:
```bash
chntpw -u John -e /mnt/windows/Windows/System32/config/SAM
# Then select option 1: Clear (blank) user password
```

### Edit Password Hash

```bash
chntpw -u John -e /mnt/windows/Windows/System32/config/SAM
```

**Interactive Options**:
```
1 - Clear (blank) user password
2 - Edit user password hash
3 - Promote user (make admin)
4 - Unlock user account
5 - Disable user account
q - Quit
```

### Extract Hashes for Offline Cracking

```bash
# Extract hashes using samdump2 or creddump7
samdump2 /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SAM > hashes.txt

# Or with creddump7
creddump7 -s /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SAM > hashes.txt
```

### Crack Extracted Hashes

```bash
# With John the Ripper
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# With Hashcat
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Offline Password Reset

**Boot from Live USB/DVD**:
```bash
# 1. Boot from Kali Live USB
# 2. Mount Windows partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 3. Reset passwords
chntpw /mnt/windows/Windows/System32/config/SAM

# 4. Unmount and reboot
umount /mnt/windows
reboot
```

### Forensic Analysis

**Extract SAM and SYSTEM Hives**:
```bash
# Copy hives for analysis
cp /mnt/windows/Windows/System32/config/SAM /forensics/SAM
cp /mnt/windows/Windows/System32/config/SYSTEM /forensics/SYSTEM
cp /mnt/windows/Windows/System32/config/SECURITY /forensics/SECURITY

# Unmount
umount /mnt/windows
```

**Analyze Offline**:
```bash
# List users
chntpw -l /forensics/SAM

# Extract hashes
samdump2 /forensics/SYSTEM /forensics/SAM > hashes.txt

# Crack hashes
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Windows Registry Editing

**Mount Registry Hive**:
```bash
# Mount SYSTEM hive
chntpw -l /mnt/windows/Windows/System32/config/SYSTEM

# Mount SECURITY hive
chntpw -l /mnt/windows/Windows/System32/config/SECURITY
```

**Edit Registry Values**:
```bash
chntpw /mnt/windows/Windows/System32/config/SYSTEM
# Then use interactive menu to edit values
```

### Batch Operations

**Reset All User Passwords**:
```bash
# List users
chntpw -l /mnt/windows/Windows/System32/config/SAM | grep -E "^0x" | awk '{print $2}' > users.txt

# Reset each user
for user in $(cat users.txt); do
    chntpw -u $user -r /mnt/windows/Windows/System32/config/SAM
done
```

**Extract All Hashes**:
```bash
# Extract SAM hash
samdump2 /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SAM > sam_hashes.txt

# Extract cached domain credentials
cachedump /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SECURITY > cached_hashes.txt

# Extract LSA secrets
lsadump /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SECURITY > lsa_secrets.txt
```

## Output Interpretation

### User List Output

```
RID     User name              Admin?  Lock?
0x01f4  Administrator          *ADMIN  locked
0x01f5  Guest                  *ADMIN  
0x03e8  John                   *       locked
0x03e9  Jane                   *       locked
```

**Fields**:
- **RID**: Relative Identifier (user ID)
- **User name**: Account name
- **Admin?**: Administrative privileges
- **Lock?**: Account locked status

### Password Reset Output

```
chntpw 1.0 - (c) 2005-2015,2017 Bernie Hargrove
SAM registry peaks: rid 0xf00, last modified: 2024-01-15 14:30:22

===== chntpw Edit Mode ===
   1 - Clear (blank) user password
   2 - Edit user password hash
   3 - Promote user (make admin)
   4 - Unlock user account
   5 - Disable user account
   q - Quit editing, save changes
   0 - Quit, discard changes

Select: [q] > 1

Password cleared!
```

## Integration with Other Tools

### SAM + SYSTEM Hives → Hash Extraction

```bash
# Extract hashes
samdump2 SYSTEM SAM > hashes.txt

# Or with creddump7
creddump7 -s SYSTEM SAM > hashes.txt

# Crack with John
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with Hashcat
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Cached Domain Credentials

```bash
# Extract cached domain credentials
cachedump SYSTEM SECURITY > cached_hashes.txt

# Crack with John
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### LSA Secrets

```bash
# Extract LSA secrets
lsadump SYSTEM SECURITY > lsa_secrets.txt

# View LSA secrets
cat lsa_secrets.txt
```

## Operational Security

### Offline Editing

```bash
# Always work with copies
cp SAM SAM.backup
cp SYSTEM SYSTEM.backup

# Edit backup
chntpw SAM.backup
```

### Hash Extraction

```bash
# Extract hashes without modifying SAM
samdump2 SYSTEM SAM > hashes.txt

# Verify extraction
cat hashes.txt
```

## Common Issues and Troubleshooting

### Issue: "SAM file not found"

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
sudo chntpw /mnt/windows/Windows/System32/config/SAM
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

### Backup Before Editing

```bash
# Create backups
cp SAM SAM.backup.$(date +%Y%m%d)
cp SYSTEM SYSTEM.backup.$(date +%Y%m%d)
cp SECURITY SECURITY.backup.$(date +%Y%m%d)
```

### Hash Extraction Workflow

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

# 4. Extract hashes
samdump2 /forensics/SYSTEM /forensics/SAM > hashes.txt

# 5. Crack hashes
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

## Resources

- **Homepage**: https://pogostick.net/~pnh/ntpasswd/
- **Kali Linux Tools**: https://www.kali.org/tools/chntpw/
- **Documentation**: https://pogostick.net/~pnh/ntpasswd/
- **License**: GPL-2.0

## Related Tools

- **creddump7**: Python tool for extracting credentials from Windows registry hives
- **samdump2**: Extract password hashes from Windows SAM database
- **mimikatz**: Windows credential dumping tool (runs on Windows)
- **Responder**: LLMNR/NBT-NS/MDNS poisoner for credential capture
