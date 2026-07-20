---
title: "udptunnel - UDP Encapsulation over TCP"
description: "Tunnel UDP packets through TCP connections for bypassing UDP-blocking firewalls"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "udptunnel"
kali_install: "sudo apt install udptunnel"
version: "1.1"
---

# udptunnel

## Overview

udptunnel encapsulates UDP packets within TCP connections, allowing UDP-based services to traverse networks that block UDP traffic. It creates a TCP tunnel that carries UDP packets, enabling services like DNS, SIP, and gaming protocols to work through TCP-only firewalls.

udptunnel works by creating a TCP connection between a client and server. The client captures UDP packets on a specified port, encapsulates them in the TCP stream, and sends them to the server. The server receives the TCP stream, extracts the UDP packets, and forwards them to their destinations. Responses follow the reverse path.

This approach is useful in environments where UDP is blocked but TCP is permitted. It can also be used to traverse NAT devices that may not properly handle UDP traffic, and to provide reliability for UDP-based protocols that are sensitive to packet loss.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install udptunnel
```

### From Source

```bash
wget https://www.wains.be/pub/networking/udptunnel-1.1.tar.gz
tar xzf udptunnel-1.1.tar.gz
cd udptunnel-1.1
./configure
make
sudo make install
```

### Verify Installation

```bash
udptunnel -h
```

## Chapter 2: Basic Usage

### Start Server

```bash
udptunnel -s -p 5000
```

**Explanation**: Starts the udptunnel server listening on TCP port 5000. The server receives encapsulated UDP packets and forwards them to their destinations.

### Start Client

```bash
udptunnel -c SERVER_IP:5000 -l 5353 -d TARGET_IP:53
```

**Explanation**: `-c` specifies the server address, `-l` sets the local UDP listen port, `-d` specifies the destination. UDP packets sent to localhost:5353 are tunneled through TCP to the target's DNS port.

### Test Tunnel

```bash
dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: Sends a DNS query through the tunnel. The query is encapsulated in TCP, sent to the udptunnel server, and forwarded as UDP to the target DNS server.

### Verify Connection

```bash
# Check if tunnel is listening
ss -ulnp | grep 5353

# Check TCP connection
ss -tnp | grep 5000
```

**Explanation**: Verify the tunnel is running by checking the listening UDP port and the TCP connection to the server.

## Chapter 3: DNS Tunneling

### DNS over TCP

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 5353 -d DNS_SERVER:53

# Query
dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: DNS queries are typically UDP, but some networks block UDP DNS. udptunnel carries DNS queries over TCP, bypassing UDP restrictions.

### DNS with Multiple Servers

```bash
# Client with multiple DNS servers
udptunnel -c SERVER:5000 -l 5353 -d DNS1:53
udptunnel -c SERVER:5000 -l 5354 -d DNS2:53
```

**Explanation**: Multiple udptunnel instances can tunnel to different DNS servers, each on a different local port.

### DNS over TCP with Encryption

```bash
# Use stunnel for additional encryption
stunnel -d 5353 -r 127.0.0.1:5354 -c

# udptunnel client
udptunnel -c SERVER:5000 -l 5354 -d DNS:53
```

**Explanation**: Combine udptunnel with stunnel for additional encryption. The DNS query goes: dig -> stunnel -> udptunnel -> TCP -> server -> UDP -> DNS.

## Chapter 4: VoIP Tunneling

### SIP over TCP

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 5060 -d SIP_SERVER:5060
```

**Explanation**: SIP signaling typically uses UDP port 5060. udptunnel encapsulates SIP packets in TCP, allowing VoIP traffic through TCP-only networks.

### RTP over TCP

```bash
# Server
udptunnel -s -p 5001

# Client
udptunnel -c SERVER:5001 -l 10000 -d SIP_SERVER:10000
udptunnel -c SERVER:5001 -l 10001 -d SIP_SERVER:10001
```

**Explanation**: RTP media traffic uses UDP ports 10000-20000. Tunnel each RTP stream separately through TCP.

### Complete VoIP Setup

```bash
# Tunnel SIP signaling
udptunnel -c SERVER:5000 -l 5060 -d SIP_SERVER:5060

# Tunnel RTP media (multiple ports)
for port in $(seq 10000 10010); do
    udptunnel -c SERVER:5001 -l $port -d SIP_SERVER:$port &
