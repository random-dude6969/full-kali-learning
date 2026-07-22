---
tool_name: ddrescue
display_name: GNU ddrescue
mitre_tactic: collection
mitre_tactic_id: TA0009
install_command: sudo apt install gddrescue
github_repo: https://github.com/Distrotech/ddrescue
kali_tools_page: https://www.kali.org/tools/ddrescue/
difficulty_level: intermediate
tags:
  - forensics
  - disk-imaging
  - data-recovery
  - bad-sectors
  - dfir
related_tools:
  - dc3dd
  - dcfldd
  - guymager
  - photorec
---

# GNU ddrescue — Data Recovery from Damaged Media

## Chapter 1: Overview

GNU ddrescue (also known as gddrescue on Debian/Kali) is a data recovery tool that copies data from one block device to another, minimizing data loss on media with errors. Unlike standard `dd` or forensic imaging tools, ddrescue is specifically designed to handle failing or damaged storage media by using intelligent retry strategies.

### Why ddrescue Exists

When a hard drive, SSD, or optical disc starts failing, standard imaging tools like `dd` or `dc3dd` will stop at the first bad sector. ddrescue takes a fundamentally different approach: it copies all readable sectors first, then goes back to retry bad sectors with increasingly aggressive strategies. This maximizes the amount of data recovered from failing media.

### Core Differentiators

| Feature | dd | dc3dd | ddrescue |
|---|---|---|---|
| Bad sector handling | Stops | Fills zeros | Retries intelligently |
| Retry strategy | None | None | Multi-pass with backoff |
| Mapfile/resume | No | No | Yes (full state persistence) |
| Split output | No | Yes | Yes |
| Direct disc mode | No | No | Yes (bypasses OS cache) |
| Trimming | No | No | Yes (skip bad areas) |
| Forward/backward | No | No | Yes (optimize read order) |
| Speed | Fast (healthy) | Fast (healthy) | Adaptive (damaged media) |

### When to Use ddrescue

- Imaging failing hard drives (clicking, bad sectors, degraded media)
- Recovering data from scratched CDs/DVDs/Blu-rays
- Imaging drives with firmware issues causing intermittent errors
- Any scenario where maximizing recovered data is critical
- First-pass imaging before running file carving tools
- Situations where time-to-first-image matters (emergency triage)

### When NOT to Use ddrescue

- Healthy drives where dc3dd with hash verification is preferred (ddrescue does not compute hashes natively)
- Quick bit-for-bit copies (dd is faster on healthy media)
- When you need on-the-fly hash verification for chain of custody

### The Mapfile: ddrescue's Secret Weapon

The mapfile is ddrescue's most important feature. It records which sectors have been successfully copied, which have failed, and which haven't been attempted yet. This means:

1. If imaging is interrupted (power loss, system crash), ddrescue can resume exactly where it left off
2. Multiple passes can progressively improve recovery without re-copying good sectors
3. The mapfile serves as forensic documentation of the recovery process

### Limitations

- No built-in hash computation (use md5sum/sha256sum afterward)
- Not designed for write-blocking (pair with hardware write blockers for forensics)
- Complex options can be confusing for beginners
- Direct disc mode requires root privileges
- Mapfile must be on a different filesystem than the image output

