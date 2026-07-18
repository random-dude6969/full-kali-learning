# Fcrackzip Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install fcrackzip
```

### Basic Commands

#### Dictionary Attack
```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt archive.zip
```

#### Brute Force (Letters)
```bash
fcrackzip -u -b -c a -l 1-6 archive.zip
```

#### Brute Force (Numbers)
```bash
fcrackzip -u -b -c 1 -l 1-6 archive.zip
```

### Attack Options

#### Dictionary
```bash
fcrackzip -u -D -p wordlist.txt archive.zip
```

#### Brute Force
```bash
fcrackzip -u -b -c aA1 -l 1-6 archive.zip
```

### Charset Options

| Charset | Description | Example |
|---------|-------------|---------|
| `a` | Lowercase | `fcrackzip -c a archive.zip` |
| `A` | Uppercase | `fcrackzip -c A archive.zip` |
| `1` | Numbers | `fcrackzip -c 1 archive.zip` |
| `!` | Symbols | `fcrackzip -c ! archive.zip` |
| `aA1!` | All | `fcrackzip -c aA1! archive.zip` |

### Length Options

#### Specific Length
```bash
fcrackzip -u -b -c a -l 4-4 archive.zip
```

#### Range
```bash
fcrackzip -u -b -c aA1 -l 4-8 archive.zip
```

### Output Options

#### Verbose
```bash
fcrackzip -v archive.zip
```

#### Unzip on Success
```bash
fcrackzip -u archive.zip
```

### Workflow

1. **Try** dictionary attack first
2. **Try** brute force if needed
3. **Note** password
4. **Extract** files

### Integration

#### With John
```bash
# Extract hash
zip2john archive.zip > hash.txt

# Crack
john hash.txt
```

#### With Hashcat
```bash
# Extract hash
zip2john archive.zip > hash.txt

# Crack
hashcat -m 17200 hash.txt rockyou.txt
```

### Troubleshooting

#### File Not Found
```bash
ls -la archive.zip
fcrackzip -u -D -p wordlist.txt archive.zip
```

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **zip2john:** ZIP hash extractor
