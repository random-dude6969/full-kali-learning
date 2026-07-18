# Kali Linux Tools — Master Index & Learning Roadmap

> A comprehensive educational resource for mastering Kali Linux tools, organized by the MITRE ATT&CK framework. Each tool includes exhaustive documentation, real-world scenarios, and quick-reference cheat sheets.

## How to Use This Resource

1. **Start with prerequisites** if you're new to cybersecurity concepts
2. **Follow the phases in order** (01-16) for a structured learning path
3. **Use cheat sheets** (`CHEATSHEET.md` in each tool directory) for quick reference
4. **Check the master cheat sheet** (`CHEATSHEET.md` at project root) for all tools in one place

---

## Prerequisites (Foundational Knowledge)

Before diving into tools, ensure you understand these core concepts:

### [01Networking](prerequisites/01_networking/)
- [Networking Fundamentals](prerequisites/01_networking/networking.md) — OSI model, TCP/IP, subnetting
- [Packet Analysis](prerequisites/01_networking/packet_analysis.md) — Wireshark, packet structure, protocols
- [Port Scanning Concepts](prerequisites/01_networking/port_scanning.md) — scan types, ports, services
- [DNS Basics](prerequisites/01_networking/dns_basics.md) — resolution, records, zone transfers
- [Layer 2 Concepts](prerequisites/01_networking/layer2_concepts.md) — MAC, ARP, switching, VLANs
- [MITM Concepts](prerequisites/01_networking/mitm_concepts.md) — interception techniques, countermeasures
- [Tunneling Concepts](prerequisites/01_networking/tunneling_concepts.md) — encapsulation, VPNs, proxy chains
- [Wireless Concepts](prerequisites/01_networking/wireless_concepts.md) — 802.11, authentication, encryption
- [VoIP Concepts](prerequisites/01_networking/voip_concepts.md) — SIP, RTP, VoIP security

### [02Programming](prerequisites/02_programming/)
- [Programming Fundamentals](prerequisites/02_programming/programming.md) — variables, control flow, functions
- [Python Basics](prerequisites/02_programming/python_basics.md) — syntax, libraries, scripting
- [Bash Scripting](prerequisites/02_programming/bash_scripting.md) — shell scripting, automation
- [Exploit Development](prerequisites/02_programming/exploit_dev.md) — buffer overflows, shellcode, payload dev
- [Shellcode Development](prerequisites/02_programming/shellcode_dev.md) — assembly, encoding, payloads
- [Reverse Engineering](prerequisites/02_programming/reverse_engineering.md) — disassembly, debugging, analysis

### [03Cryptography](prerequisites/03_cryptography/)
- [Cryptography Fundamentals](prerequisites/03_cryptography/cryptography.md) — symmetric, asymmetric, hashing
- [Encryption Algorithms](prerequisites/03_cryptography/encryption.md) — AES, RSA, ECC, use cases
- [Hash Functions](prerequisites/03_cryptography/hash_functions.md) — MD5, SHA, bcrypt, rainbow tables
- [Kerberos](prerequisites/03_cryptography/kerberos.md) — authentication protocol, attacks
- [SSL/TLS](prerequisites/03_cryptography/ssl_tls.md) — certificates, handshakes, vulnerabilities

### [04Web Development](prerequisites/04_web_development/)
- [Web Development Fundamentals](prerequisites/04_web_development/web_dev.md) — HTML, CSS, JavaScript, servers
- [HTTP/HTTPS](prerequisites/04_web_development/http_https.md) — methods, headers, status codes, cookies
- [Web Application Architecture](prerequisites/04_web_development/web_app_architecture.md) — MVC, APIs, sessions
- [Web Vulnerabilities](prerequisites/04_web_development/web_vulnerabilities.md) — OWASP Top 10, injection, XSS

