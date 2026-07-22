---
tool_name: "volatility"
display_name: "Volatility"
mitre_tactic: "forensics"
mitre_tactic_id: "TA0007"
mitre_subcategory: "memory_forensics"
install_command: "sudo apt install volatility"
version: "2.6.1"
github_repo: "https://github.com/volatilityfoundation/volatility"
kali_tools_page: "https://www.kali.org/tools/volatility/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["forensics", "memory", "incident-response", "malware", "analysis"]
related_tools: ["volatility3", "rekall", "linpmem"]
---

# Volatility

## Chapter 1: Introduction & Overview

### What is Volatility?
Volatility is the world's most widely used framework for extracting digital artifacts from volatile memory (RAM) samples. It's used for incident response and malware analysis.

### History & Background
- **Created:** 2007 by Volatility Foundation
- **Maintained:** Volatility Foundation
- **License:** GNU General Public License v2
- **Purpose:** Memory forensics analysis

Volatility is the industry standard for memory forensics.

### When to Use This Tool
- **Incident response:** Analyze compromised systems
- **Malware analysis:** Extract malware from memory
- **Digital forensics:** Recover artifacts
- **CTF challenges:** Solve forensics challenges

### Key Concepts
- **Memory Image:** RAM dump file
- **Profile:** OS-specific memory layout
- **Plugin:** Analysis function
- **Artifact:** Extracted data

### Supported Platforms
| Platform | Memory Format |
|----------|---------------|
| Windows | .raw, .mem, .vmem |
| Linux | .raw, .lime, .vmem |
| Mac OS X | .raw, .vmem |

---

## Chapter 2: Installation & Setup

### System Requirements
- **RAM:** 4 GB minimum (8 GB+ recommended)
- **Disk Space:** 200 MB
- **Python:** 2.7 (Volatility 2) or 3.x (Volatility 3)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install volatility
```

#### Method 2: From Source (Volatility 2)
```bash
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility
pip install -r requirements.txt
```

#### Method 3: Volatility 3
```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
pip install -r requirements.txt
```

### Verification
```bash
volatility --version
# or
vol.py --version
```

---

## Chapter 3: Basic Usage

### First Analysis

#### Identify Image Profile
```bash
volatility -f memory.raw imageinfo
```
**Output:** Shows suggested profiles for the memory image.

#### List Processes
```bash
volatility -f memory.raw --profile=Win7SP1x64 pslist
```
**Output:** Lists all processes in memory.

### Essential Commands

#### Image Info
```bash
volatility -f memory.raw imageinfo
```
**Explanation:** Identifies the memory image profile.

#### Process List
```bash
volatility -f memory.raw --profile=Win7SP1x64 pslist
```
**Explanation:** Lists all processes.

#### Network Connections
```bash
volatility -f memory.raw --profile=Win7SP1x64 connections
```
**Explanation:** Shows network connections.

#### Hash Dump
```bash
volatility -f memory.raw --profile=Win7SP1x64 hashdump
```
**Explanation:** Extracts password hashes.

### Complete Plugin Reference

#### System Information
| Plugin | Description | Example |
|--------|-------------|---------|
| `imageinfo` | Image profile | `volatility -f mem.raw imageinfo` |
| `kdbgscan` | Kernel debugger | `volatility -f mem.raw kdbgscan` |
| `psinfo` | Process info | `volatility -f mem.raw --profile=Win7SP1x64 psinfo PID` |
| `envars` | Environment vars | `volatility -f mem.raw --profile=Win7SP1x64 envars` |

#### Process Analysis
| Plugin | Description | Example |
|--------|-------------|---------|
| `pslist` | Process list | `volatility -f mem.raw --profile=Win7SP1x64 pslist` |
| `pstree` | Process tree | `volatility -f mem.raw --profile=Win7SP1x64 pstree` |
| `psscan` | Hidden processes | `volatility -f mem.raw --profile=Win7SP1x64 psscan` |
| `psxview` | Cross-view | `volatility -f mem.raw --profile=Win7SP1x64 psxview` |
| `procexediff` | Diff processes | `volatility -f mem.raw --profile=Win7SP1x64 procexediff` |

#### Network Analysis
| Plugin | Description | Example |
|--------|-------------|---------|
| `connections` | Network connections | `volatility -f mem.raw --profile=Win7SP1x64 connections` |
| `connscan` | Connection scan | `volatility -f mem.raw --profile=Win7SP1x64 connscan` |
| `netscan` | Network scan | `volatility -f mem.raw --profile=Win7SP1x64 netscan` |
| `sockscan` | Socket scan | `volatility -f mem.raw --profile=Win7SP1x64 sockscan` |

#### Memory Dump
| Plugin | Description | Example |
|--------|-------------|---------|
| `procdump` | Dump process | `volatility -f mem.raw --profile=Win7SP1x64 procdump PID` |
| `memdump` | Dump memory | `volatility -f mem.raw --profile=Win7SP1x64 memdump PID` |
| `dumpfiles` | Dump files | `volatility -f mem.raw --profile=Win7SP1x64 dumpfiles PID` |

#### Credential Analysis
| Plugin | Description | Example |
|--------|-------------|---------|
| `hashdump` | Password hashes | `volatility -f mem.raw --profile=Win7SP1x64 hashdump` |
| `lsadump` | LSA secrets | `volatility -f mem.raw --profile=Win7SP1x64 lsadump` |
| `mimikatz` | Mimikatz output | `volatility -f mem.raw --profile=Win7SP1x64 mimikatz` |

#### Malware Analysis
| Plugin | Description | Example |
|--------|-------------|---------|
| `malfind` | Find malware | `volatility -f mem.raw --profile=Win7SP1x64 malfind` |
| `ldrscan` | Loader scan | `volatility -f mem.raw --profile=Win7SP1x64 ldrscan` |
| `threads` | Thread scan | `volatility -f mem.raw --profile=Win7SP1x64 threads` |

#### Registry Analysis
| Plugin | Description | Example |
|--------|-------------|---------|
| `hivelist` | Registry hives | `volatility -f mem.raw --profile=Win7SP1x64 hivelist` |
| `hashdump` | SAM hashes | `volatility -f mem.raw --profile=Win7SP1x64 hashdump` |
| `printkey` | Registry key | `volatility -f mem.raw --profile=Win7SP1x64 printkey KEY` |

#### File System
| Plugin | Description | Example |
|--------|-------------|---------|
| `filescan` | File scan | `volatility -f mem.raw --profile=Win7SP1x64 filescan` |
| `dumpfiles` | Dump files | `volatility -f mem.raw --profile=Win7SP1x64 dumpfiles` |
| `mftparser` | MFT parser | `volatility -f mem.raw --profile=Win7SP1x64 mftparser` |

---

## Chapter 4: Intermediate Usage

### Advanced Analysis Techniques

#### Hidden Process Detection
```bash
# Compare pslist with psscan
volatility -f mem.raw --profile=Win7SP1x64 pslist > pslist.txt
volatility -f mem.raw --profile=Win7SP1x64 psscan > psscan.txt