done
```

**Explanation**: A complete VoIP setup requires tunneling both SIP signaling and RTP media. Each RTP stream needs its own tunnel.

## Chapter 5: Configuration

### Command Line Options

```bash
udptunnel -s -p 5000 -b 1000
```

**Explanation**: `-s` starts server mode, `-p` sets the listen port, `-b` sets the bandwidth limit in bytes per second.

### Client Options

```bash
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060 -b 5000
```

**Explanation**: The `-b` flag limits bandwidth to prevent saturating the TCP connection. This is useful for maintaining tunnel stability.

### Server Options

```bash
udptunnel -s -p 5000 -b 10000 -d
```

**Explanation**: The `-d` flag enables debug mode, showing detailed packet information. The `-b` flag limits bandwidth.

### Configuration File

```
# /etc/udptunnel.conf
server_port=5000
bandwidth_limit=10000
debug=0
```

**Explanation**: udptunnel can be configured via command line options. Some implementations support configuration files for persistent settings.

## Chapter 6: Performance

### Bandwidth Limitations

TCP encapsulation of UDP introduces:
- TCP overhead (headers, acknowledgments)
- Head-of-line blocking
- Congestion control
- Retransmission delays

**Explanation**: UDP's real-time properties are degraded when tunneled through TCP. Time-sensitive applications may experience latency and jitter.

### Latency

```bash
# Measure tunnel latency
time dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: TCP encapsulation adds latency due to connection management and congestion control. The additional latency depends on network conditions and tunnel configuration.

### Throughput Testing

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 12345 -d SERVER:12345

# Test with iperf (UDP mode)
iperf -s -u -p 12345 &
iperf -c 127.0.0.1 -p 12345 -u -b 10M
```

**Explanation**: Test tunnel throughput using iperf in UDP mode. The traffic is encapsulated in TCP by udptunnel.

### Optimization Tips

```bash
# Use larger MTU if possible
udptunnel -c SERVER:5000 -l 5353 -d DNS:53 -m 1500

# Increase TCP buffer size
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
```

**Explanation**: Optimize tunnel performance by adjusting MTU and buffer sizes. Larger MTU reduces overhead per packet.

## Chapter 7: Security

### Traffic Analysis

udptunnel traffic appears as:
- TCP connections on the tunnel port
- Regular data flow patterns
- No UDP signatures visible to network monitors

**Explanation**: The TCP encapsulation hides the original UDP traffic. Network monitors see only TCP connections, not the underlying UDP protocols.

### Firewall Evasion

```bash
# Bypass UDP blocking
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060
```

**Explanation**: Networks that block UDP but permit TCP can be traversed using udptunnel. The TCP tunnel carries UDP packets through the firewall.

### Detection Methods

- High volume of TCP traffic on single port
- Unusual TCP payload patterns
- Long-lived TCP connections
- Correlation of TCP and UDP traffic patterns

**Explanation**: While udptunnel hides UDP traffic within TCP, sophisticated monitoring can detect the tunnel through traffic analysis.

### Evasion Techniques

```bash
# Use standard HTTPS port
udptunnel -s -p 443

# Use standard DNS port
udptunnel -c SERVER:443 -l 5353 -d DNS:53
```

**Explanation**: Using standard ports like 443 (HTTPS) or 53 (DNS) makes the tunnel traffic look more legitimate.

## Chapter 8: Advanced Features

### Multiple Tunnels

```bash
# Terminal 1: DNS tunnel
udptunnel -c SERVER:5000 -l 5353 -d DNS:53

# Terminal 2: SIP tunnel
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060

# Terminal 3: NTP tunnel
udptunnel -c SERVER:5000 -l 123 -d NTP:123
```

**Explanation**: Multiple udptunnel instances can run simultaneously, each tunneling a different UDP service through TCP.

### Bandwidth Control

```bash
udptunnel -c SERVER:5000 -l 5353 -d DNS:53 -b 10000
```

**Explanation**: Bandwidth limiting prevents the tunnel from consuming all available bandwidth. This is important for maintaining tunnel stability and not impacting other network services.

### Connection Persistence

```bash
# Server with persistent connections
udptunnel -s -p 5000 -t 3600

