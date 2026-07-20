---
title: "ptunnel - ICMP Tunneling"
description: "Tunnel TCP connections through ICMP echo requests and replies"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "ptunnel"
kali_install: "sudo apt install ptunnel"
version: "0.61"
---

# ptunnel

## Overview

ptunnel tunnels TCP connections through ICMP echo (ping) requests and replies. It encapsulates TCP data within ICMP packets, bypassing firewalls that allow ICMP traffic but block TCP. ptunnel is useful in restrictive environments where only ICMP is permitted for outbound connectivity.

ptunnel works by creating a proxy (relay) that listens for ICMP packets and a client that encapsulates TCP data in ICMP echo requests. The proxy receives the ICMP packets, extracts the TCP data, and forwards it to the destination. Responses follow the reverse path through ICMP echo replies.

The tool is particularly useful in highly restrictive network environments where only ICMP echo requests are permitted. It can tunnel SSH, HTTP, and other TCP-based protocols through ICMP, providing connectivity when all other protocols are blocked.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install ptunnel
```

### From Source

```bash
git clone https://github.com/lwang-ssh/ptunnel.git
cd ptunnel
./configure
make
sudo make install
```

### Verify Installation

```bash
ptunnel -h
```

### Check Dependencies

```bash
# ptunnel requires root for ICMP
# Check if running as root
whoami

# Check ICMP permissions
ping -c 1 8.8.8.8
```

## Chapter 2: Basic Usage

### Start Proxy (Relay)

```bash
sudo ptunnel
```

**Explanation**: Starts ptunnel in proxy mode, listening for ICMP tunnel connections. The proxy relays TCP connections between the client and target. Must run as root for ICMP access.

### Connect Client

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET_IP -dp 22
```

**Explanation**: `-p` specifies the proxy address, `-lp` sets the local listen port, `-da` specifies the destination address, `-dp` sets the destination port. This creates a local port 8000 that tunnels to the target's SSH port.

### Connect via Tunnel

```bash
ssh -p 8000 user@127.0.0.1
```

**Explanation**: Connects to the local port 8000, which is tunneled through ICMP to the target's SSH service. The SSH connection appears to come from the proxy's network.

### Verify Connection

```bash
# Check if local port is listening
ss -tlnp | grep 8000

# Test connectivity
nc -zv 127.0.0.1 8000
```

**Explanation**: Verify the tunnel is working by checking the local port and testing connectivity.

## Chapter 3: Authentication

### Password Protection

```bash
# Proxy
sudo ptunnel -pass mypassword

# Client
ptunnel -p PROXY_IP -pass mypassword -lp 8000 -da TARGET:22 -dp 22
```

**Explanation**: The `-pass` flag sets a password for authentication. Both proxy and client must use the same password. This prevents unauthorized use of the tunnel.

### Key-Based Authentication

```bash
# Generate key
openssl rand -hex 16 > tunnel.key

# Proxy
sudo ptunnel -pass $(cat tunnel.key)

# Client
ptunnel -p PROXY_IP -pass $(cat tunnel.key) -lp 8000 -da TARGET:22 -dp 22
```

**Explanation**: Use a random key for stronger authentication. Store the key securely and distribute it through secure channels.

## Chapter 4: Tunnel Configuration

### Custom Ports

```bash
ptunnel -p PROXY_IP -lp 9000 -da TARGET_IP -dp 3389
```

**Explanation**: Tunnels RDP traffic (port 3389) through ICMP to the proxy. Local port 9000 forwards to the target's RDP service.

### Multiple Tunnels

```bash
# Terminal 1: SSH tunnel
ptunnel -p PROXY_IP -lp 8000 -da TARGET_IP -dp 22

# Terminal 2: HTTP tunnel
ptunnel -p PROXY_IP -lp 8080 -da TARGET_IP -dp 80

# Terminal 3: RDP tunnel
ptunnel -p PROXY_IP -lp 3389 -da TARGET_IP -dp 3389
```

**Explanation**: Multiple ptunnel instances can run simultaneously, each tunneling a different service. Each instance uses its own ICMP connection.

### Custom Interface

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET_IP -dp 22 -i eth1
```

**Explanation**: The `-i` flag specifies the network interface to use. Useful when multiple network interfaces are available.

### Custom ICMP Identifier

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -id 12345
```

**Explanation**: The `-id` flag sets a custom ICMP identifier field. This can help distinguish tunnel traffic from legitimate ICMP traffic.

## Chapter 5: Performance

### Bandwidth Limitations

