# libfreefare Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install libfreefare-bin
```

### Available Tools

| Tool | Description |
|------|-------------|
| `mifare-classic-format` | Format Classic cards |
| `mifare-desfire-info` | DESFire card info |
| `mifare-desfire-create-ndef` | Create NDEF data |

### Basic Commands

#### Format Mifare Classic
```bash
mifare-classic-format
```

#### DESFire Info
```bash
mifare-desfire-info
```

#### Create NDEF Text
```bash
mifare-desfire-create-ndef -t "Hello World"
```

#### Create NDEF URL
```bash
mifare-desfire-create-ndef -u "http://example.com"
```

### Card Types

| Type | Tool |
|------|------|
| Mifare Classic | `mifare-classic-format` |
| Mifare Ultralight | Use libnfc |
| Mifare DESFire | `mifare-desfire-info` |

### Workflow

1. **Connect** NFC reader
2. **Place** card on reader
3. **Run** appropriate tool
4. **Follow** prompts

### Integration

#### With libnfc
```bash
# List cards
nfc-list

# Format
mifare-classic-format

# Read
nfc-mfclassic r b card.dump
```

### Troubleshooting

#### No Card Found
```bash
# Place card on reader
nfc-list
```

### Related Tools
- **libnfc:** NFC library
- **mfcuk:** Key recovery
- **mfoc:** Card dump
