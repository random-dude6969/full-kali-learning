# MyRescue — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic recovery
sudo myrescue /dev/sda image.raw

# With more retries
sudo myrescue -r 5 /dev/sda image.raw

# Specific sector range
sudo myrescue -s 0 -e 2097152 /dev/sda image.raw

# From optical media
sudo myrescue /dev/cdrom image.raw

# Verbose output
sudo myrescue -v /dev/sda image.raw
```

## Installation

```bash
# Kali Linux
sudo apt install myrescue

# From source
git clone https://github.com/thomaswilke/myrescue.git
cd myrescue && make && sudo make install
```

## Key Options

| Option | Description | Default |
|--------|-------------|---------|
| `-b <size>` | Block size in bytes | 512 |
| `-r <n>` | Retries per sector | 3 |
| `-s <sector>` | Start sector | 0 |
| `-e <sector>` | End sector | end of disk |
| `-v` | Verbose output | off |

## Common Scenarios

### Damaged Hard Drive
```bash
sudo myrescue -r 5 -v /dev/sda /evidence/image.raw
```

### Scratched CD/DVD
```bash
sudo myrescue -r 10 /dev/cdrom /evidence/cd_image.raw
```

### Specific Partition
```bash
sudo myrescue -s 2048 -e 1026047 /dev/sda /evidence/partition.raw
```

### Resume Interrupted
```bash
# Just run the same command again
sudo myrescue /dev/sda /evidence/image.raw
```

## Workflow

### Step-by-Step

```
1. Document:    sudo fdisk -l /dev/sda
2. Write block: sudo blockdev --setro /dev/sda
3. Image:       sudo myrescue -v /dev/sda /evidence/raw.img
4. Verify:      ls -lh /evidence/raw.img
5. Hash:        md5sum /evidence/raw.img
6. Analyze:     fls -r -d /evidence/raw.img
```

## Integration

```bash
# With TestDisk
sudo testdisk /evidence/image.raw

# With PhotoRec
sudo photorec /evidence/image.raw

# With The Sleuth Kit
fls -r -d /evidence/image.raw

# With Safecopy (compare results)
sudo safecopy /dev/sda /evidence/safecopy_image.raw
```

## Comparison

| Tool | Best For | Complexity |
|------|----------|------------|
| MyRescue | Simplest recovery | Very low |
| Safecopy | Maximum recovery | Low |
| ddrescue | Advanced recovery | Medium |
| dd | Healthy disks | Very low |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` |
| No space left | Check `df -h` |
| Stuck | Check `ps aux \| grep myrescue` |
| Slow | Increase block size `-b 4096` |
| Incomplete | Run again (resumes) |

## Block Size Guide

| Media Type | Recommended Block Size |
|------------|----------------------|
| Hard drive | 512 (default) |
| SSD | 4096 |
| Optical media | 2048 |
| USB drive | 512 |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
