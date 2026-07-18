# Ophcrack Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install ophcrack
```

### Basic Commands

#### Launch GUI
```bash
ophcrack
```

#### Command Line
```bash
ophcrack -f sam_file -t rainbow_table_dir
```

### Rainbow Tables

#### Free Tables
| Table | Description | Size |
|-------|-------------|------|
| XP_pro_fast | Fast, limited | 400MB |
| XP_pro_big | Slow, comprehensive | 700MB |
| Vista_precomputed | Vista/7 | 500MB |

#### Download Tables
```bash
wget http://ophcrack.sourceforge.net/tables.php
```

### Usage Modes

#### GUI Mode
```bash
ophcrack
```

#### Console Mode
```bash
ophcrack -c -f sam_file -t tables/
```

#### Audit Mode
```bash
ophcrack -a sam_file -t tables/
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-f` | SAM file | `ophcrack -f sam` |
| `-t` | Table dir | `ophcrack -t tables/` |
| `-d` | Table dir | `ophcrack -d tables/` |
| `-o` | Output | `ophcrack -o results.txt` |
| `-v` | Verbose | `ophcrack -v` |
| `-c` | Console | `ophcrack -c` |

### SAM Extraction

#### Windows
```bash
reg save HKLM\SAM sam_file
reg save HKLM\SYSTEM system_file
```

#### From Mounted Drive
```bash
chntpw -r /mnt/windows/WINDOWS/system32/config/SAM
```

### Workflow

1. **Extract** SAM file
2. **Select** rainbow tables
3. **Run** Ophcrack
4. **View** cracked passwords

### Table Selection

#### Fast (Less Coverage)
```bash
ophcrack -f sam -t tables/XP_pro_fast
```

#### Large (More Coverage)
```bash
ophcrack -f sam -t tables/XP_pro_big
```

### Troubleshooting

#### SAM Not Found
```bash
ls -la sam_file
ophcrack -f /path/to/sam -t tables/
```

#### Table Not Found
```bash
ophcrack -l tables/
```

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **chntpw:** Windows password change
