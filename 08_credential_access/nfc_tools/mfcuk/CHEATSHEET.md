# MFCUK Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install mfcuk
```

### Basic Commands

#### Key Recovery
```bash
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF
```

#### Save Keys
```bash
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF -o keys.txt
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-C` | Check for card | `mfcuk -C` |
| `-R` | Recover keys | `mfcuk -R` |
| `-W` | Write keys | `mfcuk -W` |
| `-f` | Target key | `mfcuk -f 0xFFFFFFFFFFFF` |
| `-o` | Output file | `mfcuk -o keys.txt` |
| `-i` | Input file | `mfcuk -i keys.txt` |
| `-v` | Verbose | `mfcuk -v 3` |

### Usage

#### Basic Recovery
```bash
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF
```

#### Specific Key
```bash
mfcuk -C -R -v 3 -f 0xA0A1A2A3A4A5
```

### Workflow

1. **Connect** NFC reader
2. **Run** MFCUK
3. **Place** card on reader
4. **Wait** for key recovery
5. **Save** recovered keys

### Integration

#### With MFOC
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt

# Dump card
mfoc -k keys.txt -r card.dump
```

#### With NFC-List
```bash
# List card info
nfc-list

# Recover keys
mfcuk -C -R -v 3 -o keys.txt
```

### Common Keys

| Key | Description |
|-----|-------------|
| `FFFFFFFFFFFF` | Default key A |
| `A0A1A2A3A4A5` | MAD key |
| `D3F7D3F7D3F7` | NDEF key |

### Troubleshooting

#### No Reader Found
```bash
pcscd status
nfc-list
```

### Related Tools
- **mfoc:** Mifare Classic dump tool
- **libnfc:** NFC library
- **mfterm:** Mifare terminal
