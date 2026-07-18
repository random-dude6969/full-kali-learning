# HashID Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install hashid
```

### Basic Commands

#### Identify Hash
```bash
hashid '5d41402abc4b2a76b9719d911017c592'
```

#### Show Hashcat Modes
```bash
hashid -m 'hash_string'
```

#### Show John Formats
```bash
hashid -j 'hash_string'
```

#### Extended Output
```bash
hashid -e 'hash_string'
```

#### Force Identification
```bash
hashid -f 'hash_string'
```

### Common Hash Types

| Hash | Example | Hashcat Mode |
|------|---------|--------------|
| MD5 | `5d41402abc4b2a76b9719d911017c592` | 0 |
| SHA-1 | `aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d` | 100 |
| SHA-256 | `2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c...` | 1400 |
| SHA-512 | `cf83e1357eefb8bdf1542850d66d8007d620e405...` | 1700 |
| NTLM | `aad3b435b51404eeaad3b435b51404ee` | 1000 |
| bcrypt | `$2a$10$...` | 3200 |

### File Operations

#### From File
```bash
hashid -m -f hashes.txt
```

#### Output to File
```bash
hashid -m -o results.txt 'hash_string'
```

#### Raw Output (No Colors)
```bash
hashid -r -m 'hash_string'
```

### Scripting

#### Bash One-Liner
```bash
for hash in $(cat hashes.txt); do hashid -m "$hash"; done
```

#### With Hashcat
```bash
hashid -m 'hash_string' && hashcat -m [mode] 'hash_string' rockyou.txt
```

### Flags Reference

| Flag | Description |
|------|-------------|
| `-m` | Show hashcat modes |
| `-j` | Show John formats |
| `-e` | Extended output |
| `-f` | Force identification |
| `-o` | Output file |
| `-r` | Raw output (no colors) |

### Workflow

1. **Identify** hash type with hashid
2. **Note** hashcat mode or john format
3. **Crack** with appropriate tool

### Common Patterns

| Length | Possible Types |
|--------|----------------|
| 32 | MD5 |
| 40 | SHA-1 |
| 64 | SHA-256 |
| 128 | SHA-512 |
| 34 | bcrypt |

### Troubleshooting

#### Unknown Hash
```bash
# Try extended mode
hashid -e 'hash_string'

# Try force mode
hashid -f 'hash_string'
```

#### Multiple Matches
```bash
# Use hashcat modes for precision
hashid -m 'hash_string'
```

### Related Tools
- **hash-identifier:** Alternative hash identifier
- **hashcat:** Password cracker
- **john:** John the Ripper
