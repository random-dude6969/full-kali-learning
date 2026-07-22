# Binwalk — Quick Reference Cheatsheet

## Essential Commands

```bash
# Scan firmware
binwalk firmware.bin

# Extract identified files
binwalk -e firmware.bin

# Recursive extraction
binwalk -M firmware.bin

# Extract to specific directory
binwalk -e -C output/ firmware.bin

# Hex dump
binwalk -hex firmware.bin

# Scan specific length
binwalk -l 1024 firmware.bin

# Scan from offset
binwalk -o 1000 firmware.bin
```

## Installation

```bash
# Kali Linux (pre-installed)
sudo apt install binwalk

# From source
git clone https://github.com/ReFirmLabs/binwalk.git
cd binwalk && pip3 install .

# Verify
binwalk --version
```

## Key Options

| Option | Description |
|--------|-------------|
| `-e` | Extract identified files |
| `-M` | Recursive extraction |
| `-C <dir>` | Output directory |
| `-hex` | Display hex dump |
| `-l <len>` | Number of bytes to analyze |
| `-o <offset>` | Start at offset |
| `-q` | Quiet mode |
| `-x <type>` | Exclude file type |

## Supported Formats

| Category | Formats |
|----------|---------|
| File Systems | squashfs, cramfs, JFFS2, UBIFS, ext2/3/4 |
| Archives | ZIP, RAR, 7Z, TAR.GZ, TAR.BZ2 |
| Executables | ELF, PE, Mach-O |
| Boot Loaders | U-Boot, GRUB, LILO |
| Firmware | ASUS, Cisco, Linksys, Netgear |

## Common Scenarios

### Quick Firmware Scan
```bash
binwalk firmware.bin
```

### Complete Firmware Extraction
```bash
binwalk -e -M -C output/ firmware.bin
```

### IoT Device Analysis
```bash
binwalk -e -M router_firmware.bin
find output/ -name "*.conf" -exec grep -l "password" {} \;
```

### Malware Analysis
```bash
binwalk -e suspicious.bin
find output/ -name "*.sh" -o -name "*.py"
```

## Workflow

### Firmware Analysis

```
1. Hash:      md5sum firmware.bin
2. Scan:      binwalk firmware.bin
3. Extract:   binwalk -e -M -C output/ firmware.bin
4. Analyze:   Review extracted files
5. Search:    Find credentials, configs
6. Report:    Document findings
```

## Output Structure

```
output/
├── squashfs-root/       # Extracted file system
│   ├── bin/
│   ├── etc/
│   ├── lib/
│   └── ...
├── _firmware.bin.extracted/
│   ├── 0/
│   └── ...
└── ...
```

## Integration

```bash
# With strings (string extraction)
strings firmware.bin | head -50

# With hexwalk (hex analysis)
hexwalk -l 64 firmware.bin

# With YARA (malware scanning)
yara -r rules/ output/

# With The Sleuth Kit (file system analysis)
fls -r -d output/squashfs-root/
```

## Finding Secrets

```bash
# Find passwords
grep -r "password" output/ 2>/dev/null

# Find configurations
find output/ -name "*.conf" -o -name "*.cfg"

# Find web interfaces
find output/ -name "*.html" -o -name "*.php"

# Find certificates
find output/ -name "*.pem" -o -name "*.crt"
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` |
| No files extracted | Check file type, try `-M` |
| Extraction fails | Install squashfs-tools, etc. |
| Disk full | Use selective extraction |
| Unknown format | Check binwalk signature database |

## Dependencies

```bash
# Install extraction tools
sudo apt install squashfs-tools cramfs-tools mtd-utils p7zip-full unzip

# For disassembly
sudo apt install binutils
```

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
