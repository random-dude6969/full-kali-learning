# Foremost — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic recovery
foremost -i image.raw -o output/

# From device
sudo foremost -i /dev/sda -o output/

# Specific file types
foremost -i image.raw -o output/ -t jpg,png,pdf

# Custom configuration
foremost -i image.raw -o output/ -c my.conf

# Audit mode (see what would be carved)
foremost -i image.raw -o output/ -T

# Verbose output
foremost -i image.raw -o output/ -v
```

## Installation

```bash
# Kali Linux (pre-installed)
sudo apt install foremost

# From source
git clone https://github.com/korczis/foremost.git
cd foremost && make && sudo make install

# Verify
foremost -v
```

## Configuration File

```bash
# Default location
/etc/foremost.conf

# Copy and customize
cp /etc/foremost.conf ./my.conf

# Format:
# extension  case_sensitive  header  footer  max_size
```

## Common Configs

### JPEG Only
```
jpg     y  0xffd8ff  0xffd9  20000000
```

### All Images
```
jpg     y  0xffd8ff      0xffd9        20000000
png     y  0x89504e47    0xae426082    30000000
gif     y  0x47494638    0x3b          30000000
bmp     y  0x424d        0xffffffff    0
```

### All Documents
```
pdf     y  0x25504446    0x2525454f46  50000000
doc     y  0xd0cf11e0    0x00000000    0
docx    y  0x504b0304    0x504b0506    50000000
```

### All Archives
```
zip     y  0x504b0304  0x504b0506  100000000
rar     y  0x52617221  0x00        0
7z      y  0x377abcaf  0x00        0
```

## Key Options

| Option | Description |
|--------|-------------|
| `-i <file>` | Input image/device |
| `-o <dir>` | Output directory |
| `-c <file>` | Configuration file |
| `-t <types>` | File types to recover |
| `-T` | Audit mode (dry run) |
| `-v` | Verbose output |
| `-s` | Skip header validation |

## Output Structure

```
output/
├── jpg/
│   ├── 00000000.jpg
│   └── ...
├── png/
├── pdf/
├── audit.txt      # Detailed recovery log
└── filetypes.txt  # File type statistics
```

## Common Scenarios

### Recover from Formatted Drive
```bash
foremost -i /dev/sda1 -o output/
```

### Recover Only Images
```bash
foremost -i image.raw -o output/ -t jpg,png,gif
```

### Test Configuration
```bash
# Audit mode first
foremost -i image.raw -o output/ -T
cat output/audit.txt
```

### From E01 Image
```bash
sudo ewfmount evidence.E01 /mnt/ewf
foremost -i /mnt/ewf/ewf1 -o output/
sudo umount /mnt/ewf
```

## Workflow

### Forensic Recovery

```
1. Document:    sudo fdisk -l /dev/sda
2. Write block: sudo blockdev --setro /dev/sda
3. Image:       sudo dd if=/dev/sda of=raw.img bs=4M
4. Hash:        md5sum raw.img
5. Audit:       foremost -i raw.img -o output/ -T
6. Recover:     foremost -i raw.img -o output/
7. Analyze:     cat output/audit.txt
8. Sort:        Move files to categorized directories
9. Verify:      find output/ -type f -exec file {} \;
10. Hash files: find output/ -type f -exec md5sum {} \;
```

## Integration

```bash
# With TestDisk
sudo testdisk /evidence/original.raw

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
| No files recovered | Check config, try more types |
| Permission denied | Use `sudo` |
| Output dir exists | Delete it first |
| Too many false positives | Enable footer matching |
| Slow recovery | Limit file types |

## Comparison

| Tool | Best For | Audit Mode |
|------|----------|------------|
| Foremost | Simple, reliable | Yes |
| Scalpel | Fast, configurable | No |
| PhotoRec | Maximum types | No |
| RecoverDM | Quick recovery | No |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