### [05System Administration](prerequisites/05_system_administration/)
- [Linux Administration](prerequisites/05_system_administration/linux_admin.md) — filesystem, permissions, services
- [Windows Administration](prerequisites/05_system_administration/windows_admin.md) — registry, groups, policies
- [Active Directory](prerequisites/05_system_administration/active_directory.md) — domains, GPO, Kerberos
- [Forensics Basics](prerequisites/05_system_administration/forensics_basics.md) — evidence handling, chain of custody

### [06RF & Wireless](prerequisites/06_rf_wireless/)
- [RF Basics](prerequisites/06_rf_wireless/rf_basics.md) — frequency, modulation, propagation
- [WiFi Protocols](prerequisites/06_rf_wireless/wifi_protocols.md) — 802.11 standards, WPA/WPA2/WPA3
- [Bluetooth Basics](prerequisites/06_rf_wireless/bluetooth_basics.md) — BLE, pairing, profiles
- [NFC Basics](prerequisites/06_rf_wireless/nfc_basics.md) — contactless communication, tags
- [SDR Concepts](prerequisites/06_rf_wireless/sdr_concepts.md) — software-defined radio, SDR hardware

---

## Phase 01: Reconnaissance

> **MITRE ATT&CK:** TA0043 — Reconnaissance
> **Goal:** Gather information about target systems, networks, and people
> **Estimated time:** 40-60 hours

### [01 Host Information](01_reconnaissance/01_host_information/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Metagoofil](01_reconnaissance/01_host_information/metagoofil/metagoofil.md) | Metadata extraction from public documents | Beginner | [Quick Ref](01_reconnaissance/01_host_information/metagoofil/CHEATSHEET.md) |
| [SpiderFoot](01_reconnaissance/01_host_information/spiderfoot/spiderfoot.md) | OSINT automation framework | Intermediate | [Quick Ref](01_reconnaissance/01_host_information/spiderfoot/CHEATSHEET.md) |

### [02 Identity Information](01_reconnaissance/02_identity_information/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [EmailHarvester](01_reconnaissance/02_identity_information/emailharvester/emailharvester.md) | Email address harvesting | Beginner | [Quick Ref](01_reconnaissance/02_identity_information/emailharvester/CHEATSHEET.md) |
| [Instaloader](01_reconnaissance/02_identity_information/instaloader/instaloader.md) | Instagram data downloading | Beginner | [Quick Ref](01_reconnaissance/02_identity_information/instaloader/CHEATSHEET.md) |
| [LinkedIn2Username](01_reconnaissance/02_identity_information/linkedin2username/linkedin2username.md) | LinkedIn profile to username converter | Beginner | [Quick Ref](01_reconnaissance/02_identity_information/linkedin2username/CHEATSHEET.md) |
| [Photon](01_reconnaissance/02_identity_information/photon/photon.md) | Fast web crawler for OSINT | Intermediate | [Quick Ref](01_reconnaissance/02_identity_information/photon/CHEATSHEET.md) |
| [Sherlock](01_reconnaissance/02_identity_information/sherlock/sherlock.md) | Username enumeration across social networks | Beginner | [Quick Ref](01_reconnaissance/02_identity_information/sherlock/CHEATSHEET.md) |
| [Tookie-OSINT](01_reconnaissance/02_identity_information/tookie-osint/tookie-osint.md) | OSINT tool for social media | Intermediate | [Quick Ref](01_reconnaissance/02_identity_information/tookie-osint/CHEATSHEET.md) |

