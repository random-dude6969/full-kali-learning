---
title: "Sickle"
tool_name: "sickle-pdk"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Intermediate"
platform: "Linux (cross-platform shellcode)"
language: "Python"
license: "MIT"
repository: "https://github.com/wiregh0/sickle"
kali_package: "sickle"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "shellcode-development"
  - "shell-storm"
  - "format-conversion"
  - "assembly"
date_created: "2015-01-01"
last_updated: "2023-01-01"
author: "wiregh0"
---

# Sickle — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

Sickle is a shellcode development toolkit that provides format conversion, shellcode generation, and integration with the Shell-Storm shellcode database. It bridges the gap between raw shellcode writing and practical deployment by offering quick conversion between common formats and direct access to community-contributed shellcode.

### Core Features

- **Format conversion**: Assembly ↔ hex, Python, C, raw binary, Ruby
- **Shell-Storm integration**: Download and convert shellcode directly from the database
- **Bad character filtering**: Automatically identify and report bad characters
- **Multi-architecture**: x86 and x64 shellcode support
- **Quick generation**: One-liner shellcode creation for common payloads

### Relationship to ShellNoob

While ShellNoob focuses on development workflow and debugging, Sickle emphasizes practical shellcode generation and database access. They complement each other: ShellNoob for writing, Sickle for sourcing and deploying.

---

## Chapter 2: Installation & Setup

### Kali Linux

```bash
sudo apt update && sudo apt install sickle
```

**Explanation:** Installs Sickle from the Kali repository.

### From Source

```bash
git clone https://github.com/wiregh0/sickle.git
cd sickle
pip3 install -r requirements.txt
chmod +x sickle.py
```

**Explanation:** Clones the repository and installs Python dependencies.

### Verify Installation

```bash
sickle -h
```

Expected output:

```
Sickle - Shellcode development toolkit

Usage: sickle [options]

Options:
  -h, --help            show this help message and exit
  -s SHELLCODE          shellcode to work with
  -f FORMAT             source format of shellcode (raw, hex, c, python, ruby)
  -o OUTPUT             output format (raw, hex, c, python, ruby)
  -c                    check for bad characters
  -b BADCHARS           bad characters to check for (e.g., \x00\x0a)
  -S SHELLSTORM_ID      download shellcode from Shell-Storm by ID
  -a ARCH               architecture (x86, x64)
  -v                    verbose output
```

---

## Chapter 3: Basic Usage

### Format Conversion

```bash
# Convert hex to C array
sickle -s "9089c331d2" -f hex -o c

# Convert C array to Python
sunny -s "\x90\x89\xc3\x31\xd2" -f c -o python

# Convert hex to raw binary file
sickle -s "9089c331d2" -f hex -o raw -o shellcode.bin
```

**Explanation:** Converts shellcode between formats for different delivery mechanisms.

### Bad Character Detection

```bash
sunny -s "9089c331d2000a" -f hex -c
```

**Explanation:** Checks the shellcode for common bad characters and reports any found.

### Bad Character Filtering with Custom List

```bash
sunny -s "9089c331d2000a0d" -f hex -c -b "\x00\x0a\x0d"
```

**Explanation:** Checks for specific bad characters that may break shellcode delivery.

### Complete Flags Reference

| Flag | Description |
|------|-------------|
| **-s** | Shellcode input (raw string) |
| **-f** | Source format: `raw`, `hex`, `c`, `python`, `ruby` |
| **-o** | Output format: `raw`, `hex`, `c`, `python`, `ruby` |
| **-c** | Check for bad characters |
| **-b** | Custom bad character list (e.g., `\x00\x0a\x20`) |
| **-S** | Download from Shell-Storm by ID |
| **-a** | Architecture: `x86`, `x64` |
| **-v** | Verbose output |
| **-h** | Display help |

---

## Chapter 4: Intermediate Usage

### Shell-Storm Integration

```bash
# Download shellcode by ID from Shell-Storm
sunny -S 837
```

**Explanation:** Downloads a specific shellcode from the Shell-Storm database by its ID number, converting it to the default output format.

### Multi-Format Pipeline

```bash
# Generate, check, and convert in sequence
# Step 1: Get shellcode from shell-storm
sunny -S 837 -f raw -o hex > shell.hex

# Step 2: Check for bad characters
sunny -s "$(cat shell.hex)" -f hex -c -b "\x00\x0a"

# Step 3: Convert to Python for injection script
sunny -s "$(cat shell.hex)" -f hex -o python
```

**Explanation:** Chains Sickle commands for a complete shellcode workflow — download, validate, and convert.

### Verbose Output

```bash
sunny -s "9089c331d2" -f hex -o c -v
```

**Explanation:** Verbose mode displays detailed information about the conversion process, useful for debugging format issues.

---

## Chapter 5: Advanced Usage

### Custom Bad Character Analysis

```bash
# Comprehensive bad character check
sunny -s "fc e8 82 00 00 00 60 89 e5 31 c0 64 8b 50 30" \
  -f hex -c -b "\x00\x0a\x0d\x20\x26\x2f\x3f\x5c"
```

**Explanation:** Checks against the full set of commonly problematic characters for web exploitation, network protocols, and buffer overflow constraints.

### Shell-Storm Database Exploration

```bash
# Browse available shellcode
# Visit http://shell-storm.org/shellcode/ to find IDs
# Then download specific ones:
sunny -S 1337 -f raw -o c
```

**Explanation:** Shell-Storm provides a curated database of community shellcodes for various platforms and purposes.

### Integration with Other Tools

```bash
# Download shellcode and feed to Shellter
sunny -S 837 -f raw -o raw > /tmp/shellcode.bin
# Then use with Shellter, BDF, or custom loaders

# Convert to MSFvenom-compatible format
sunny -S 1337 -f hex -o raw | msfvenom -p linux/x86/exec -f elf -
```

