---
tool_name: dc3dd
display_name: DC3DD
mitre_tactic: collection
mitre_tactic_id: TA0009
install_command: sudo apt install dc3dd
github_repo: https://github.com/resurrecting-open-source-projects/dc3dd
kali_tools_page: https://www.kali.org/tools/dc3dd/
difficulty_level: beginner
tags:
  - forensics
  - disk-imaging
  - evidence
  - bitstream
  - dfir
related_tools:
  - dcfldd
  - ddrescue
  - guymager
  - dd
---

# DC3DD — Defense Cyber Crime Center Disk Dump

## Chapter 1: Overview

DC3DD is a forensic-grade disk imaging tool developed by the Defense Cyber Crime Center (DC3), the cyber crime unit of the U.S. Department of Defense. It is a patched version of the traditional `dd` utility, enhanced with features specifically designed for digital forensics and incident response (DFIR) workflows.

### Why DC3DD Exists

The classic `dd` tool has been a Unix staple for decades, but it lacks critical forensic capabilities: no progress reporting during long operations, no built-in hashing for evidence verification, no flexible pattern matching, and no integrated logging. DC3DD addresses every one of these gaps while maintaining command-line compatibility with `dd`.

### Core Differentiators

| Feature | dd | dc3dd |
|---|---|---|
| Progress reporting | No | Yes (real-time) |
| On-the-fly hashing | No | Yes (MD5, SHA-1, SHA-256, SHA-512) |
| Log file generation | No | Yes (detailed operation logs) |
| Pattern matching | No | Yes (search while imaging) |
| Split output | Manual | Built-in with automatic naming |
| Verify mode | No | Yes (compare source to destination) |
| Wipe pass | No | Yes (DoD 5220.22-M patterns) |

### When to Use DC3DD

- Creating bit-for-bit forensic images of storage media
- Generating chain-of-custody documentation with hash verification
- Searching for specific patterns during acquisition (e.g., finding credit card numbers)
- Splitting large images across multiple storage devices
- Wiping media before disposal or reuse
- Any scenario requiring auditable, forensically sound disk imaging

### Legal and Compliance

DC3DD produces forensically sound images when used correctly. The combination of on-the-fly hashing, detailed logging, and write-blocking (when used with hardware write blockers) meets the evidence handling requirements specified in:
- Federal Rules of Evidence
- NIST SP 800-86 (Guide to Integrating Forensic Techniques into Incident Response)
- ASTM E2763-10 (Standard Practice for Computer Forensics)

### Limitations

- DC3DD does not bypass hardware write blockers; it still writes to the destination
- Extremely large images may take hours; plan for power and time accordingly
- Pattern matching is single-pass only; complex regex requires separate tools
- No graphical interface; all operations are command-line driven
- Cannot image encrypted volumes without the decryption key

## Chapter 2: Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install dc3dd
```

### Verify Installation

```bash
dc3dd --version
```

Expected output:
```
dc3dd 7.3.1
```

### From Source (Alternative)

```bash
git clone https://github.com/resurrecting-open-source-projects/dc3dd.git
cd dc3dd
./configure
make
sudo make install
```

### Dependencies

DC3DD has minimal dependencies:
- Standard C library (libc)
- POSIX headers
- Make (for source builds)
- GCC (for source builds)

### Installation Verification

```bash
# Confirm the binary is in PATH
which dc3dd

