# chntpw Cheatsheet

## Quick Reference

### Mount Windows Partition

```bash
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows
```

### List Users

```bash
chntpw -l /mnt/windows/Windows/System32/config/SAM
```

### Reset Password

```bash
# Interactive reset
chntpw -R /mnt/windows/Windows/System32/config/SAM

# Reset specific user
chntpw -u John -r /mnt/windows/Windows/System32/config/SAM
```

### Edit Password Hash

```bash
chntpw -u John -e /mnt/windows/Windows/System32/config/SAM
```

### Extract Hashes

```bash
# With samdump2
samdump2 /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SAM > hashes.txt

# With creddump7
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

```bash
# Copy hives for analysis
cp /mnt/windows/Windows/System32/config/SAM /forensics/SAM
cp /mnt/windows/Windows/System32/config/SYSTEM /forensics/SYSTEM
cp /mnt/windows/Windows/System32/config/SECURITY /forensics/SECURITY

# Unmount
umount /mnt/windows

# Analyze offline
chntpw -l /forensics/SAM
samdump2 /forensics/SYSTEM /forensics/SAM > hashes.txt
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Batch Operations

```bash
# Reset all user passwords
chntpw -l /mnt/windows/Windows/System32/config/SAM | grep -E "^0x" | awk '{print $2}' > users.txt
for user in $(cat users.txt); do
    chntpw -u $user -r /mnt/windows/Windows/System32/config/SAM
done

# Extract all hashes
samdump2 /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SAM > sam_hashes.txt
cachedump /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SECURITY > cached_hashes.txt
lsadump /mnt/windows/Windows/System32/config/SYSTEM /mnt/windows/Windows/System32/config/SECURITY > lsa_secrets.txt
```

## Command Options

| Option | Description |
|--------|-------------|
| `-l` | List all users in SAM database |
| `-u USERNAME` | Specify username to edit |
| `-r` | Reset password (set to empty) |
| `-R` | Reset password (interactive) |
| `-e` | Edit password hash directly |
| `-h` | Show help |
| `-v` | Verbose output |

## Interactive Menu Options

| Option | Description |
|--------|-------------|
| 1 | Clear (blank) user password |
| 2 | Edit user password hash |
| 3 | Promote user (make admin) |
| 4 | Unlock user account |
| 5 | Disable user account |
| q | Quit, save changes |
| 0 | Quit, discard changes |

## Common Workflows

### Password Reset Workflow

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. List users
chntpw -l /mnt/windows/Windows/System32/config/SAM

# 3. Reset password
chntpw -u John -r /mnt/windows/Windows/System32/config/SAM

# 4. Unmount
umount /mnt/windows
```

### Hash Extraction Workflow

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Copy hives
cp /mnt/windows/Windows/System32/config/SAM /forensics/
cp /mnt/windows/Windows/System32/config/SYSTEM /forensics/

# 3. Unmount
umount /mnt/windows

# 4. Extract hashes
samdump2 /forensics/SYSTEM /forensics/SAM > hashes.txt

# 5. Crack hashes
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Forensic Analysis Workflow

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Copy all hives
cp /mnt/windows/Windows/System32/config/SAM /forensics/
cp /mnt/windows/Windows/System32/config/SYSTEM /forensics/
cp /mnt/windows/Windows/System32/config/SECURITY /forensics/

# 3. Unmount
umount /mnt/windows

# 4. Analyze
chntpw -l /forensics/SAM
samdump2 /forensics/SYSTEM /forensics/SAM > sam_hashes.txt
cachedump /forensics/SYSTEM /forensics/SECURITY > cached_hashes.txt
lsadump /forensics/SYSTEM /forensics/SECURITY > lsa_secrets.txt
```

## Troubleshooting

### Common Errors

**"SAM file not found"**:
```bash
ls /mnt/windows/Windows/System32/config/SAM
ls /mnt/windows/WINDOWS/System32/config/SAM
ls /mnt/windows/WinNT/System32/config/SAM
```

**"Permission denied"**:
```bash
mount -t ntfs-3g -o rw /dev/sda1 /mnt/windows
sudo chntpw /mnt/windows/Windows/System32/config/SAM
```

**"SAM is locked"**:
```bash
powercfg -h off
mount -t ntfs-3g -o rw,recover /dev/sda1 /mnt/windows
```

**"Hash extraction failed"**:
```bash
creddump7 -s SYSTEM SAM > hashes.txt
secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
```

## OpSec Tips

1. **Always backup**: Create backups before editing SAM
2. **Work offline**: Use Live USB for password resets
3. **Extract hashes**: Use samdump2/creddump7 for offline cracking
4. **Verify changes**: List users after password reset

## Resources

- **Homepage**: https://pogostick.net/~pnh/ntpasswd/
- **Kali Docs**: https://www.kali.org/tools/chntpw/
