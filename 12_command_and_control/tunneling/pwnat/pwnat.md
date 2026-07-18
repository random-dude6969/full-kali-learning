---
title: "pwnat - NAT Traversal"
description: "Traverse NATs and firewalls without port forwarding using UDP hole punching"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
github: "https://github.com/schmittjoseph/pwnat"
kali_install: "sudo apt install pwnat"
version: "0.3"
---

# pwnat

## Overview

pwnat traverses NATs and firewalls without port forwarding using UDP hole punching. It allows direct connections between two hosts behind NAT devices by using a third-party "airlock" server. pwnat is useful for establishing direct peer-to-peer connections in restrictive network environments.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install pwnat
```

### From Source

```bash
git clone https://github.com/schmittjoseph/pwnat.git
cd pwnat
make
sudo make install
```

### Verify Installation

```bash
pwnat -h
```

## Chapter 2: Basic Usage

### Start Airlock Server

```bash
pwnat -s -p 35000
```

**Explanation**: Starts the airlock server on port 35000. The airlock server coordinates the connection between two NAT'd clients. It must be publicly accessible.

### Connect Client

```bash
pwnat -c AIRLOCK_IP -l 127.0.0.1:8080 -r TARGET:22
```

**Explanation**: `-c` specifies the airlock server, `-l` sets the local listen address, `-r` specifies the remote target. The client connects to the airlock and establishes a tunnel to the target.

### Connect via Tunnel

```bash
ssh -p 8080 user@127.0.0.1
```

**Explanation**: Connects to the local port 8080, which is tunneled through the NAT traversal to the target's SSH service.

## Chapter 3: NAT Traversal Concepts

### How UDP Hole Punching Works

UDP hole punching uses the NAT mapping created by outbound connections to allow inbound connections. When both clients connect to the airlock, the NAT creates mappings that can be exploited for direct communication.

**Explanation**: The airlock server coordinates the exchange of NAT mapping information between clients. Once both clients have created NAT mappings, they can communicate directly without going through the airlock.

### Requirements

- Both clients behind NAT (not symmetric NAT)
- Airlock server with public IP
- UDP connectivity through NAT
- No symmetric NAT on either side

**Explanation**: UDP hole punching works with most NAT types except symmetric NAT. The airlock server must be publicly accessible to coordinate the connection.

## Chapter 4: Configuration

### Server Configuration

```bash
pwnat -s -p 35000 -i eth0
```

**Explanation**: `-s` starts the server, `-p` sets the listen port, `-i` specifies the network interface. The server listens for client connections on the specified port.

### Client Configuration

```bash
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:22
```

**Explanation**: The client connects to the airlock server and establishes a tunnel. Local port 8080 forwards to the target's port 22.

### Multiple Tunnels

```bash
# Terminal 1: SSH tunnel
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22

# Terminal 2: HTTP tunnel
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:80

# Terminal 3: RDP tunnel
pwnat -c AIRLOCK:35000 -l 127.0.0.1:3389 -r TARGET:3389
```

**Explanation**: Multiple pwnat instances can run simultaneously, each tunneling a different service. Each uses its own UDP connection for NAT traversal.

## Chapter 5: Performance

### Bandwidth

pwnat provides reasonable bandwidth for TCP connections:
- SSH: Full interactive performance
- HTTP: Web browsing speed
- File transfer: Limited by NAT translation overhead

**Explanation**: UDP hole punching introduces minimal overhead for TCP connections. Performance is generally good for interactive protocols but may degrade for bulk transfers.

### Latency

```bash
# Test latency
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22
ssh -p 8000 user@127.0.0.1 "ping -c 5 8.8.8.8"
```

**Explanation**: The initial connection may have higher latency due to NAT traversal. Once established, the connection performs similarly to a direct TCP connection.

## Chapter 6: Security

### Connection Security

pwnat tunnels are not encrypted by default. Use SSH or SSL for encrypted communication through the tunnel.

**Explanation**: The UDP hole punching mechanism does not provide encryption. Sensitive data should be encrypted at the application layer (e.g., SSH, TLS).

### NAT Traversal Detection

- Unusual UDP traffic patterns
- Connection attempts to known airlock servers
- NAT binding changes
- UDP hole punching signatures

**Explanation**: Network monitoring tools can detect NAT traversal attempts through traffic analysis. Using standard UDP patterns and common ports can reduce detection risk.

## Chapter 7: Advanced Features

### Custom Ports

```bash
pwnat -c AIRLOCK:35000 -l 127.0.0.1:5000 -r TARGET:5000
```

**Explanation**: Custom ports allow tunneling any TCP service through the NAT traversal. Both local and remote ports can be specified.

### Verbose Mode

```bash
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:22 -v
```

**Explanation**: Verbose mode shows connection establishment details, NAT traversal status, and tunnel activity. Useful for debugging connection issues.

### Daemon Mode

```bash
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:22 -d
```

**Explanation**: Daemon mode runs pwnat in the background. Useful for persistent tunnels that should survive terminal disconnection.

## Chapter 8: Practical Scenarios

### Remote Access

```bash
# Airlock on VPS
pwnat -s -p 35000

# Client behind NAT
pwnat -c AIRLOCK_IP:35000 -l 127.0.0.1:8000 -r TARGET:22

# SSH through tunnel
ssh -p 8000 user@127.0.0.1
```

**Explanation**: Establishes SSH access to a remote system through NAT traversal. The airlock server coordinates the connection between two NAT'd hosts.

### Peer-to-Peer

```bash
# Host A
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r HOST_B:22

# Host B
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r HOST_A:22
```

**Explanation**: Both hosts connect to the airlock and establish direct communication. This enables peer-to-peer connections without port forwarding.

## Resources

- [pwnat GitHub Repository](https://github.com/schmittjoseph/pwnat)
- [UDP Hole Punching](https://en.wikipedia.org/wiki/UDP_hole_punching)
- [NAT Traversal Techniques](https://en.wikipedia.org/wiki/NAT_traversal)