# Check available options
dc3dd --help 2>&1 | head -50
```

## Chapter 3: Basic Usage

### Syntax

DC3DD mirrors `dd` syntax with extended options:

```
dc3dd [if=<input_file>] [of=<output_file>] [bs=<block_size>] [count=<blocks>] [skip=<blocks>] [hash=<algorithm>] [log=<logfile>] [pattern=<hex_string>]
```

### Core Parameters

| Parameter | Short | Description | Default |
|---|---|---|---|
| `if=` | `-if` | Input file (source device or image) | stdin |
| `of=` | `-of` | Output file (destination image) | stdout |
| `bs=` | `-bs` | Block size in bytes | 512 |
| `count=` | `-c` | Number of blocks to copy | all |
| `skip=` | `-s` | Skip N blocks at start of input | 0 |
| `seek=` | `-os` | Skip N blocks at start of output | 0 |
| `hash=` | `-h` | Hash algorithm(s) to compute | none |
| `log=` | `-l` | Log file path | none |
| `pattern=` | `-p` | Hex pattern to search for | none |
| `rhlog=` | `-rh` | Read hash log (for verification) | none |
| `whlog=` | `-wh` | Write hash log (for verification) | none |
| `conv=` | `-conv` | Conversion flags (noerror,sync) | none |
| `hashwindow=` | `-hw` | Compute hash every N blocks | 0 (disabled) |
| `trim=` | `-t` | Trim zero blocks from output | no |
| `wmode=` | `-wm` | Wipe mode before writing | no |

### Hash Algorithms

| Algorithm | Flag | Output Length | Speed |
|---|---|---|---|
| MD5 | `hash=md5` | 128-bit | Fast |
| SHA-1 | `hash=sha1` | 160-bit | Fast |
| SHA-256 | `hash=sha256` | 256-bit | Moderate |
| SHA-512 | `hash=sha512` | 512-bit | Moderate |
| Multiple | `hash=md5,sha1,sha256` | Multiple | Varies |

### Sample Commands and Output

#### Basic Disk Image

```bash
sudo dc3dd if=/dev/sda of=/evidence/sda_image.dd hash=md5 log=/evidence/sda.log
```

Output:
```
dc3dd 7.3.1
  input /dev/sda (/dev/sda), 500107862016 bytes (976773168 sectors), 500107862016 bytes input
  output /evidence/sda_image.dd (/evidence/sda_image.dd), 500107862016 bytes (976773168 sectors)
  hash md5
  bytes 500107862016 (500107862016)
  md5: 3d4f2bf07dc1faba79b92a3796499564
  976773168 sectors (500107862016 bytes) in 4321.54 seconds (115.7 MB/s)
  total 500107862016 bytes (500107862016 bytes) in 4321.54 seconds (115.7 MB/s)
```

#### Partition Imaging (Skip First 2048 Sectors)

```bash
sudo dc3dd if=/dev/sda of=/evidence/sda_partition2.dd skip=2048 count=1000000 hash=sha256 log=/evidence/sda_part2.log
```

Output:
```
dc3dd 7.3.1
  input /dev/sda (/dev/sda), 500107862016 bytes (976773168 sectors), 500107862016 bytes input
  output /evidence/sda_partition2.dd (/evidence/sda_partition2.dd), 512000000 bytes (1000000 sectors)
  skip 2048 blocks (1048576 bytes)
  hash sha256
  bytes 512000000 (512000000)
  sha256: 7b8e9a1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a
  1000000 sectors (512000000 bytes) in 4423.12 seconds (115.7 MB/s)
  total 512000000 bytes (512000000 bytes) in 4423.12 seconds (115.7 MB/s)
```

#### Image with Split Output

```bash
sudo dc3dd if=/dev/sdb of=/evidence/sdb_%f.dd hash=md5 log=/evidence/sdb.log ofsz=4294967296
```

This splits the output into 4 GB chunks (files named `sdb_000001.dd`, `sdb_000002.dd`, etc.).

#### Search While Imaging

```bash
sudo dc3dd if=/dev/sdc of=/evidence/sdc_image.dd pattern="50415353574f5244" hash=md5 log=/evidence/sdc.log
```

The hex string `50415353574f5244` decodes to "PASSWORD". DC3DD will report any matches during the imaging process.

#### Pattern Matching Output

```
pattern match at offset 12345678
pattern match at offset 98765432
pattern match at offset 456789012
```

#### Verify Image Against Source

```bash
# First, create the image with hash logs
sudo dc3dd if=/dev/sda of=/evidence/sda_image.dd hash=md5 rhlog=/evidence/sda_src.md5 whlog=/evidence/sda_dst.md5 log=/evidence/sda.log

# Then verify
dc3dd if=/dev/sda of=/evidence/sda_image.dd rhlog=/evidence/sda_src.md5 whlog=/evidence/sda_dst.md5 log=/evidence/sda_verify.log
```

Verify output:
```
rhlog and whlog match
```

#### Error Handling During Imaging

```bash
sudo dc3dd if=/dev/sdb of=/evidence/sdb_image.dd conv=noerror,sync hash=sha1 log=/evidence/sdb.log
```

When `noerror` is specified, DC3DD continues past read errors, filling unreadable sectors with zeros (`sync` ensures the block count remains consistent).

Error output example:
```
dc3dd: /dev/sdb: Input/output error
1234+0 records in
1234+0 records out
```

#### Wipe Media

```bash
sudo dc3dd if=/dev/zero of=/dev/sdb ofsz=1048576 hash=md5 wmode=dod522022m log=/evidence/wipe.log
```

Wipe modes:

| Mode | Description | Passes |
|---|---|---|
| `dod522022m` | DoD 5220.22-M | 3 |
| `dodshort` | DoD Short | 3 |
| `bsi` | BSI/TL-03423 | 2 |
| `gutmann` | Gutmann method | 35 |
| `prng` | Pseudorandom | 1 |
| `zero` | All zeros | 1 |
| `one` | All ones | 1 |
| `oneszero` | Ones then zeros | 2 |
| `custom` | User-defined pattern | N |

## Chapter 4: Intermediate Usage and Scripting

### Automated Evidence Collection Script

```bash
#!/bin/bash
# evidence_collector.sh — Automated forensic imaging with verification

