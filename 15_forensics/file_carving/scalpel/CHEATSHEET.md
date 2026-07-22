# Scalpel — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic carving from disk image
scalpel image.raw -o output/

# Carve with custom configuration
scalpel -c custom.conf image.raw -o output/

# Carve from device (forensics: use image first!)
sudo scalpel /dev/sda -o output/

# Only carved deleted files
scalpel -u image.raw -o output/

# Verbose output
scalpel -v image.raw -o output/

# Skip header validation
scalpel -s image.raw -o output/
```

## Installation

```bash
# Kali Linux (pre-installed)
sudo apt install scalpel

# From source
git clone https://github.com/sleuthkit/scalpel.git
cd scalpel && mkdir build && cd build
cmake .. && make && sudo make install
```

## Configuration

```bash
# Default config location
/etc/scalpel/scalpel.conf

# Copy and customize
cp /etc/scalpel/scalpel.conf ./my.conf

# Config format:
# name  case_sensitive  header_hex  footer_hex  max_size
```

## Common Configs

### JPEG Only
```
jpg  y  0xffd8ff  0xffd9  20000000
```

### All Documents
```
pdf     y  0x25504446    0x2525454f46  50000000
doc     y  0xd0cf11e0    0x00000000    0
docx    y  0x504b0304    0x504b0506    50000000
xlsx    y  0x504b0304    0x504b0506    50000000
```

### All Archives
```
zip     y  0x504b0304  0x504b0506  100000000
rar     y  0x52617221  0x00        0
7z      y  0x377abcaf  0x00        0
```

### All Media
```
jpg     y  0xffd8ff      0xffd9        20000000
png     y  0x89504e47    0xae426082    30000000
mp3     y  0x494433      0x00000000    0
mp4     y  0x00000018    0x00000000    0
```

## Output Analysis

```bash
# View carving stats
cat output/scalpel-output.txt

# Count by type
find output/ -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Verify file types
find output/ -type f -exec file {} \;

# Check sizes
find output/ -type f -exec ls -lh {} \;

# Remove empty/junk files
find output/ -type f -size 0 -delete
find output/ -type f -size -100c -delete
```

## Workflow

### Step-by-Step

```
1. Create image:     dd if=/dev/sda of=raw.img bs=4M
2. Verify hash:      md5sum raw.img /dev/sda
3. Configure:        Edit scalpel.conf for needed types
4. Carve:            scalpel -c my.conf raw.img -o output/
5. Analyze:          Review output/scalpel-output.txt
6. Sort:             Move files to categorized directories
7. Verify:           Use file command on all recovered files
8. Document:         Record all steps and hashes
```

## Integration

```bash
# With The Sleuth Kit
fls -r -d image.raw > tsk_output.txt

# With ExifTool (metadata extraction)
find output/ -name "*.jpg" -exec exiftool {} \;

# With YARA (malware scanning)
yara -r rules/ output/

# With Autopsy
# Import carved files into Autopsy case
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No files carved | Check config, try more file types |
| Permission denied | Use `sudo` |
| Output dir exists | Delete it first: `rm -rf output/` |
| Too many false positives | Enable footer matching |
| Slow performance | Use SSD, limit file types |
| Junk files | Set max_size in config |

## Speed Tips

```bash
# Use faster storage for output
scalpel image.raw -o /ssd/output/

# Only carve needed file types
# (edit config to disable unused types)

# Split large images for parallel processing
split -b 1G image.raw chunk_
for f in chunk_*; do
    scalpel "$f" -o "out_$f/" &
done
wait
```

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
