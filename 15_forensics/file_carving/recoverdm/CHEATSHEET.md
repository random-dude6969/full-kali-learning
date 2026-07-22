# RecoverDM — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic file recovery
sudo recoverdm -d /dev/sda -o output/

# Recover JPEG files only
sudo recoverdm -d /dev/sda -t jpg -o output/

# Recover from disk image
recoverdm -d image.raw -o output/

# Scan specific sector range
sudo recoverdm -d /dev/sda -s 0 -e 1000000 -o output/

# Verbose mode
sudo recoverdm -d /dev/sda -o output/ -v
```

## Installation

```bash
# Kali Linux
sudo apt install recoverdm

# From source
git clone https://github.com/Gnuckies/recoverdm.git
cd recoverdm && make && sudo make install
```

## Supported File Types

| Type | Header (Hex) | Flag |
|------|--------------|------|
| JPEG | `FF D8 FF E0` | `-t jpg` |
| PNG | `89 50 4E 47` | `-t png` |
| GIF | `47 49 46 38` | `-t gif` |
| PDF | `25 50 44 46` | `-t pdf` |
| ZIP | `50 4B 03 04` | `-t zip` |
| RAR | `52 61 72 21` | `-t rar` |
| MP3 | `49 44 33` | `-t mp3` |
| MP4 | `00 00 00 18` | `-t mp4` |

## Common Scenarios

### Recover from Formatted Drive
```bash
sudo recoverdm -d /dev/sda1 -o /evidence/formatted/
```

### Recover from Specific Location
```bash
sudo recoverdm -d /dev/sda -s 1000000 -e 2000000 -o /evidence/specific/
```

### Batch Recovery
```bash
for type in jpg png gif pdf zip; do
    sudo recoverdm -d /dev/sda -t $type -o output/$type/
done
```

## Key Options

| Option | Description |
|--------|-------------|
| `-d` | Device or image to scan |
| `-o` | Output directory |
| `-t` | File type to recover |
| `-s` | Start sector |
| `-e` | End sector |
| `-v` | Verbose output |
| `-h` | Help |

## Workflow

### Step-by-Step

```
1. Document:    sudo fdisk -l /dev/sda
2. Image:       sudo dd if=/dev/sda of=raw.img bs=4M
3. Hash:        md5sum raw.img
4. Recover:     sudo recoverdm -d raw.img -o output/
5. Verify:      find output/ -type f -exec file {} \;
6. Sort:        Move files to categorized directories
7. Hash files:  find output/ -type f -exec md5sum {} \;
8. Report:      Document all steps
```

## Integration

```bash
# With The Sleuth Kit
fls -r -d image.raw > tsk_output.txt

# With Scalpel (additional recovery)
scalpel image.raw -o scalpel_output/

# With ExifTool (metadata)
find output/ -name "*.jpg" -exec exiftool {} \;

# With YARA (malware scanning)
yara -r rules/ output/
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` |
| No files recovered | Try more file types, scan different sectors |
| Device not found | Check `lsblk` |
| Slow scan | Scan specific sectors, use disk image |
| False positives | Verify with `file` command |

## Comparison

| Tool | Best For | File Types |
|------|----------|------------|
| RecoverDM | Quick, simple recovery | Few |
| Scalpel | Bulk recovery, many types | Many |
| PhotoRec | Maximum recovery | Many |
| Foremost | Simple configuration | Medium |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