## Chapter 2: Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install gddrescue
```

Note: The package is named `gddrescue` on Debian/Kali but the binary is `ddrescue`.

### Verify Installation

```bash
ddrescue --version
```

Expected output:
```
ddrescue: version 1.27
```

### Important: Multiple ddrescue Implementations

There are two main implementations:
- **GNU ddrescue** (gddrescue): The one installed on Kali. Version 1.27+. This is the recommended version.
- **dd_rescue** (dtc): A different tool by Kurt Garloff. Avoid confusion.

Always use `ddrescue` (GNU version) for forensics and data recovery.

### From Source

```bash
wget https://ftp.gnu.org/gnu/ddrescue/ddrescue-1.27.tar.gz
tar xzf ddrescue-1.27.tar.gz
cd ddrescue-1.27
./configure
make
sudo make install
```

### Post-Installation Verification

```bash
which ddrescue
ddrescue --help 2>&1 | head -30
```

## Chapter 3: Basic Usage

### Syntax

```
ddrescue [options] <input_file> <output_file> <mapfile>
```

The three positional arguments are mandatory:
1. **input_file**: Source device or image
2. **output_file**: Destination image file
3. **mapfile**: Logfile for resuming (can be `/dev/null` if not needed)

### Core Parameters

| Parameter | Short | Description | Default |
|---|---|---|---|
| `-d` | `-d` | Direct disc access (bypass OS cache) | off |
| `-r<n>` | `-r` | Retry passes on bad sectors (n=-1 for unlimited) | 0 |
| `-b<n>` | `-b` | Block size for retries (power of 2) | 512 |
| `-c<n>` | `-c` | Cluster size (sectors to copy at once) | 128 |
| `-e<n>` | `-e` | Starting error size (clusters) | 256 |
| `-g<n>` | `-g` | Generate mapfile only (don't copy) | off |
| `-G` | `-G` | Mapfile only mode (assume output exists) | off |
| `-f` | `-f` | Force (overwrite output, ignore warnings) | off |
| `-n` | `-n` | No-scrape (skip scraping phase) | off |
| `-N` | `-N` | No-trim (skip trimming phase) | off |
| `-T<n>` | `-T` | Timeout for error areas (seconds) | none |
| `-W` | `-W` | Wipe mapfile (start fresh) | off |
| `-w` | `-w` | Wipe output (fill with zeros before copy) | off |
| `-x<n>` | `-x` | Max retry time per area (seconds) | none |
| `-Y` | `-Y` | Allocate output space upfront | off |
| `-Z<n>` | `-Z` | Minimum read size (bytes) | 1 |

### Phases of ddrescue Operation

ddrescue operates in four distinct phases, each with a specific purpose:

| Phase | Name | Description |
|---|---|---|
| 1 | Trimming | Identify error boundaries (mark bad areas quickly) |
| 2 | Scraping | Read bad areas sector-by-sector with small blocks |
| 3 | Retrying | Retry failed areas with various strategies |
| 4 | Splitting | Copy any remaining fragments between bad areas |

### Sample Commands and Output

#### Basic Image (Healthy Drive)

```bash
sudo ddrescue -d /dev/sda /evidence/sda_image.dd /evidence/sda_mapfile.log
```

Output:
```
GNU ddrescue 1.27
About to copy 466 GiB from /dev/sda to /evidence/sda_image.dd
    Starting positions:源 0 B, destination 0 B
    Block sizes: 1024 B, 128 B
    Direct disc access: yes
     Error sizes: 1.5 MiB, 256 B
       Aligned: yes
       Timeout: none
Max retry time: none
       Pause: none
   Retries finished: 0
         Trimmed: 0/0, 0 B
        Scraped: 0/0, 0 B
          Retried: 0/0, 0 B
        Trimming: 0.00%, 0 B
       Scraping: 0.00%, 0 B
   Retrying failed: 0 B
Splitting failed: 0 B
Copying finished: done.
   Copied: 466 GiB
   Elapsed: 4102.3s
   Speed: 117.2 MB/s
   Errors: 0
```

#### Image Failing Drive (Multiple Passes)

```bash
# Pass 1: Fast copy of all readable data
sudo ddrescue -d -n /dev/sdb /evidence/sdb_image.dd /evidence/sdb_mapfile.log

# Pass 2: Retry bad areas with increasing cluster size
sudo ddrescue -d -r1 /dev/sdb /evidence/sdb_image.dd /evidence/sdb_mapfile.log

# Pass 3: More aggressive retries
sudo ddrescue -d -r2 /dev/sdb /evidence/sdb_image.dd /evidence/sdb_mapfile.log

# Pass 4: Maximum effort
sudo ddrescue -d -r3 /dev/sdb /evidence/sdb_image.dd /evidence/sdb_mapfile.log
```

#### Using Trimming to Identify Bad Areas First

```bash
# Trim first (fast scan of error boundaries)
sudo ddrescue -d -N /dev/sdb /evidence/sdb_image.dd /evidence/sdb_mapfile.log

