# Hash-Identifier Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install hash-identifier
```

### Basic Commands

#### Start Interactive Mode
```bash
hash-identifier
```

#### Pipe Hash
```bash
echo "5d41402abc4b2a76b9719d911017c592" | hash-identifier
```

### Usage Workflow

1. **Launch** tool
2. **Paste** hash at prompt
3. **View** possible hash types
4. **Note** most likely type
5. **Crack** with hashcat or john

### Interactive Session

```
$ hash-identifier
> 5d41402abc4b2a76b9719d911017c592

Possible Hashes:
[+] MD5
[+] MD4
[+] MD2
```

### Batch Processing

#### From File
```bash
for hash in $(cat hashes.txt); do
    echo "Hash: $hash"
    echo "$hash" | hash-identifier
    echo ""
done
```

#### With Output
```bash
echo "hash" | hash-identifier 2>&1 | grep "Possible"
```

### Common Hash Lengths

| Length | Hash Type |
|--------|-----------|
| 16 | MD5 (partial) |
| 32 | MD5, MD4, MD2 |
| 40 | SHA-1 |
| 56 | SHA-224 |
| 64 | SHA-256 |
| 96 | SHA-384 |
| 128 | SHA-512 |

### Comparison with HashID

| Feature | hash-identifier | hashid |
|---------|-----------------|--------|
| Interactive mode | Yes | No |
| Hashcat modes | No | Yes |
| John formats | No | Yes |
| Batch processing | Limited | Yes |

### Integration

#### With Hashcat
```bash
# Identify
echo "hash" | hash-identifier

# Crack
hashcat -m 0 hash.txt rockyou.txt
```

#### With John
```bash
# Identify
echo "hash" | hash-identifier

# Crack
john --format=raw-md5 hash.txt
```

### Troubleshooting

#### Unknown Hash
```bash
# Try different tool
hashid -e 'hash_string'
```

#### No Output
```bash
# Check hash format
echo "hash" | hash-identifier
```

### Related Tools
- **hashid:** More detailed hash identification
- **hashcat:** Password cracker
- **john:** John the Ripper
