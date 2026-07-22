# DCFDLD CHEATSHEET

## Quick Reference

```bash
# Basic image with MD5 hash
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 hashlog=/evidence/image.hash

# Image with multiple hashes
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5,sha256 hashlog=/evidence/image.md5 hashlog2=/evidence/image.sha256

# Image with error tolerance
sudo dcfldd if=/dev/sdb of=/evidence/image.dd conv=noerror,sync hash=md5 hashlog=/evidence/image.hash

# Split image into 4GB chunks
sudo dcfldd if=/dev/sda of=/evidence/image_%f.dd split=4G hash=md5 hashlog=/evidence/image.hash

# Image specific partition
sudo dcfldd if=/dev/sda of=/evidence/partition.dd skip=2048 count=1000000 hash=md5 hashlog=/evidence/partition.hash

# Search pattern while imaging
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 pattern="50415353" hashlog=/evidence/image.hash

# Verify image
dcfldd if=/dev/sda verify=/evidence/src.hash

# Zero-fill device
sudo dcfldd if=/dev/zero of=/dev/sdb bs=4096 hash=md5 hashlog=/dev/null
```

## Parameters

| Parameter | Description |
|---|---|
| `if=` | Input file/device |
| `of=` | Output file/device |
| `bs=` | Block size (default: 512) |
| `count=` | Number of blocks to copy |
| `skip=` | Skip N blocks at input start |
| `seek=` | Skip N blocks at output start |
| `hash=` | Hash algorithm(s) |
| `hashlog=` | Hash log file 1 |
| `hashlog2=` | Hash log file 2 |
| `pattern=` | Hex pattern to search |
| `verify=` | Verification log file |
| `split=` | Split output size |
| `conv=` | Conversion flags (noerror,sync) |
| `bufsize=` | Buffer size (default: 4096) |
| `padpad=` | Pad character (default: 0) |

## Hash Algorithms

| Algorithm | Flag | Notes |
|---|---|---|
| MD5 | `hash=md5` | Fastest, lowest security |
| SHA-1 | `hash=sha1` | Moderate |
| SHA-256 | `hash=sha256` | Recommended |
| SHA-384 | `hash=sha384` | High security |
| SHA-512 | `hash=sha512` | Highest security |
| Multiple | `hash=md5,sha256` | Comma-separated |

## Common Workflows

```bash
# Forensic imaging workflow
# 1. Connect write blocker
# 2. Identify device
lsblk
# 3. Image with hashes
sudo dcfldd if=/dev/sdb of=/evidence/case/image.dd hash=md5,sha256 hashlog=/evidence/case/md5.hash hashlog2=/evidence/case/sha256.hash conv=noerror,sync
# 4. Verify
dcfldd if=/dev/sdb verify=/evidence/case/md5.hash

# Resume interrupted imaging
WRITTEN=$(stat -c%s /evidence/image.dd)
SECTORS=$((WRITTEN / 512))
sudo dcfldd if=/dev/sda of=/evidence/image.dd skip=${SECTORS} seek=${SECTORS} hash=md5 hashlog=/evidence/resume.hash

# Batch image multiple devices
for dev in /dev/sd{b,c,d}; do
    name=$(basename ${dev})
    sudo dcfldd if=${dev} of=/evidence/${name}.dd hash=md5 hashlog=/evidence/${name}.hash conv=noerror,sync
done
```

## Error Handling

```bash
# Continue past errors, pad with zeros
sudo dcfldd if=/dev/sdb of=/evidence/image.dd conv=noerror,sync hash=md5 hashlog=/evidence/image.hash

# Check for errors
grep -c "Input/output error" /evidence/image.log
```

## Network Transfer

```bash
# Source: stream image over network
sudo dcfldd if=/dev/sda hash=md5 | nc -l -p 9999

# Destination: receive image
nc 192.168.1.100 9999 > /evidence/image.dd
```

## Forensic Workflow

1. Write-block the source device
2. Document chain of custody
3. Image with multiple hashes: `hash=md5,sha256`
4. Write hash logs: `hashlog=` and `hashlog2=`
5. Verify image integrity with `verify=`
6. Create independent hash verification
7. Store evidence securely
8. Document all steps