ICMP tunneling has significant bandwidth limitations:
- ICMP packet size limits (typically 64KB max)
- Network latency for each ping/pong exchange
- ICMP rate limiting on many networks
- Reliability depends on ICMP being permitted

**Explanation**: ICMP tunneling is suitable for lightweight C2 channels but not for bulk data transfers. Expect throughput in the range of 10-100 kbps depending on network conditions.

### MTU Considerations

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -mtu 512
```

**Explanation**: The `-mtu` flag sets the Maximum Transmission Unit for the tunnel. Lower MTU values may improve reliability on networks with strict ICMP size limits.

### Latency

```bash
# Test ICMP latency
ping -c 10 PROXY_IP

# Test tunnel latency
time ssh -p 8000 user@127.0.0.1 "echo test"
```

**Explanation**: ICMP tunneling adds latency due to the ping/pong exchange. The additional latency depends on network conditions and ICMP response times.

### Throughput Testing

```bash
# Test SSH throughput
dd if=/dev/zero bs=1M count=10 | ssh -p 8000 user@127.0.0.1 "cat > /dev/null"

# Test HTTP throughput
curl -o /dev/null -w "Speed: %{speed_download} bytes/sec\n" http://127.0.0.1:8080/large-file
```

**Explanation**: Measure tunnel throughput using real-world applications. ICMP tunneling bandwidth is typically limited to tens of kbps.

## Chapter 6: Security

### ICMP Traffic Analysis

ICMP tunneling can be detected by:
- High volume of ICMP traffic
- Unusual ICMP payload sizes
- ICMP traffic to unusual destinations
- Timing analysis of ping/pong patterns

**Explanation**: ICMP traffic is commonly monitored in enterprise environments. Using standard ICMP patterns and limiting tunnel activity can reduce detection risk.

### ICMP Filtering

Many networks restrict ICMP:
- Some block ICMP echo entirely
- Others limit ICMP rate
- Some inspect ICMP payloads
- Firewall rules may filter by ICMP type

**Explanation**: Before using ptunnel, verify that ICMP echo requests are permitted through the network. Test with standard `ping` to confirm connectivity.

### Detection Methods

```bash
# Monitor ICMP traffic
tcpdump -i eth0 icmp -w icmp.pcap

# Analyze ICMP payloads
tshark -r icmp.pcap -Y "icmp.type == 8" -T fields -e data | head -5

# Check for unusual ICMP patterns
tshark -r icmp.pcap -Y "icmp.type == 8" -T fields -e icmp.ident | sort | uniq -c
```

**Explanation**: ICMP tunnel traffic can be detected through traffic analysis. Monitoring for unusual ICMP patterns can identify tunnel activity.

### Evasion Techniques

```bash
# Use standard ICMP patterns
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -id 0

# Limit tunnel activity
# Only transfer data when needed

# Use random delays
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -delay 100
```

**Explanation**: Reduce detection risk by using standard ICMP patterns and limiting tunnel activity.

## Chapter 7: Advanced Features

### Verbose Mode

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -v
```

**Explanation**: The `-v` flag enables verbose output, showing ICMP packet details and tunnel establishment process. Useful for debugging connection issues.

### Daemon Mode

```bash
sudo ptunnel -daemon
```

**Explanation**: Runs ptunnel as a daemon in the background. Useful for long-running tunnel services.

### Log File

```bash
sudo ptunnel -log /var/log/ptunnel.log
```

**Explanation**: Enable logging to a file for troubleshooting and monitoring.

### Connection Limit

```bash
sudo ptunnel -maxconn 10
```

**Explanation**: Limit the number of concurrent connections through the tunnel. This prevents resource exhaustion on the proxy.

## Chapter 8: Practical Scenarios

### SSH Through ICMP

```bash
# Proxy
sudo ptunnel

# Client
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22

# Connect
ssh -p 8000 user@127.0.0.1
```

**Explanation**: The most common use case is tunneling SSH through ICMP. This provides remote access when TCP connections are blocked but ICMP is allowed.

### Pivoting

```bash
# SSH through ICMP
ptunnel -p PROXY_IP -lp 8000 -da PIVOT:22 -dp 22

# SOCKS proxy through SSH
ssh -D 1080 -p 8000 user@127.0.0.1

# Scan internal network
proxychains4 nmap -sT 10.10.10.0/24
```

**Explanation**: ptunnel can be combined with SSH SOCKS proxy for multi-hop pivoting. Traffic flows: client -> ICMP -> proxy -> TCP -> pivot -> SOCKS -> internal network.

### Web Application Access

```bash
# HTTP through ICMP
ptunnel -p PROXY_IP -lp 8080 -da WEB_SERVER:80 -dp 80

# Access web application
curl http://127.0.0.1:8080

# Or use browser
# Set proxy to 127.0.0.1:8080
```

