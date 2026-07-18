---
title: "tlssled Quick Reference"
description: "Quick reference guide for tlssled SSL/TLS assessment wrapper"
categories:
  - "ssl"
  - "tls"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "ssl"
  - "tls"
  - "wrapper"
---

# tlssled CHEATSHEET

## Quick Start

**Basic scan**: `tlssled 192.168.1.100 443`

**Custom port**: `tlssled 192.168.1.100 8443`

**Verbose mode**: `tlssled -v 192.168.1.100 443`

**Save output**: `tlssled 192.168.1.100 443 | tee results.txt`

**Multiple hosts**: `for host in web1 web2 web3; do tlssled "$host" 443; done`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-p PORT` | Target port | `tlssled -p 8443 192.168.1.100` |
| `-v` | Verbose mode | `tlssled -v 192.168.1.100 443` |
| `--no-color` | Disable colors | `tlssled --no-color 192.168.1.100 443` |
| `--quiet` | Suppress info | `tlssled --quiet 192.168.1.100 443` |
| `--help` | Show help | `tlssled --help` |
| `--version` | Show version | `tlssled --version` |

## Examples

### Basic Assessment
```bash
tlssled 192.168.1.100 443
```

### Custom Port
```bash
tlssled 192.168.1.100 8443
```

### Verbose Mode
```bash
tlssled -v 192.168.1.100 443
```

### Default Port
```bash
tlssled 192.168.1.100
```

### Clean Output
```bash
tlssled 192.168.1.100 443 2>/dev/null
```

### Save Results
```bash
tlssled 192.168.1.100 443 2>&1 | tee results.txt
```

### Multiple Hosts
```bash
for host in web1 web2 web3; do
  echo "=== $host ==="
  tlssled "$host" 443
done
```

### Paginated Output
```bash
tlssled 192.168.1.100 443 | less
```

### Network Scan
```bash
for i in $(seq 1 100); do
  tlssled 192.168.1.$i 443
done
```

### Comparison
```bash
diff <(tlssled server1 443) <(tlssled server2 443)
```

## Chaining

### Chain with nmap
```bash
# Find HTTPS hosts, then assess
nmap -p 443 192.168.1.0/24 -oG - | grep "443/open" | awk '{print $2}' | while read host; do
  echo "=== $host ==="
  tlssled "$host" 443
done
```

### Chain with sslscan/sslyze
```bash
# Use tlssled for quick overview, then detailed tools
tlssled 192.168.1.100 443 > overview.txt
sslscan --bugs 192.168.1.100:443
sslyze --regular --json detailed.json 192.168.1.100:443
```

### Batch Processing
```bash
cat > targets.txt << 'EOF'
192.168.1.10
192.168.1.20
192.168.1.30
EOF
while read host; do
  tlssled "$host" 443 > "$host.txt"
done < targets.txt
```

## Troubleshooting

### Tool not found
- Install dependencies: `sudo apt install sslscan sslyze`

### Connection refused
- Verify port: `nmap -p 443 192.168.1.100`
- Check firewall: `iptables -L -n | grep 443`

### Partial results
- Server may not support all protocols
- Try individual tools:
  ```bash
  sslscan 192.168.1.100:443
  sslyze 192.168.1.100:443
  ```

### Timeout issues
- Check network: `ping -c 2 192.168.1.100`
- Server may be overloaded

### No output
- Verify syntax: `tlssled 192.168.1.100 443`
- Check tools: `which sslscan sslyze`

### Inconsistent results
- Load balancers may have different configs
- Run tools separately for detailed comparison