### [03 Network Information](01_reconnaissance/03_network_information/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Nmap](01_reconnaissance/03_network_information/nmap/nmap.md) | Network discovery and security auditing | Beginner-Advanced | [Quick Ref](01_reconnaissance/03_network_information/nmap/CHEATSHEET.md) |
| [Amass](01_reconnaissance/03_network_information/amass/amass.md) | Network mapping of attack surfaces | Intermediate | [Quick Ref](01_reconnaissance/03_network_information/amass/CHEATSHEET.md) |
| [AutoRecon](01_reconnaissance/03_network_information/autorecon/autorecon.md) | Automated network reconnaissance | Beginner | [Quick Ref](01_reconnaissance/03_network_information/autorecon/CHEATSHEET.md) |
| [Dmitry](01_reconnaissance/03_network_information/dmitry/dmitry.md) | Deep Information gathering | Beginner | [Quick Ref](01_reconnaissance/03_network_information/dmitry/CHEATSHEET.md) |
| [Legion](01_reconnaissance/03_network_information/legion/legion.md) | Network reconnaissance scanner | Beginner | [Quick Ref](01_reconnaissance/03_network_information/legion/CHEATSHEET.md) |
| [TheHarvester](01_reconnaissance/03_network_information/theharvester/theharvester.md) | Email, subdomain, name harvesting | Beginner | [Quick Ref](01_reconnaissance/03_network_information/theharvester/CHEATSHEET.md) |
| [Unicornscan](01_reconnaissance/03_network_information/unicornscan/unicornscan.md) | Asynchronous SYN/ACK scanning | Intermediate | [Quick Ref](01_reconnaissance/03_network_information/unicornscan/CHEATSHEET.md) |
| [Zenmap](01_reconnaissance/03_network_information/zenmap/zenmap.md) | Nmap GUI frontend | Beginner | [Quick Ref](01_reconnaissance/03_network_information/zenmap/CHEATSHEET.md) |

### [04 DNS Reconnaissance](01_reconnaissance/04_dns_reconnaissance/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [DNSEnum](01_reconnaissance/04_dns_reconnaissance/dnsenum/dnsenum.md) | DNS enumeration tool | Beginner | [Quick Ref](01_reconnaissance/04_dns_reconnaissance/dnsenum/CHEATSHEET.md) |
| [DNSRecon](01_reconnaissance/04_dns_reconnaissance/dnsrecon/dnsrecon.md) | DNS reconnaissance tool | Intermediate | [Quick Ref](01_reconnaissance/04_dns_reconnaissance/dnsrecon/CHEATSHEET.md) |
| [DNSMap](01_reconnaissance/04_dns_reconnaissance/dnsmap/dnsmap.md) | DNS brute-force tool | Beginner | [Quick Ref](01_reconnaissance/04_dns_reconnaissance/dnsmap/CHEATSHEET.md) |
| [MassDNS](01_reconnaissance/04_dns_reconnaissance/massdns/massdns.md) | High-performance DNS resolver | Advanced | [Quick Ref](01_reconnaissance/04_dns_reconnaissance/massdns/CHEATSHEET.md) |
| [DNSTracer](01_reconnaissance/04_dns_reconnaissance/dnstracer/dnstracer.md) | DNS trace tool | Intermediate | [Quick Ref](01_reconnaissance/04_dns_reconnaissance/dnstracer/CHEATSHEET.md) |
| [DNSWalk](01_reconnaissance/04_dns_reconnaissance/dnswalk/dnswalk.md) | DNS zone transfer tester | Intermediate | [Quick Ref](01_reconnaissance/04_dns_reconnaissance/dnswalk/CHEATSHEET.md) |