set -euo pipefail

# Configuration
EVIDENCE_DIR="/evidence/case_$(date +%Y%m%d_%H%M%S)"
SOURCE_DEVICE="/dev/sdb"
HASH_ALGOS="md5,sha1,sha256"
SPLIT_SIZE="4294967296"
CASE_ID="CASE-2026-0042"
OPERATOR="jdoe"

# Create evidence directory
mkdir -p "${EVIDENCE_DIR}"
cd "${EVIDENCE_DIR}"

# Get source device info
echo "[*] Source device info:"
lsblk -o NAME,SIZE,MODEL,SERIAL "${SOURCE_DEVICE}"
BLOCK_COUNT=$(blockdev --getsize64 "${SOURCE_DEVICE}" 2>/dev/null || sudo blockdev --getsize64 "${SOURCE_DEVICE}")
echo "[*] Total size: ${BLOCK_COUNT} bytes"

# Create chain of custody header
cat > chain_of_custody.txt << EOF
Case ID:       ${CASE_ID}
Operator:      ${OPERATOR}
Date:          $(date -u +"%Y-%m-%d %H:%M:%S UTC")
Source:        ${SOURCE_DEVICE}
Source Size:   ${BLOCK_COUNT} bytes
Hash Algorithms: ${HASH_ALGOS}
Split Size:    ${SPLIT_SIZE} bytes
Tool:          dc3dd $(dc3dd --version 2>/dev/null || echo "unknown")
EOF

# Image with multiple hashes
echo "[*] Starting forensic image..."
dc3dd \
    if="${SOURCE_DEVICE}" \
    of="evidence_%f.dd" \
    ofsz="${SPLIT_SIZE}" \
    hash="${HASH_ALGOS}" \
    log="imaging.log" \
    conv=noerror,sync

echo "[*] Imaging complete."

# Compute independent hashes for verification
echo "[*] Computing independent hashes..."
for f in evidence_*.dd; do
    md5sum "${f}" >> independent_hashes.md5
    sha1sum "${f}" >> independent_hashes.sha1
    sha256sum "${f}" >> independent_hashes.sha256
done

echo "[*] Verification hashes computed."
echo "[*] Evidence collection complete at ${EVIDENCE_DIR}"
```

### Parallel Imaging Across Multiple Devices

```bash
#!/bin/bash
# parallel_image.sh — Image multiple drives simultaneously

EVIDENCE_BASE="/evidence"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
MAX_PARALLEL=3

declare -A DEVICES
DEVICES[sdb]="/dev/sdb"
DEVICES[sdc]="/dev/sdc"
DEVICES[sdd]="/dev/sdd"

image_device() {
    local dev_name=$1
    local dev_path=$2
    local out_dir="${EVIDENCE_BASE}/${TIMESTAMP}_${dev_name}"
    
    mkdir -p "${out_dir}"
    
    echo "[${dev_name}] Starting image at $(date)"
    sudo dc3dd \
        if="${dev_path}" \
        of="${out_dir}/image_%f.dd" \
        ofsz=4294967296 \
        hash=md5,sha256 \
        log="${out_dir}/imaging.log" \
        conv=noerror,sync 2>&1 | tee "${out_dir}/console.log"
    echo "[${dev_name}] Complete at $(date)"
}

export -f image_device
export EVIDENCE_BASE TIMESTAMP

# Run with GNU parallel or background jobs
jobs=0
for dev_name in "${!DEVICES[@]}"; do
    image_device "${dev_name}" "${DEVICES[$dev_name]}" &
    ((jobs++))
    if (( jobs >= MAX_PARALLEL )); then
        wait -n 2>/dev/null || wait
        ((jobs--))
    fi
done
wait

