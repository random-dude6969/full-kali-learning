# blkcalc CHEATSHEET - Block Address Calculator

## Quick Reference

```bash
# Convert byte offset to block number
blkcalc image.dd BYTE_OFFSET

# Check block allocation status
blkcalc -v -b BLOCK_SIZE image.dd BYTE_OFFSET

# With file system type specified
blkcalc -f ext2 -b 4096 image.dd 1048576

# For EWF images
blkcalc -t ewf evidence.E01 1048576
```

---

## Common Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-b SIZE` | Block size in bytes | `-b 4096` |
| `-f TYPE` | File system type | `-f ext2` |
| `-t TYPE` | Image type | `-t ewf` |
| `-o OFFSET` | Volume offset (sectors) | `-o 2048` |
| `-v` | Verbose output | `-v` |
| `-V` | Version | `-V` |
| `-h` | Help | `-h` |

---

## File System Types

| Type | Description |
|------|-------------|
| `ext2` | Linux ext2/ext3/ext4 |
| `ntfs` | Windows NTFS |
| `fat` | FAT12/FAT16/FAT32 |
| `hfs` | Mac HFS/HFS+ |
| `ufs` | BSD UFS |

---

## Output Status Values

| Status | Meaning |
|--------|---------|
| `ALLOCATED` | Block in use by file system |
| `UNALLOCATED` | Block is free/available |
| `RESERVED` | Block reserved by file system |

---

## Bash One-Liners

```bash
# Get block number for a byte offset
BLOCK=$(blkcalc image.dd 1048576 | grep -oP '\d+')

# Check if block is unallocated
blkcalc -b 4096 image.dd $((BLOCK * 4096)) | grep -q "UNALLOCATED" && echo "Free"

# Scan range of offsets
for off in $(seq 0 4096 1048576); do blkcalc -b 4096 image.dd $off; done

# Find all unallocated blocks in range
for i in $(seq 0 100); do
    off=$((i * 4096))
    blkcalc -b 4096 image.dd $off 2>/dev/null | grep -q "UNALLOCATED" && echo "Block $i"
done

# Convert and immediately extract
blkcat image.dd $(blkcalc -b 4096 image.dd 1048576 | grep -oP '\d+') > extracted.bin
```

---

## Python Integration

```python
import subprocess, re

def offset_to_block(image, offset, block_size=4096, fs_type="ext2"):
    cmd = ["blkcalc", "-b", str(block_size), "-f", fs_type, image, str(offset)]
    result = subprocess.run(cmd, capture_output=True, text=True)
    match = re.search(r'Block:\s*(\d+)', result.stdout)
    return int(match.group(1)) if match else None

def block_status(image, block, block_size=4096):
    offset = block * block_size
    cmd = ["blkcalc", "-v", "-b", str(block_size), image, str(offset)]
    result = subprocess.run(cmd, capture_output=True, text=True)
    match = re.search(r'(ALLOCATED|UNALLOCATED|RESERVED)', result.stdout)
    return match.group(1) if match else "UNKNOWN"

# Usage
block = offset_to_block("image.dd", 1048576)
status = block_status("image.dd", block)
```

---

## Common Byte Offsets

| Offset | Equivalent |
|--------|------------|
| 1,048,576 | 1 MiB |
| 2,097,152 | 2 MiB |
| 5,242,880 | 5 MiB |
| 10,485,760 | 10 MiB |
| 512 | 1 sector |
| 2048 | 4 sectors |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Unable to open image" | Check file exists and permissions |
| Wrong block number | Verify block size with `fsstat` |
| Wrong file system | Use `-f` to specify type |
| Slow performance | Process specific ranges, not full image |
| Different results than manual calc | Check volume offset with `mmls` |

---

## Typical Workflow

```
1. fsstat image.dd          # Get block size, FS type
2. mmls image.dd            # Get partition layout
3. blkcalc image.dd OFFSET  # Convert offset to block
4. blkcat image.dd BLOCK    # Extract block contents
5. blkls image.dd           # List unallocated blocks
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error (file not found, invalid args) |
| 2 | Memory allocation failure |

---

## See Also

- `blkcat` - Extract block contents
- `blkls` - List unallocated blocks
- `blkstat` - Display block status
- `fsstat` - File system statistics
- `mmls` - Partition layout