# Find differences
diff pslist.txt psscan.txt
```

#### Network Forensics
```bash
# Find malware communication
volatility -f mem.raw --profile=Win7SP1x64 connections > connections.txt
grep -E "ESTABLISHED|SYN" connections.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated memory analysis
MEMORY="memory.raw"
PROFILE="Win7SP1x64"
OUTPUT_DIR="analysis_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Analyzing $MEMORY with profile $PROFILE"

# Process list
volatility -f $MEMORY --profile=$PROFILE pslist > $OUTPUT_DIR/pslist.txt

# Network connections
volatility -f $MEMORY --profile=$PROFILE connections > $OUTPUT_DIR/connections.txt

# Hash dump
volatility -f $MEMORY --profile=$PROFILE hashdump > $OUTPUT_DIR/hashdump.txt

# Malware detection
volatility -f $MEMORY --profile=$PROFILE malfind > $OUTPUT_DIR/malfind.txt

echo "[+] Analysis complete: $OUTPUT_DIR"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess
import os

def volatility_analysis(memory_file, profile, plugin):
    """Run volatility plugin"""
    cmd = f"volatility -f {memory_file} --profile={profile} {plugin}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Analyze memory
output = volatility_analysis("memory.raw", "Win7SP1x64", "pslist")
print(output)
```

### Combining with Other Tools

#### With YARA
```bash
# Find malware with YARA
volatility -f mem.raw --profile=Win7SP1x64 malfind -D dump/
yara -r rules.yar dump/
```

#### With Autopsy
```bash
# Export artifacts for Autopsy
volatility -f mem.raw --profile=Win7SP1x64 dumpfiles -D exported/
# Import exported files to Autopsy
```

### Advanced Configurations

#### Custom Profiles
```bash
# Create custom profile
volatility -f mem.raw kdbgscan > kdbg.txt
# Use kdbg value for profile
```

### Performance Optimization

#### Large Memory Images
```bash
# Use specific plugins
volatility -f mem.raw --profile=Win7SP1x64 pslist

# Limit output
volatility -f mem.raw --profile=Win7SP1x64 pslist | head -50
```

---

## Chapter 5: Advanced Usage

### Advanced Analysis Techniques

#### Malware Detection
```bash
# Find injected code
volatility -f mem.raw --profile=Win7SP1x64 malfind -D malware/

# Analyze with YARA
yara -r malware_rules.yar malware/
```

#### Timeline Analysis
```bash
# Create timeline
volatility -f mem.raw --profile=Win7SP1x64 timeliner > timeline.txt

# Analyze timeline
cat timeline.txt | grep -E "CREATE|MODIFY|ACCESS"
```

### Integration in Pentest Workflow

#### Phase 1: Acquisition
```bash
# Capture memory (on target)
# Windows: winpmem, DumpIt
# Linux: LiME, avml
```

#### Phase 2: Analysis
```bash
# Identify profile
volatility -f memory.raw imageinfo

# Analyze processes
volatility -f memory.raw --profile=Win7SP1x64 pslist

