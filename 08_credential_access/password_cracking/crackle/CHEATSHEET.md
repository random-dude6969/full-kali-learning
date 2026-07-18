# Crackle Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install crackle
```

### Basic Commands

#### Dictionary Attack
```bash
crackle -d wordlist.txt archive.zip
```

#### Brute Force
```bash
crackle -b archive.zip
```

### Archive Types

| Type | Command |
|------|---------|
| ZIP | `crackle archive.zip` |
| RAR | `crackle archive.rar` |
| 7z | `crackle archive.7z` |

### Attack Options

#### Dictionary
```bash
crackle -d /usr/share/wordlists/rockyou.txt archive.zip
```

#### Brute Force
```bash
crackle -b archive.zip
```

### Output Options

#### Verbose
```bash
crackle -v archive.zip
```

#### Output File
```bash
crackle -o results.txt archive.zip
```

### Workflow

1. **Identify** archive type
2. **Try** dictionary attack
3. **Try** brute force if needed
4. **Extract** with password

### Integration

#### With Fcrackzip
```bash
# Try crackle first
crackle -d wordlist.txt archive.zip

# Then fcrackzip
fcrackzip -u -D -p wordlist.txt archive.zip
```

#### With John
```bash
# Extract hash
zip2john archive.zip > hash.txt

# Crack
john hash.txt
```

### Troubleshooting

#### No Password Found
```bash
# Try larger wordlist
crackle -d /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt archive.zip
```

### Related Tools
- **fcrackzip:** ZIP password cracker
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
