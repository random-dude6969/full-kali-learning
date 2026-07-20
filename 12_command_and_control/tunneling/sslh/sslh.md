---
title: "sslh - Protocol Multiplexer"
description: "Share a single port for multiple protocols by detecting and routing connections based on protocol"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "sslh"
kali_install: "sudo apt install sslh"
version: "2.1.6"
---

# sslh

## Overview

sslh (SSL/SSH/HTTP/HTTPS multiplexer) listens on a single port and detects the incoming protocol, then routes the connection to the appropriate backend service. It can multiplex TLS, SSH, HTTP, and other protocols on a single port (typically 443), making different services appear as a single HTTPS endpoint.

sslh works by examining the first bytes of each connection to determine the protocol. It then forwards the connection to the appropriate backend service. This process is transparent to both the client and the server — no modifications are required to either.

The tool is particularly useful for exposing multiple services through a single firewall port. By running sslh on port 443, you can serve HTTPS, SSH, and other protocols while appearing as a single HTTPS server to network monitors.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sslh
```

### From Source

```bash
wget https://github.com/yrutschle/sslh/archive/refs/tags/v2.1.6.tar.gz
tar xzf v2.1.6.tar.gz
cd sslh-2.1.6
make
sudo make install
```

### Verify Installation

```bash
sslh -V
```

### Check Service Status

```bash
systemctl status sslh
```

## Chapter 2: Basic Usage

### Start Multiplexer

```bash
sslh -p 0.0.0.0:443 --ssl 127.0.0.1:4443 --ssh 127.0.0.1:22 --http 127.0.0.1:80
```

**Explanation**: Listens on port 443 and routes TLS to port 4443, SSH to port 22, and HTTP to port 80. All three services share port 443.

### Start with systemd

```bash
sudo systemctl start sslh
```

**Explanation**: Starts sslh as a systemd service. The default configuration is in `/etc/sslh/sslh.conf`.

### Enable on Boot

```bash
sudo systemctl enable sslh
```

**Explanation**: Enables sslh to start automatically on system boot.

### Stop Service

```bash
sudo systemctl stop sslh
```

**Explanation**: Stops the sslh service.

### Restart Service

```bash
sudo systemctl restart sslh
```

**Explanation**: Restarts the sslh service with the current configuration.

## Chapter 3: Configuration

### Configuration File

```
# /etc/sslh/sslh.conf
listen:
{
    host: 0.0.0.0;
    port: 443;
};

protocols:
(
    { name: "ssl"; host: "127.0.0.1"; port: "4443"; },
    { name: "ssh"; host: "127.0.0.1"; port: "22"; },
    { name: "http"; host: "127.0.0.1"; port: "80"; }
);
```

**Explanation**: The configuration file defines listening addresses and protocol routing rules. Each protocol has a name, backend host, and port.

### Command Line Options

```bash
sslh -p 0.0.0.0:443 \
    --ssl 127.0.0.1:4443 \
    --ssh 127.0.0.1:22 \
    --http 127.0.0.1:80 \
    --anyprot 127.0.0.1:4444