echo "[*] All imaging tasks complete."
```

### Hash Verification Script

```bash
#!/bin/bash
# verify_image.sh — Verify a forensic image against known hashes

IMAGE_FILE="$1"
KNOWN_HASH="$2"
ALGORITHM="${3:-md5}"

case "${ALGORITHM}" in
    md5)    COMPUTED=$(md5sum "${IMAGE_FILE}" | awk '{print $1}') ;;
    sha1)   COMPUTED=$(sha1sum "${IMAGE_FILE}" | awk '{print $1}') ;;
    sha256) COMPUTED=$(sha256sum "${IMAGE_FILE}" | awk '{print $1}') ;;
    *)      echo "Unsupported algorithm: ${ALGORITHM}"; exit 1 ;;
esac

echo "Computed:  ${COMPUTED}"
echo "Expected:  ${KNOWN_HASH}"

if [ "${COMPUTED}" = "${KNOWN_HASH}" ]; then
    echo "[PASS] Hash verification successful"
    exit 0
else
    echo "[FAIL] Hash mismatch — image may be corrupted"
    exit 1
fi
```

### Monitoring Imaging Progress

```bash
#!/bin/bash
# monitor_imaging.sh — Monitor dc3dd progress via log file

LOG_FILE="$1"
INTERVAL=5

while true; do
    if ! pgrep -x dc3dd > /dev/null; then
        echo "[*] dc3dd process completed."
        break
    fi
    
    BYTES=$(grep -oP 'bytes \K[0-9]+' "${LOG_FILE}" 2>/dev/null | tail -1)
    TOTAL=$(stat -c%s /dev/sdb 2>/dev/null || echo "unknown")
    
    if [ -n "${BYTES}" ] && [ "${TOTAL}" != "unknown" ]; then
        PCT=$(( (BYTES * 100) / TOTAL ))
        echo "[$(date +%H:%M:%S)] Progress: ${PCT}% (${BYTES}/${TOTAL} bytes)"
    fi
    
    sleep "${INTERVAL}"
done
```

### Using Hash Logs for Chain of Custody

```bash
# Step 1: Create image with hash logs
sudo dc3dd if=/dev/sda \
    of=/evidence/case001/sda_image.dd \
    hash=md5,sha256 \
    rhlog=/evidence/case001/source_hashes.log \
    whlog=/evidence/case001/image_hashes.log \
    log=/evidence/case001/imaging.log

# Step 2: Verify hashes match
dc3dd if=/dev/sda \
    of=/evidence/case001/sda_image.dd \
    rhlog=/evidence/case001/source_hashes.log \
    whlog=/evidence/case001/image_hashes.log \
    log=/evidence/case001/verify.log

# Output: "rhlog and whlog match"
```

### Batch Processing Multiple Partitions

```bash
#!/bin/bash
# image_partitions.sh — Image all partitions on a device

DEVICE="$1"
OUT_DIR="$2"

mkdir -p "${OUT_DIR}"

# Get partition layout
PARTITIONS=$(lsblk -ln -o NAME "${DEVICE}" | grep -v "$(basename ${DEVICE})" | sed 's/^[[:space:]]*//')

for PART in ${PARTITIONS}; do
    PART_PATH="/dev/${PART}"
    PART_NAME=$(basename "${PART}")
    PART_OFFSET=$(sudo fdisk -l "${DEVICE}" 2>/dev/null | grep "^${PART_PATH} " | awk '{print $2}')
    PART_SIZE=$(sudo fdisk -l "${DEVICE}" 2>/dev/null | grep "^${PART_PATH} " | awk '{print $4}')
    
    echo "[*] Imaging ${PART_NAME} (${PART_SIZE} sectors, offset ${PART_OFFSET})"
    
    sudo dc3dd \
        if="${DEVICE}" \
        of="${OUT_DIR}/${PART_NAME}.dd" \
        skip="${PART_OFFSET}" \
        count="${PART_SIZE}" \
        hash=md5,sha256 \
        log="${OUT_DIR}/${PART_NAME}.log" \
        conv=noerror,sync
done