# Then scrape bad areas
sudo ddrescue -d /dev/sdb /evidence/sdb_image.dd /evidence/sdb_mapfile.log
```

#### Image Scratched DVD

```bash
sudo ddrescue -d -n /dev/sr0 /evidence/dvd_image.iso /evidence/dvd_mapfile.log
sudo ddrescue -d -r3 /dev/sr0 /evidence/dvd_image.iso /evidence/dvd_mapfile.log
```

#### Force Overwrite Existing Image

```bash
sudo ddrescue -f -d /dev/sda /evidence/new_image.dd /evidence/new_mapfile.log
```

#### Wipe Output Before Imaging

```bash
sudo ddrescue -w -d /dev/sda /evidence/image.dd /evidence/mapfile.log
```

#### Allocate Output Space Upfront

```bash
sudo ddrescue -Y -d /dev/sda /evidence/image.dd /evidence/mapfile.log
```

#### Generate Mapfile Without Copying

```bash
# Create mapfile from existing image
sudo ddrescue -g -d /dev/sda /evidence/image.dd /evidence/mapfile.log
```

#### Set Timeout for Bad Areas

```bash
# Give up on each bad area after 30 seconds
sudo ddrescue -d -T30 /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

#### Direct Disc Access (Bypass OS Cache)

```bash
sudo ddrescue -d /dev/sda /evidence/image.dd /evidence/mapfile.log
```

#### Cluster Size Tuning

```bash
# Small clusters for damaged media (slower but more thorough)
sudo ddrescue -d -c64 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Large clusters for fast imaging
sudo ddrescue -d -c1024 /dev/sda /evidence/image.dd /evidence/mapfile.log
```

## Chapter 4: Intermediate Usage and Scripting

### Automated Recovery Script

```bash
#!/bin/bash
# auto_recover.sh — Multi-pass automated data recovery

set -euo pipefail

SOURCE="$1"
OUTPUT="$2"
MAPFILE="${2}.mapfile"
MAX_RETRIES=5

echo "[*] Starting automated recovery: ${SOURCE} -> ${OUTPUT}"

# Pass 1: Fast copy (no retries)
echo "[*] Pass 1: Fast copy of readable data..."
sudo ddrescue -d -n -f "${SOURCE}" "${OUTPUT}" "${MAPFILE}"

# Get initial stats
echo "[*] Initial recovery statistics:"
ddrescue -d -r-1 "${SOURCE}" "${OUTPUT}" "${MAPFILE}" 2>&1 | grep -E "Copied|Errors"

# Passes 2+: Progressive retries
for ((pass=1; pass<=MAX_RETRIES; pass++)); do
    echo "[*] Pass $((pass+1)): Retry pass ${pass}..."
    sudo ddrescue -d -r${pass} -f "${SOURCE}" "${OUTPUT}" "${MAPFILE}"
    
    # Check if we made progress
    STATS=$(ddrescue -d -r-1 "${SOURCE}" "${OUTPUT}" "${MAPFILE}" 2>&1)
    echo "${STATS}" | grep -E "Copied|Errors"
    
    ERRORS=$(echo "${STATS}" | grep "Errors:" | awk '{print $2}')
    if [ "${ERRORS}" = "0" ]; then
        echo "[*] All sectors recovered."
        break
    fi
    
    echo "[*] ${ERRORS} error areas remaining."
done

# Final verification
echo "[*] Final mapfile status:"
ddrescue --show-map "${MAPFILE}"
```

### Recovery with Progress Monitoring

```bash
#!/bin/bash
# monitor_recovery.sh — Monitor ddrescue progress

MAPFILE="$1"
INTERVAL=10

echo "Monitoring recovery progress (Ctrl+C to stop)..."
echo "Time | Copied | Errors | Status"
echo "-----|--------|--------|-------"

while pgrep -x ddrescue > /dev/null; do
    STATS=$(ddrescue --show-map "${MAPFILE}" 2>/dev/null)
    COPIED=$(echo "${STATS}" | grep -oP 'Copied:\s+\K[0-9.]+ [A-Z]+')
    ERRORS=$(echo "${STATS}" | grep -oP 'error size:\s+\K[0-9]+')
    echo "$(date +%H:%M:%S) | ${COPIED} | ${ERRORS} error areas"
    sleep "${INTERVAL}"
done

echo "[*] ddrescue process completed."
```

