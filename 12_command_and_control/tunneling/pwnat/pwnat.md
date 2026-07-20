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

## Chapter 9: Command Reference

### Server Options

| Option | Description | Default |
|--------|-------------|---------|
| `-s` | Server mode | client mode |
| `-p, --port` | Listen port | 35000 |
| `-i, --interface` | Network interface | all |
| `-v` | Verbose output | quiet |

### Client Options

| Option | Description | Default |
|--------|-------------|---------|
| `-c, --connect` | Airlock server address | none (required) |
| `-l, --local` | Local listen address | 127.0.0.1:8080 |
| `-r, --remote` | Remote target address | none (required) |
| `-p, --port` | Airlock port | 35000 |
| `-d` | Daemon mode | foreground |
| `-v` | Verbose output | quiet |

### NAT Types

| Type | Description | pwnat Support |
|------|-------------|---------------|
| Full Cone | Any external host can send | Yes |
| Restricted Cone | Only hosts seen by internal | Yes |
| Port-Restricted Cone | Like restricted + port | Yes |
| Symmetric | Different mapping per dest | No |

### UDP Hole Punching Process

```
1. Client A connects to airlock (creates NAT mapping)
2. Client B connects to airlock (creates NAT mapping)
3. Airlock shares NAT mapping info between clients
4. Clients send packets to each other's public endpoints
5. NAT devices accept packets (hole punched)
6. Direct communication established
```

## Chapter 10: Real-World Scenarios

### Scenario 1: Remote Access Without Port Forwarding

```bash
# Airlock server on VPS
pwnat -s -p 35000

# Home computer (behind NAT)
pwnat -c VPS_IP:35000 -l 127.0.0.1:8000 -r home-server:22

# Connect via tunnel
ssh -p 8000 user@127.0.0.1
```

**Explanation**: pwnat provides SSH access to a home server without port forwarding on the router. The airlock coordinates the connection.

### Scenario 2: Peer-to-Peer File Sharing

```bash
# Host A
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r host-a:22

# Host B
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r host-b:22

# Host A connects to Host B
scp -P 8000 file.txt user@127.0.0.1
```

**Explanation**: Two hosts behind NAT can establish direct connections through the airlock for file sharing.

### Scenario 3: Multi-Service Access

```bash
# Terminal 1: SSH
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22

# Terminal 2: HTTP
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:80

# Terminal 3: RDP
pwnat -c AIRLOCK:35000 -l 127.0.0.1:3389 -r TARGET:3389

# Terminal 4: MySQL
pwnat -c AIRLOCK:35000 -l 127.0.0.1:3306 -r TARGET:3306
```

**Explanation**: Multiple services can be accessed through separate pwnat tunnels.

### Scenario 4: Gaming Through NAT

```bash
# Game server behind NAT
pwnat -c AIRLOCK:35000 -l 127.0.0.1:27015 -r game-server:27015

# Game client connects to local port
# Game settings: server 127.0.0.1:27015
```

**Explanation**: pwnat enables gaming connections between NAT'd hosts without port forwarding.

### Scenario 5: VoIP Through Strict NAT

```bash
# SIP server behind NAT
pwnat -c AIRLOCK:35000 -l 127.0.0.1:5060 -r sip-server:5060

# RTP port range
pwnat -c AIRLOCK:35000 -l 127.0.0.1:10000 -r sip-server:10000

# Configure SIP client
# SIP Proxy: 127.0.0.1:5060
# RTP Port: 10000
```

**Explanation**: VoIP traffic can be tunneled through NAT for communication in restrictive environments.

## Chapter 11: Detection and Defense

### Network Detection

```bash
# Monitor for UDP traffic to airlock ports
tcpdump -i eth0 udp port 35000

# Check for pwnat signatures
# - UDP packets with specific header format
# - Traffic to known airlock IPs
# - Unusual UDP patterns
```

### Host-Based Detection

```bash
# Check for pwnat processes
ps aux | grep pwnat

# Monitor network connections
ss -ulnp | grep pwnat
netstat -ulnp | grep pwnat

# Check for airlock server
ps aux | grep "pwnat -s"
```

### IDS/IPS Signatures

```bash
# Snort rule for pwnat
alert udp any any -> any 35000 (
    msg:"pwnat Airlock Connection";
    content:"|00|";
    depth:1;
    sid:2024001; rev:1;
)
```

### Defense Recommendations

1. Monitor for unusual UDP traffic patterns
2. Block outbound UDP to non-standard ports
3. Implement egress filtering
4. Monitor for airlock servers
5. Use network segmentation

### Traffic Analysis

```bash
# Capture UDP traffic
tcpdump -i eth0 udp -w pwnat.pcap

# Analyze packet sizes
tshark -r pwnat.pcap -T fields -e udp.length | sort | uniq -c

# Check for tunnel signatures
tshark -r pwnat.pcap -Y "udp.port == 35000" -T fields -e data | head -5
```

## Chapter 12: Troubleshooting

### Connection Issues

```bash
# Test airlock connectivity
nc -uzv AIRLOCK_IP 35000

# Check firewall rules
iptables -L -n | grep 35000
ufw status | grep 35000

# Verify airlock is running
ps aux | grep "pwnat -s"

# Test UDP connectivity
echo "test" | nc -u AIRLOCK_IP 35000
```

### NAT Type Issues

```bash
# Check NAT type
# Use online NAT type checker or

# Test with pwnat verbose mode
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22 -v

# If symmetric NAT, pwnat won't work
# Consider alternative: SSH tunnel, VPN, etc.
```

### Performance Issues

```bash
# Check latency
ping -c 10 AIRLOCK_IP

# Monitor UDP packet loss
mtr AIRLOCK_IP

# Use verbose mode for diagnostics
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22 -vvv
```

### Daemon Mode

```bash
# Start in background
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22 -d

# Check if running
ps aux | grep pwnat

# Stop daemon
killall pwnat
```

### Multiple Connections

```bash
# Each tunnel needs separate instance
# Terminal 1
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22

# Terminal 2
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:80

# Verify both running
ps aux | grep pwnat
```

## Chapter 13: Comparison with Alternatives

### pwnat vs ngrok

| Feature | pwnat | ngrok |
|---------|-------|-------|
| Protocol | UDP hole punching | TCP tunnel |
| Server required | Airlock | ngrok cloud |
| Encryption | No | Yes (HTTPS) |
| Cost | Free | Free/Paid |
| Setup | Simple | Simple |
| Reliability | NAT dependent | More reliable |

### pwnat vs frp

| Feature | pwnat | frp |
|---------|-------|-----|
| Architecture | P2P via airlock | Client-server |
| Encryption | No | Yes (TLS) |
| Features | Basic tunneling | Full proxy |
| Performance | Good | Better |
| Configuration | Minimal | Extensive |

### pwnat vs SSH Tunnel

| Feature | pwnat | SSH Tunnel |
|---------|-------|------------|
| NAT traversal | Built-in | Requires port forwarding |
| Encryption | No | Yes |
| Setup | Simple | Simple |
| Reliability | NAT dependent | More reliable |
| Performance | Good | Better |

## Resources

- [pwnat GitHub Repository](https://github.com/schmittjoseph/pwnat)
- [UDP Hole Punching](https://en.wikipedia.org/wiki/UDP_hole_punching)
- [NAT Traversal Techniques](https://en.wikipedia.org/wiki/NAT_traversal)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [NAT Types RFC 3489](https://tools.ietf.org/html/rfc3489)
