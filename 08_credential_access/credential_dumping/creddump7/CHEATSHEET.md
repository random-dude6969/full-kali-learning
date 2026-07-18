# creddump7 Cheatsheet

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

### Extract Password Hashes (pwdump)

```bash
python3 pwdump.py $SYSTEM $SAM
```

### Extract Cached Domain Credentials (cachedump)

```bash
# Windows Vista/7/8/10/11
python3 cachedump.py $SYSTEM $SECURITY true

# Windows XP/2003
python3 cachedump.py $SYSTEM $SECURITY false
```

### Extract LSA Secrets (lsadump)

```bash
python3 lsadump.py $SYSTEM $SECURITY
```

### Complete Credential Extraction

```bash
# Create output directory
mkdir -p /forensics/credentials

# Extract all credentials
python3 pwdump.py $SYSTEM $SAM > /forensics/credentials/sam_hashes.txt
python3 cachedump.py $SYSTEM $SECURITY true > /forensics/credentials/cached_hashes.txt
python3 lsadump.py $SYSTEM $SECURITY > /forensics/credentials/lsa_secrets.txt
```

### Crack Extracted Hashes

```bash
# LM/NT hashes (SAM)
john --format=nt sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
hashcat -m 1000 sam_hashes.txt /usr/share/wordlists/rockyou.txt

# Cached domain credentials
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
hashcat -m 2100 cached_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Offline Forensic Analysis

```bash
# Copy hives
cp $SYSTEM /forensics/
cp $SAM /forensics/
cp $SECURITY /forensics/

# Unmount
umount /mnt/windows

# Extract all credentials
python3 pwdump.py /forensics/SYSTEM /forensics/SAM > sam_hashes.txt
python3 cachedump.py /forensics/SYSTEM /forensics/SECURITY true > cached_hashes.txt
python3 lsadump.py /forensics/SYSTEM /forensics/SECURITY > lsa_secrets.txt

# Crack hashes
john --format=nt sam_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
john --format=mscash2 cached_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Batch Operations

```bash
# Extract from multiple systems
for system in /forensics/*/SYSTEM; do
    dir=$(dirname $system)
    python3 pwdump.py $system $dir/SAM > ${dir}_hashes.txt &
done
wait
```

## Output Formats

### LM/NT Hash Format

```
username:RID:LM_hash:NT_hash:::
```

**Example**:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

### Cached Domain Credential Format

```
username:DCC2_hash:domain:domain_fqdn
```

**Example**:
```
nharpsis:6b29dfa157face3f3d8db489aec5cc12:acme:acme.local
```

### LSA Secret Format

```
secret_name: 
  hex: hex_value
  dec: decimal_value
  str: string_value
```

## Common Workflows

### Password Hash Extraction

```bash
# 1. Mount partition
mkdir /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows

# 2. Extract hashes
python3 pwdump.py $SYSTEM $SAM > sam_hashes.txt

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
python3 cachedump.py $SYSTEM $SECURITY true > cached_hashes.txt

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
python3 lsadump.py $SYSTEM $SECURITY > lsa_secrets.txt

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
sudo python3 pwdump.py SYSTEM SAM
```

**"SAM is locked"**:
```bash
powercfg -h off
mount -t ntfs-3g -o rw,recover /dev/sda1 /mnt/windows
```

**"Python version error"**:
```bash
python3 pwdump.py SYSTEM SAM
python3 --version
```

## OpSec Tips

1. **Always backup**: Create backups before extraction
2. **Work offline**: Use Live USB for forensic analysis
3. **Secure hashes**: Store extracted hashes securely
4. **Verify extraction**: Check output for complete hashes

## Resources

- **GitHub**: https://github.com/CiscoCXSecurity/creddump7
- **Kali Docs**: https://www.kali.org/tools/creddump7/
