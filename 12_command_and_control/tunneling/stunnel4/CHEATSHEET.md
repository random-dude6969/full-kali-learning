---
title: "stunnel4 CHEATSHEET"
description: "Quick reference for stunnel4 SSL tunnel commands"
---

# stunnel4 CHEATSHEET

## Basic Usage

```bash
# Start stunnel
sudo stunnel /etc/stunnel/stunnel.conf

# Start in foreground
sudo stunnel -f /etc/stunnel/stunnel.conf

# Show version
stunnel -version
```

## Configuration

```
# /etc/stunnel/stunnel.conf

# Client mode
[ssh-tunnel]
client = yes
accept = 127.0.0.1:2222
connect = TARGET:22

# Server mode
[ssh-server]
client = no
accept = 443
connect = 127.0.0.1:22
cert = /etc/stunnel/cert.pem
key = /etc/stunnel/key.pem
```

## Certificate Generation

```bash
# Self-signed certificate
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
cat cert.pem key.pem > /etc/stunnel/stunnel.pem

# Combined PEM file
cert = /etc/stunnel/stunnel.pem
```

## Common Configurations

```bash
# SSH over SSL
[ssh]
client = yes
accept = 127.0.0.1:2222
connect = TARGET:22

# HTTP TLS termination
[https]
client = no
accept = 443
connect = 127.0.0.1:80
cert = /etc/stunnel/cert.pem

# SMTP over SSL
[smtp]
client = yes
accept = 127.0.0.1:2525
connect = mail.example.com:587

# SOCKS proxy over TLS
[socks]
client = no
accept = 443
connect = 0
protocol = socks
```

## Common Options

| Option | Description |
|--------|-------------|
| `client = yes/no` | Client or server mode |
| `accept = ADDR:PORT` | Listen address |
| `connect = ADDR:PORT` | Backend address |
| `cert = FILE` | Certificate file |
| `key = FILE` | Private key file |
| `verify = LEVEL` | Certificate verification |
| `CAfile = FILE` | CA certificate file |
| `protocol = PROTO` | Protocol negotiation |
| `retry = yes` | Auto-reconnect |
| `debug = LEVEL` | Debug level (0-7) |
| `output = FILE` | Log file |

## Service Management

```bash
# Start
sudo stunnel /etc/stunnel/stunnel.conf

# Reload
sudo kill -HUP $(cat /var/run/stunnel4/stunnel.pid)

# Stop
sudo kill $(cat /var/run/stunnel4/stunnel.pid)

# Check status
ps aux | grep stunnel
```

## Practical Workflow

```bash
# 1. Generate certificate
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
cat cert.pem key.pem > /etc/stunnel/stunnel.pem

# 2. Configure
cat > /etc/stunnel/stunnel.conf << 'EOF'
[ssh]
client = yes
accept = 127.0.0.1:2222
connect = TARGET:22
EOF

# 3. Start
sudo stunnel /etc/stunnel/stunnel.conf

# 4. Connect
ssh -p 2222 user@127.0.0.1
```

## Troubleshooting

```bash
# Check logs
cat /var/log/stunnel.log

# Check if running
ps aux | grep stunnel

# Check port
ss -tlnp | grep 2222

# Test connection
openssl s_client -connect 127.0.0.1:2222
```
