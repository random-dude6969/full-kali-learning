# regripper — Windows Registry Parser

## Table of Contents

1. [Overview](#overview)
2. [Understanding the Windows Registry](#understanding-the-windows-registry)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Registry Hive Files](#registry-hive-files)
7. [Profiles and Plugins](#profiles-and-plugins)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other Tools](#integration-with-other-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

**RegRipper** is an open-source Windows registry forensics tool written in Perl. It parses Windows Registry hive files and extracts forensic artifacts using a plugin-based architecture.

RegRipper is essential for Windows forensics because the registry contains a wealth of forensic evidence including:

- User activity and preferences
- Installed software
- System configuration
- Network settings
- USB device history
- Recent file access
- Startup programs
- Userassist entries
- And much more

### Key Features

- Plugin-based architecture (Perl scripts)
- Supports all major registry hive types
- Timeline analysis support (TLN format)
- Command-line interface
- Extensible via custom plugins

---

## Understanding the Windows Registry

### What Is the Registry?

The Windows Registry is a hierarchical database that stores configuration settings and options for Microsoft Windows. It contains:

- Hardware configuration
- Software settings
- User preferences
- System state information

### Registry Hives

| Hive File | Location | Contents |
|-----------|----------|----------|
| SAM | `%SystemRoot%\system32\config\SAM` | User accounts, passwords |
| SECURITY | `%SystemRoot%\system32\config\SECURITY` | Security policies |
| SOFTWARE | `%SystemRoot%\system32\config\SOFTWARE` | Installed software |
| SYSTEM | `%SystemRoot%\system32\config\SYSTEM` | Hardware, boot config |
| NTUSER.DAT | `%UserProfile%\NTUSER.DAT` | User-specific settings |
| UsrClass.dat | `%UserProfile%\AppData\Local\Microsoft\Windows\UsrClass.dat` | User class data |

### Registry Key Structure

```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
├── SecurityHealth
│   └── "C:\Program Files\Windows Defender\MSASCuiL.exe"
└── Windows Defender
    └── "%ProgramFiles%\Windows Defender\MSASCuiL.exe"
```

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install regripper
```

### Dependencies

```bash
# RegRipper requires Perl and specific modules
sudo apt install perl libparse-win32registry-perl
```

### Verify Installation

```bash
rip -V
```

---

## Command Syntax

```bash
rip [options] -r <hive_file>
```

### Basic Usage

```bash
# Auto-detect hive type and run appropriate plugins
rip -r /evidence/SOFTWARE

# Run specific profile
rip -r /evidence/SOFTWARE -f software

# Run specific plugin
rip -r /evidence/NTUSER.DAT -p userassist

# List all available plugins
rip -l

# Guess the hive type
rip -r /evidence/hive_file -g
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-r hive_file` | Registry hive file to parse |
| `-f profile` | Use a specific profile (list of plugins) |
| `-p plugin` | Run a specific plugin |
| `-a` | Auto-run hive-specific plugins |
| `-aT` | Auto-run hive-specific TLN plugins |
| `-d` | Check if hive is dirty |
| `-g` | Guess the hive file type |
| `-l` | List all plugins |
| `-c` | Output plugin list in CSV format |
| `-s systemname` | System name (for TLN support) |
| `-u username` | Username (for TLN support) |
| `-uP` | Update default profiles |
| `-h` | Help |

---

## Registry Hive Files

### Locating Hive Files

On a Windows system:

```bash
# System hives
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SECURITY
C:\Windows\System32\config\SOFTWARE
C:\Windows\System32\config\SYSTEM

# User hives
C:\Users\<username>\NTUSER.DAT
C:\Users\<username>\AppData\Local\Microsoft\Windows\UsrClass.dat
```

### From a Disk Image

```bash
# First, identify partitions
mmls evidence.E01

# Extract hive files using TSK
icat -f ntfs -o <offset> evidence.E01 <inode> > SAM

# Or use fls to find the files
fls -f ntfs -o <offset> -r evidence.E01 | grep -i "SAM\|NTUSER\|SOFTWARE"
```

---

## Profiles and Plugins

### Built-in Profiles

| Profile | Hive File | Description |
|---------|-----------|-------------|
| `software` | SOFTWARE | Installed software analysis |
| `system` | SYSTEM | System configuration |
| `sam` | SAM | User accounts |
| `security` | SECURITY | Security policies |
| `ntuser` | NTUSER.DAT | User activity |
| `usrclass` | UsrClass.dat | User class data |

### Key Plugins

| Plugin | Description |
|--------|-------------|
| `userassist` | Recently executed programs |
| `shimcache` | Application compatibility cache |
| `amcache` | Application execution history |
| `mrulist` | Most recently used lists |
| `usb` | USB device history |
| `network` | Network configuration |
| `services` | Windows services |
| `run` | Startup programs |
| `uninstall` | Installed software |
| `typedurls` | Typed URLs in browser |

### Listing All Plugins

```bash
rip -l
```

### Running a Specific Plugin

```bash
# UserAssist entries
rip -r /evidence/NTUSER.DAT -p userassist

# USB history
rip -r /evidence/NTUSER.DAT -p usb

# Startup programs
rip -r /evidence/NTUSER.DAT -p run
```

### Running All Plugins for a Hive

```bash
# Auto-detect and run all appropriate plugins
rip -r /evidence/SOFTWARE -a

# Or use a profile
rip -r /evidence/SOFTWARE -f software
```

---

## Practical Lab Exercises

### Lab 1: Analyzing SOFTWARE Hive

```bash
# Auto-detect and analyze
rip -r /evidence/SOFTWARE -a

# Or use profile
rip -r /evidence/SOFTWARE -f software

# Save output
rip -r /evidence/SOFTWARE -a > software_analysis.txt
```

### Lab 2: User Activity Analysis

```bash
# Analyze NTUSER.DAT
rip -r /evidence/NTUSER.DAT -a

# Focus on UserAssist
rip -r /evidence/NTUSER.DAT -p userassist

# Focus on MRU lists
rip -r /evidence/NTUSER.DAT -p mrulist
```

### Lab 3: USB Device History

```bash
# Extract USB history from NTUSER.DAT
rip -r /evidence/NTUSER.DAT -p usb

# Also check SYSTEM hive
rip -r /evidence/SYSTEM -p usbstor
```

### Lab 4: Network Configuration

```bash
# Analyze network settings
rip -r /evidence/SOFTWARE -p network

# Check SYSTEM for network adapters
rip -r /evidence/SYSTEM -p network
```

### Lab 5: Startup Programs Analysis

```bash
# Check Run keys
rip -r /evidence/NTUSER.DAT -p run

# Check SOFTWARE for startup
rip -r /evidence/SOFTWARE -p run
```

---

## Integration with Other Tools

### RegRipper + Registry Explorer

```bash
# RegRipper for automated analysis
rip -r /evidence/SOFTWARE -a > regripper_output.txt

# Registry Explorer for manual browsing
# (Eric Zimmerman's tool - Windows only)
```

### RegRipper + Autopsy

```bash
# Autopsy can use RegRiper plugins
# Or analyze hives directly in Autopsy
```

### RegRipper + Timeline Analysis

```bash
# Generate TLN (Time Line) output
rip -r /evidence/NTUSER.DAT -aT -s "WORKSTATION1" -u "jsmith"

# Combine with other timeline data
cat tlnt_output.txt >> timeline_body.txt
mactime -b timeline_body.txt
```

### RegRipper + Harlan Carvey's Tools

```bash
# Use reg解析 for additional analysis
# Check Windows Forensic Analysis toolkit
```

---

## Common Forensic Scenarios

### Scenario 1: User Activity Timeline

```bash
# Analyze user activity
rip -r /evidence/NTUSER.DAT -p userassist
rip -r /evidence/NTUSER.DAT -p mrulist
rip -r /evidence/NTUSER.DAT -p typedurls

# Combine for timeline
rip -r /evidence/NTUSER.DAT -aT > user_timeline.txt
```

### Scenario 2: Malware Persistence

```bash
# Check startup locations
rip -r /evidence/NTUSER.DAT -p run
rip -r /evidence/SOFTWARE -p run

# Check services
rip -r /evidence/SYSTEM -p services

# Check scheduled tasks
rip -r /evidence/SOFTWARE -p scheduled
```

### Scenario 3: USB Device Investigation

```bash
# Extract USB history
rip -r /evidence/NTUSER.DAT -p usb
rip -r /evidence/SYSTEM -p usbstor

# Correlate with event logs
```

### Scenario 4: Software Installation History

```bash
# Check installed software
rip -r /evidence/SOFTWARE -p uninstall

# Check ShimCache
rip -r /evidence/SOFTWARE -p shimcache

# Check Amcache
rip -r /evidence/SOFTWARE -p amcache
```

### Scenario 5: Browser Forensics

```bash
# Typed URLs
rip -r /evidence/NTUSER.DAT -p typedurls

# Recent documents
rip -r /evidence/NTUSER.DAT -p mrulist
```

---

## Troubleshooting

### "Cannot open hive file"

```bash
# Check file permissions
ls -la /evidence/SOFTWARE

# Check if file is corrupted
rip -r /evidence/SOFTWARE -g

# Ensure you have the correct file
file /evidence/SOFTWARE
```

### "Plugin not found"

```bash
# List available plugins
rip -l

# Check plugin name
rip -l | grep -i "plugin_name"
```

### "Dirty hive"

```bash
# Check hive status
rip -r /evidence/SOFTWARE -d

# Process transaction logs first
# (use yarp + registryFlush.py or rla.exe from Eric Zimmerman)
```

### "No output"

```bash
# Check if hive has data
rip -r /evidence/SOFTWARE -g

# Try verbose mode
rip -r /evidence/SOFTWARE -a -v
```

---

## Best Practices

1. **Always process transaction logs first** — dirty hives may have incomplete data
2. **Use `-g` to identify hive type** — before running plugins
3. **Run all plugins** — `-a` for comprehensive analysis
4. **Use TLN output** — `-aT` for timeline integration
5. **Document system name and user** — `-s` and `-u` for TLN
6. **Check multiple hives** — SYSTEM, SOFTWARE, NTUSER.DAT
7. **Combine with other tools** — RegRipper for extraction, Registry Explorer for verification
8. **Save all output** — redirect to files for later analysis
9. **Custom plugins** — write your own for specific needs
10. **Verify findings** — cross-reference with other evidence

---

## Summary

RegRipper is essential for Windows registry forensics. Key capabilities:

- Parses all major registry hive types
- Plugin-based architecture for flexibility
- Timeline analysis support (TLN)
- Extensible via custom plugins
- Works with raw hive files from disk images

### Quick Reference

```bash
# Auto-analyze a hive
rip -r /evidence/SOFTWARE -a

# Run specific plugin
rip -r /evidence/NTUSER.DAT -p userassist

# Timeline output
rip -r /evidence/NTUSER.DAT -aT -s "WORKSTATION" -u "user"

# List plugins
rip -l

# Guess hive type
rip -r /evidence/file -g
```

---

## References

- RegRipper GitHub: https://github.com/keydet89/RegRipper3.0
- Harlan Carvey's Windows Forensic Analysis
- SANS Windows Forensic Analysis: https://www.sans.org/windows-forensic-analysis/
- Kali Linux RegRipper: https://www.kali.org/tools/regripper/