# Find malware
volatility -f memory.raw --profile=Win7SP1x64 malfind
```

#### Phase 3: Reporting
```bash
# Generate report
volatility -f memory.raw --profile=Win7SP1x64 pslist > report.txt
volatility -f memory.raw --profile=Win7SP1x64 connections >> report.txt
volatility -f memory.raw --profile=Win7SP1x64 hashdump >> report.txt
```

### Advanced Configurations

#### Custom Plugins
```bash
# Use custom plugin
volatility -f mem.raw --profile=Win7SP1x64 --plugins=custom/ plugin_name
```

### Performance Optimization

#### Parallel Analysis
```bash
# Run multiple plugins in parallel
volatility -f mem.raw --profile=Win7SP1x64 pslist &
volatility -f mem.raw --profile=Win7SP1x64 connections &
wait
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Find hidden flag

**Steps:**
```bash
# 1. Identify profile
volatility -f memory.raw imageinfo

# 2. List processes
volatility -f memory.raw --profile=Win7SP1x64 pslist

# 3. Find hidden processes
volatility -f memory.raw --profile=Win7SP1x64 psscan

# 4. Dump suspicious process
volatility -f memory.raw --profile=Win7SP1x64 procdump PID

# 5. Analyze dumped process
strings dumped.exe | grep flag
```

### Scenario 2: Incident Response
**Objective:** Analyze compromised system

**Steps:**
```bash
# 1. Identify profile
volatility -f memory.raw imageinfo

# 2. List processes
volatility -f memory.raw --profile=Win7SP1x64 pslist

# 3. Find malware
volatility -f memory.raw --profile=Win7SP1x64 malfind

# 4. Check network connections
volatility -f memory.raw --profile=Win7SP1x64 connections

# 5. Extract credentials
volatility -f memory.raw --profile=Win7SP1x64 hashdump
```

### Scenario 3: Malware Analysis
**Objective:** Analyze malware behavior

**Steps:**
```bash
# 1. Find malware
volatility -f memory.raw --profile=Win7SP1x64 malfind -D malware/

# 2. Analyze process tree
volatility -f memory.raw --profile=Win7SP1x64 pstree

# 3. Check network activity
volatility -f memory.raw --profile=Win7SP1x64 connections

# 4. Dump malware
volatility -f memory.raw --profile=Win7SP1x64 procdump PID -D malware/

# 5. Analyze with YARA
yara -r malware_rules.yar malware/
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Memory Forensics

#### System Indicators
```
# Memory dump tools
# Suspicious process access
# Unusual network connections
```

#### Log Analysis
```bash
# Check for memory dump tools
# Monitor for suspicious process access
# Alert on unusual connections
```

### Mitigation Strategies

#### Anti-Forensics
```bash
# Encrypt memory
# Use anti-forensics tools
# Clear traces
```

#### Detection
```bash
# Monitor for dump tools
# Detect memory access
# Alert on anomalies
```

### Blue Team Response
1. **Monitor memory access** - Detect dump attempts
2. **Use anti-forensics** - Protect memory
3. **Encrypt sensitive data** - Prevent extraction
4. **Monitor network** - Detect C2 communication
5. **Regular analysis** - Find compromises early

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No suitable address space"
**Cause:** Wrong profile
**Solution:**
```bash
# Identify correct profile
volatility -f memory.raw imageinfo

# Use suggested profile
volatility -f memory.raw --profile=Win7SP1x64 pslist
```

#### Error: "Plugin not found"
**Cause:** Plugin name wrong
**Solution:**
```bash
# List available plugins
volatility --info | grep Plugin

# Use correct name
volatility -f memory.raw --profile=Win7SP1x64 pslist
```

### Performance Optimization

#### Slow Analysis
```bash
# Use specific plugins
volatility -f memory.raw --profile=Win7SP1x64 pslist

# Limit output
volatility -f memory.raw --profile=Win7SP1x64 pslist | head -50
```

### FAQ

**Q: Is volatility legal to use?**
A: Yes, volatility is a legitimate forensic tool. However, analyzing systems without authorization is illegal. Always get written permission before analysis.

**Q: How do I update volatility?**
A: `sudo apt update && sudo apt upgrade volatility`

**Q: What's the difference between Volatility 2 and 3?**
A: Volatility 3 is the latest version with Python 3 support and improved features.

**Q: Can I analyze Linux memory?**
A: Yes, Volatility supports Linux memory analysis with appropriate profiles.

**Q: How do I find the correct profile?**
A: Use `imageinfo` plugin: `volatility -f memory.raw imageinfo`

---

## Resources

### Official Documentation
- [Volatility Foundation](https://www.volatilityfoundation.org/)
- [GitHub Repository](https://github.com/volatilityfoundation/volatility)
- [Volatility Documentation](https://github.com/volatilityfoundation/volatility/wiki)

### Tutorials
- [Volatility Tutorial - HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/forensics)
- [Volatility Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Forensic.md)

### Related Tools
- **volatility3:** Latest version
- **rekall:** Alternative memory forensics
- **linpmem:** Linux memory acquisition
- **winpmem:** Windows memory acquisition

### Practice Resources
- [Volatility Labs](https://www.volatility-labs.blogspot.com/)
- [Memory Forensics CTF](https://www.magnetuserdays.com/)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