**Explanation**: Access web applications through ICMP when TCP is blocked.

### Database Access

```bash
# MySQL through ICMP
ptunnel -p PROXY_IP -lp 3306 -da DB_SERVER:3306 -dp 3306

# Connect to database
mysql -h 127.0.0.1 -P 3306 -u user -p
```

**Explanation**: Access databases through ICMP when direct TCP connections are blocked.

## Chapter 9: Detection and Defense

### Network Signatures

- High volume of ICMP echo requests
- Large ICMP payloads
- Unusual ICMP identifier patterns
- Long-lived ICMP sessions

### IDS/IPS Detection

```
# Suricata rule for ICMP tunnel
alert icmp any any -> any any (msg:"ICMP Tunnel Detected"; \
  dsize:>64; \
  threshold:type both,track by_src, count 100, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor ICMP traffic
tcpdump -i eth0 icmp -w icmp.pcap

# Analyze ICMP patterns
tshark -r icmp.pcap -Y "icmp.type == 8"

# Check for large ICMP payloads
tshark -r icmp.pcap -Y "icmp.type == 8" -T fields -e data.len | sort -n
```

### Defense Recommendations

1. Monitor for high-volume ICMP traffic
2. Limit ICMP rate at network perimeter
3. Inspect ICMP payloads for anomalies
4. Implement ICMP type filtering
5. Use network segmentation

## Chapter 10: Troubleshooting

### Connection Issues

```bash
# Test ICMP connectivity
ping -c 5 PROXY_IP

# Check proxy is running
ps aux | grep ptunnel

# Check firewall rules
iptables -L -n | grep icmp
```

### Permission Issues

```bash
# ptunnel requires root for ICMP
sudo ptunnel

# Or set capabilities
sudo setcap cap_net_raw+ep /usr/bin/ptunnel
```

### Performance Issues

```bash
# Check ICMP latency
ping -c 10 PROXY_IP

# Monitor tunnel throughput
iftop -i icmp

# Adjust MTU
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -mtu 512
```

### Tunnel Not Working

```bash
# Check ICMP is permitted
ping -c 1 PROXY_IP

# Test with verbose mode
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -vvv

# Check proxy logs
tail -f /var/log/ptunnel.log
```

## Chapter 11: Comparison with Alternatives

### ptunnel vs icmpdoor

| Feature | ptunnel | icmpdoor |
|---------|---------|----------|
| Encryption | No | Yes |
| Authentication | Password | No |
| Protocol | TCP over ICMP | Raw ICMP |
| Performance | Good | Better |
| Detection | Medium | Low |

### ptunnel vs icmpsh

| Feature | ptunnel | icmpsh |
|---------|---------|--------|
| Direction | Bidirectional | Client-server |
| TCP tunneling | Yes | No |
| Shell support | No | Yes |
| Encryption | No | No |
| Setup | Simple | Simple |

### ptunnel vs Hans

| Feature | ptunnel | Hans |
|---------|---------|------|
| Encryption | No | Yes (Blowfish) |
| IPv4 tunneling | No | Yes |
| Performance | Good | Better |
| Configuration | Simple | Complex |
| NAT traversal | No | Yes |

## Chapter 12: Command Reference

### Proxy Options

| Option | Description | Default |
|--------|-------------|---------|
| `-daemon` | Run as daemon | foreground |
| `-pass` | Authentication password | none |
| `-log` | Log file path | syslog |
| `-maxconn` | Maximum connections | unlimited |

### Client Options

| Option | Description | Default |
|--------|-------------|---------|
| `-p` | Proxy address | required |
| `-lp` | Local listen port | required |
| `-da` | Destination address | required |
| `-dp` | Destination port | required |
| `-pass` | Authentication password | none |
| `-mtu` | Maximum Transmission Unit | 1400 |
| `-id` | ICMP identifier | auto |
| `-v` | Verbose output | quiet |

### ICMP Types Used

| Type | Code | Description |
|------|------|-------------|
| 8 | 0 | Echo request (ping) |
| 0 | 0 | Echo reply |

## Resources

- [ptunnel Homepage](https://www.kali.org/tools/ptunnel/)
- [ICMP Tunneling Techniques](https://en.wikipedia.org/wiki/ICMP_tunnel)
- [RFC 792 - ICMP](https://tools.ietf.org/html/rfc792)
- [ptunnel GitHub](https://github.com/lwang-ssh/ptunnel)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [Network Security RFC 4890](https://tools.ietf.org/html/rfc4890)