### [05 Web Scanning](01_reconnaissance/05_web_scanning/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Arjun](01_reconnaissance/05_web_scanning/arjun/arjun.md) | HTTP parameter discovery | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/arjun/CHEATSHEET.md) |
| [Assetfinder](01_reconnaissance/05_web_scanning/assetfinder/assetfinder.md) | Find domains and subdomains | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/assetfinder/CHEATSHEET.md) |
| [Dirb](01_reconnaissance/05_web_scanning/dirb/dirb.md) | Web content scanner | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/dirb/CHEATSHEET.md) |
| [DirBuster](01_reconnaissance/05_web_scanning/dirbuster/dirbuster.md) | Directory/file brute-forcer | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/dirbuster/CHEATSHEET.md) |
| [DirSearch](01_reconnaissance/05_web_scanning/dirsearch/dirsearch.md) | Web path brute-forcer | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/dirsearch/CHEATSHEET.md) |
| [Feroxbuster](01_reconnaissance/05_web_scanning/feroxbuster/feroxbuster.md) | Fast recursive content discovery | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/feroxbuster/CHEATSHEET.md) |
| [FFuf](01_reconnaissance/05_web_scanning/ffuf/ffuf.md) | Fast web fuzzer | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/ffuf/CHEATSHEET.md) |
| [FinalRecon](01_reconnaissance/05_web_scanning/finalrecon/finalrecon.md) | All-in-one reconnaissance | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/finalrecon/CHEATSHEET.md) |
| [Findomain](01_reconnaissance/05_web_scanning/findomain/findomain.md) | Cross-platform subdomain enumerator | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/findomain/CHEATSHEET.md) |
| [Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md) | Directory/DNS brute-forcer | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/gobuster/CHEATSHEET.md) |
| [GoSpider](01_reconnaissance/05_web_scanning/gospider/gospider.md) | Fast web spider | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/gospider/CHEATSHEET.md) |
| [LBD](01_reconnaissance/05_web_scanning/lbd/lbd.md) | Load balancing detector | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/lbd/CHEATSHEET.md) |
| [Parsero](01_reconnaissance/05_web_scanning/parsero/parsero.md) | Robots.txt analyzer | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/parsero/CHEATSHEET.md) |
| [Recon-ng](01_reconnaissance/05_web_scanning/recon-ng/recon-ng.md) | Full-featured reconnaissance framework | Advanced | [Quick Ref](01_reconnaissance/05_web_scanning/recon-ng/CHEATSHEET.md) |
| [Subfinder](01_reconnaissance/05_web_scanning/subfinder/subfinder.md) | Subdomain discovery tool | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/subfinder/CHEATSHEET.md) |
| [Sublist3r](01_reconnaissance/05_web_scanning/sublist3r/sublist3r.md) | Subdomain enumeration tool | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/sublist3r/CHEATSHEET.md) |
| [Uro](01_reconnaissance/05_web_scanning/uro/uro.md) | URL filtering for pentesting | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/uro/CHEATSHEET.md) |
| [Wfuzz](01_reconnaissance/05_web_scanning/wfuzz/wfuzz.md) | Web application fuzzer | Advanced | [Quick Ref](01_reconnaissance/05_web_scanning/wfuzz/CHEATSHEET.md) |
| [WPProbe](01_reconnaissance/05_web_scanning/wpprobe/wpprobe.md) | WordPress plugin scanner | Beginner | [Quick Ref](01_reconnaissance/05_web_scanning/wpprobe/CHEATSHEET.md) |
| [URLCrazy](01_reconnaissance/05_web_scanning/urlcrazy/urlcrazy.md) | Domain typo permutation engine | Intermediate | [Quick Ref](01_reconnaissance/05_web_scanning/urlcrazy/CHEATSHEET.md) |

### [06 Vulnerability Scanning](01_reconnaissance/06_vulnerability_scanning/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [GVM](01_reconnaissance/06_vulnerability_scanning/gvm/gvm.md) | Greenbone Vulnerability Manager | Advanced | [Quick Ref](01_reconnaissance/06_vulnerability_scanning/gvm/CHEATSHEET.md) |
| [Heartleech](01_reconnaissance/06_vulnerability_scanning/heartleech/heartleech.md) | Heartbleed vulnerability scanner | Intermediate | [Quick Ref](01_reconnaissance/06_vulnerability_scanning/heartleech/CHEATSHEET.md) |

