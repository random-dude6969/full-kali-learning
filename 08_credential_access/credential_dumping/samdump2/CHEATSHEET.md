# samdump2 Cheatsheet

## Quick Reference

### Mount Windows Partition

```bash
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# Define hive paths
SYSTEM="/mnt/windows/Windows/System32/config/SYSTEM"
SAM="/mnt/windows/Windows/System32/config/SAM"
SECURITY="/mnt/windows/Windows/System32/config/SECURITY"
```

### Extract Password Hashes

```bash
samdump2 $SYSTEM $SAM
```

### Output to File

```bash
samdump2 $SYSTEM $SAM > hashes.txt
```

### With Custom Boot Key

```bash
samdump2 $SYSTEM $SAM -b 1234567890abcdef1234567890abcdef
```

### Offline Forensic Analysis

```bash
# Copy hives
cp $SYSTEM /forensics/
cp $SAM /forensics/

# Unmount
umount /mnt/windows

# Extract hashes
samdump2 /forensics/SYSTEM /forensics/SAM > hashes.txt

# Crack hashes
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Complete Credential Extraction

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

### Batch Operations

```bash
# Extract from multiple systems
for system in /forensics/*/SYSTEM; do
    dir=$(dirname $system)
    samdump2 $system $dir/SAM > ${dir}_hashes.txt &
done
wait
```

### Forensic Workflow

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

## Output Format

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

## Common Workflows

### Password Hash Extraction

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Extract hashes
samdump2 $SYSTEM $SAM > sam_hashes.txt

# 3. Unmount
umount /mnt/windows

# 4. Crack hashes
john --format=nt sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Cached Credentials Extraction

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Extract cached credentials
cachedump $SYSTEM $SECURITY > cached_hashes.txt

# 3. Unmount
umount /mnt/windows

# 4. Crack cached credentials
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### LSA Secrets Extraction

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Extract LSA secrets
lsadump $SYSTEM $SECURITY > lsa_secrets.txt

# 3. Unmount
umount /mnt/windows

# 4. View secrets
cat lsa_secrets.txt
```

## Troubleshooting

### Common Errors

**"File not found"**:
```bash
ls /mnt/windows/Windows/System32/config/SAM
ls /mnt/windows/WINDOWS/System32/config/SAM
ls /mnt/windows/WinNT/System32/config/SAM
```

**"Permission denied"**:
```bash
mount -t ntfs-3g -o rw /dev/sda1 /mnt/windows
sudo samdump2 SYSTEM SAM
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

1. **Always backup**: Create backups before extraction
2. **Work offline**: Use Live USB for forensic analysis
3. **Secure hashes**: Store extracted hashes securely
4. **Verify extraction**: Check output for complete hashes

## Resources

- **SourceForge**: https://sourceforge.net/projects/ophcrack/files/samdump2/
- **Kali Docs**: https://www.kali.org/tools/samdump2/