echo "[*] All partitions imaged."
```

## Chapter 5: Advanced Configurations

### Custom Hash Logging Formats

DC3DD generates hash logs in a structured format. Understanding this format enables integration with custom verification tools:

```
dc3dd hash log v1
input /dev/sda
output /evidence/sda_image.dd
hash md5
blocksize 512
blocks 976773168
md5 3d4f2bf07dc1faba79b92a3796499564
```

### Integration with Forensic Suites

DC3DD images are directly usable by:
- **Autopsy/Sleuth Kit**: Mount images with `-o loop,ro`
- **Volatility**: Analyze memory dumps with `volatility -f <image>`
- **FTK Imager**: Open raw images for viewing
- **X-Ways Forensics**: Import as evidence item

### Performance Tuning

| Block Size | Best For | Trade-off |
|---|---|---|
| 512 | Compatibility | Slowest throughput |
| 4096 | General use | Good balance |
| 65536 | Fast imaging | May miss alignment issues |
| 1048576 | Maximum throughput | Requires ample RAM |

```bash
# Benchmark different block sizes
for bs in 512 4096 65536 1048576; do
    echo "Testing bs=${bs}"
    sudo dc3dd if=/dev/zero of=/dev/null bs=${bs} count=1000000 2>&1 | grep -E 'seconds|MB/s'
done
```

### Wipe Patterns in Detail

| Pattern | Description | Security Level |
|---|---|---|
| `zero` | Single pass zeros | Low (data recovery possible) |
| `one` | Single pass ones | Low |
| `dod522022m` | DoD 5220.22-M (3-pass) | Medium |
| `gutmann` | Gutmann 35-pass | High (theoretical) |
| `prng` | Pseudorandom data | Medium |
| `bsi` | BSI TL-03423 (2-pass) | Medium |

### Custom Wipe Patterns

```bash
# Write specific pattern repeatedly
sudo dc3dd if=/dev/zero of=/dev/sdb bs=512 count=1024 ofsz=524288 conv=notrunc
```

### Network-Based Imaging

```bash
# Image over network using SSH pipe
sudo dc3dd if=/dev/sda bs=65536 | ssh investigator@192.168.1.100 "cat > /evidence/remote_image.dd"

# Image from network device
sudo dc3dd if=/dev/ram0 of=/dev/null  # test with RAM disk first
```

### Handling Bad Sectors

```bash
# Imaging with error tolerance
sudo dc3dd if=/dev/sdb of=/evidence/image.dd \
    conv=noerror,sync \
    hash=md5 \
    log=/evidence/image.log

# The 'sync' pad ensures the output image maintains the correct sector count
# Bad sectors are filled with zeros in the output
```

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Data Breach Investigation

**Context**: A financial institution suspects unauthorized access to a server. The server must remain operational while evidence is collected.

**Approach**:
```bash
# 1. Verify write blocker is connected
lsblk | grep sdb  # Should show the blocked device

# 2. Create forensic image with full documentation
sudo dc3dd if=/dev/sdb \
    of="/evidence/breach_2026_001/server_image.dd" \
    ofsz=4294967296 \
    hash=md5,sha1,sha256 \
    log="/evidence/breach_2026_001/imaging.log" \
    rhlog="/evidence/breach_2026_001/src_hash.log" \
    whlog="/evidence/breach_2026_001/dst_hash.log" \
    conv=noerror,sync

# 3. Generate chain of custody document
cat > "/evidence/breach_2026_001/custody.txt" << EOF
CASE: breach_2026_001
DATE: $(date -u +"%Y-%m-%d %H:%M:%S UTC")
OPERATOR: $(whoami)
HOSTNAME: $(hostname)
SOURCE: /dev/sdb (write-blocked)
HASH ALGORITHMS: MD5, SHA-1, SHA-256
TOOL: dc3dd $(dc3dd --version 2>&1 | head -1)
EOF

# 4. Verify image integrity
dc3dd if=/dev/sdb \
    of="/evidence/breach_2026_001/server_image.dd" \
    rhlog="/evidence/breach_2026_001/src_hash.log" \
    whlog="/evidence/breach_2026_001/dst_hash.log" \
    log="/evidence/breach_2026_001/verify.log"
```

**Result**: A forensically sound image with triple-hash verification and chain of custody documentation.

### Scenario 2: Finding Credit Card Numbers on Disk

**Context**: During an investigation, analysts need to locate potential payment card data on a suspect's hard drive.

**Approach**:
```bash
# Luhn-valid PAN pattern (simplified)
# Track 2 format: ;PAN=expiry?
# Hex for track 2 start: 3B (semicolon)
# We search for patterns matching card number formats

# Visa: starts with 4, 13-19 digits
# Mastercard: starts with 51-55, 16 digits
# Amex: starts with 34 or 37, 15 digits

# Search for potential Track 2 data markers
sudo dc3dd if=/dev/sda \
    of=/evidence/cc_search.dd \
    hash=md5 \
    pattern="3B" \
    log=/evidence/cc_search.log

