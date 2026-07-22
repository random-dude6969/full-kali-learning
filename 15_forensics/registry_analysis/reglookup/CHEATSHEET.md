# reglookup Cheatsheet — Windows Registry Lookup Tool

## Quick Commands

```bash
reglookup [options] registry-file
reglookup-recover [options] registry-file
reglookup-timeline registry-file
```

## Installation

```bash
sudo apt install reglookup
```

## Most Common Commands

```bash
# Read entire registry
reglookup /evidence/SOFTWARE

# Filter by path
reglookup -p /Microsoft /evidence/SOFTWARE

# Filter by type
reglookup -t SZ /evidence/SOFTWARE

# Recover deleted entries
reglookup-recover /evidence/SOFTWARE > recovered.csv

# Generate timeline
reglookup-timeline /evidence/SOFTWARE > timeline.csv
```

## Options Reference

### reglookup

| Option | Description |
|--------|-------------|
| `-p prefix` | Path prefix filter |
| `-t type` | Type filter |
| `-h` / `-H` | Enable/disable header |
| `-i` | Inherit parent timestamp |
| `-s` / `-S` | Enable/disable security descriptors |
| `-v` | Verbose output |

### reglookup-recover

| Option | Description |
|--------|-------------|
| `-l` | Show unparsable cells |
| `-r` | Show raw cell contents |
| `-v` | Verbose output |

### Type Filters

| Type | Meaning |
|------|---------|
| SZ | String |
| EXPAND_SZ | Expandable string |
| BINARY | Binary data |
| DWORD | 32-bit integer |
| MULTI_SZ | Multiple strings |
| QWORD | 64-bit integer |
| KEY | Registry key |

## Workflow

```bash
# 1. Identify registry hive
ls /evidence/

# 2. Query specific paths
reglookup -p /path /evidence/SOFTWARE > results.csv

# 3. Recover deleted data
reglookup-recover /evidence/SOFTWARE > recovered.csv

# 4. Generate timeline
reglookup-timeline /evidence/SOFTWARE > timeline.csv

# 5. Analyze CSV output
cat results.csv | head -20
```

## Common Queries

```bash
# Installed software
reglookup -p /Microsoft/Windows/CurrentVersion/Uninstall /evidence/SOFTWARE

# Startup programs
reglookup -p /Microsoft/Windows/CurrentVersion/Run /evidence/NTUSER.DAT

# Network settings
reglookup -p /ControlSet001/Services/Tcpip /evidence/SYSTEM

# USB history
reglookup -p /Microsoft/Windows/CurrentVersion/Explorer/MountPoints2 /evidence/NTUSER.DAT
```

## Integration

```bash
# reglookup + csvkit analysis
reglookup /evidence/SOFTWARE > reg.csv
csvcut -c 2,3,4 reg.csv | head -20

# reglookup + grep
reglookup /evidence/SOFTWARE | grep -i "password"

# reglookup + timeline tools
reglookup-timeline /evidence/SOFTWARE > reg_timeline.csv
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Cannot open hive | Check file permissions and validity |
| No output | Try `-p /` to see all keys |
| CSV encoding | Handle %XX encoding in scripts |
| Timeline empty | Verify timestamps present in hive |
