# John the Ripper Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install john
```

### Basic Commands

#### Wordlist Attack
```bash
john --wordlist=rockyou.txt hashes.txt
```

#### Show Cracked
```bash
john --show hashes.txt
```

#### List Formats
```bash
john --list=formats
```

### Attack Modes

#### Single Crack
```bash
john --single hashes.txt
```

#### Wordlist
```bash
john --wordlist=rockyou.txt hashes.txt
```

#### Brute-force
```bash
john --incremental hashes.txt
```

#### Mask Attack
```bash
john --mask="?a?a?a?a?a?a" hashes.txt
```

### Rules

#### Enable Rules
```bash
john --wordlist=rockyou.txt --rules hashes.txt
```

#### Specific Rules
```bash
john --wordlist=rockyou.txt --rules=single hashes.txt
```

#### List Rules
```bash
john --list-rules
```

### Hash Formats

| Format | Flag |
|--------|------|
| MD5 | `--format=raw-md5` |
| SHA-1 | `--format=raw-sha1` |
| NTLM | `--format=nt` |
| bcrypt | `--format=bcrypt` |
| Kerberos | `--format=krb5-tgs` |

### Session Management

#### Start Session
```bash
john --session=crack1 --wordlist=rockyou.txt hashes.txt
```

#### Check Status
```bash
john --status=crack1
```

#### Restore
```bash
john --restore=crack1
```

### Mask Attacks

#### Placeholders
```bash
# ?a = all printable
# ?l = lowercase
# ?u = uppercase
# ?d = digits
# ?s = symbols
john --mask="?a?a?a?a?a?a" hashes.txt
```

### Hash Extraction

#### Linux
```bash
unshadow /etc/passwd /etc/shadow > hashes.txt
```

#### Windows
```bash
secretsdump.py user:pass@target > hashes.txt
```

### Performance

#### Fork Processes
```bash
john --fork=8 --wordlist=rockyou.txt hashes.txt
```

#### GPU
```bash
john --format=raw-md5-opencl hashes.txt
```

### Pot File

#### View
```bash
cat ~/.john/john.pot
```

#### Reset
```bash
rm ~/.john/john.pot
```

### Troubleshooting

#### No Hashes Loaded
```bash
john --list=formats | grep md5
john --format=raw-md5 hashes.txt
```

#### Unknown Format
```bash
john hashes.txt  # Auto-detect
hashid 'hash'    # Identify
```

### Related Tools
- **hashcat:** GPU password cracker
- **johnny:** GUI for John
- **hashid:** Hash identifier