### Batch Recovery from Multiple Sources

```bash
#!/bin/bash
# batch_recover.sh — Recover from multiple failing drives

declare -A DRIVES
DRIVES[sdb]="/dev/sdb"
DRIVES[sdc]="/dev/sdc"
DRIVES[sdd]="/dev/sdd"

OUTDIR="/evidence/recovery_$(date +%Y%m%d)"
mkdir -p "${OUTDIR}"

for name in "${!DRIVES[@]}"; do
    dev="${DRIVES[$name]}"
    echo "[${name}] Starting recovery from ${dev}"
    
    sudo ddrescue -d -n "${dev}" \
        "${OUTDIR}/${name}_image.dd" \
        "${OUTDIR}/${name}_mapfile.log" &
    
    echo "[${name}] Recovery started in background (PID: $!)"
done

echo "[*] All recovery jobs started. Waiting..."
wait
echo "[*] All recovery jobs complete."
```

### Post-Recovery Hash Verification

```bash
#!/bin/bash
# verify_recovery.sh — Verify recovered images

IMAGE="$1"

echo "[*] Computing hashes for recovered image..."
echo "MD5:    $(md5sum "${IMAGE}" | awk '{print $1}')"
echo "SHA1:   $(sha1sum "${IMAGE}" | awk '{print $1}')"
echo "SHA256: $(sha256sum "${IMAGE}" | awk '{print $1}')"

# Check mapfile for completeness
MAPFILE="${IMAGE}.mapfile"
if [ -f "${MAPFILE}" ]; then
    echo ""
    echo "[*] Mapfile status:"
    ddrescue --show-map "${MAPFILE}"
fi
```

### Analyzing Mapfile

```bash
# View mapfile statistics
ddrescue --show-map /evidence/image.mapfile

# View detailed mapfile
ddrescue --mapfile=/evidence/image.mapfile --show-status

# Generate mapfile from existing image
sudo ddrescue -g /dev/sda /evidence/image.dd /evidence/image.mapfile
```

### Combining ddrescue with File Carving

```bash
#!/bin/bash
# recover_and_carve.sh — Recover data then carve files

SOURCE="/dev/sdb"
OUTPUT="/evidence/recovered.dd"
MAPFILE="/evidence/recovered.mapfile"
CARVED="/evidence/carved"

# Step 1: Recover all readable data
sudo ddrescue -d -n "${SOURCE}" "${OUTPUT}" "${MAPFILE}"
sudo ddrescue -d -r3 "${SOURCE}" "${OUTPUT}" "${MAPFILE}"

# Step 2: Carve files from recovered image
mkdir -p "${CARVED}"
foremost -i "${OUTPUT}" -o "${CARVED}"

# Step 3: Also run photorec for additional recovery
photorec /cmd "${OUTPUT}" fileopt,everything,disable,jpg,enable, search

echo "[*] Recovery and carving complete."
```

## Chapter 5: Advanced Configurations

### Custom Retry Strategies

```bash
# Aggressive: unlimited retries with 1-second timeout per area
sudo ddrescue -d -r-1 -x1 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Conservative: 2 retries, 10-second timeout
sudo ddrescue -d -r2 -x10 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Patient: unlimited retries, no timeout
sudo ddrescue -d -r-1 /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

### Cluster Size Optimization

| Cluster Size | Use Case | Trade-off |
|---|---|---|
| 16 | Severely damaged media | Very slow, maximizes recovery |
| 64 | Damaged media | Slow, good recovery |
| 128 | Moderate damage | Balanced (default) |
| 512 | Light damage | Fast, may miss some data |
| 1024 | Healthy media | Fastest, skip if errors found |

### Direct Disc Access Mode

Direct disc access (`-d`) bypasses the operating system's disk cache and reads directly from the hardware. This is important for:

- Accurate imaging of failing drives
- Avoiding OS-level error handling that might mask issues
- Getting true read error counts
- Preventing OS from marking bad sectors as recovered

```bash
# Always use -d for failing drives
sudo ddrescue -d /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

