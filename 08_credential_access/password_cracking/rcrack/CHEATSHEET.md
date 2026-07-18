# Rcrack Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install rcrack
```

### Basic Commands

#### Crack with Rainbow Tables
```bash
rcrack -t table_dir hashes.txt
```

#### List Tables
```bash
rcrack -l table_dir
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Table directory | `rcrack -t tables/ hashes.txt` |
| `-f` | Hash file | `rcrack -t tables/ -f hashes.txt` |
| `-l` | List tables | `rcrack -l tables/` |
| `-o` | Output | `rcrack -o results.txt hashes.txt` |
| `-v` | Verbose | `rcrack -v hashes.txt` |

### Rainbow Table Sources

| Source | URL |
|--------|-----|
| RainbowCrack | project-rainbowcrack.com |
| Freerable Tables | rcracki.sourceforge.net |

### Usage

#### Basic Cracking
```bash
rcrack -t /path/to/tables hashes.txt
```

#### Multiple Table Sets
```bash
rcrack -t tables1/ -t tables2/ hashes.txt
```

### Workflow

1. **Download** rainbow tables
2. **Identify** hash type
3. **Run** rcrack
4. **View** results

### Hash Types Supported

| Type | Description |
|------|-------------|
| MD5 | 32 hex chars |
| SHA1 | 40 hex chars |
| NTLM | 32 hex chars |
| LM | 32 hex chars |

### Troubleshooting

#### No Tables Found
```bash
ls tables/
rcrack -t /correct/path/ hashes.txt
```

### Related Tools
- **ophcrack:** Windows rainbow table cracker
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
