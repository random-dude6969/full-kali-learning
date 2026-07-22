---
tool_name: dcfldd
display_name: DCFDLD
mitre_tactic: collection
mitre_tactic_id: TA0009
install_command: sudo apt install dcfldd
github_repo: https://github.com/resurrecting-open-source-projects/dcfldd
kali_tools_page: https://www.kali.org/tools/dcfldd/
difficulty_level: beginner
tags:
  - forensics
  - disk-imaging
  - evidence
  - hash
  - dfir
related_tools:
  - dc3dd
  - ddrescue
  - guymager
  - dd
---

# DCFDLD — Defense Cyber Crime Center Forensic DD

## Chapter 1: Overview

DCFDLD is a forensic enhancement of the traditional `dd` utility, developed by the Defense Cyber Crime Center (DC3). It provides built-in hashing, pattern matching, splitting, and verification capabilities essential for forensically sound disk imaging operations.

### Historical Context

DCFDLD was one of the first `dd` enhancements specifically targeting forensic needs. It was originally distributed through the DC3 lab and has since become a standard tool in many forensic toolkits. While `dc3dd` is a more recent fork with additional features, `dcfldd` remains widely deployed due to its stability and extensive documentation.

### Core Capabilities

| Feature | Description |
|---|---|
| On-the-fly hashing | MD5, SHA-1, SHA-256, SHA-384, SHA-512 |
| Pattern matching | Hex string search during imaging |
| Split output | Automatic file splitting at specified sizes |
| Hash logging | Write hash values to files for verification |
| Verify mode | Compare source against destination hashes |
| Memory buffer | Configurable buffer sizes for performance |
| Padding | Zero-pad incomplete blocks |

### When to Use DCFDLD

- Forensic disk imaging with hash verification
- Incident response evidence collection
- Media analysis with pattern detection
- Training and education (simpler than dc3dd)
- Environments where dc3dd is not available

### Comparison with Similar Tools

| Feature | dd | dcfldd | dc3dd | ddrescue |
|---|---|---|---|---|
| Hash | No | Yes | Yes | No |
| Pattern | No | Yes | Yes | No |
| Split | No | Yes | Yes | Yes |
| Verify | No | Yes | Yes | No |
| Error handling | Basic | Yes | Yes | Advanced |
| Read errors | Fail | Continue | Continue | Retry |

### Limitations

- No on-the-fly progress reporting (use `status=progress` where supported)
- Pattern matching is single-pass only
- No built-in wipe capabilities
- Limited to command-line operation
- Does not handle bad sectors as gracefully as ddrescue

## Chapter 2: Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install dcfldd
```

### Verify Installation

```bash
dcfldd --version
```

### From Source

```bash
git clone https://github.com/resurrecting-open-source-projects/dcfldd.git
cd dcfldd
./configure
make
sudo make install
```

### Dependencies

- Standard C library
- POSIX headers
- Make and GCC (for source builds)

### Post-Installation Verification

```bash
# Verify binary location
which dcfldd

