# ssldump Cheatsheet

## Quick Start

```bash
# Basic capture
ssldump -i eth0

# Capture on port 443
ssldump -i eth0 -p 443

# With decryption key
ssldump -i eth0 -k server.key
```

## Essential Commands

```bash
# Verbose output
ssldump -i eth0 -v -p 443

# ASCII application data
ssldump -i eth0 -A -p 443

# Hex dump
ssldump -i eth0 -X -p 443

# Read PCAP file
ssldump -r capture.pcap -k server.key
```

## Filtering Options

```bash
# By host
ssldump -i eth0 -h target.local

# By port
ssldump -i eth0 -p 443

# By host and port
ssldump -i eth0 -h target.local -p 443

# By source/destination
ssldump -i eth0 -s 192.168.1.100 -d 10.0.0.1
```

## Decryption

```bash
# With RSA private key
ssldump -i eth0 -k server.key -p 443

# With SSLKEYLOGFILE
ssldump -i eth0 -l keylog.txt

# From PCAP with key
ssldump -r traffic.pcap -k server.key
```

## Output Options

```bash
# ASCII mode
ssldump -i eth0 -A -p 443

# Hex mode
ssldump -i eth0 -X -p 443

# Timestamp
ssldump -i eth0 -t -p 443

# Quiet mode
ssldump -i eth0 -q -p 443

# Write to PCAP
ssldump -i eth0 -w output.pcap -k server.key
```

## Attack Scenarios

### Web Traffic Analysis

```bash
# Capture HTTPS traffic
ssldump -i eth0 -p 443 -k server.key -A 2>&1 | tee web_traffic.txt

# Filter specific host
ssldump -i eth0 -h web.target.local -p 443 -k server.key -A
```

### Credential Extraction

```bash
# Extract authentication headers
ssldump -i eth0 -p 443 -k server.key -A | grep -i "authorization\|password\|token\|cookie"

# Save extracted credentials
ssldump -i eth0 -p 443 -k server.key -A | grep -i "password" > creds.txt
```

### API Analysis

```bash
# Analyze API traffic
ssldump -i eth0 -h api.target.local -p 443 -k server.key -v -A

# Extract JSON responses
ssldump -i eth0 -h api.target.local -p 443 -k server.key -A | grep -E "\{.*\}" > api_responses.json
```

### Forensic Analysis

```bash
# Analyze PCAP evidence
ssldump -r evidence.pcap -k server.key -A 2>&1 | tee forensic.txt

# Extract specific connections
ssldump -r evidence.pcap -k server.key -h target.local -A > extracted.txt
```

## Protocol Analysis

```bash
# SSL/TLS handshake details
ssldump -i eth0 -v -p 443 | grep -E "ClientHello|ServerHello|Certificate"

# Cipher suite analysis
ssldump -i eth0 -v -p 443 | grep -i "cipher"

# Certificate details
ssldump -i eth0 -v -p 443 | grep -A 5 "Certificate"
```

## Integration

### With tcpdump

```bash
# Capture with tcpdump
tcpdump -i eth0 -w ssl.pcap port 443

# Analyze with ssldump
ssldump -r ssl.pcap -k server.key
```

### With Wireshark

```bash
# Export decrypted traffic
ssldump -i eth0 -k server.key -w decrypted.pcap

# Open in Wireshark
wireshark decrypted.pcap
```

### With OpenSSL

```bash
# Verify key matches certificate
openssl x509 -noout -modulus -in server.crt | openssl md5
openssl rsa -noout -modulus -in server.key | openssl md5

# Test key decryption
openssl rsa -in server.key -check
```

## Debugging

```bash
# Verbose debug output
ssldump -i eth0 -k server.key -vvv

# Check interface traffic
tcpdump -i eth0 -c 10

# Verify SSLKEYLOGFILE
cat keylog.txt | head -5
```

## Common Patterns

```bash
# Capture all HTTPS
ssldump -i eth0 -p 443

# Capture specific subnet
ssldump -i eth0 -s 192.168.1.0/24

# Capture HTTP and HTTPS
ssldump -i eth0 -p 80,443

# Capture from multiple hosts
ssldump -i eth0 -h "web|api|auth"
```

## Cleanup

```bash
# No persistent changes made
# ssldump is passive capture only

# Check captured files
ls -la *.pcap
ls -la *.txt
```
