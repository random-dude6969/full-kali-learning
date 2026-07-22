# Safecopy — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic recovery (all 3 stages)
safecopy /dev/sda image.raw

# Stage 1 only (fast read)
safecopy -s /dev/sda image.raw

# Stage 2 only (restricted read)
safecopy -S /dev/sda image.raw

# Stage 3 only (unrestricted read)
safecopy -U /dev/sda image.raw

# With custom block size
safecopy -b 4096 /dev/sda image.raw

# Verbose output
safecopy -l 3 /dev/sda image.raw
```

## Installation

```bash
# Kali Linux (pre-installed)
sudo apt install safecopy

# From source
git clone https://github.com/claunia/safecopy.git
cd safecopy && ./configure && make && sudo make install
```

## Three-Stage Recovery

| Stage | Flag | Description | Speed |
|-------|------|-------------|-------|
| 1 | `-s` / `-1` | Fast read, skips errors | Fast |
| 2 | `-S` / `-2` | Restricted read, retries | Medium |
| 3 | `-U` / `-3` | Unrestricted, maximum retry | Slow |

## Common Scenarios

### Damaged Hard Drive
```bash
# Run all stages for maximum recovery
sudo safecopy /dev/sda /evidence/image.raw

# Check results
grep "STATS" /evidence/image.raw.log
```

### Scratched CD/DVD
```bash
sudo safecopy /dev/cdrom /evidence/cd_image.raw
```

### USB Drive / SD Card
```bash
sudo safecopy /dev/sdb /evidence/usb_image.raw
```

### Specific Partition
```bash
# Offset in bytes, size in bytes
sudo safecopy -o $((2048*512)) -l $((100*1024*1024*1024)) /dev/sda /evidence/part.raw
```

## Key Options

| Option | Description |
|--------|-------------|
| `-s` / `-1` | Stage 1 only |
| `-S` / `-2` | Stage 2 only |
| `-U` / `-3` | Stage 3 only |
| `-b <size>` | Block size (default 512) |
| `-o <offset>` | Start offset in source |
| `-l <length>` | Length to copy |
| `-l <level>` | Log verbosity (0-3) |
| `-f` | Force overwrite destination |

## Output Files

```
image.raw        # Recovered disk image
image.raw.log    # Recovery log with statistics
```

## Log Analysis

```bash
# Count errors
grep "\[ERROR\]" image.raw.log | wc -l

# View recovery statistics
grep "STATS" image.raw.log

# Find most damaged areas
grep "\[ERROR\]" image.raw.log | awk '{print $4}' | sort -n
```

## Workflow

### Step-by-Step

```
1. Document:    sudo fdisk -l /dev/sda
2. Write block: sudo blockdev --setro /dev/sda
3. Image:       sudo safecopy /dev/sda /evidence/raw.img
4. Log:         cat /evidence/raw.img.log
5. Analyze:     fls -r -d /evidence/raw.img
6. Hash:        md5sum /evidence/raw.img
```

## Integration

```bash
# With TestDisk
sudo testdisk /evidence/image.raw

# With PhotoRec
sudo photorec /evidence/image.raw

# With The Sleuth Kit
fls -r -d /evidence/image.raw
icat /evidence/image.raw inode > recovered_file

# With Scalpel
scalpel /evidence/image.raw -o /evidence/carved/
```

## Comparison

| Tool | Best For | Error Handling |
|------|----------|----------------|
| Safecopy | Damaged media | Excellent |
| dd | Healthy media | Poor |
| ddrescue | Damaged media | Excellent |
| dc3dd | Forensics | Good |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` |
| No space left | Check `df -h` |
| Stuck on read | Expected with errors; check log |
| Slow recovery | Use larger block size `-b 4096` |
| Incomplete image | Run again; safecopy resumes |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
