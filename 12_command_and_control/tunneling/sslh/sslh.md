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

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install sslh
```

### Verify Installation

```bash
sslh -V
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

## Resources

- [sslh Homepage](https://www.apsalu.org/sslh/)
- [sslh man page](https://linux.die.net/man/8/sslh)
- [Kali Linux sslh Page](https://www.kali.org/tools/sslh/)
- [sslh GitHub](https://github.com/yrutschle/sslh)
