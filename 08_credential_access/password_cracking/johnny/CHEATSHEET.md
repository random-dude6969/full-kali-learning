# Johnny Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install johnny
```

### Basic Commands

#### Launch GUI
```bash
johnny
```

### Interface Components

| Component | Description |
|-----------|-------------|
| Hash Input | Enter/paste hashes |
| Wordlist | Select wordlist file |
| Rules | Enable/disable rules |
| Start | Begin cracking |
| Stop | Stop cracking |
| Show | Display results |

### Workflow

1. **Launch** Johnny
2. **Load** hash file or paste hashes
3. **Select** wordlist
4. **Enable** rules (optional)
5. **Start** cracking
6. **View** results

### Features

#### Auto-Detection
- Johnny automatically detects hash types
- Or select manually from dropdown

#### Session Management
- Save/load sessions
- Resume cracking

#### Rule Selection
- Built-in rule sets
- Custom rules

### Comparison with John

| Feature | Johnny | John |
|---------|--------|------|
| Interface | GUI | CLI |
| Features | Basic | Full |
| Speed | Same | Same |
| Ease of use | Easier | Harder |

### Tips

#### For Beginners
- Start with Johnny for ease of use
- Graduate to John for advanced features

#### For Advanced Users
- Use Johnny for quick visual feedback
- Use John for scripting/automation

### Troubleshooting

#### John Not Found
```bash
sudo apt install john
```

#### No Hashes Loaded
```bash
# Check hash format
hashid 'hash_string'
```

### Related Tools
- **john:** John the Ripper (backend)
- **hashcat:** GPU password cracker
- **ophcrack:** Windows password cracker
