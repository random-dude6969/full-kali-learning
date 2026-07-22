# MagicRescue — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic recovery
sudo magicrescue -d /output/ /dev/sda

# With recipe directory
sudo magicrescue -d /output/ -r /usr/share/magicrescue/recipes/ /dev/sda

# Single recipe
sudo magicrescue -d /output/ -R /path/to/jpeg.recipe /dev/sda

# From disk image
magicrescue -d /output/ image.raw

# Specific offset and length
sudo magicrescue -d /output/ -s 0 -l 1G /dev/sda
```

## Installation

```bash
# Kali Linux
sudo apt install magicrescue

# From source
git clone https://github.com/jbj/magicrescue.git
cd magicrescue && make && sudo make install
```

## Recipe Format

```
# name    header    footer    max_size
jpeg      ffd8ffe0  ffd9      20000000
png       89504e47  ae426082  30000000
pdf       25504446  2525454f46 50000000
zip       504b0304  504b0506  100000000
```

## Common Recipes

### JPEG Only
```
jpeg    ffd8ffe0    ffd9    20000000
```

### All Images
```
jpeg    ffd8ffe0      ffd9        20000000
png     89504e47      ae426082    30000000
gif     47494638      3b          30000000
bmp     424d          ffffff      0
```

### All Documents
```
pdf     25504446      2525454f46  50000000
doc     d0cf11e0      00000000    0
docx    504b0304      504b0506    50000000
```

## Key Options

| Option | Description |
|--------|-------------|
| `-d <dir>` | Output directory |
| `-r <dir>` | Recipe directory |
| `-R <file>` | Single recipe file |
| `-s <offset>` | Start offset |
| `-l <size>` | Length to scan |
| `-v` | Verbose output |

## Common Scenarios

### Recover from Formatted Drive
```bash
sudo magicrescue -d /output/ /dev/sda1
```

### Recover Only JPEG Files
```bash
sudo magicrescue -d /output/ -R jpeg.recipe /dev/sda
```

### Batch Process Multiple Devices
```bash
for dev in /dev/sd{a,b,c}; do
    sudo magicrescue -d "output_$(basename $dev)" "$dev"
done
```

## Workflow

### Step-by-Step

```
1. Document:    sudo fdisk -l /dev/sda
2. Write block: sudo blockdev --setro /dev/sda
3. Image:       sudo dd if=/dev/sda of=raw.img bs=4M
4. Hash:        md5sum raw.img
5. Recover:     sudo magicrescue -d output/ raw.img
6. Verify:      find output/ -type f -exec file {} \;
7. Sort:        Move files to categorized directories
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
| No files recovered | Try different recipes, check device |
| Permission denied | Use `sudo` |
| Recipe not found | Check `/usr/share/magicrescue/recipes/` |
| Slow scan | Scan specific offset/length |
| False positives | Verify with `file` command |

## Recipe Locations

```bash
# System recipes
/usr/share/magicrescue/recipes/

# List available recipes
ls /usr/share/magicrescue/recipes/

# Create custom recipe
cat > my.recipe << 'EOF'
myformat    4d59464d    00000000    0
EOF
```

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
