# Twofi Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install twofi
```

### Basic Commands

#### Generate Wordlist
```bash
twofi -o wordlist.txt
```

#### With Data Directory
```bash
twofi -d /path/to/data -o wordlist.txt
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Data directory | `twofi -d /path/to/data` |
| `-o` | Output file | `twofi -o wordlist.txt` |
| `-m` | Min word length | `twofi -m 4 -o wordlist.txt` |
| `-M` | Max word length | `twofi -M 8 -o wordlist.txt` |

### Usage

#### Basic Generation
```bash
twofi -o wordlist.txt
```

#### With Custom Data
```bash
twofi -d /path/to/corpus -o wordlist.txt
```

#### Filter Length
```bash
twofi -m 6 -M 8 -o wordlist.txt
```

### Workflow

1. **Run** twofi
2. **Generate** frequency list
3. **Use** with cracking tool

### Integration

#### With Hydra
```bash
# Generate
twofi -o wordlist.txt

# Use
hydra -l admin -P wordlist.txt ssh://192.168.1.100
```

### Troubleshooting

#### No Data Found
```bash
ls /path/to/data
twofi -d /correct/path -o wordlist.txt
```

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **pack:** Password analysis
