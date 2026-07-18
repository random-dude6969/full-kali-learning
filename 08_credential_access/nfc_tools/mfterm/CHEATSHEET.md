# Mfterm Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install mfterm
```

### Basic Commands

#### Start Interactive Shell
```bash
mfterm
```

#### Read Card
```bash
mfterm -r
```

### Interactive Commands

#### Card Operations
| Command | Description |
|---------|-------------|
| `read` | Read card |
| `write` | Write card |
| `dump` | Dump card |
| `load` | Load dump |

#### Key Operations
| Command | Description |
|---------|-------------|
| `keys` | Show keys |
| `setkey` | Set key |
| `loadkeys` | Load keys |

#### Navigation
| Command | Description |
|---------|-------------|
| `cd` | Change sector |
| `ls` | List blocks |
| `cat` | Read block |
| `echo` | Write block |

#### Utility
| Command | Description |
|---------|-------------|
| `help` | Show help |
| `exit` | Exit shell |

### Usage

#### Interactive Session
```bash
mfterm
> read
> keys
> cat 0
> dump card.dump
```

#### Set Custom Key
```bash
mfterm
> setkey A 0xA0A1A2A3A4A5
```

#### Load Keys
```bash
mfterm
> loadkeys custom_keys.txt
```

### Workflow

1. **Start** mfterm
2. **Read** card
3. **Check** keys
4. **Navigate** sectors
5. **Dump** data

### Integration

#### With MFOC
```bash
# Dump card
mfoc -r card.dump

# Analyze with mfterm
mfterm
> load card.dump
```

### Mifare Classic Structure

| Element | Description |
|---------|-------------|
| Sector 0 | Manufacturer block |
| Sector 1-15 | Data sectors |
| Block 3 | Sector trailer (keys) |

### Troubleshooting

#### No Reader Found
```bash
pcscd status
nfc-list
```

### Related Tools
- **mfcuk:** Key recovery
- **mfoc:** Card dump
- **libnfc:** NFC library