# Check available options
dcfldd --help 2>&1 | head -40
```

## Chapter 3: Basic Usage

### Syntax

```
dcfldd [if=<input>] [of=<output>] [bs=<block_size>] [count=<blocks>] [hash=<algo>] [hashlog=<file>] [hashlog2=<file>] [pattern=<hex>] [verify=<log>] [split=<size>] [conv=<flags>]
```

### Core Parameters

| Parameter | Description | Default |
|---|---|---|
| `if=` | Input file/device | stdin |
| `of=` | Output file/device | stdout |
| `bs=` | Block size | 512 |
| `count=` | Blocks to copy | all |
| `skip=` | Skip input blocks | 0 |
| `seek=` | Skip output blocks | 0 |
| `hash=` | Hash algorithm | none |
| `hashlog=` | Hash log file 1 | none |
| `hashlog2=` | Hash log file 2 | none |
| `pattern=` | Hex pattern | none |
| `verify=` | Verification log | none |
| `split=` | Split size | none |
| `conv=` | Conversion flags | none |
| `padpad=` | Pad character | 0 |
| `bufsize=` | Buffer size | 4096 |

### Hash Algorithms

| Algorithm | Flag | Speed | Security |
|---|---|---|---|
| MD5 | `hash=md5` | Fastest | Low |
| SHA-1 | `hash=sha1` | Fast | Moderate |
| SHA-256 | `hash=sha256` | Moderate | High |
| SHA-384 | `hash=sha384` | Moderate | High |
| SHA-512 | `hash=sha512` | Slower | High |
| Multiple | `hash=md5,sha256` | Varies | Varies |

### Sample Commands and Output

#### Basic Forensic Image

```bash
sudo dcfldd if=/dev/sda of=/evidence/sda_image.dd hash=md5 hashlog=/evidence/sda.hash
```

Output:
```
488397168 blocks (488397168 Mb) written.
Total (md5): 3d4f2bf07dc1faba79b92a3796499564
```

Hash log content (`/evidence/sda.hash`):
```
3d4f2bf07dc1faba79b92a3796499564  /evidence/sda_image.dd
```

#### Image with Multiple Hashes

```bash
sudo dcfldd if=/dev/sda of=/evidence/sda_image.dd hash=md5,sha256 hashlog=/evidence/sda.md5 hashlog2=/evidence/sda.sha256
```

Output:
```
488397168 blocks (488397168 Mb) written.
Total (md5): 3d4f2bf07dc1faba79b92a3796499564
Total (sha256): 7b8e9a1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a
```

#### Image with Error Handling

```bash
sudo dcfldd if=/dev/sdb of=/evidence/sdb_image.dd hash=md5 conv=noerror,sync hashlog=/evidence/sdb.hash
```

#### Pattern Search During Imaging

```bash
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 pattern="50415353" hashlog=/evidence/image.hash
```

Pattern matches appear during imaging:
```
pattern match at offset 12345678
pattern match at offset 98765432
```

#### Split Output

```bash
sudo dcfldd if=/dev/sda of=/evidence/sda_%f.dd hash=md5 split=4G hashlog=/evidence/sda.hash
```

Creates files: `sda_000001.dd`, `sda_000002.dd`, etc.

#### Verify Image

```bash
# Create image with hash log
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 hashlog=/evidence/src.hash

# Verify against hash log
dcfldd if=/dev/sda verify=/evidence/src.hash
```

Output:
```
Total (md5): 3d4f2bf07dc1faba79b92a3796499564
Verify pass: hash matches
```

#### Copy Specific Sectors

```bash
# Image only the first 1GB
sudo dcfldd if=/dev/sda of=/evidence/first_1gb.dd count=2097152 hash=md5 hashlog=/evidence/1gb.hash

# Skip first 1GB, image next 500MB
sudo dcfldd if=/dev/sda of=/evidence/500mb.dd skip=2097152 count=1048576 hash=md5 hashlog=/evidence/500mb.hash
```

#### Copy with Larger Block Size

```bash
# Faster imaging with 64KB blocks
sudo dcfldd if=/dev/sda of=/evidence/image.dd bs=65536 hash=md5 hashlog=/evidence/image.hash
```

#### Zero-Fill Device

```bash
sudo dcfldd if=/dev/zero of=/dev/sdb bs=4096 hash=md5 hashlog=/dev/null
```

## Chapter 4: Intermediate Usage and Scripting

### Automated Evidence Collection

```bash
#!/bin/bash
# collect_evidence.sh — Automated forensic imaging

set -euo pipefail

# Configuration
EVIDENCE_DIR="/evidence/case_$(date +%Y%m%d_%H%M%S)"
SOURCE="/dev/sdb"
HASH="md5,sha256"
SPLIT_SIZE="4G"

# Setup
mkdir -p "${EVIDENCE_DIR}"
TIMESTAMP=$(date -u +"%Y-%m-%d %H:%M:%S UTC")

echo "[*] Evidence collection started at ${TIMESTAMP}"
echo "[*] Source: ${SOURCE}"
echo "[*] Destination: ${EVIDENCE_DIR}"

# Create chain of custody header
cat > "${EVIDENCE_DIR}/custody.txt" << EOF
Case ID: case_$(date +%Y%m%d)
Timestamp: ${TIMESTAMP}
Operator: $(whoami)
Source Device: ${SOURCE}
Source Model: $(lsblk -dno MODEL ${SOURCE} 2>/dev/null || echo "unknown")
Source Serial: $(lsblk -dno SERIAL ${SOURCE} 2>/dev/null || echo "unknown")
Source Size: $(blockdev --getsize64 ${SOURCE} 2>/dev/null || echo "unknown") bytes
Hash Algorithms: ${HASH}
Tool: dcfldd
EOF

