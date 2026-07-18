# PACK Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install pack
```

### Basic Commands

#### Analyze Hashes
```bash
pack -i hashes.txt
```

#### Generate Masks
```bash
pack -i hashes.txt -o masks.txt
```

#### Top Masks
```bash
pack -i hashes.txt -t 10
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Input file | `pack -i hashes.txt` |
| `-o` | Output file | `pack -i hashes.txt -o masks.txt` |
| `-t` | Top masks | `pack -i hashes.txt -t 10` |
| `-v` | Verbose | `pack -i hashes.txt -v` |

### Usage

#### Basic Analysis
```bash
pack -i hashes.txt
```

#### Generate Hashcat Masks
```bash
pack -i hashes.txt -t 10 -o masks.txt
```

#### With Hashcat
```bash
# Generate masks
pack -i hashes.txt -o masks.txt

# Use with hashcat
hashcat -a 3 -m 0 hashes.txt masks.txt
```

### Workflow

1. **Collect** hashes
2. **Analyze** patterns
3. **Generate** masks
4. **Crack** with hashcat

### Integration

#### With Hashcat
```bash
pack -i hashes.txt -o masks.txt
hashcat -a 3 -m 0 hashes.txt masks.txt
```

### Troubleshooting

#### No Hashes Loaded
```bash
head -1 hashes.txt
pack -i hashes.txt -m 0
```

### Related Tools
- **hashcat:** Advanced password recovery
- **john:** John the Ripper
- **crunch:** Wordlist generator