### [07 Web Vulnerability Scanning](01_reconnaissance/07_web_vulnerability_scanning/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md) | Web application security testing platform | Intermediate-Advanced | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/CHEATSHEET.md) |
| [Caido](01_reconnaissance/07_web_vulnerability_scanning/caido/caido.md) | Lightweight web security auditing proxy | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/caido/CHEATSHEET.md) |
| [CRLFuzz](01_reconnaissance/07_web_vulnerability_scanning/crlfuzz/crlfuzz.md) | CRLF injection fuzzer | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/crlfuzz/CHEATSHEET.md) |
| [DavTest](01_reconnaissance/07_web_vulnerability_scanning/davtest/davtest.md) | WebDAV testing tool | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/davtest/CHEATSHEET.md) |
| [JoomScan](01_reconnaissance/07_web_vulnerability_scanning/joomscan/joomscan.md) | Joomla vulnerability scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/joomscan/CHEATSHEET.md) |
| [Nikto](01_reconnaissance/07_web_vulnerability_scanning/nikto/nikto.md) | Web server scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/nikto/CHEATSHEET.md) |
| [Nuclei](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md) | Template-based vulnerability scanner | Intermediate-Advanced | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/nuclei/CHEATSHEET.md) |
| [Paros](01_reconnaissance/07_web_vulnerability_scanning/paros/paros.md) | Java-based web proxy scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/paros/CHEATSHEET.md) |
| [Skipfish](01_reconnaissance/07_web_vulnerability_scanning/skipfish/skipfish.md) | Active web application security reconnaissance | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/skipfish/CHEATSHEET.md) |
| [SSTImap](01_reconnaissance/07_web_vulnerability_scanning/sstimap/sstimap.md) | Server-side template injection detection | Advanced | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/sstimap/CHEATSHEET.md) |
| [Subjack](01_reconnaissance/07_web_vulnerability_scanning/subjack/subjack.md) | Subdomain takeover vulnerability scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/subjack/CHEATSHEET.md) |
| [Tinja](01_reconnaissance/07_web_vulnerability_scanning/tinja/tinja.md) | Template injection finder | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/tinja/CHEATSHEET.md) |
| [Wapiti](01_reconnaissance/07_web_vulnerability_scanning/wapiti/wapiti.md) | Web application vulnerability scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/wapiti/CHEATSHEET.md) |
| [Watobo](01_reconnaissance/07_web_vulnerability_scanning/watobo/watobo.md) | Web application testing robot | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/watobo/CHEATSHEET.md) |
| [WCVS](01_reconnaissance/07_web_vulnerability_scanning/wcvs/wcvs.md) | Comprehensive web vulnerability scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/wcvs/CHEATSHEET.md) |
| [WebScarab](01_reconnaissance/07_web_vulnerability_scanning/webscarab/webscarab.md) | Web application analysis tool | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/webscarab/CHEATSHEET.md) |
| [WhatWeb](01_reconnaissance/07_web_vulnerability_scanning/whatweb/whatweb.md) | Web technology fingerprinter | Beginner | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/whatweb/CHEATSHEET.md) |
| [WPScan](01_reconnaissance/07_web_vulnerability_scanning/wpscan/wpscan.md) | WordPress security scanner | Intermediate | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/wpscan/CHEATSHEET.md) |
| [ZAProxy](01_reconnaissance/07_web_vulnerability_scanning/zaproxy/zaproxy.md) | OWASP Zed Attack Proxy | Intermediate-Advanced | [Quick Ref](01_reconnaissance/07_web_vulnerability_scanning/zaproxy/CHEATSHEET.md) |

### [08 Bluetooth](01_reconnaissance/08_bluetooth/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Bettercap](01_reconnaissance/08_bluetooth/bettercap/bettercap.md) | Network attack and monitoring framework | Intermediate-Advanced | [Quick Ref](01_reconnaissance/08_bluetooth/bettercap/CHEATSHEET.md) |
| [BlueLog](01_reconnaissance/08_bluetooth/bluelog/bluelog.md) | Bluetooth scanner/logger | Beginner | [Quick Ref](01_reconnaissance/08_bluetooth/bluelog/CHEATSHEET.md) |
| [BlueRanger](01_reconnaissance/08_bluetooth/blueranger/blueranger.md) | Bluetooth device locator | Beginner | [Quick Ref](01_reconnaissance/08_bluetooth/blueranger/CHEATSHEET.md) |
| [Bluesnarfer](01_reconnaissance/08_bluetooth/bluesnarfer/bluesnarfer.md) | Bluetooth information gathering | Intermediate | [Quick Ref](01_reconnaissance/08_bluetooth/bluesnarfer/CHEATSHEET.md) |
| [BTScanner](01_reconnaissance/08_bluetooth/btscanner/btscanner.md) | Bluetooth scanner | Beginner | [Quick Ref](01_reconnaissance/08_bluetooth/btscanner/CHEATSHEET.md) |
| [RedFang](01_reconnaissance/08_bluetooth/redfang/redfang.md) | Bluetooth sniffing tool | Intermediate | [Quick Ref](01_reconnaissance/08_bluetooth/redfang/CHEATSHEET.md) |
| [Spooftooph](01_reconnaissance/08_bluetooth/spooftooph/spooftooph.md) | Bluetooth spoofing tool | Intermediate | [Quick Ref](01_reconnaissance/08_bluetooth/spooftooph/CHEATSHEET.md) |
| [Ubertooth-Util](01_reconnaissance/08_bluetooth/ubertooth-util/ubertooth-util.md) | Ubertooth One utilities | Advanced | [Quick Ref](01_reconnaissance/08_bluetooth/ubertooth-util/CHEATSHEET.md) |

