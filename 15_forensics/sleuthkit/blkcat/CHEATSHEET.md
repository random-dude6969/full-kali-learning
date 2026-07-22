# blkcat CHEATSHEET - Block Content Extractor

## Quick Reference

```bash
# Extract single block
blkcat image.dd BLOCK_NUMBER > output.bin

# Extract with specific block size
blkcat -b 4096 image.dd 100 > block_100.bin

# Extract range of blocks
blkcat -r 100-200 image.dd > range.bin

# Extract from EWF image
blkcat -t ewf evidence.E01 100 > block.bin

# With metadata
blkcat -e -b 4096 image.dd 100
```

---

## Common Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-b SIZE` | Block size in bytes | `-b 4096` |
| `-f TYPE` | File system type | `-f ext2` |
| `-t TYPE` | Image type | `-t ewf` |
| `-o OFFSET` | Volume offset (sectors) | `-o 2048` |
| `-e` | Include metadata | `-e` |
| `-r RANGE` | Extract block range | `-r 100-200` |
| `-v` | Verbose output | `-v` |
| `-V` | Version | `-V` |
| `-h` | Help | `-h` |

---

## Image Types

| Flag | Format | Extensions |
|------|--------|------------|
| `raw` | Raw disk image | `.dd`, `.raw`, `.img` |
| `ewf` | EnCase Evidence File | `.E01`, `.Ex01` |
| `aff` | Advanced Forensics Format | `.aff`, `.afd` |
| `split` | Split raw image | `.001`, `.002` |

---

## Bash One-Liners

```bash
# Extract and hex dump
blkcat image.dd 100 | xxd

# Extract and search for strings
blkcat image.dd 100 | strings -n 10

# Extract and detect file type
blkcat image.dd 100 | file -

# Extract and calculate hash
blkcat image.dd 100 | sha256sum

# Extract multiple blocks in parallel
parallel -j 4 blkcat image.dd {} ::: $(seq 0 100)

# Extract block and pipe to grep
blkcat image.dd 100 | grep -a "password"

# Extract from unallocated block
blkcat image.dd $(blkls image.dd | head -1) > first_unalloc.bin

# Extract range and concatenate
for i in $(seq 100 200); do blkcat image.dd $i; done > combined.bin
```

---

## Common Workflows

### Recover Deleted File Fragments
```bash
# 1. Find deleted file blocks
fls -r -d image.dd | grep "filename"

# 2. Extract the block
blkcat -b 4096 image.dd BLOCK_NUM > recovered_fragment.bin

# 3. Analyze
strings recovered_fragment.bin
xxd recovered_fragment.bin | head -50
```

### Analyze Unallocated Space
```bash
# 1. List unallocated blocks
blkls image.dd > unallocated_blocks.txt

# 2. Extract first 100 unallocated blocks
head -100 unallocated_blocks.txt | while read block; do
    blkcat image.dd $block > "unalloc_${block}.bin"
done

# 3. Search for interesting content
grep -rl "password" unalloc_*.bin
grep -rl "secret" unalloc_*.bin
```

### Extract and Compare Blocks
```bash
# Extract two blocks
blkcat image.dd 100 > block1.bin
blkcat image.dd 200 > block2.bin

# Compare
diff <(xxd block1.bin) <(xxd block2.bin)
md5sum block1.bin block2.bin
```

---

## Python Integration

```python
import subprocess
import hashlib

def extract_block(image, block, block_size=4096, output_file=None):
    cmd = ["blkcat", "-b", str(block_size), image, str(block)]
    result = subprocess.run(cmd, capture_output=True)
    
    if result.returncode != 0:
        return None
    
    content = result.stdout
    
    if output_file:
        with open(output_file, 'wb') as f:
            f.write(content)
    
    return content

def extract_and_analyze(image, block, block_size=4096):
    content = extract_block(image, block, block_size)
    if not content:
        return None
    
    return {
        'block': block,
        'size': len(content),
        'md5': hashlib.md5(content).hexdigest(),
        'sha256': hashlib.sha256(content).hexdigest(),
        'printable': sum(32 <= b < 127 for b in content) / len(content)
    }

# Usage
block_data = extract_and_analyze("image.dd", 100)
print(block_data)
```

---

## File Signatures to Watch For

| Signature | Hex | File Type |
|-----------|-----|-----------|
| PNG | `89 50 4E 47` | PNG image |
| JPEG | `FF D8 FF` | JPEG image |
| ZIP | `50 4B 03 04` | ZIP archive |
| GZIP | `1F 8B 08` | GZIP compressed |
| PDF | `25 50 44 46` | PDF document |
| ELF | `7F 45 4C 46` | Linux executable |
| PE | `4D 5A` | Windows executable |
| RAR | `52 61 72 21` | RAR archive |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Unable to open image" | Check file path and permissions |
| Empty output | Verify block number with `blkcalc` |
| Wrong content | Check block size with `fsstat` |
| Slow extraction | Use parallel processing |
| Corrupted output | Block may be overwritten |

---

## Size Calculations

| Blocks | Block Size | Total Size |
|--------|------------|------------|
| 1 | 512 B | 512 B |
| 1 | 4096 B | 4 KB |
| 100 | 4096 B | 400 KB |
| 1024 | 4096 B | 4 MB |
| 10000 | 4096 B | 40 MB |

---

## Typical Workflow

```
1. fsstat image.dd          # Get block size
2. mmls image.dd            # Get partition layout
3. fls -r image.dd          # Find files of interest
4. blkcalc image.dd OFFSET  # Find block for offset
5. blkcat image.dd BLOCK    # Extract block contents
6. blkls image.dd           # List unallocated blocks
```

---

## See Also

- `blkcalc` - Convert between byte offsets and block addresses
- `blkls` - List unallocated blocks
- `blkstat` - Display block status
- `icat` - Extract file by inode
- `fls` - List file system entries
