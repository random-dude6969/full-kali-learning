# sslsplit Cheatsheet

## Quick Start

```bash
# Basic HTTPS interception
sslsplit -k ca.key -c ca.pem https 0.0.0.0 8443

# With logging
sslsplit -l connections.log -k ca.key -c ca.pem https 0.0.0.0 8443
```

## Essential Commands

```bash
# Debug mode
sslsplit -D -k ca.key -c ca.pem https 0.0.0.0 8443

# Connection logging
sslsplit -l /var/log/sslsplit/connections.log -k ca.key -c ca.pem https 0.0.0.0 8443

# Content logging
sslsplit -S /var/log/sslsplit/content/ -k ca.key -c ca.pem https 0.0.0.0 8443

# PCAP logging
sslsplit -X output.pcap -k ca.key -c ca.pem https 0.0.0.0 8443
```

## Proxy Specifications

```bash
# HTTPS with static destination
sslsplit https 0.0.0.0 8443 192.168.1.10 443

# HTTP with static destination
sslsplit http 0.0.0.0 8080 192.168.1.10 80

# SNI-based routing
sslsplit https 0.0.0.0 8443 sni 443

# Generic TCP
sslsplit tcp 0.0.0.0 10025

# STARTTLS (autossl)
sslsplit autossl 0.0.0.0 8465 192.168.1.10 25

# Multiple specs
sslsplit https 0.0.0.0 8443 http 0.0.0.0 8080 tcp 0.0.0.0 10025
```

## Certificate Options

```bash
# CA certificate and key
sslsplit -k ca.key -c ca.pem

# Certificate chain
sslsplit -k ca.key -c ca.pem -C intermediate.pem

# Target-specific certificates
sslsplit -t /path/to/certs/

# Fallback certificate
sslsplit -t /path/to/certs/ -A fallback.pem

# Generate and store certificates
sslsplit -w /var/log/sslsplit/certs/
```

## Logging Options

```bash
# Connection log (one line per connection)
sslsplit -l connections.log

# Content log (full data)
sslsplit -L content.log

# Content to directory
sslsplit -S /var/log/content/

# Content with path substitution
sslsplit -F "/var/log/%T-%s-%d.log"

# PCAP log
sslsplit -X output.pcap

# PCAP to directory
sslsplit -Y /var/log/pcap/

# Master keys (for Wireshark)
sslsplit -M masterkeys.log
```

## Attack Scenarios

### Full Network Interception

```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure iptables
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-ports 8443

# Start sslsplit
sslsplit -D -l /var/log/sslsplit/connections.log \
  -S /var/log/sslsplit/content/ \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443 \
  http 0.0.0.0 8080
```

### SMTP STARTTLS Interception

```bash
# Redirect SMTP
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 25 -j REDIRECT --to-ports 8465

# Start sslsplit for STARTTLS
sslsplit -D -l smtp.log -S /var/log/smtp/ \
  -k ca.key -c ca.pem \
  autossl 0.0.0.0 8465 192.168.1.10 25
```

### Forensic Analysis

```bash
# Comprehensive logging
sslsplit -D -l /forensics/connections.log \
  -L /forensics/content.log \
  -X /forensics/pcap/ \
  -M /forensics/masterkeys.log \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443
```

### Targeted Domains

```bash
# Prepare certificates
mkdir -p /etc/sslsplit/certs/
cp target.com.crt /etc/sslsplit/certs/
cp target.com.key /etc/sslsplit/certs/

# Start with targeted certs
sslsplit -t /etc/sslsplit/certs/ \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443
```

## Stealth Options

```bash
# Deny OCSP requests
sslsplit -O -k ca.key -c ca.pem https 0.0.0.0 8443

# Pass through when can't split
sslsplit -P -k ca.key -c ca.pem https 0.0.0.0 8443

# Use pre-generated certs
sslsplit -t /etc/sslsplit/certs/ -k ca.key -c ca.pem https 0.0.0.0 8443

# Deny OCSP + pass through
sslsplit -O -P -k ca.key -c ca.pem https 0.0.0.0 8443
```

## Wireshark Integration

```bash
# Log master keys
sslsplit -M masterkeys.log -k ca.key -c ca.pem https 0.0.0.0 8443

# In Wireshark:
# Edit > Preferences > Protocols > TLS
# (Pre)-Master-Secret log filename: /path/to/masterkeys.log
```

## iptables/nftables Setup

```bash
# iptables REDIRECT
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-ports 8443

# nftables
sudo nft add rule nat prerouting iif eth0 tcp dport 80 redirect to :8080
sudo nft add rule nat prerouting iif eth0 tcp dport 443 redirect to :8443

# Remove rules
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-ports 8443
```

## Configuration File

```bash
# Create config file
cat > /etc/sslsplit.conf << 'EOF'
ssl 0.0.0.0 8443 192.168.1.10 443
http 0.0.0.0 8080 192.168.1.10 80
EOF

# Start with config
sslsplit -f /etc/sslsplit.conf -k ca.key -c ca.pem
```

## Debugging

```bash
# Debug mode
sslsplit -D -k ca.key -c ca.pem https 0.0.0.0 8443

# Check iptables
sudo iptables -t nat -L -n

# Check IP forwarding
sysctl net.ipv4.ip_forward

# Check listening ports
netstat -tlnp | grep 8443

# Test connection
curl -x http://localhost:8443 https://example.com
```

## Performance Tuning

```bash
# Increase file descriptors
ulimit -n 65536

# Reduce logging for high throughput
sslsplit -l /dev/null -k ca.key -c ca.pem https 0.0.0.0 8443

# Use chroot for security
sslsplit -j /tmp/sslsplit/ -k ca.key -c ca.pem https 0.0.0.0 8443
```

## Cleanup

```bash
# Remove iptables rules
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-ports 8443

# Disable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=0

# Kill sslsplit
sudo killall sslsplit
```
