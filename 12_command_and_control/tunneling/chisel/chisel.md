---
title: "chisel - Fast TCP/UDP Tunnel over HTTP"
description: "Go-based tunneling tool for port forwarding, SOCKS proxying, and network pivoting over HTTP with SSH encryption"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
github: "https://github.com/jpillora/chisel"
kali_install: "sudo apt install chisel"
version: "1.9.1"
---

# chisel

## Overview

chisel is a fast TCP/UDP tunnel transported over HTTP and secured via SSH. Written in Go, it provides a single executable that functions as both client and server. chisel is designed for passing through firewalls, establishing secure endpoints into networks, and creating encrypted C2 channels that blend with normal HTTP traffic.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install chisel
```

### From GitHub

```bash
# Download latest release
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_linux_amd64.gz
gunzip chisel_linux_amd64.gz
chmod +x chisel_linux_amd64
mv chisel_linux_amd64 /usr/local/bin/chisel
```

### Docker

```bash
docker pull jpillora/chisel
```

### Verify Installation

```bash
chisel version
```

## Chapter 2: Basic Usage

### Start Server

```bash
chisel server -p 8080
```

**Explanation**: Starts the chisel server on port 8080, listening for client connections. The server generates an ECDSA key pair and displays its fingerprint for client verification.

### Connect Client

```bash
chisel client SERVER_IP:8080 R:8081:127.0.0.1:80
```

**Explanation**: The client connects to the server and creates a reverse tunnel. `R:8081:127.0.0.1:80` means the server listens on port 8081 and forwards traffic through the client to `127.0.0.1:80` on the client's network.

## Chapter 3: Port Forwarding

### Local Forward

```bash
# Server
chisel server -p 8080

# Client
chisel client SERVER_IP:8080 8081:target_host:80
```

**Explanation**: Local port forwarding. The client listens on port 8081 and forwards connections through the tunnel to the target host's port 80 on the server's network.

### Reverse Forward

```bash
# Server
chisel server -p 8080 --reverse

# Client
chisel client SERVER_IP:8080 R:8081:target_host:80
```

**Explanation**: Reverse port forwarding. The server listens on port 8081 and forwards connections through the client to the target. This exposes internal services to the server's network.

### UDP Forwarding

```bash
# Client
chisel client SERVER_IP:8080 5353/udp:target_host:53/udp
```

**Explanation**: Forwards UDP traffic through the chisel tunnel. The `/udp` suffix specifies UDP protocol. Useful for DNS, SNMP, and other UDP-based services.

## Chapter 4: Authentication

### Server with Authentication

```bash
chisel server -p 8080 --auth user:password
```

**Explanation**: The `--auth` flag requires clients to authenticate with username and password. Only clients providing matching credentials can establish tunnels.

### Client with Authentication

```bash
chisel client SERVER_IP:8080 --auth user:password R:8081:target:80
```

**Explanation**: The client provides authentication credentials to the server. The server validates these against its configured credentials before allowing the tunnel.

### Auth File

```bash
# users.json
{
  "admin:secretpass": ["*"],
  "readonly:pass123": ["^10\\.0\\.0\\.1:.*$"]
}

chisel server -p 8080 --authfile users.json
```

**Explanation**: An auth file defines multiple users with access control rules. Each user can have specific address patterns they're allowed to tunnel to, providing fine-grained access control.

## Chapter 5: SOCKS Proxy

### Server SOCKS5

```bash
chisel server -p 8080 --socks5
```

**Explanation**: Enables the SOCKS5 proxy on the server. Clients connecting to the server can use it as a SOCKS5 proxy endpoint.

### Client SOCKS5

```bash
chisel client SERVER_IP:8080 socks
```

**Explanation**: Creates a local SOCKS5 proxy on port 1080. Applications configured to use this proxy can route traffic through the chisel tunnel to reach the server's network.

### Custom SOCKS Port

```bash
chisel client SERVER_IP:8080 5000:socks
```

**Explanation**: Binds the SOCKS5 proxy to port 5000 instead of the default 1080. Applications connect to `127.0.0.1:5000` as their SOCKS proxy.

## Chapter 6: Security Features

### Key Fingerprint Verification

```bash
# Server displays fingerprint on startup
# Client verifies with --fingerprint
chisel client --fingerprint SERVER_FINGERPRINT SERVER_IP:8080 R:8081:target:80
```

**Explanation**: The `--fingerprint` flag validates the server's public key, preventing man-in-the-middle attacks. The fingerprint is displayed when the server starts.

### TLS Encryption

```bash
# With Let's Encrypt
chisel server -p 443 --tls-domain example.com

# With custom certificate
chisel server -p 443 --tls-key key.pem --tls-cert cert.pem
```

**Explanation**: TLS wraps the HTTP transport layer. The inner SSH layer still encrypts and authenticates, providing double encryption. Let's Encrypt automatically provisions certificates.

### Client TLS Verification

```bash
chisel client --tls-ca ca.pem https://chisel.example.com 3000
```

**Explanation**: The client validates the server's TLS certificate against a custom CA. This is essential for self-signed or internal CA certificates.

## Chapter 7: Advanced Features

### Auto-Reconnect

```bash
chisel client SERVER_IP:8080 --max-retry-count 10 R:8081:target:80
```

**Explanation**: The client automatically reconnects with exponential backoff. `--max-retry-count` limits reconnection attempts. The default is unlimited retries.

### HTTP Proxy Pass-Through

```bash
chisel client SERVER_IP:8080 --proxy http://proxy:8080 R:8081:target:80
```

**Explanation**: The client connects through an HTTP proxy. This allows chisel to operate in environments where direct outbound connections are blocked but HTTP proxy access is available.

### Custom Headers

```bash
chisel client SERVER_IP:8080 --header "X-Custom: value" R:8081:target:80
```

**Explanation**: Custom HTTP headers are added to the WebSocket upgrade request. This can help blend chisel traffic with legitimate HTTP traffic or satisfy proxy requirements.

### Stdio Mode

```bash
ssh -o ProxyCommand='chisel client chiselserver stdio:%h:%p' user@example.com
```

**Explanation**: chisel operates over stdio, enabling SSH through HTTP tunnels. The SSH connection is tunneled through chisel's HTTP transport.

## Chapter 8: Operational Considerations

### Behind CDN

```bash
chisel server -p 443 --tls-domain tunnel.example.com
chisel client https://tunnel.example.com R:8081:target:80
```

**Explanation**: chisel works through CDNs like Cloudflare that support WebSockets. Enable WebSockets in the CDN configuration and use HTTPS for client connections.

### Environment Variables

```bash
# Tune timeouts
CHISEL_WS_TIMEOUT=30s chisel server -p 8080
CHISEL_SSH_TIMEOUT=20s chisel client SERVER_IP:8080 R:8081:target:80
```

**Explanation**: Environment variables control timeouts, buffer sizes, and other parameters. The `CHISEL_` prefix is required for all variables.

### Performance Tuning

```bash
# Disable keepalive for high-throughput
chisel client SERVER_IP:8080 --keepalive 0 R:8081:target:80

# Increase buffer size
CHISEL_WS_BUFF_SIZE=32768 chisel server -p 8080
```

**Explanation**: Adjust keepalive and buffer sizes for specific use cases. Disabling keepalive reduces overhead for high-throughput tunnels, while larger buffers improve throughput for bulk data transfers.

## Resources

- [chisel GitHub Repository](https://github.com/jpillora/chisel)
- [chisel Releases](https://github.com/jpillora/chisel/releases)
- [Go Documentation](https://golang.org/doc/)