### Using with Hardware Write Blockers

```bash
# Ensure write blocker is connected
lsblk

# Use ddrescue with direct access through write blocker
sudo ddrescue -d /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

### Mapfile Manipulation

```bash
# View mapfile in human-readable format
ddrescue --show-map /evidence/mapfile.log

# View mapfile as hex dump
xxd /evidence/mapfile.log | head -50

# Copy mapfile to separate media for archival
cp /evidence/mapfile.log /backup/case_mapfile.log
```

### Network-Based Recovery

```bash
# Recover over network via SSH
sudo ddrescue -d -n /dev/sda - | ssh investigator@192.168.1.100 "cat > /evidence/image.dd"

# Recover from network device
sudo ddrescue -d -n ssh://192.168.1.100:/dev/sdb /evidence/image.dd /evidence/mapfile.log
```

### Using with LVM and RAID

```bash
# Image LVM logical volume
sudo ddrescue -d /dev/vg0/lv_root /evidence/lvm_image.dd /evidence/lvm_mapfile.log

# Image RAID array
sudo ddrescue -d /dev/md0 /evidence/raid_image.dd /evidence/raid_mapfile.log
```

## Chapter 6: Real-World Scenarios

### Scenario 1: Failing Hard Drive with Clicking Noise

**Context**: A client's laptop hard drive is making clicking sounds. The drive contains irreplaceable family photos and important documents.

**Approach**:
```bash
# Step 1: Fast first pass — grab all healthy sectors
sudo ddrescue -d -n /dev/sdb /evidence/clicking_drive.dd /evidence/clicking_mapfile.log

# Step 2: Check what we got
echo "[*] Fast pass complete."
ddrescue --show-map /evidence/clicking_mapfile.log

# Step 3: Aggressive retries on bad areas
sudo ddrescue -d -r1 -T30 /dev/sdb /evidence/clicking_drive.dd /evidence/clicking_mapfile.log
sudo ddrescue -d -r2 -T60 /dev/sdb /evidence/clicking_drive.dd /evidence/clicking_mapfile.log
sudo ddrescue -d -r3 -T120 /dev/sdb /evidence/clicking_drive.dd /evidence/clicking_mapfile.log

# Step 4: Final pass with unlimited retries
sudo ddrescue -d -r-1 -x300 /dev/sdb /evidence/clicking_drive.dd /evidence/clicking_mapfile.log

# Step 5: Compute hashes
md5sum /evidence/clicking_drive.dd
sha256sum /evidence/clicking_drive.dd

# Step 6: Mount and extract files
mkdir -p /mnt/recovered
mount -o loop,ro /evidence/clicking_drive.dd /mnt/recovered
cp -r /mnt/recovered/Users/*/Pictures /evidence/photos/
cp -r /mnt/recovered/Users/*/Documents /evidence/documents/
umount /mnt/recovered
```

### Scenario 2: Scratched Installation DVD

**Context**: A software installation DVD is scratched and won't read. The original installer is no longer available.

**Approach**:
```bash
# Identify the optical drive
lsblk | grep sr0

# First pass: fast copy
sudo ddrescue -n /dev/sr0 /evidence/installer.iso /evidence/installer_mapfile.log

# Check progress
ddrescue --show-map /evidence/installer_mapfile.log

# Retry bad areas
sudo ddrescue -r3 /dev/sr0 /evidence/installer.iso /evidence/installer_mapfile.log

# If still errors, try with different block size
sudo ddrescue -r2 -b1024 /dev/sr0 /evidence/installer.iso /evidence/installer_mapfile.log