```

**Explanation**: Command line options override the configuration file. `--anyprot` is a catch-all for unrecognized protocols.

### Multiple Listen Addresses

```
listen:
(
    { host: 0.0.0.0; port: 443; },
    { host: 0.0.0.0; port: 8443; }
);
```

**Explanation**: sslh can listen on multiple addresses and ports simultaneously.

### Protocol List

```
protocols:
(
    { name: "ssl"; host: "127.0.0.1"; port: "4443"; },
    { name: "ssh"; host: "127.0.0.1"; port: "22"; },
    { name: "http"; host: "127.0.0.1"; port: "80"; },
    { name: "openvpn"; host: "127.0.0.1"; port: "1194"; },
    { name: "anyprot"; host: "127.0.0.1"; port: "4444"; }
);
```

**Explanation**: The protocol list defines which protocols are supported and where to route them. The order matters — sslh checks protocols in the order they appear.

## Chapter 4: Protocol Detection

### How sslh Works

sslh examines the first bytes of each connection to determine the protocol:
- **TLS**: Starts with `\x16\x03` (TLS handshake)
- **SSH**: Starts with `SSH-`
- **HTTP**: Starts with HTTP methods (GET, POST, etc.)

**Explanation**: Protocol detection is based on the initial bytes sent by the client. This allows sslh to route connections without modifying the traffic.

### Protocol Compatibility

| Protocol | Detection Method | Port |
|----------|-----------------|------|
| TLS/SSL | `\x16\x03` | 443 |
| SSH | `SSH-` | 22 |
| HTTP | HTTP methods | 80 |
| OpenVPN | `\x00\x0e` | 1194 |
| SNI | TLS ClientHello | 443 |

**Explanation**: sslh supports many protocols through protocol-specific detection. The SNI extension allows routing based on the hostname in TLS connections.

### SNI Detection

```
# SNI-based routing
protocols:
(
    { name: "ssl"; host: "127.0.0.1"; port: "4443"; sni: "web.example.com"; },
    { name: "ssl"; host: "127.0.0.1"; port: "8443"; sni: "api.example.com"; }
);
```

**Explanation**: SNI-based routing directs TLS connections to different backends based on the hostname. This allows hosting multiple HTTPS services on different domains.

### Protocol Detection Timeout

```
# Increase timeout for slow clients
timeout: 5;
```

**Explanation**: The timeout controls how long sslh waits for protocol detection. Increase this value for slow or unreliable connections.

## Chapter 5: Practical Scenarios

### HTTPS and SSH on Port 443

```bash
sslh -p 0.0.0.0:443 \
    --ssl 127.0.0.1:4443 \
    --ssh 127.0.0.1:22
```

**Explanation**: Serves both HTTPS and SSH on port 443. The HTTPS server handles TLS traffic, while SSH connections are routed to the SSH daemon. This makes SSH appear as HTTPS to network monitors.

### Multiple Services

```bash
sslh -p 0.0.0.0:443 \
    --ssl 127.0.0.1:4443 \
    --ssh 127.0.0.1:22 \
    --http 127.0.0.1:80 \
    --openvpn 127.0.0.1:1194
```

**Explanation**: Multiplexes TLS, SSH, HTTP, and OpenVPN on a single port. This is useful for exposing multiple services through a single firewall port.

### Behind Load Balancer

```bash
sslh -p 0.0.0.0:443 \
    --ssl 127.0.0.1:4443 \
    --ssh 127.0.0.1:22