# Cross-reference with bulk-extractor for comprehensive search
bulk-extractor -o /evidence/bulk_output /evidence/cc_search.dd
```

### Scenario 3: Damaged Media Recovery

**Context**: A USB drive that was physically damaged needs imaging. Some sectors may be unreadable.

**Approach**:
```bash
# Use dc3dd with error tolerance
sudo dc3dd if=/dev/sdb \
    of=/evidence/damaged_usb.dd \
    conv=noerror,sync \
    hash=md5,sha256 \
    log=/evidence/damaged.log

# Count errors
grep -c "Input/output error" /evidence/damaged.log

# For severely damaged media, consider ddrescue first for best-effort recovery
ddrescue -f -n /dev/sdb /evidence/rescue_firstpass.log /evidence/rescue.dd
ddrescue -f -d -r3 /dev/sdb /evidence/rescue.dd /evidence/rescue_secondpass.log
```

## Chapter 7: Detection and Defense

### Detecting DC3DD Usage

**Log Analysis**:
```bash
# Check for dc3dd execution in system logs
grep -r "dc3dd" /var/log/syslog* /var/log/auth.log* 2>/dev/null
journalctl | grep dc3dd

# Check process accounting (if enabled)
lastcomm dc3dd
sa -c | grep dc3dd

# Check bash history
grep dc3dd ~/.bash_history
```

**File System Artifacts**:
```bash
# DC3DD log files have distinctive headers
find / -name "*.log" -exec grep -l "dc3dd hash log" {} \; 2>/dev/null

# Hash log files
find / -name "*hash*.log" -newer /etc/hostname 2>/dev/null
```

### Defensive Measures

- Monitor for `dc3dd` in process execution logs (auditd, Sysmon)
- Alert on `if=/dev/sd*` patterns in command-line logging
- Use filesystem monitoring (inotifywait) on `/dev/sd*` to detect imaging
- Implement USB device control policies to prevent unauthorized imaging

### Audit Trail Configuration

```bash
# Add to /etc/audit/rules.d/forensics.rules
-a always,exit -F arch=b64 -S openat -F path=/dev/sd* -F key=forensics_imaging
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/dc3dd -F key=forensics_dc3dd
```

## Chapter 8: Troubleshooting

### Common Issues

| Error | Cause | Solution |
|---|---|---|
| `Permission denied` | Not running as root | Use `sudo` |
| `Input/output error` | Bad sectors | Add `conv=noerror,sync` |
| `No space left on device` | Destination full | Use `ofsz=` to split output |
| `Invalid hash algorithm` | Typo in hash name | Check supported algorithms |
| `Cannot open /dev/sdX` | Device not present | Run `lsblk` to verify |

### Performance Issues

```bash
# If imaging is slow, try larger block sizes
sudo dc3dd if=/dev/sda of=/evidence/image.dd bs=65536 hash=md5

# Check I/O wait
iostat -x 1

# Ensure no other processes are using the disk
iotop -o
```

### Recovery from Interrupted Imaging

```bash
# If imaging was interrupted, you can resume using skip/seek
# Determine how many sectors were written
WRITTEN=$(stat -c%s /evidence/image.dd)
SECTORS_WRITTEN=$((WRITTEN / 512))

echo "Resuming from sector ${SECTORS_WRITTEN}..."

sudo dc3dd if=/dev/sda \
    of=/evidence/image.dd \
    skip="${SECTORS_WRITTEN}" \
    seek="${SECTORS_WRITTEN}" \
    hash=md5 \
    log=/evidence/resume.log
```

### Hash Mismatch Investigation

```bash
# If verification fails:
# 1. Re-run the original imaging
# 2. Check for write-blocking issues
# 3. Verify the source device is consistent
# 4. Check for hardware issues on source or destination

# Compare with independent hash
md5sum /evidence/image.dd
sha256sum /evidence/image.dd
```

## Resources

- **Official Documentation**: https://github.com/resurrecting-open-source-projects/dc3dd
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response
- **DC3 Forensics Lab**: https://www.dc3.mil/
- **ASTM E2763-10**: Standard Practice for Computer Forensics
- **SANS DFIR**: https://www.sans.org/dfir/
- **Kali Linux Forensics Documentation**: https://www.kali.org/docs/forensics/
- **The Sleuth Kit**: https://www.sleuthkit.org/

---

*Last updated: 2026-07-18*