### [09 WiFi](01_reconnaissance/09_wifi/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Asleap](01_reconnaissance/09_wifi/asleap/asleap.md) | LEAP/CHAP password recovery | Intermediate | [Quick Ref](01_reconnaissance/09_wifi/asleap/CHEATSHEET.md) |
| [Kismet](01_reconnaissance/09_wifi/kismet/kismet.md) | Wireless network detector/sniffer | Intermediate | [Quick Ref](01_reconnaissance/09_wifi/kismet/CHEATSHEET.md) |
| [Sparrow-WiFi](01_reconnaissance/09_wifi/sparrow-wifi/sparrow-wifi.md) | WiFi analysis tool | Intermediate | [Quick Ref](01_reconnaissance/09_wifi/sparrow-wifi/CHEATSHEET.md) |
| [Wash](01_reconnaissance/09_wifi/wash/wash.md) | WPS scanning tool | Intermediate | [Quick Ref](01_reconnaissance/09_wifi/wash/CHEATSHEET.md) |

### [10 Radio Frequency](01_reconnaissance/10_radio_frequency/)
| Tool | Description | Difficulty | Cheat Sheet |
|------|-------------|------------|-------------|
| [Chirp](01_reconnaissance/10_radio_frequency/chirp/chirp.md) | Radio programming tool | Intermediate | [Quick Ref](01_reconnaissance/10_radio_frequency/chirp/CHEATSHEET.md) |
| [GnuRadio](01_reconnaissance/10_radio_frequency/gnuradio/gnuradio.md) | Signal processing framework | Advanced | [Quick Ref](01_reconnaissance/10_radio_frequency/gnuradio/CHEATSHEET.md) |
| [Gqrx](01_reconnaissance/10_radio_frequency/gqrx/gqrx.md) | SDR receiver | Intermediate | [Quick Ref](01_reconnaissance/10_radio_frequency/gqrx/CHEATSHEET.md) |
| [HackRF](01_reconnaissance/10_radio_frequency/hackrf/hackrf.md) | SDR transceiver tool | Advanced | [Quick Ref](01_reconnaissance/10_radio_frequency/hackrf/CHEATSHEET.md) |
| [RFcat](01_reconnaissance/10_radio_frequency/rfcat/rfcat.md) | RF analysis tool | Advanced | [Quick Ref](01_reconnaissance/10_radio_frequency/rfcat/CHEATSHEET.md) |

---

## Phase 02: Resource Development
> **MITRE ATT&CK:** TA0042 — Resource Development
> **Estimated time:** 20-30 hours
> *Coming soon — tools for disassemblers, debuggers, exploit resources, fuzzing, Java/Android reversing, shellcode/payload development*

## Phase 03: Initial Access
> **MITRE ATT&CK:** TA0001 — Initial Access
> **Estimated time:** 20-30 hours
> *Coming soon — Commix, Gophish, Metasploit Framework, SQLMap, SET Toolkit, etc.*