# Image with multiple hashes
dcfldd \
    if="${SOURCE}" \
    of="${EVIDENCE_DIR}/image_%f.dd" \
    split="${SPLIT_SIZE}" \
    hash="${HASH}" \
    hashlog="${EVIDENCE_DIR}/hash.md5" \
    hashlog2="${EVIDENCE_DIR}/hash.sha256" \
    conv=noerror,sync

echo "[*] Imaging complete."
echo "[*] Hash logs: ${EVIDENCE_DIR}/hash.md5, ${EVIDENCE_DIR}/hash.sha256"
```

### Batch Hashing

```bash
#!/bin/bash
# batch_hash.sh — Hash multiple evidence files

EVIDENCE_DIR="$1"

for f in "${EVIDENCE_DIR}"/*.dd; do
    echo "Hashing: ${f}"
    md5sum "${f}" >> "${EVIDENCE_DIR}/all_hashes.md5"
    sha256sum "${f}" >> "${EVIDENCE_DIR}/all_hashes.sha256"
done

echo "All hashes computed."
cat "${EVIDENCE_DIR}/all_hashes.md5"
```

### Monitoring Imaging Progress

```bash
#!/bin/bash
# monitor.sh — Monitor dcfldd output size

OUTPUT_FILE="$1"
INTERVAL=5

while pgrep -x dcfldd > /dev/null; do
    SIZE=$(stat -c%s "${OUTPUT_FILE}" 2>/dev/null || echo 0)
    SIZE_MB=$((SIZE / 1048576))
    echo "[$(date +%H:%M:%S)] Current size: ${SIZE_MB} MB"
    sleep "${INTERVAL}"
done

echo "[*] Imaging complete."
```

### Parallel Imaging

```bash
#!/bin/bash
# parallel_image.sh — Image multiple drives

DEVICES=("/dev/sdb" "/dev/sdc" "/dev/sdd")
OUTDIR="/evidence/multi_$(date +%Y%m%d)"

mkdir -p "${OUTDIR}"

for dev in "${DEVICES[@]}"; do
    name=$(basename "${dev}")
    echo "[*] Starting image of ${dev}"
    sudo dcfldd if="${dev}" \
        of="${OUTDIR}/${name}.dd" \
        hash=md5,sha256 \
        hashlog="${OUTDIR}/${name}.md5" \
        hashlog2="${OUTDIR}/${name}.sha256" \
        conv=noerror,sync &
done

echo "[*] All imaging jobs started. Waiting..."
wait
echo "[*] All imaging jobs complete."
```

### Verify All Images in Directory

```bash
#!/bin/bash
# verify_all.sh — Verify all images against hash logs

EVIDENCE_DIR="$1"
FAILURES=0

for hash_file in "${EVIDENCE_DIR}"/*.hash; do
    [ -f "${hash_file}" ] || continue
    echo "Verifying: ${hash_file}"
    dcfldd if=/dev/null verify="${hash_file}" 2>&1
    if [ $? -ne 0 ]; then
        echo "[FAIL] ${hash_file}"
        ((FAILURES++))
    fi
done

if [ ${FAILURES} -eq 0 ]; then
    echo "[PASS] All verifications successful"
else
    echo "[FAIL] ${FAILURES} verification(s) failed"
fi
```

## Chapter 5: Advanced Configurations

### Custom Buffer Sizes

```bash
# Increase buffer for faster imaging (default: 4096)
sudo dcfldd if=/dev/sda of=/evidence/image.dd bs=65536 hash=md5 hashlog=/evidence/image.hash

# Decrease buffer for lower memory usage
sudo dcfldd if=/dev/sda of=/evidence/image.dd bs=512 hash=md5 hashlog=/evidence/image.hash
```

### Padding Configuration

```bash
# Pad incomplete blocks with zeros (default)
sudo dcfldd if=/dev/sda of=/evidence/image.dd padpad=0 hash=md5 hashlog=/evidence/image.hash

# Pad with specific character
sudo dcfldd if=/dev/sda of=/evidence/image.dd padpad=ff hash=md5 hashlog=/evidence/image.hash
```

### Dual Hash Logging

DCFDLD supports writing hash logs to two separate files simultaneously:

```bash
sudo dcfldd if=/dev/sda of=/evidence/image.dd \
    hash=md5,sha256 \
    hashlog=/evidence/image_md5.log \
    hashlog2=/evidence/image_sha256.log
```

### Integration with Other Tools

```bash
# Image then analyze with autopsy
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 hashlog=/evidence/image.hash
autopsy /evidence/image.dd

# Image then carve with scalpel
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 hashlog=/evidence/image.hash
scalpel /evidence/image.dd -o /evidence/carved/

# Image then search with strings
sudo dcfldd if=/dev/sda of=/evidence/image.dd hash=md5 hashlog=/evidence/image.hash
strings /evidence/image.dd | grep -i "password"
```

### Network Transfer

```bash
# Image over network (source)
sudo dcfldd if=/dev/sda hash=md5 | nc -l -p 9999

# Receive on forensic workstation
nc 192.168.1.100 9999 > /evidence/image.dd
md5sum /evidence/image.dd
```

### Handling Large Images

```bash
# For images larger than available disk, split and transfer
sudo dcfldd if=/dev/sda \
    of=/mnt/external/image_%f.dd \
    split=4G \
    hash=md5 \
    hashlog=/mnt/external/image.hash

# Or stream to multiple locations
sudo dcfldd if=/dev/sda \
    of=/mnt/external1/image.dd \
    hash=md5 \
    hashlog=/mnt/external1/image.hash | \
tee /mnt/external2/image.dd > /dev/null
```

## Chapter 6: Real-World Scenarios

### Scenario 1: Incident Response Triage

**Context**: A suspected malware infection on a Windows workstation. The machine is still running but needs immediate evidence preservation.

**Approach**:
```bash
# 1. Create memory dump first (if volatile memory is critical)
sudo dd if=/proc/kcore of=/evidence/memory.dump bs=1M

# 2. Image the hard drive
sudo dcfldd if=/dev/sda \
    of=/evidence/disk_image.dd \
    hash=md5,sha256 \
    hashlog=/evidence/disk_md5.hash \
    hashlog2=/evidence/disk_sha256.hash \
    conv=noerror,sync

# 3. Verify image
dcfldd if=/dev/sda verify=/evidence/disk_md5.hash

# 4. Begin triage
strings /evidence/disk_image.dd | grep -iE "(malware|payload|c2)" > /evidence/triage.txt
```

### Scenario 2: Mobile Device Forensics

**Context**: A seized Android phone needs image extraction. The device is connected via USB in forensic mode.

**Approach**:
```bash
# Identify the device
lsblk
# Assume the phone appears as /dev/sdb

# Create forensic image
sudo dcfldd if=/dev/sdb \
    of=/evidence/phone_image.dd \
    hash=md5,sha256 \
    hashlog=/evidence/phone_md5.hash \
    hashlog2=/evidence/phone_sha256.hash \
    conv=noerror,sync

# Extract deleted data using specialized tools
strings /evidence/phone_image.dd | grep -E "\+[0-9]{10}" > /evidence/phone_numbers.txt
```

### Scenario 3: Legal Hold Imaging

**Context**: A law firm requires a forensic image of a corporate laptop for litigation purposes, with strict chain of custody requirements.

**Approach**:
```bash
#!/bin/bash
# legal_hold_imaging.sh — Forensic imaging with legal documentation

EVIDENCE="/evidence/legal_$(date +%Y%m%d_%H%M%S)"
DEVICE="/dev/sda"

mkdir -p "${EVIDENCE}"

# Comprehensive chain of custody
cat > "${EVIDENCE}/chain_of_custody.txt" << EOF
LEGAL HOLD FORENSIC IMAGE
=========================
Date: $(date -u +"%Y-%m-%d %H:%M:%S UTC")
Operator: $(whoami) ($(id -un))
Workstation: $(hostname) ($(hostname -I | awk '{print $1}'))
Source Device: ${DEVICE}
Device Model: $(lsblk -dno MODEL ${DEVICE} 2>/dev/null || echo "unknown")
Device Serial: $(lsblk -dno SERIAL ${DEVICE} 2>/dev/null || echo "unknown")
Device Size: $(blockdev --getsize64 ${DEVICE} 2>/dev/null) bytes
Tool: dcfldd $(dcfldd --version 2>/dev/null || echo "unknown")
Hash Algorithms: MD5, SHA-256
Write Blocker: $(lsusb 2>/dev/null | grep -i "write" || echo "Verify manually")
EOF

# Image with dual hash logs
dcfldd \
    if="${DEVICE}" \
    of="${EVIDENCE}/laptop_image.dd" \
    hash=md5,sha256 \
    hashlog="${EVIDENCE}/md5.hash" \
    hashlog2="${EVIDENCE}/sha256.hash" \
    conv=noerror,sync

# Independent verification
echo "" >> "${EVIDENCE}/chain_of_custody.txt"
echo "INDEPENDENT VERIFICATION" >> "${EVIDENCE}/chain_of_custody.txt"
echo "MD5: $(md5sum ${EVIDENCE}/laptop_image.dd | awk '{print $1}')" >> "${EVIDENCE}/chain_of_custody.txt"
echo "SHA-256: $(sha256sum ${EVIDENCE}/laptop_image.dd | awk '{print $1}')" >> "${EVIDENCE}/chain_of_custody.txt"

echo "[*] Legal hold imaging complete. Evidence at: ${EVIDENCE}"
```

## Chapter 7: Detection and Defense

### Detecting DCFDLD Usage

**System Monitoring**:
```bash
# Check process history
lastcomm dcfldd
sa -c | grep dcfldd

# Check system logs
grep -r "dcfldd" /var/log/syslog* /var/log/auth.log* 2>/dev/null

# Check bash history
grep dcfldd ~/.bash_history
grep dcfldd /root/.bash_history
```

**File System Indicators**:
```bash
# Hash log files
find / -name "*.hash" -o -name "*hash*.log" 2>/dev/null

# Large .dd files
find / -name "*.dd" -size +1G 2>/dev/null

# DC3DD-style log headers
grep -r "dcfldd" /evidence/ 2>/dev/null
```

**Network Monitoring**:
```bash
# Check for network transfers during imaging
netstat -an | grep -E ":(9999|4444|8888)"
tcpdump -i any port 9999
```

### Defensive Measures

- Monitor for `dcfldd` in command execution logs
- Alert on direct access to `/dev/sd*` devices
- Implement device access controls via udev rules
- Use USB device whitelisting
- Enable filesystem monitoring on evidence directories

### Audit Configuration

```bash
# Add to /etc/audit/rules.d/forensics.rules
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/dcfldd -F key=forensics_dcfldd
-a always,exit -F arch=b64 -S openat -F path=/dev/sd* -F key=forensics_devices
```

## Chapter 8: Troubleshooting

### Common Issues

| Error | Cause | Solution |
|---|---|---|
| `Permission denied` | Not root | Use `sudo` |
| `No space left on device` | Destination full | Use `split=` parameter |
| `Input/output error` | Bad sectors | Add `conv=noerror,sync` |
| `Hash mismatch` | Corruption or tampering | Re-image with write blocker |
| `Device busy` | Another process using device | Kill conflicting processes |

### Performance Issues

```bash
# Check I/O utilization
iostat -x 1 5

# Check for competing processes
iotop -o

# Try larger block size
sudo dcfldd if=/dev/sda of=/evidence/image.dd bs=65536 hash=md5 hashlog=/evidence/image.hash
```

### Hash Mismatch Investigation

```bash
# 1. Verify the hash log file integrity
cat /evidence/src.hash

# 2. Re-compute hash on source
dcfldd if=/dev/sda hash=md5

# 3. Re-compute hash on image
md5sum /evidence/image.dd

# 4. Compare
echo "Source hash: [from step 2]"
echo "Image hash: [from step 3]"

# If mismatch, re-image with write blocker
```

### Resume Interrupted Imaging

```bash
# Calculate how much was written
WRITTEN=$(stat -c%s /evidence/image.dd)
SECTORS=$((WRITTEN / 512))

# Resume from that point
sudo dcfldd if=/dev/sda \
    of=/evidence/image.dd \
    skip=${SECTORS} \
    seek=${SECTORS} \
    hash=md5 \
    hashlog=/evidence/resume.hash \
    conv=noerror,sync
```

### Verifying Tool Integrity

```bash
# Check dcfldd binary hasn't been tampered with
dpkg -V dcfldd

# Compare checksum against package
sha256sum /usr/bin/dcfldd
apt-file search /usr/bin/dcfldd
```

## Resources

- **Official Repository**: https://github.com/resurrecting-open-source-projects/dcfldd
- **DC3 Forensics**: https://www.dc3.mil/
- **NIST Forensics Guidelines**: https://www.nist.gov/
- **SANS DFIR**: https://www.sans.org/dfir/
- **Kali Linux Documentation**: https://www.kali.org/docs/

---

*Last updated: 2026-07-18*