**Explanation:** Sickle's output integrates with the broader toolchain for injection and delivery.

### Shell-Storm Database Common IDs

| ID | Platform | Size | Description |
|----|----------|------|-------------|
| 837 | Linux x86 | 21 bytes | execve /bin/sh |
| 1337 | Linux x86 | 23 bytes | execve /bin/sh (variant) |
| 45 | Linux x86 | 28 bytes | reverse TCP shell |
| 606 | Linux x86 | 71 bytes | bind shell |
| 885 | Linux x64 | 27 bytes | execve /bin/sh |

**Explanation:** These are commonly used shellcode entries. IDs may change — always verify against the current Shell-Storm database.

### Multi-Format Conversion Chain

```bash
# Complete conversion pipeline
# Download → Validate → Convert → Verify

# Step 1: Download from Shell-Storm
sunny -S 837 -f raw > /tmp/shellcode.raw

# Step 2: Convert to hex for analysis
sunny -s "$(cat /tmp/shellcode.raw)" -f raw -o hex > /tmp/shellcode.hex

# Step 3: Validate no bad characters
sunny -s "$(cat /tmp/shellcode.hex)" -f hex -c -b "\x00\x0a"

# Step 4: Convert to C for embedding
sunny -s "$(cat /tmp/shellcode.hex)" -f hex -o c > /tmp/shellcode.c

# Step 5: Convert to Python for scripting
sunny -s "$(cat /tmp/shellcode.hex)" -f hex -o python > /tmp/shellcode.py
```

**Explanation:** A systematic conversion pipeline ensures shellcode is validated and available in all needed formats before deployment.

### Custom Shellcode Verification

```bash
# After writing custom shellcode, verify it compiles and runs correctly

# Convert to executable
snoob --from-asm custom.asm --to-exe custom

# Run under strace to verify syscalls
snoob --to-strace custom.asm

# Check file size
ls -la custom
wc -c custom
```

**Explanation:** Verification ensures custom shellcode compiles cleanly, makes the expected system calls, and stays within size constraints.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Shellcode Sourcing

During a CTF, a buffer overflow challenge required specific shellcode. The team needed a working execve shellcode quickly:

```bash
# Search shell-storm for execve /bin/sh
# Found ID: 837 (Linux x86 execve /bin/sh - 21 bytes)

# Download and convert to Python
sunny -S 837 -f raw -o python
# Output: b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"

# Verify no bad characters
sunny -S 837 -f raw -c -b "\x00\x0a"
```

**Explanation:** Sickle's Shell-Storm integration provides instant access to proven shellcode without writing from scratch.

### Scenario 2: Payload Development with Bad Character Constraints

```bash
# Initial shellcode has null bytes - problematic for most exploits
sunny -s "\x31\xc0\x50\x68\x2f\x2f\x73\x68" -f raw -c
# Reports: \x00 not present (good!)

# Check the full shellcode
sunny -s "$(sunny -S 837 -f raw)" -f raw -c -b "\x00\x0a\x0d\x20"
```

**Explanation:** Quick bad character analysis prevents wasted time during exploit development.

### Scenario 3: Cross-Platform Shellcode Development

```bash
# Get x86 shellcode
sunny -S 837 -f raw -o hex
# 31c050682f2f7368682f62696e89e3505389e1b00bcd80

# Convert to C for embedding
sunny -S 837 -f raw -o c
# \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80

# Convert to Python for scripting
sunny -S 837 -f raw -o python
```

**Explanation:** The same shellcode is quickly converted to multiple formats for different deployment scenarios.

---

## Chapter 7: Detection & Defense

### Shell-Storm Database Monitoring

- Monitor access to shell-storm.org for reconnaissance indicators
- Track download patterns that suggest offensive preparation

### Shellcode Signatures

- Known Shell-Storm shellcodes have predictable byte patterns
- Deploy YARA rules for commonly downloaded shellcodes

### Defensive Measures

1. **Block** access to shell-storm.org and similar databases
2. **Monitor** for shellcode format conversion tools in logs
3. **Deploy** endpoint detection for common shellcode patterns
4. **Implement** memory protection (ASLR, NX/DEP)

---

## Chapter 8: Troubleshooting

### Shell-Storm Download Fails

**Cause:** Network connectivity or Shell-Storm server issues.

```bash
# Manual download alternative
wget http://shell-storm.org/api/shellcode/?id=837 -O shellcode.bin
```

**Explanation:** If the Shell-Storm API is unavailable, download shellcodes directly from the website.

### "Invalid format" Error

**Cause:** The input format doesn't match the specified format flag.

```bash
# Verify format matches actual content
sunny -s "9089c331d2" -f hex -o c    # Correct
sunny -s "\x90\x89\xc3\x31\xd2" -f c -o hex  # Correct
```

**Explanation:** Ensure the `-f` flag accurately describes the input format. Hex strings use no prefixes; C arrays use `\x` notation.

### Missing Python Dependencies

**Cause:** Required Python packages not installed.

```bash
pip3 install -r requirements.txt
# Or install manually:
pip3 install requests
```

**Explanation:** Sickle requires the `requests` library for Shell-Storm API access.

---

## Resources

- [Sickle GitHub](https://github.com/wiregh0/sickle) — Source code
- [Shell-Storm Database](http://shell-storm.org/shellcode/) — Community shellcode repository
- [ShellNoob](https://github.com/reyammer/shellnoob) — Complementary shellcode toolkit
- [Metasploit Shellcode](https://www.metasploit.com/) — MSFvenom shellcode reference

---

*Last updated: July 2026*