# Client with keepalive
udptunnel -c SERVER:5000 -l 5353 -d DNS:53 -k
```

**Explanation**: Some implementations support connection persistence and keepalive options to maintain long-lived tunnels.

### Logging

```bash
# Enable logging
udptunnel -s -p 5000 -l /var/log/udptunnel.log

# Debug mode
udptunnel -s -p 5000 -d 3
```

**Explanation**: Enable logging for troubleshooting and monitoring. Debug levels provide increasing amounts of detail.

## Chapter 9: Practical Scenarios

### VoIP Through Firewall

```bash
# Server on public IP
udptunnel -s -p 5000

# Client behind firewall
udptunnel -c PUBLIC_IP:5000 -l 5060 -d SIP_SERVER:5060

# Configure SIP client to use 127.0.0.1:5060
```

**Explanation**: SIP and RTP traffic can be tunneled through TCP-only firewalls, enabling VoIP connectivity in restricted environments.

### Gaming Protocol

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 27015 -d GAME_SERVER:27015
```

**Explanation**: Many game protocols use UDP for low-latency communication. udptunnel carries game traffic through TCP when UDP is blocked.

### DNS Backup

```bash
# Primary DNS over UDP (normal)
dig @dns-server example.com

# Backup DNS over TCP tunnel
udptunnel -c SERVER:5000 -l 5353 -d dns-server:53
dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: Use udptunnel as a backup DNS channel when UDP DNS is blocked or unreliable.

### Network Monitoring

```bash
# Tunnel SNMP over TCP
udptunnel -c SERVER:5000 -l 161 -d MONITOR:161

# Query SNMP through tunnel
snmpwalk -v2c -c public 127.0.0.1
```

**Explanation**: SNMP typically uses UDP. Tunnel it through TCP for monitoring across restrictive networks.

## Chapter 10: Detection and Defense

### Network Signatures

- High volume of TCP traffic on single port
- Unusual TCP payload patterns
- Long-lived TCP connections
- Correlation of TCP and UDP traffic

### IDS/IPS Detection

```
# Suricata rule for udptunnel
alert tcp any any -> any any (msg:"UDP over TCP Tunnel"; \
  flow:to_server,established; \
  dsize:>100; \
  threshold:type both,track by_src, count 100, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor TCP connections
ss -tnp | grep -E "ESTAB.*:5000"

# Check for unusual payloads
tcpdump -i eth0 port 5000 -w udptunnel.pcap

# Analyze traffic patterns
tshark -r udptunnel.pcap -Y "tcp.port == 5000"
```

### Defense Recommendations

1. Monitor for high-volume TCP traffic on single ports
2. Implement deep packet inspection
3. Use application-layer firewalls
4. Monitor for long-lived TCP connections
5. Implement network segmentation

## Chapter 11: Troubleshooting

### Connection Refused

```bash
# Check server is running
ps aux | grep udptunnel

# Check port is listening
ss -tlnp | grep 5000

# Check firewall rules
iptables -L -n | grep 5000
```

### DNS Resolution Issues

```bash
# Test DNS through tunnel
dig @127.0.0.1 -p 5353 example.com

# Check DNS server
dig @dns-server example.com

# Verify tunnel is forwarding
tcpdump -i lo port 5353
```

### Performance Issues

```bash
# Check bandwidth
iperf -c SERVER -p 5000

# Monitor TCP retransmissions
ss -ti | grep retrans

# Adjust buffer sizes
sysctl -w net.core.rmem_max=16777216
```

### Packet Loss

```bash
# Test UDP connectivity
nc -uzv 127.0.0.1 5353

# Check TCP health
ss -tnp | grep 5000

# Monitor packet counts
tcpdump -i eth0 port 5000 -c 100
```

### Bandwidth Monitoring

```bash
# Monitor tunnel bandwidth
iftop -i tun0 -f "port 5000"

# Check TCP throughput
iperf3 -c SERVER -p 5000 -t 10

# Monitor connection count
ss -s
```

## Resources

- [udptunnel Homepage](https://www.kali.org/tools/udptunnel/)
- [UDP Protocol RFC 768](https://tools.ietf.org/html/rfc768)
- [TCP Protocol RFC 793](https://tools.ietf.org/html/rfc793)
- [DNS over TCP RFC 7766](https://tools.ietf.org/html/rfc7766)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