# Verify ISO integrity
md5sum /evidence/installer.iso
# Compare against known good hash from vendor
```

### Scenario 3: Virtual Machine Disk Recovery

**Context**: A VDI/VMDK virtual machine disk image has become corrupted. Need to recover the virtual machine's data.

**Approach**:
```bash
# Treat the VM disk file as input
sudo ddrescue -n /evidence/corrupted.vmdk /evidence/recovered.vmdk /evidence/vm_mapfile.log

# Retry
sudo ddrescue -r3 /evidence/corrupted.vmdk /evidence/recovered.vmdk /evidence/vm_mapfile.log

# Check if the recovered image is mountable
sudo fsstat /evidence/recovered.vmdk 2>/dev/null

# If NTFS, try mounting with offset
fdisk -l /evidence/recovered.vmdk
# Note the partition offset, then mount
sudo mount -o loop,ro,offset=65536 /evidence/recovered.vmdk /mnt/recovered
```

## Chapter 7: Detection and Defense

### Detecting ddrescue Usage

**Log Analysis**:
```bash
# Check system logs
grep -r "ddrescue" /var/log/syslog* /var/log/auth.log* 2>/dev/null

# Check process accounting
lastcomm ddrescue
sa -c | grep ddrescue

# Check bash history
grep ddrescue ~/.bash_history
grep ddrescue /root/.bash_history
```

**File System Indicators**:
```bash
# Mapfiles have distinctive binary content
find / -name "*.logfile" -newer /etc/hostname 2>/dev/null

# Large .dd files being created
find / -name "*.dd" -size +1G -newer /etc/hostname 2>/dev/null

# ddrescue mapfile magic bytes
find / -type f -exec file {} \; 2>/dev/null | grep -i "ddrescue"
```

**Process Monitoring**:
```bash
# Monitor for ddrescue processes
ps aux | grep ddrescue
top -c | grep ddrescue
```

### Defensive Measures

- Monitor for `ddrescue` execution in audit logs
- Alert on large sequential reads from block devices
- Implement USB device control policies
- Use filesystem monitoring (inotify) on /dev/sd* devices
- Monitor disk I/O patterns for sequential read signatures

### Audit Configuration

```bash
# Add to /etc/audit/rules.d/forensics.rules
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/ddrescue -F key=forensics_ddrescue
-a always,exit -F arch=b64 -S openat -F path=/dev/sd* -F key=forensics_devices
```

## Chapter 8: Troubleshooting

### Common Issues

| Error | Cause | Solution |
|---|---|---|
| `Permission denied` | Not root | Use `sudo` |
| `Mapfile already in use` | Another ddrescue running | Kill other process first |
| `No space left on device` | Output partition full | Use larger destination |
| `Input/output error` | Expected with bad media | Normal; ddrescue handles this |
| `Mapfile not found` | Corrupted or missing | Use `-W` to wipe and restart |

### Performance Issues

```bash
# Check I/O utilization
iostat -x 1 5

# Check for competing processes
iotop -o

# Try direct disc access
sudo ddrescue -d -n /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

### Interrupted Recovery

```bash
# ddrescue can always be resumed using the mapfile
# Simply run the same command again
sudo ddrescue -d /dev/sdb /evidence/image.dd /evidence/mapfile.log

# The mapfile tracks all progress
```

### Corrupted Mapfile

```bash
# If mapfile is corrupted, you can try to recover it
# or start fresh (will re-copy everything)
sudo ddrescue -W -d /dev/sdb /evidence/image.dd /evidence/new_mapfile.log
```

### Drive Keeps Failing

```bash
# Cool down the drive (thermal issues)
# If clicking persists, consider professional clean room recovery
# You can still try to maximize what ddrescue gets:
sudo ddrescue -d -r-1 -x60 -T5 /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

## Resources

- **GNU ddrescue Manual**: https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html
- **GNU ddrescue Homepage**: https://www.gnu.org/software/ddrescue/
- **Data Recovery Guide**: https://www.sleuthkit.org/
- **SANS DFIR**: https://www.sans.org/dfir/
- **Kali Linux Documentation**: https://www.kali.org/docs/

---

*Last updated: 2026-07-18*
