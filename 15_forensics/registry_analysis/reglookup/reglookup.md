# reglookup — Windows Registry Lookup Tool

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Command Syntax](#command-syntax)
4. [Options Reference](#options-reference)
5. [Understanding the Output](#understanding-the-output)
6. [Registry Data Types](#registry-data-types)
7. [Practical Lab Exercises](#practical-lab-exercises)
8. [Integration with Other Tools](#integration-with-other-tools)
9. [Common Forensic Scenarios](#common-forensic-scenarios)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Summary](#summary)

---

## Overview

**RegLookup** is a system for direct analysis of Windows NT-based registry files. It provides command-line tools, a C API, and a Python module for accessing registry data structures.

RegLookup is focused on digital forensics investigations and includes algorithms for:

- Reading entire registry hives
- Outputting data in standardized CSV format
- Filtering by registry path and data type
- Recovering deleted data structures from registry hives
- Generating timeline data from registry timestamps

### Key Tools

| Tool | Purpose |
|------|---------|
| `reglookup` | Read and query registry entries |
| `reglookup-recover` | Recover deleted registry data |
| `reglookup-timeline` | Generate timeline from registry |

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install reglookup
```

### Dependencies

```bash
# RegLookup requires libregfi
sudo apt install libregfi1t64 python3-pyregfi
```

### Verify Installation

```bash
reglookup --help
```

---

## Command Syntax

### reglookup

```bash
reglookup [options] registry-file
```

### reglookup-recover

```bash
reglookup-recover [options] registry-file
```

### reglookup-timeline

```bash
reglookup-timeline registry-file
```

---

## Options Reference

### reglookup Options

| Option | Description |
|--------|-------------|
| `-p prefix` | Path prefix filter |
| `-t type` | Type filter (SZ, BINARY, DWORD, etc.) |
| `-h` | Print column header (default) |
| `-H` | Disable column header |
| `-i` | Inherit timestamp from parent key |
| `-s` | Add security descriptor columns |
| `-S` | Disable security descriptors (default) |
| `-v` | Verbose output |

### reglookup-recover Options

| Option | Description |
|--------|-------------|
| `-v` | Verbose output |
| `-h` | Print column header (default) |
| `-H` | Disable column header |
| `-l` | Display unparsable cells |
| `-L` | Don't display unparsable cells (default) |
| `-r` | Display raw cell contents |
| `-R` | Don't display raw contents (default) |

### Type Filter Values

| Type | Description |
|------|-------------|
| NONE | No value |
| SZ | String |
| EXPAND_SZ | Expandable string |
| BINARY | Binary data |
| DWORD | 32-bit integer |
| DWORD_BE | Big-endian DWORD |
| LINK | Symbolic link |
| MULTI_SZ | Multiple strings |
| RSRC_LIST | Resource list |
| RSRC_DESC | Resource descriptor |
| RSRC_REQ_LIST | Resource requirements list |
| QWORD | 64-bit integer |
| KEY | Registry key |

---

## Understanding the Output

### CSV Output Format

```
KEY,PATH,MTIME,TYPE,VALUE
```

| Column | Description |
|--------|-------------|
| KEY | Registry key type (KEY, VALUE) |
| PATH | Full registry path |
| MTIME | Modification timestamp |
| TYPE | Data type |
| VALUE | Data value |

### Example Output

```
KEY,/ControlSet001,2024-01-15 10:30:00,
KEY,/ControlSet001/Services,2024-01-15 10:30:00,
VALUE,/ControlSet001/Services/Tcpip,2024-01-15 10:30:00,SZ,TCP/IP Driver
VALUE,/ControlSet001/Services/Tcpip/Parameters,2024-01-15 10:30:00,BINARY,a1b2c3...
```

### Security Descriptor Output

With `-s` flag:

```
KEY,PATH,MTIME,TYPE,VALUE,OWNER,GROUP,SACL,DACL,CLASS
```

---

## Practical Lab Exercises

### Lab 1: Basic Registry Reading

```bash
# Read entire registry
reglookup /evidence/SOFTWARE

# Filter by path
reglookup -p /Microsoft /evidence/SOFTWARE

# Filter by type
reglookup -t SZ /evidence/SOFTWARE
```

### Lab 2: Finding Installed Software

```bash
# List all strings under Uninstall key
reglookup -p /Microsoft/Windows/CurrentVersion/Uninstall -t SZ /evidence/SOFTWARE

# Or grep for software names
reglookup /evidence/SOFTWARE | grep -i "uninstall"
```

### Lab 3: Network Configuration

```bash
# Find network settings
reglookup -p /ControlSet001/Services/Tcpip /evidence/SYSTEM

# Find WiFi profiles
reglookup -p /Microsoft/Wlansvc /evidence/SOFTWARE
```

### Lab 4: Deleted Registry Data Recovery

```bash
# Recover deleted entries
reglookup-recover /evidence/SOFTWARE > recovered.csv

# Show unparsable cells
reglookup-recover -l /evidence/SOFTWARE > recovered_all.csv

# Include raw data
reglookup-recover -r /evidence/SOFTWARE > recovered_raw.csv
```

### Lab 5: Timeline Generation

```bash
# Generate timeline from registry
reglookup-timeline /evidence/SOFTWARE > timeline.csv

# Combine with other timelines
cat timeline.csv >> full_timeline.csv
```

---

## Integration with Other Tools

### RegLookup + RegRipper

```bash
# RegLookup for specific queries
reglookup -p /path /evidence/SOFTWARE > query_results.csv

# RegRipper for comprehensive analysis
rip -r /evidence/SOFTWARE -a > regripper_output.txt
```

### RegLookup + Plaso/log2timeline

```bash
# Generate timeline
reglookup-timeline /evidence/SOFTWARE > registry_timeline.csv

# Import into Plaso
log2timeline.py --storage-file timeline.plaso registry_timeline.csv
```

### RegLookup + Python (pyregfi)

```python
# Python bindings for custom analysis
import pyregfi

# Open hive
hive = pyregfi.Hive('/evidence/SOFTWARE')

# Iterate keys
for key in hive:
    print(key.name)
```

### RegLookup + CSV Analysis

```bash
# Export to CSV
reglookup /evidence/SOFTWARE > registry.csv

# Analyze with csvkit
csvcut -c 2,3,4 registry.csv | head -20

# Or with awk
reglookup /evidence/SOFTWARE | awk -F',' '{print $2, $4}' | sort | uniq -c
```

---

## Common Forensic Scenarios

### Scenario 1: Software Inventory

```bash
# List all installed software
reglookup -p /Microsoft/Windows/CurrentVersion/Uninstall /evidence/SOFTWARE

# List recently executed (UserAssist)
reglookup -p /Microsoft/Windows/CurrentVersion/Explorer/UserAssist /evidence/NTUSER.DAT
```

### Scenario 2: Network Investigation

```bash
# WiFi connections
reglookup -p /Microsoft/Wlansvc /evidence/SOFTWARE

# Network adapters
reglookup -p /ControlSet001/Services/Tcpip /evidence/SOFTWARE

# Proxy settings
reglookup -p /Microsoft/Windows/CurrentVersion/Internet\ Settings /evidence/NTUSER.DAT
```

### Scenario 3: USB Device History

```bash
# USB device records
reglookup -p /Microsoft/Windows/CurrentVersion/Explorer/MountPoints2 /evidence/NTUSER.DAT

# USBSTOR in SYSTEM
reglookup -p /ControlSet001/Enum/USBSTOR /evidence/SYSTEM
```

### Scenario 4: Startup Programs

```bash
# Run keys
reglookup -p /Microsoft/Windows/CurrentVersion/Run /evidence/NTUSER.DAT
reglookup -p /Microsoft/Windows/CurrentVersion/RunOnce /evidence/NTUSER.DAT

# Services
reglookup -p /ControlSet001/Services /evidence/SYSTEM
```

### Scenario 5: Deleted Data Recovery

```bash
# Recover deleted registry entries
reglookup-recover /evidence/SOFTWARE > recovered.csv

# Look for interesting entries
grep -i "password\|secret\|key" recovered.csv
```

---

## Troubleshooting

### "Cannot open hive file"

```bash
# Check file exists and is readable
ls -la /evidence/SOFTWARE

# Verify it's a valid registry hive
file /evidence/SOFTWARE

# Check for transaction logs
ls -la /evidence/SOFTWARE.LOG*
```

### "No output"

```bash
# Check if hive has data
reglookup /evidence/SOFTWARE | head -5

# Try verbose mode
reglookup -v /evidence/SOFTWARE

# Check path filter
reglookup -p / /evidence/SOFTWARE
```

### "CSV parsing errors"

```bash
# The output uses %XX encoding for special characters
# Handle in scripts:
cat output.csv | sed 's/%20/ /g' | sed 's/%2C/,/g'
```

### "Timeline not working"

```bash
# Ensure timestamps are present
reglookup /evidence/SOFTWARE | head -5

# Check file permissions
ls -la /evidence/SOFTWARE
```

---

## Best Practices

1. **Use path filters** — `-p` to focus on specific registry areas
2. **Use type filters** — `-t` to find specific data types
3. **Include headers** — `-h` for CSV parsing
4. **Check for deleted data** — use `reglookup-recover`
5. **Generate timelines** — `reglookup-timeline` for temporal analysis
6. **Combine with RegRipper** — different tools for different needs
7. **Export to CSV** — for spreadsheet analysis
8. **Document findings** — save all output
9. **Check transaction logs** — dirty hives need log processing
10. **Verify with multiple tools** — cross-reference findings

---

## Summary

RegLookup provides direct access to Windows registry data. Key capabilities:

- Reads and queries registry entries
- Recovers deleted data structures
- Generates timeline from timestamps
- Outputs in CSV format
- Supports Python scripting

### Quick Reference

```bash
# Read registry
reglookup /evidence/SOFTWARE

# Filter by path
reglookup -p /path /evidence/SOFTWARE

# Filter by type
reglookup -t SZ /evidence/SOFTWARE

# Recover deleted
reglookup-recover /evidence/SOFTWARE > recovered.csv

# Timeline
reglookup-timeline /evidence/SOFTWARE > timeline.csv
```

---

## Advanced Registry Analysis

### Understanding Registry Data Types

Each registry value has a specific data type that determines how the data is interpreted:

```bash
# List all string values
reglookup -t SZ /evidence/SOFTWARE

# List all binary values
reglookup -t BINARY /evidence/SOFTWARE

# List all DWORD values
reglookup -t DWORD /evidence/SOFTWARE

# List all QWORD values
reglookup -t QWORD /evidence/SOFTWARE
```

### Security Descriptor Analysis

Registry keys have security descriptors that control access:

```bash
# Show security descriptors
reglookup -s -p /ControlSet001 /evidence/SYSTEM

# Output includes: OWNER, GROUP, SACL, DACL
# DACL shows who can access the key
```

### Registry Key Timestamps

Registry keys have modification timestamps that are valuable for forensics:

```bash
# All timestamps
reglookup -p /ControlSet001 /evidence/SOFTWARE | head -20

# Filter by time range
reglookup -p /Microsoft /evidence/SOFTWARE | grep "2024-01"
```

### Python Scripting with pyregfi

```python
#!/usr/bin/env python3
import pyregfi

# Open a registry hive
hive = pyregfi.Hive('/evidence/SOFTWARE')

# Iterate through keys
for key in hive:
    print(f"Key: {key.name}")
    print(f"Timestamp: {key.mtime}")
    
    # Iterate through values
    for value in key.values:
        print(f"  Value: {value.name} = {value.value}")
```

### Comparing Registry Hives

```bash
# Export both hives
reglookup /evidence/SOFTWARE.before > before.csv
reglookup /evidence/SOFTWARE.after > after.csv

# Compare
diff before.csv after.csv

# Find new entries
comm -23 after.csv before.csv

# Find removed entries
comm -13 after.csv before.csv
```

### Registry Analysis for Malware Detection

```bash
# Check Run keys for persistence
reglookup -p /Microsoft/Windows/CurrentVersion/Run /evidence/NTUSER.DAT
reglookup -p /Microsoft/Windows/CurrentVersion/RunOnce /evidence/NTUSER.DAT

# Check Services for malicious services
reglookup -p /ControlSet001/Services /evidence/SYSTEM

# Check AppInit_DLLs for injection
reglookup -p /Microsoft/Windows NT/CurrentVersion /evidence/SOFTWARE

# Check Winlogon for shell replacement
reglookup -p /Microsoft/Windows NT/CurrentVersion/Winlogon /evidence/SOFTWARE
```

### Registry Analysis for Network Forensics

```bash
# WiFi profiles and passwords
reglookup -p /Microsoft/Wlansvc /evidence/SOFTWARE

# Network connections history
reglookup -p /Microsoft/Windows/CurrentVersion/Internet Settings /evidence/NTUSER.DAT

# Proxy settings
reglookup -p /Microsoft/Windows/CurrentVersion/Internet Settings /evidence/NTUSER.DAT | grep -i proxy
```

### Registry Analysis for User Activity

```bash
# Recent documents
reglookup -p /Microsoft/Windows/CurrentVersion/Explorer/RecentDocs /evidence/NTUSER.DAT

# OpenSaveMRU
reglookup -p /Microsoft/Windows/CurrentVersion/Explorer/ComDlg32/OpenSaveMRU /evidence/NTUSER.DAT

# Typed paths
reglookup -p /Microsoft/Windows/CurrentVersion/Explorer/TypedPaths /evidence/NTUSER.DAT
```

---

## Registry Analysis Reference

| Analysis Type | Key Path | Hive |
|---------------|----------|------|
| Installed software | `/Microsoft/Windows/CurrentVersion/Uninstall` | SOFTWARE |
| Startup programs | `/Microsoft/Windows/CurrentVersion/Run` | NTUSER.DAT |
| Services | `/ControlSet001/Services` | SYSTEM |
| USB history | `/Microsoft/Windows/CurrentVersion/Explorer/MountPoints2` | NTUSER.DAT |
| WiFi profiles | `/Microsoft/Wlansvc` | SOFTWARE |
| UserAssist | `/Microsoft/Windows/CurrentVersion/Explorer/UserAssist` | NTUSER.DAT |
| ShimCache | `/Microsoft/Windows NT/CurrentVersion/AppCompatFlags/AppCompatCache` | SYSTEM |
| Amcache | `/Microsoft/Windows/CurrentVersion/AppCompatFlags/AppCompatCache/Amcache.hve` | SOFTWARE |

---

## References

- RegLookup Homepage: https://web.archive.org/web/20240804024044/http://projects.sentinelchicken.org/reglookup/
- Kali Linux RegLookup: https://www.kali.org/tools/reglookup/
- Windows Registry Forensics
- Harlan Carvey, "Windows Registry Forensic Analysis"
- SANS Windows Forensic Analysis: https://www.sans.org/windows-forensic-analysis/
