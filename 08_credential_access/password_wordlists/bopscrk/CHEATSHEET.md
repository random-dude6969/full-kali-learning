# Bopscrk Cheatsheet

## Quick Reference

### Installation
```bash
pip3 install bopscrk
```

### Basic Commands

#### Interactive Mode
```bash
bopscrk -i
```

#### Generate Wordlist
```bash
bopscrk -n "John Doe" -b "1990" -o wordlist.txt
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-n` | Name | `bopscrk -n "John Doe"` |
| `-b` | Birthday | `bopscrk -b "1990"` |
| `-p` | Partner name | `bopscrk -p "Jane"` |
| `-c` | Company | `bopscrk -c "Acme"` |
| `-k` | Keywords | `bopscrk -k "admin,root"` |
| `-o` | Output file | `bopscrk -o wordlist.txt` |
| `-i` | Interactive | `bopscrk -i` |

### Usage

#### Interactive
```bash
bopscrk -i
```

#### Command Line
```bash
bopscrk -n "John Doe" -b "1990-05-15" -p "Jane" -c "Acme" -o wordlist.txt
```

### Workflow

1. **Gather** target info
2. **Run** bopscrk
3. **Input** details
4. **Save** wordlist
5. **Use** with cracking tool

### Integration

#### With Hydra
```bash
# Generate
bopscrk -n "John Doe" -b "1990" -o wordlist.txt

# Use
hydra -l john -P wordlist.txt ssh://192.168.1.100
```

#### With Crunch
```bash
# Generate base
bopscrk -n "John Doe" -o base.txt

# Generate variations
crunch 8 8 -t @@@@%%%% -o variations.txt

# Combine
cat base.txt variations.txt | sort -u > final.txt
```

### Tips

#### Gather Info From
- Social media
- Company website
- Public records
- OSINT tools

### Troubleshooting

#### Module Not Found
```bash
pip3 install -r requirements.txt
```

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **rsmangler:** Wordlist mangler