## Phase 04: Execution
> **MITRE ATT&CK:** TA0002 — Execution
> **Estimated time:** 10-15 hours
> *Coming soon — Armitage, BeEF, Nishang, PowerShellSploit, XSSer, etc.*

## Phase 05: Persistence
> **MITRE ATT&CK:** TA0003 — Persistence
> **Estimated time:** 10-15 hours
> *Coming soon — Backdoor Factory, Cymothoa, Laudanum, Webshells, Weevely, etc.*

## Phase 06: Privilege Escalation
> **MITRE ATT&CK:** TA0004 — Privilege Escalation
> **Estimated time:** 15-20 hours
> *Coming soon — LinPEAS, WinPEAS, Lynis, BloodyAD, etc.*

## Phase 07: Defense Evasion
> **MITRE ATT&CK:** TA0005 — Defense Evasion
> **Estimated time:** 15-20 hours
> *Coming soon — Shellter, Veil, Impacket, Steghide, etc.*

## Phase 08: Credential Access
> **MITRE ATT&CK:** TA0006 — Credential Access
> **Estimated time:** 25-35 hours
> *Coming soon — Hydra, John, Hashcat, Mimikatz, Responder, etc.*

## Phase 09: Discovery
> **MITRE ATT&CK:** TA0007 — Discovery
> **Estimated time:** 30-40 hours
> *Coming soon — BloodHound, CrackMapExec, NetExec, SNMP tools, etc.*

## Phase 10: Lateral Movement
> **MITRE ATT&CK:** TA0008 — Lateral Movement
> **Estimated time:** 10-15 hours
> *Coming soon — Impacket-PsExec, Impacket-SmbExec, RDesktop, etc.*

## Phase 11: Collection
> **MITRE ATT&CK:** TA0009 — Collection
> **Estimated time:** 15-20 hours
> *Coming soon — Evilginx2, MitMProxy, SSLsplit, Wifipumpkin3, etc.*

## Phase 12: Command and Control
> **MITRE ATT&CK:** TA0011 — Command and Control
> **Estimated time:** 20-30 hours
> *Coming soon — Metasploit C2, Cobalt Strike alternatives, Chisel, Ligolo-ng, etc.*

## Phase 13: Exfiltration
> **MITRE ATT&CK:** TA0010 — Exfiltration
> **Estimated time:** 5-10 hours
> *Coming soon — GoSHS, Impacket-SMBServer, etc.*

## Phase 14: Impact
> **MITRE ATT&CK:** TA0040 — Impact
> **Estimated time:** 10-15 hours
> *Coming soon — Slowhttptest, Siege, T50, MDK3/4, etc.*

## Phase 15: Forensics
> **MITRE ATT&CK:** Analysis / Forensics
> **Estimated time:** 30-40 hours
> *Coming soon — Volatility, Autopsy, Sleuth Kit, YARA, Binwalk, etc.*

## Phase 16: Services & Other
> **Estimated time:** 15-20 hours
> *Coming soon — DVWA, Juice Shop, Dradis, Maltego, Faraday, etc.*

---

## Master Cheat Sheet

See [CHEATSHEET.md](CHEATSHEET.md) for a consolidated quick-reference of all tools across all phases.

---

## Learning Path Recommendations

### Beginner Track (60-80 hours)
1. Prerequisites: Networking, Python, Linux Admin
2. Phase 01: Nmap, TheHarvester, Gobuster, FFuf, Nikto, WhatWeb
3. Phase 03: SQLMap, SET Toolkit
4. Phase 06: LinPEAS, WinPEAS

### Intermediate Track (100-120 hours)
1. All prerequisites
2. Phase 01: All tools
3. Phase 03-06: Core tools
4. Phase 08: Hydra, John, Hashcat, Responder
5. Phase 09: BloodHound, CrackMapExec

### Advanced Track (200+ hours)
1. All prerequisites including exploit dev and reverse engineering
2. All 16 phases
3. Custom tool development
4. Red team operations methodology

---

## License

This educational resource is provided for authorized security testing and learning purposes only. Always obtain proper authorization before testing on any systems you do not own.
