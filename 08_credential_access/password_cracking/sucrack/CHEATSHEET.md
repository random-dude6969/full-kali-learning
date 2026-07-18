# Sucrack Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install sucrack
```

### Basic Commands

#### Dictionary Attack
```bash
sudo sucrack -w /usr/share/wordlists/rockyou.txt
```

#### Target User
```bash
sudo sucrack -u admin -w wordlist.txt
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Wordlist file | `sucrack -w wordlist.txt` |
| `-u` | Username | `sucrack -u admin -w wordlist.txt` |
| `-s` | Sleep time | `sucrack -s 1 -w wordlist.txt` |
| `-v` | Verbose | `sucrack -v wordlist.txt` |

### Usage

#### Basic Brute Force
```bash
sudo sucrack -w /usr/share/wordlists/rockyou.txt
```

#### With Sleep
```bash
sudo sucrack -s 1 -w wordlist.txt
```

### Workflow

1. **Gain** low-privilege access
2. **Run** sucrack
3. **Wait** for password
4. **Escalate** to root

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| Multiple su attempts | Authentication failures |
| High CPU usage | Brute force activity |

### Mitigation

- Strong root password
- Account lockout
- PAM modules

### Troubleshooting

#### Permission Denied
```bash
sudo sucrack -w wordlist.txt
```

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **hydra:** Network login cracker
