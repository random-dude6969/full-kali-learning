# dc3dd — Enhanced dd with Hashing

## Table of Contents

1. [Overview](#overview)
2. [How dc3dd Works](#how-dc3dd-works)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Forensic Imaging Workflow](#forensic-imaging-workflow)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other Tools](#integration-with-other-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

**dc3dd** is a patched version of GNU `dd` with added features specifically designed for computer forensics. Developed by the Department of Defense Cyber Crime Center (DC3), it provides:

- **On-the-fly hashing** — MD5, SHA-1, SHA-256, SHA-512
- **Progress reporting** — Shows copying progress
- **Error logging** — Writes errors to a file
- **Pattern wiping** — Overwrite disks with patterns
- **Output splitting** — Split output into multiple files
- **Hash verification** — Verify output matches input

### Why dc3dd Instead of dd?

| Feature | dd | dc3dd |
|---------|-----|-------|
| On-the-fly hashing | No | Yes |
| Progress reporting | Manual (`status=progress`) | Built-in |
| Error logging | Limited | Full logging |
| Pattern wiping | No | Yes |
| Output splitting | Need `split` | Built-in |
| Hash verification | Manual | Automatic |

### Forensic Importance

In digital forensics, **chain of custody** requires proving that the evidence image is an exact copy of the original. dc3dd computes cryptographic hashes during the imaging process, providing immediate verification that the copy is faithful.

---

## How dc3dd Works

### Imaging Process

1. Read data from input (if=) device or file
2. Compute hashes of input data
3. Write data to output (of=) device or file
4. Compute hashes of output data (with hof=/hofs=)
5. Compare input and output hashes
6. Log all operations and results

### Hash Algorithms

| Algorithm | Bits | Speed | Security |
|-----------|------|-------|----------|
| MD5 | 128 | Fast | Deprecated for security |
| SHA-1 | 160 | Fast | Deprecated for security |
| SHA-256 | 256 | Moderate | Recommended |
| SHA-512 | 512 | Moderate | Recommended |

### Error Handling

dc3dd handles read errors by:

1. Attempting to read the sector multiple times
2. If unresolvable, writing zeros to the output
3. Logging the error with sector number
4. Continuing with the next sector
5. Generating an error report

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install dc3dd
```

### Verify Installation

```bash
dc3dd --version
```

---

## Command Syntax

```bash
dc3dd [options]
```

### Basic Usage

```bash
# Simple disk image with SHA-256 hash
dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256

# Image with logging
dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Pattern wipe a disk
dc3dd if=/dev/zero of=/dev/sdb wipe=on
```

---

## Options Reference

### Basic Options

| Option | Description |
|--------|-------------|
| `if=DEVICE/FILE` | Input device or file |
| `ofs=DEVICE/FILE` | Output device or file |
| `hof=DEVICE/FILE` | Output with hash verification |
| `ofsz=BYTES` | Maximum output file size |
| `hash=ALGORITHM` | Hash algorithm (md5, sha1, sha256, sha512) |
| `log=FILE` | Log I/O statistics and hashes |
| `hlog=FILE` | Log hash values only |
| `mlog=FILE` | Machine-readable hash log |

### Advanced Options

| Option | Description |
|--------|-------------|
| `fhod=DEVICE` | Full hash output device |
| `rec=off` | Don't write zeros on read errors |
| `wipe=DEVICE` | Wipe device with zeros or pattern |
| `hwipe=DEVICE` | Wipe and verify |
| `pat=HEX` | Hex pattern for wiping |
| `tpat=TEXT` | Text pattern for wiping |
| `cnt=SECTORS` | Read only N sectors |
| `iskip=SECTORS` | Skip N input sectors |
| `oskip=SECTORS` | Skip N output sectors |
| `app=on` | Append to output file |
| `ssz=BYTES` | Set sector size |
| `bufsz=BYTES` | Set buffer size |
| `verb=on` | Verbose reporting |
| `nwspc=on` | Compact reporting |
| `b10=on` | Base-10 bytes (1000=1KB) |

### Hash Options

| Option | Description |
|--------|-------------|
| `hash=md5` | Compute MD5 hash |
| `hash=sha1` | Compute SHA-1 hash |
| `hash=sha256` | Compute SHA-256 hash |
| `hash=sha512` | Compute SHA-512 hash |

### Unit Suffixes

| Suffix | Value |
|--------|-------|
| `c` | 1 byte |
| `w` | 2 bytes |
| `b` | 512 bytes |
| `kB` | 1000 bytes |
| `K` | 1024 bytes |
| `MB` | 1000² bytes |
| `M` | 1024² bytes |
| `GB` | 1000³ bytes |
| `G` | 1024³ bytes |

---

## Understanding the Output

### Standard Output

```
dc3dd 7.3.1 started at 2024-01-15 10:30:00 -0500
compiled options:
command line: dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256
sector size: 512 bytes (assumed)
    42015925 bytes ( 40 M ) copied ( 100% ),    0 s, 185 M/s

input results for file `/dev/sda':
   82062 sectors + 181 bytes in
   2dd19a4fd6bdebddf2ce754f4b09f7914b7cb7433efc5b8678ff437d9e138579 (sha256)

output results for file `/evidence/disk.img':
   82062 sectors + 181 bytes out

dc3dd completed at 2024-01-15 10:30:01 -0500
```

### Output Fields

| Field | Description |
|-------|-------------|
| Start time | When imaging started |
| Bytes copied | Total data transferred |
| Percentage | Completion percentage |
| Speed | Transfer rate |
| Sectors in/out | Sector count |
| Hash value | Cryptographic hash |
| End time | When imaging completed |

### Log File Format

```
dc3dd log started at 2024-01-15 10:30:00 -0500

input results for file `/dev/sda':
   sectors in: 82062
   bytes in: 42015925

output results for file `/evidence/disk.img':
   sectors out: 82062
   bytes out: 42015925

hash results for input:
   sha256: 2dd19a4fd6bdebddf2ce754f4b09f7914b7cb7433efc5b8678ff437d9e138579

hash results for output:
   sha256: 2dd19a4fd6bdebddf2ce754f4b09f7914b7cb7433efc5b8678ff437d9e138579

dc3dd log completed at 2024-01-15 10:30:01 -0500
```

---

## Forensic Imaging Workflow

### Step-by-Step Process

```bash
# Step 1: Identify the source disk
lsblk
sudo fdisk -l

# Step 2: Write-protect the source (use hardware write blocker)
# Or use software write block
sudo blockdev --setro /dev/sdb

# Step 3: Create forensic image with dc3dd
sudo dc3dd if=/dev/sdb of=/evidence/source_disk.img hash=sha256 log=/evidence/imaging.log

# Step 4: Verify the image
dc3dd if=/evidence/source_disk.img log=/dev/null hash=sha256

# Step 5: Compare hashes
# The hash in imaging.log should match the verification hash

# Step 6: Document everything
echo "Source: /dev/sdb" > /evidence/chain_of_custody.txt
echo "Image: /evidence/source_disk.img" >> /evidence/chain_of_custody.txt
echo "Hash: $(sha256sum /evidence/source_disk.img)" >> /evidence/chain_of_custody.txt
echo "Date: $(date)" >> /evidence/chain_of_custody.txt
echo "Imager: $(whoami)" >> /evidence/chain_of_custody.txt
```

### Multiple Hash Algorithms

```bash
# Compute multiple hashes simultaneously
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=md5 hash=sha1 hash=sha256 hash=sha512 log=/evidence/imaging.log
```

### Splitting Output

```bash
# Split output into 1GB files
sudo dc3dd if=/dev/sdb of=/evidence/disk.img ofsz=1GB hash=sha256 log=/evidence/imaging.log

# Creates:
# disk.img
# disk.img.001
# disk.img.002
# ...
```

### Pattern Wiping

```bash
# Wipe with zeros
sudo dc3dd if=/dev/zero of=/dev/sdb wipe=on

# Wipe with pattern
sudo dc3dd if=/dev/zero of=/dev/sdb wipe=on pat=0xff

# Wipe with text pattern
sudo dc3dd if=/dev/zero of=/dev/sdb wipe=on tpat="ERASED"
```

---

## Practical Lab Exercises

### Lab 1: Basic Disk Imaging

```bash
# Create a test disk image
dd if=/dev/zero of=test_source.img bs=1M count=100
mkfs.ext4 test_source.img

# Create forensic image with dc3dd
dc3dd if=test_source.img of=test_image.img hash=sha256 log=test_imaging.log

# Verify
cat test_imaging.log
dc3dd if=test_image.img hash=sha256
```

### Lab 2: Multiple Hash Algorithms

```bash
# Create image with multiple hashes
dc3dd if=test_source.img of=test_image.img hash=md5 hash=sha256 log=test_multi.log

# Review all hashes
grep "hash:" test_multi.log
```

### Lab 3: Error Handling

```bash
# Create a source with bad sectors (simulated)
dd if=/dev/zero of=test_source.img bs=1M count=100

# Create image with error logging
dc3dd if=test_source.img of=test_image.img hash=sha256 log=test_errors.log

# Check for errors
grep -i "error\|bad" test_errors.log
```

### Lab 4: Pattern Wiping

```bash
# Create test disk
dd if=/dev/urandom of=test_wipe.img bs=1M count=10

# Wipe with zeros
dc3dd if=/dev/zero of=test_wipe.img wipe=on

# Verify wipe
hexdump -C test_wipe.img | head -20
```

### Lab 5: Split Output

```bash
# Create large test file
dd if=/dev/zero of=test_large.img bs=1M count=500

# Split into 100MB chunks
dc3dd if=test_large.img of=test_split.img ofsz=100M hash=sha256 log=test_split.log

# List output files
ls -la test_split.img*
```

---

## Integration with Other Tools

### dc3dd + Sleuth Kit

```bash
# Image with dc3dd
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Analyze with TSK
mmls /evidence/disk.img
fls -r -m / -f ntfs -o 63 /evidence/disk.img > body.txt
mactime -b body.txt -d > timeline.csv
```

### dc3dd + Autopsy

```bash
# Image with dc3dd
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Analyze with Autopsy
# Add disk image as data source
```

### dc3dd + Guymager

```bash
# dc3dd for command-line imaging
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Guymager for GUI-based imaging
sudo guymager
```

### dc3dd + Hash Verification

```bash
# Create image
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Verify later
sha256sum /evidence/disk.img

# Compare with log
grep "sha256" /evidence/imaging.log
```

---

## Common Forensic Scenarios

### Scenario 1: Live System Acquisition

```bash
# Warning: Live acquisition is risky — prefer dead acquisition
# Use write blocker if possible

# Image the disk
sudo dc3dd if=/dev/sda of=/evidence/live_image.img hash=sha256 log=/evidence/live_imaging.log

# If system is running, expect inconsistencies
```

### Scenario 2: Dead System Acquisition

```bash
# Power off the system
# Connect disk to forensic workstation
# Use write blocker

# Image the disk
sudo dc3dd if=/dev/sdb of=/evidence/dead_image.img hash=sha256 log=/evidence/dead_imaging.log

# Verify
sha256sum /evidence/dead_image.img
```

### Scenario 3: Degraded Media

```bash
# Create image with error tolerance
sudo dc3dd if=/dev/sdb of=/evidence/degraded.img hash=sha256 rec=off log=/evidence/degraded.log

# Check for errors
grep -i "error" /evidence/degraded.log

# Use ddrescue for badly damaged media
sudo ddrescue -f /dev/sdb /evidence/rescue.img /evidence/rescue.log
```

### Scenario 4: Encrypted Volume

```bash
# Image the encrypted volume
sudo dc3dd if=/dev/sdb2 of=/evidence/encrypted.img hash=sha256 log=/evidence/encrypted.log

# Decryption requires password/key
# (dc3dd cannot decrypt)
```

### Scenario 5: Multiple Evidence Items

```bash
# Image multiple disks
for disk in /dev/sdb /dev/sdc /dev/sdd; do
    name=$(basename $disk)
    sudo dc3dd if=$disk of=/evidence/${name}.img hash=sha256 log=/evidence/${name}_imaging.log
done
```

---

## Troubleshooting

### "No space left on device"

```bash
# Check available space
df -h /evidence/

# Use smaller block size
dc3dd if=/dev/sdb of=/evidence/disk.img bs=512 hash=sha256

# Or split output
dc3dd if=/dev/sdb of=/evidence/disk.img ofsz=1G hash=sha256
```

### "Permission denied"

```bash
# Run as root
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256

# Check device permissions
ls -la /dev/sdb
```

### Hash Mismatch

```bash
# If hashes don't match, the copy is corrupted
# Retry with different block size
dc3dd if=/dev/sdb of=/evidence/disk.img bs=4096 hash=sha256

# Check source for errors
badblocks /dev/sdb
```

### Slow Performance

```bash
# Increase buffer size
dc3dd if=/dev/sdb of=/evidence/disk.img bufsz=1M hash=sha256

# Use larger block size
dc3dd if=/dev/sdb of=/evidence/disk.img bs=4M hash=sha256
```

### "Read error"

```bash
# dc3dd continues past errors by default
# Check error count in log
grep -c "error" /evidence/imaging.log

# Use rec=off to stop on errors
dc3dd if=/dev/sdb of=/evidence/disk.img rec=off hash=sha256
```

---

## Best Practices

1. **Always use write blockers** — never modify original evidence
2. **Compute multiple hashes** — MD5 + SHA-256 minimum
3. **Log everything** — use `log=` parameter
4. **Verify the image** — recompute hash after imaging
5. **Document chain of custody** — who, when, what, where
6. **Use appropriate block size** — 512 for compatibility, 4M for speed
7. **Check for errors** — review the log file
8. **Store hashes securely** — keep with case documentation
9. **Test your workflow** — practice before real cases
10. **Follow agency procedures** — adhere to forensic standards

---

## Summary

dc3dd is essential for forensic disk imaging. Key capabilities:

- On-the-fly hashing (MD5, SHA-1, SHA-256, SHA-512)
- Progress reporting
- Error logging and handling
- Pattern wiping
- Output splitting
- Hash verification

### Quick Reference

```bash
# Basic imaging with hash
sudo dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Multiple hashes
sudo dc3dd if=/dev/sda of=/evidence/disk.img hash=md5 hash=sha256 log=/evidence/imaging.log

# Split output
sudo dc3dd if=/dev/sda of=/evidence/disk.img ofsz=1G hash=sha256 log=/evidence/imaging.log

# Pattern wipe
sudo dc3dd if=/dev/zero of=/dev/sdb wipe=on

# Verify image
dc3dd if=/evidence/disk.img hash=sha256
```

---

## References

- dc3dd Homepage: https://sourceforge.net/projects/dc3dd/
- Kali Linux dc3dd: https://www.kali.org/tools/dc3dd/
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response
- DoD Cyber Crime Center: https://www.dc3.mil/
