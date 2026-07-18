# libnfc Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install libnfc-utils libnfc-bin
```

### Available Tools

| Tool | Description |
|------|-------------|
| `nfc-list` | List NFC devices |
| `nfc-mfclassic` | Mifare Classic operations |
| `nfc-mfultralight` | Mifare Ultralight operations |
| `nfc-emulate` | Emulate NFC tag |

### Basic Commands

#### List Devices
```bash
nfc-list
```

#### Read Card
```bash
nfc-mfclassic r b card.dump
```

#### Write Card
```bash
nfc-mfclassic w a card.dump
```

### nfc-mfclassic Commands

| Command | Description | Example |
|---------|-------------|---------|
| `r` | Read card | `nfc-mfclassic r b dump` |
| `w` | Write card | `nfc-mfclassic w a dump` |
| `R` | Read with key A | `nfc-mfclassic R b dump` |
| `W` | Write with key A | `nfc-mfclassic W a dump` |
| `b` | Use key B | `nfc-mfclassic r b dump` |
| `a` | Use key A | `nfc-mfclassic r a dump` |

### nfc-mfultralight Commands

| Command | Description | Example |
|---------|-------------|---------|
| `r` | Read tag | `nfc-mfultralight r dump` |
| `w` | Write tag | `nfc-mfultralight w dump` |
| `dump` | Dump tag | `nfc-mfultralight dump` |

### Workflow

1. **Connect** NFC reader
2. **List** devices
3. **Read** card data
4. **Analyze** dump

### Integration

#### With MFCUK
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt

# Read card
nfc-mfclassic R b card.dump
```

#### With MFOC
```bash
# Dump card
mfoc -r card.dump
```

### Troubleshooting

#### No NFC Reader
```bash
lsusb
pcscd status
```

### Related Tools
- **mfcuk:** Key recovery
- **mfoc:** Card dump
- **mfterm:** Mifare terminal
