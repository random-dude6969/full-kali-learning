# TrueCrack Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install truecrack
```

### Basic Commands

#### Dictionary Attack
```bash
truecrack -t volume.tc -w wordlist.txt
```

#### Brute Force
```bash
truecrack -t volume.tc -b -l 8
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target volume | `truecrack -t volume.tc` |
| `-w` | Wordlist file | `truecrack -t volume.tc -w wordlist.txt` |
| `-b` | Brute force | `truecrack -t volume.tc -b` |
| `-l` | Password length | `truecrack -t volume.tc -b -l 8` |
| `-k` | Keyfile | `truecrack -t volume.tc -k keyfile.key` |
| `-g` | GPU | `truecrack -t volume.tc -g` |
| `-v` | Verbose | `truecrack -v volume.tc` |

### Usage

#### Dictionary Attack
```bash
truecrack -t volume.tc -w /usr/share/wordlists/rockyou.txt
```

#### Brute Force (8 chars)
```bash
truecrack -t volume.tc -b -l 8
```

#### With Keyfile
```bash
truecrack -t volume.tc -w wordlist.txt -k keyfile.key
```

### Workflow

1. **Find** TrueCrypt volume
2. **Run** truecrack
3. **Wait** for password
4. **Mount** volume

### GPU Acceleration

#### Check GPU Support
```bash
truecrack --help | grep gpu
```

#### Enable GPU
```bash
truecrack -t volume.tc -w wordlist.txt -g
```

### Troubleshooting

#### No GPU Found
```bash
# Use CPU mode
truecrack -t volume.tc -w wordlist.txt
```

### Related Tools
- **hashcat:** Advanced password recovery
- **john:** John the Ripper
- **veracrypt:** TrueCrypt successor
