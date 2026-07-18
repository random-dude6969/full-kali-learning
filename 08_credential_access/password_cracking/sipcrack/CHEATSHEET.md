# Sipcrack Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install sipcrack
```

### Basic Commands

#### Dictionary Attack
```bash
sipcrack -w wordlist.txt hashes.txt
```

#### Login Mode
```bash
sipcrack -l hashes.txt
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Wordlist file | `sipcrack -w wordlist.txt hashes.txt` |
| `-l` | Login mode | `sipcrack -l hashes.txt` |
| `-v` | Verbose | `sipcrack -v hashes.txt` |
| `-o` | Output | `sipcrack -o results.txt hashes.txt` |

### Usage

#### Dictionary Attack
```bash
sipcrack -w /usr/share/wordlists/rockyou.txt hashes.txt
```

#### With Custom Wordlist
```bash
sipcrack -w custom_wordlist.txt hashes.txt
```

### Workflow

1. **Capture** SIP authentication
2. **Save** hashes to file
3. **Run** sipcrack
4. **Use** recovered password

### SIP Authentication

#### Digest Authentication
- Uses MD5 hashing
- Challenge-response protocol
- Common in VoIP systems

### Troubleshooting

#### Invalid Hash Format
```bash
# Check hash format
head -1 hashes.txt

# Ensure proper SIP digest format
```

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **hydra:** Network login cracker
