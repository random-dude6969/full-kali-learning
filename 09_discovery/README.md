# Phase 09 — Discovery

88 tools for enumerating systems, networks, and services on target environments.

## Subcategories

| Category | Tools | Description |
|----------|-------|-------------|
| Account Discovery | 2 | Find user accounts (Apache-users, SMTP enum) |
| Active Directory | 7 | AD enumeration (BloodHound, SharpHound) |
| Cisco Tools | 5 | Cisco device exploitation |
| Database Discovery | 6 | Database enumeration (MySQL, Oracle, SQLite) |
| Network Security Appliances | 3 | WAF detection, firewall testing |
| Network Service Discovery | 4 | Port scanning (Masscan, Unicornscan) |
| Network Share Discovery | 5 | SMB/NFS enumeration (SMBMap, Enum4linux) |
| Network Sniffing | 16 | Traffic analysis (Wireshark, tcpdump, Ettercap) |
| Remote System Discovery | 10 | Host discovery (ARP, ping, OS fingerprinting) |
| SNMP | 3 | SNMP enumeration (Braa, Onesixtyone) |
| SSL/TLS Analysis | 3 | Certificate and protocol testing |
| System Network Config | 4 | Network configuration discovery |
| VoIP | 15 | Voice over IP attacks and enumeration |
| MX Check | 1 | Mail server enumeration |

## Learning Path

1. Start with **Nmap** and **Masscan** for port scanning
2. Learn **Wireshark** and **tcpdump** for traffic analysis
3. Study **BloodHound** for Active Directory mapping
4. Master **SMBMap** and **Enum4linux** for share enumeration
5. Explore **Ettercap** and **Bettercap** for MITM attacks
6. Practice VoIP attacks with **SIPVicious**

## Key Tools

| Tool | Lines | Use Case |
|------|-------|----------|
| Wireshark | 800+ | Packet analysis |
| BloodHound | 700+ | AD attack path mapping |
| Masscan | 600+ | Fast port scanning |
| Bettercap | 700+ | Network MITM |
| tcpdump | 600+ | Command-line packet capture |
| Ettercap | 600+ | ARP spoofing and MITM |
| SMBMap | 500+ | SMB share enumeration |

## MITRE ATT&CK Mapping

| Technique | ID | Tools |
|-----------|-----|-------|
| Network Sniffing | T1040 | Wireshark, tcpdump, Ettercap |
| Remote System Discovery | T1018 | Nmap, Masscan, ARP-scan |
| Domain Trust Discovery | T1482 | BloodHound |
| System Network Config | T1016 | ipconfig, ifconfig |
| System Owner/User Discovery | T1033 |.enum4linux, enum4linux-ng |
