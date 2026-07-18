---
title: "sslh CHEATSHEET"
description: "Quick reference for sslh protocol multiplexer commands"
---

# sslh CHEATSHEET

## Basic Usage

```bash
# Multiplex TLS, SSH, HTTP on port 443
sslh -p 0.0.0.0:443 --ssl 127.0.0.1:4443 --ssh 127.0.0.1:22 --http 127.0.0.1:80

# Start service
sudo systemctl start sslh

# Stop service
sudo systemctl stop sslh
```

## Configuration

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

## Common Configurations

```bash
# TLS + SSH only
sslh -p 0.0.0.0:443 --ssl 127.0.0.1:4443 --ssh 127.0.0.1:22

# TLS + SSH + HTTP + OpenVPN
sslh -p 0.0.0.0:443 --ssl 127.0.0.1:4443 --ssh 127.0.0.1:22 --http 127.0.0.1:80 --openvpn 127.0.0.1:1194

# Catch-all
sslh -p 0.0.0.0:443 --ssl 127.0.0.1:4443 --ssh 127.0.0.1:22 --anyprot 127.0.0.1:4444
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-p ADDR:PORT` | Listen address |
| `--ssl ADDR:PORT` | TLS backend |
| `--ssh ADDR:PORT` | SSH backend |
| `--http ADDR:PORT` | HTTP backend |
| `--openvpn ADDR:PORT` | OpenVPN backend |
| `--anyprot ADDR:PORT` | Catch-all backend |
| `-B SIZE` | Buffer size |
| `-v` | Verbose mode |
| `-f` | Foreground |

## Service Management

```bash
sudo systemctl start sslh                   # Start
sudo systemctl stop sslh                    # Stop
sudo systemctl restart sslh                 # Restart
sudo systemctl enable sslh                  # Enable on boot
sudo systemctl status sslh                  # Status
```

## Practical Workflow

```bash
# 1. Configure backends
# Web server on 4443, SSH on 22

# 2. Start sslh
sslh -p 0.0.0.0:443 --ssl 127.0.0.1:4443 --ssh 127.0.0.1:22

# 3. Connect SSH
ssh -p 443 user@server

# 4. Connect HTTPS
curl https://server/
```

## Troubleshooting

```bash
# Check if running
ps aux | grep sslh

# Check logs
journalctl -u sslh

# Test protocol detection
openssl s_client -connect server:443
ssh -p 443 user@server
curl http://server:443
```
