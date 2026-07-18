# MFOC Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install mfoc
```

### Basic Commands

#### Dump Card
```bash
mfoc -r card.dump
```

#### With Known Keys
```bash
mfoc -k keys.txt -r card.dump
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Output file | `mfoc -r card.dump` |
| `-k` | Key file | `mfoc -k keys.txt` |
| `-O` | Override | `mfoc -O card.dump` |
| `-P` | Partial dump | `mfoc -P card.dump` |
| `-t` | Target key | `mfoc -t 0xFFFFFFFFFFFF` |
| `-b` | Block range | `mfoc -b 0-63` |
| `-v` | Verbose | `mfoc -v` |

### Usage

#### Basic Dump
```bash
mfoc -r card.dump
```

#### With Recovered Keys
```bash
mfoc -k recovered_keys.txt -r card.dump
```

#### Partial Dump
```bash
mfoc -P -b 0-15 card.dump
```

### Workflow

1. **Connect** NFC reader
2. **Place** card on reader
3. **Run** mfoc
4. **Save** dump file
5. **Analyze** dump

### Integration

#### With MFCUK
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt

# Dump card
mfoc -k keys.txt -r card.dump
```

#### With NFC-Utils
```bash
# List card
nfc-list

# Dump card
mfoc -r card.dump
```

### Mifare Classic Structure

| Element | Description |
|---------|-------------|
| 16 Sectors | Memory divisions |
| 4 Blocks/Sector | Data units |
| 1 Key/Block | Access control |

### Troubleshooting

#### No Reader Found
```bash
pcscd status
nfc-list
```

#### Wrong Key
```bash
# Recover keys first
mfcuk -C -R -v 3 -o keys.txt

# Use recovered keys
mfoc -k keys.txt -r card.dump
```

### Related Tools
- **mfcuk:** Key recovery
- **libnfc:** NFC library
- **mfterm:** Mifare terminal