```

**Explanation**: sslh can sit behind a load balancer, multiplexing protocols before they reach backend servers. This allows a single load balancer port to serve multiple protocols.

### Behind Reverse Proxy

```
# nginx configuration
server {
    listen 443 ssl;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

**Explanation**: sslh can work with reverse proxies like nginx. sslh handles protocol detection, and nginx handles HTTP routing.

## Chapter 6: Security

### Protocol Confusion Attacks

Protocol detection can be bypassed by sending crafted packets that mimic the expected protocol. This can redirect connections to unintended services.

**Explanation**: sslh's protocol detection is heuristic-based and can be fooled by carefully crafted initial bytes. Using SNI-based routing and protocol-specific headers provides better security.

### Traffic Analysis

sslh traffic can be analyzed by:
- Initial packet inspection for protocol detection
- Timing analysis of connection establishment
- Volume analysis of multiplexed traffic
- SNI hostname extraction from TLS

**Explanation**: While sslh hides multiple services behind one port, traffic analysis can still detect the presence of different protocols.

### Access Control

```
# /etc/sslh/sslh.conf
access:
{
    allow: "192.168.1.0/24";
    deny: "0.0.0.0/0";
};
```

**Explanation**: Access control restricts which clients can connect to sslh. This provides basic security by limiting access to specific networks.

### Logging

```
# /etc/sslh/sslh.conf
log:
{
    file: "/var/log/sslh.log";
};
```

**Explanation**: Enable logging to track connections and detect potential attacks.

## Chapter 7: Advanced Features

### SNI Routing

```
protocols:
(
    { name: "ssl"; host: "127.0.0.1"; port: "4443"; sni: "web.example.com"; },
    { name: "ssl"; host: "127.0.0.1"; port: "8443"; sni: "api.example.com"; }
);
```

**Explanation**: SNI-based routing directs TLS connections to different backends based on the hostname. This allows hosting multiple HTTPS services on different domains.

### Transparent Proxy

```bash
sslh -p 0.0.0.0:443 --transparent
```

**Explanation**: Transparent proxy mode preserves the original client IP address. This requires iptables configuration to redirect traffic to sslh.

### SSLH Configuration with iptables

```bash
# Redirect traffic to sslh
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 443

# Allow sslh
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

**Explanation**: Use iptables to redirect traffic to sslh for transparent proxying.

### Custom Protocol Detection

```
protocols:
(
    { name: "custom"; host: "127.0.0.1"; port: "9999"; regex: "^CUSTOM"; }
);
```

**Explanation**: sslh can detect custom protocols using regex patterns. This allows routing non-standard protocols.

## Chapter 8: Performance

### Connection Handling

sslh uses a single-threaded, event-driven architecture. It handles thousands of concurrent connections efficiently.

**Explanation**: The event-driven model allows sslh to manage many connections without creating threads for each one. This provides good performance with low resource usage.

### Buffer Sizing

```bash
sslh -p 0.0.0.0:443 \
    --ssl 127.0.0.1:4443 \
    --ssh 127.0.0.1:22 \
    -B 65536
```

**Explanation**: The `-B` flag sets the buffer size for each connection. Larger buffers improve throughput but use more memory.

### Connection Limits

```
# /etc/sslh/sslh.conf
max-connections: 10000;
```

**Explanation**: Set the maximum number of concurrent connections. This prevents resource exhaustion.

### Timeout Configuration

```
# /etc/sslh/sslh.conf
timeout: 5;
```

**Explanation**: Set the protocol detection timeout. Increase this for slow or unreliable connections.

## Chapter 9: Detection and Defense

### Network Signatures

- Single port serving multiple protocols
- Protocol-specific patterns on shared port
- Unusual SNI hostname patterns

### IDS/IPS Detection

```
# Suricata rule for sslh
alert tcp any any -> any 443 (msg:"Protocol Multiplexer Detected"; \
  flow:to_server,established; \
  content:"SSH-"; \
  threshold:type both,track by_src, count 1, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor sslh logs
tail -f /var/log/sslh.log

# Check for protocol detection
tcpdump -i eth0 port 443 -w sslh.pcap

# Analyze protocol patterns
tshark -r sslh.pcap -Y "tcp.port == 443"
```

### Defense Recommendations

1. Monitor for SSH on non-standard ports
2. Implement protocol whitelisting
3. Use SNI-based routing for TLS
4. Monitor for protocol confusion attacks
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check sslh is running
ps aux | grep sslh

# Check port is listening
ss -tlnp | grep 443

# Check firewall rules
iptables -L -n | grep 443
```

### Protocol Detection Issues

```bash
# Test protocol detection
openssl s_client -connect localhost:443

# Check SSH detection
ssh -v localhost -p 443

# Increase timeout
# In sslh.conf
timeout: 10;
```

### Performance Issues

```bash
# Check connection count
ss -s

# Increase buffer size
sslh -B 131072

# Check resource usage
top -p $(pgrep sslh)
```

### Configuration Errors

```bash
# Test configuration
sslh -t -f /etc/sslh/sslh.conf

# View detailed logs
sslh -v -f /etc/sslh/sslh.conf
```

### Permission Issues

```bash
# Check sslh permissions
ls -la /usr/sbin/sslh

# Run as root for privileged ports
sudo sslh -p 443 --ssl 127.0.0.1:4443
```

## Chapter 14: Performance Optimization

### Connection Limits

```bash
# Check current limits
ulimit -n

# Increase file descriptors
ulimit -n 65536

# Set in sslh.conf
maxfds: 65536;
```

### Buffer Optimization

```bash
# Increase buffer for high throughput
sslh -B 131072 -p 0.0.0.0:443 --ssl 127.0.0.1:4443

# Monitor buffer usage
cat /proc/$(pgrep sslh)/fdinfo/* | grep -i buffer
```

### Timeout Tuning

```bash
# Reduce timeout for faster cleanup
# In sslh.conf
timeout: 30;

# For high-traffic servers
timeout: 10;
```

## Resources

- [sslh Homepage](https://www.apsalu.org/sslh/)
- [sslh man page](https://linux.die.net/man/8/sslh)
- [Kali Linux sslh Page](https://www.kali.org/tools/sslh/)
- [sslh GitHub](https://github.com/yrutschle/sslh)
- [RFC 6066 - TLS Extensions](https://tools.ietf.org/html/rfc6066)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
