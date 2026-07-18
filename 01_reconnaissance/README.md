# Kali Linux Reconnaissance — Noob to Pro

## Welcome

This is a comprehensive reconnaissance documentation project covering **80+ tools** across **10 categories**. Each tool has detailed documentation with installation, usage, automation scripts, and real-world scenarios.

---

## Learning Roadmap

### Phase 1: Foundations (Week 1-2)
Start with these essential tools to build your recon skills.

| Category | Tools to Learn | Priority |
|----------|---------------|----------|
| Network Info | Nmap, Zenmap | **Must** |
| DNS Recon | DNSRecon, DNSenum | **Must** |
| Web Scanning | WhatWeb, Nikto | **Must** |
| Identity | Sherlock, The Harvester | **Must** |

### Phase 2: Intermediate (Week 3-4)
Expand your toolkit with more specialized tools.

| Category | Tools to Learn | Priority |
|----------|---------------|----------|
| Web Scanning | Gobuster, FFUF, Subfinder | **Must** |
| Web Vuln | WPScan, Nuclei, ZAP | **Must** |
| Network | Amass, Legion | **Should** |
| Identity | SpiderFoot, Photon | **Should** |

### Phase 3: Advanced (Week 5-6)
Master advanced techniques and automation.

| Category | Tools to Learn | Priority |
|----------|---------------|----------|
| All Categories | Advanced usage, scripting | **Must** |
| Automation | Python/Bash scripting | **Must** |
| Integration | Tool chaining | **Should** |
| Evasion | Anti-detection techniques | **Should** |

### Phase 4: Expert (Week 7-8)
Master specialized areas and build custom workflows.

| Category | Tools to Learn | Priority |
|----------|---------------|----------|
| Bluetooth | BetterCap, Ubertooth | **Should** |
| WiFi | Kismet, Aircrack-ng | **Should** |
| RF | HackRF, GNU Radio | **Optional** |
| Frameworks | SpiderFoot, Recon-ng | **Should** |

---

## Categories Overview

### 01. Host Information
Tools for gathering information about hosts and infrastructure.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [Metagoofil](01_host_information/metagoofil/) | Document metadata extraction | Beginner |
| [SpiderFoot](01_host_information/spiderfoot/) | OSINT automation framework | Intermediate |

### 02. Identity Information
Tools for gathering information about people and identities.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [EmailHarvester](02_identity_information/emailharvester/) | Email address harvesting | Beginner |
| [Instaloader](02_identity_information/instaloader/) | Instagram data downloading | Beginner |
| [LinkedIn2Username](02_identity_information/linkedin2username/) | LinkedIn username generation | Beginner |
| [Photon](02_identity_information/photon/) | Fast web crawler for OSINT | Beginner |
| [Sherlock](02_identity_information/sherlock/) | Username enumeration | Beginner |
| [Toookie OSINT](02_identity_information/tookie-osint/) | Multi-tool OSINT collection | Beginner |

### 03. Network Information
Tools for network reconnaissance and scanning.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [Amass](03_network_information/amass/) | Attack surface mapping | Advanced |
| [AutoRecon](03_network_information/autorecon/) | Automated network recon | Intermediate |
| [DMitry](03_network_information/dmitry/) | Deep information gathering | Beginner |
| [Legion](03_network_information/legion/) | Network penetration testing GUI | Beginner |
| [Nmap](03_network_information/nmap/) | Network mapper | All Levels |
| [TheHarvester](03_network_information/theharvester/) | Email/subdomain harvesting | Beginner |
| [Unicornscan](03_network_information/unicornscan/) | Asynchronous port scanner | Intermediate |
| [Zenmap](03_network_information/zenmap/) | Nmap GUI frontend | Beginner |

### 04. DNS Reconnaissance
Tools for DNS enumeration and analysis.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [DNSenum](04_dns_reconnaissance/dnsenum/) | DNS enumeration | Beginner |
| [DNSmap](04_dns_reconnaissance/dnsmap/) | DNS brute-forcing | Beginner |
| [DNSRecon](04_dns_reconnaissance/dnsrecon/) | DNS reconnaissance | Beginner |
| [DnsTracer](04_dns_reconnaissance/dnstracer/) | DNS trace tool | Beginner |
| [DNSwalk](04_dns_reconnaissance/dnswalk/) | DNS zone walker | Beginner |
| [MassDNS](04_dns_reconnaissance/massdns/) | High-speed DNS resolver | Intermediate |

### 05. Web Scanning
Tools for web content discovery and scanning.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [Arjun](05_web_scanning/arjun/) | HTTP parameter discovery | Beginner |
| [Assetfinder](05_web_scanning/assetfinder/) | Domain asset finder | Beginner |
| [Dirb](05_web_scanning/dirb/) | Web content scanner | Beginner |
| [DirBuster](05_web_scanning/dirbuster/) | Directory brute-forcer | Beginner |
| [DirSearch](05_web_scanning/dirsearch/) | Web path scanner | Beginner |
| [Feroxbuster](05_web_scanning/feroxbuster/) | Fast content discovery | Beginner |
| [FFUF](05_web_scanning/ffuf/) | Fast web fuzzer | Intermediate |
| [FinalRecon](05_web_scanning/finalrecon/) | Multi-purpose recon | Beginner |
| [Findomain](05_web_scanning/findomain/) | Fast subdomain discovery | Beginner |
| [Gobuster](05_web_scanning/gobuster/) | Directory/DNS/vhost busting | Beginner |
| [GoSpider](05_web_scanning/gospider/) | Fast web crawler | Beginner |
| [LBD](05_web_scanning/lbd/) | Load balancing detector | Beginner |
| [Parsero](05_web_scanning/parsero/) | Robots.txt parser | Beginner |
| [Recon-ng](05_web_scanning/recon-ng/) | OSINT framework | Intermediate |
| [Subfinder](05_web_scanning/subfinder/) | Passive subdomain discovery | Beginner |
| [Sublist3r](05_web_scanning/sublist3r/) | Subdomain enumeration | Beginner |
| [URO](05_web_scanning/uro/) | URL deduplication | Beginner |
| [Wfuzz](05_web_scanning/wfuzz/) | Web application fuzzer | Intermediate |
| [WPProbe](05_web_scanning/wpprobe/) | WordPress plugin detector | Beginner |

### 06. Vulnerability Scanning
Tools for vulnerability detection and assessment.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [GVM](06_vulnerability_scanning/gvm/) | OpenVAS vulnerability scanner | Intermediate |
| [Heartleech](06_vulnerability_scanning/heartleech/) | Heartbleed testing | Intermediate |

### 07. Web Vulnerability Scanning
Tools for web application security testing.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [Burp Suite](07_web_vulnerability_scanning/burpsuite/) | Web vulnerability scanner/proxy | All Levels |
| [Caido](07_web_vulnerability_scanning/caido/) | Web security testing toolkit | Intermediate |
| [CRLFuzz](07_web_vulnerability_scanning/crlfuzz/) | CRLF injection fuzzer | Beginner |
| [DAVTest](07_web_vulnerability_scanning/davtest/) | WebDAV testing | Beginner |
| [JoomScan](07_web_vulnerability_scanning/joomscan/) | Joomla vulnerability scanner | Beginner |
| [Nikto](07_web_vulnerability_scanning/nikto/) | Web server scanner | Beginner |
| [Nuclei](07_web_vulnerability_scanning/nuclei/) | Template-based vuln scanner | Intermediate |
| [Paros](07_web_vulnerability_scanning/paros/) | HTTP proxy scanner | Beginner |
| [Skipfish](07_web_vulnerability_scanning/skipfish/) | Web app security scanner | Intermediate |
| [SSTImap](07_web_vulnerability_scanning/sstimap/) | SSTI detection | Intermediate |
| [Subjack](07_web_vulnerability_scanning/subjack/) | Subdomain takeover checker | Beginner |
| [Tinja](07_web_vulnerability_scanning/tinja/) | Template injection finder | Intermediate |
| [Wapiti](07_web_vulnerability_scanning/wapiti/) | Web app vulnerability scanner | Beginner |
| [WATOBO](07_web_vulnerability_scanning/watobo/) | Web app testing tool | Intermediate |
| [WCVS](07_web_vulnerability_scanning/wcvs/) | Web crawler/vuln scanner | Beginner |
| [WebScarab](07_web_vulnerability_scanning/webscarab/) | HTTP(S) proxy | Beginner |
| [WhatWeb](07_web_vulnerability_scanning/whatweb/) | Web technology identifier | Beginner |
| [WPScan](07_web_vulnerability_scanning/wpscan/) | WordPress vulnerability scanner | Beginner |
| [ZAP](07_web_vulnerability_scanning/zaproxy/) | OWASP ZAP proxy/scanner | All Levels |

### 08. Bluetooth
Tools for Bluetooth reconnaissance and testing.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [BetterCap](08_bluetooth/bettercap/) | Network attack/monitoring | Intermediate |
| [BlueLog](08_bluetooth/bluelog/) | Bluetooth logger | Beginner |
| [BlueRanger](08_bluetooth/blueranger/) | Bluetooth signal tool | Beginner |
| [Bluesnarfer](08_bluetooth/bluesnarfer/) | Bluetooth info extraction | Intermediate |
| [BTScanner](08_bluetooth/btscanner/) | Bluetooth scanner | Beginner |
| [RedFang](08_bluetooth/redfang/) | Bluetooth brute-forcer | Intermediate |
| [Spooftooph](08_bluetooth/spooftooph/) | Bluetooth spoofer | Intermediate |
| [Ubertooth](08_bluetooth/ubertooth-util/) | Ubertooth utility | Advanced |

### 09. WiFi
Tools for wireless network reconnaissance.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [Asleap](09_wifi/asleap/) | LEAP authentication cracker | Intermediate |
| [Kismet](09_wifi/kismet/) | Wireless sniffer/detector | Intermediate |
| [Sparrow WiFi](09_wifi/sparrow-wifi/) | WiFi analyzer | Intermediate |
| [Wash](09_wifi/wash/) | WPS detection | Beginner |

### 10. Radio Frequency
Tools for RF analysis and SDR.

| Tool | Description | Difficulty |
|------|-------------|------------|
| [Chirp](10_radio_frequency/chirp/) | Radio programming tool | Intermediate |
| [GNU Radio](10_radio_frequency/gnuradio/) | SDR signal processing | Advanced |
| [GQRX](10_radio_frequency/gqrx/) | SDR receiver | Intermediate |
| [HackRF](10_radio_frequency/hackrf/) | HackRF SDR tool | Advanced |
| [RFCat](10_radio_frequency/rfcat/) | RF dongle interaction | Advanced |

---

## Prerequisites

### System Requirements

- **Operating System:** Kali Linux (recommended) or any Linux distribution
- **RAM:** 8GB minimum (16GB recommended)
- **Disk Space:** 50GB+ free space
- **Network:** Internet access for tool updates and API calls
- **Hardware:** Wireless adapter for WiFi/Bluetooth tools, HackRF for RF tools

### Legal Disclaimer

**IMPORTANT:** This documentation is for educational and authorized security testing purposes only.

- **Always obtain written permission** before testing any system you don't own
- **Understand your scope** — know exactly what you're authorized to test
- **Follow responsible disclosure** — report vulnerabilities properly
- **Respect privacy laws** — don't collect more data than necessary
- **Document everything** — keep records of your authorization and activities

Unauthorized access to computer systems is illegal in most jurisdictions.

---

## How to Use This Documentation

### For Beginners

1. Start with Phase 1 tools (Nmap, WhatWeb, Sherlock)
2. Read each tool's Chapter 1 (Introduction) to understand what it does
3. Follow Chapter 2 (Installation) to set up the tool
4. Practice Chapter 3 (Basic Usage) on legal targets
5. Move to Chapter 4 (Intermediate) when comfortable

### For Intermediate Users

1. Focus on Phase 2 and 3 tools
2. Read Chapters 4-5 for automation and advanced techniques
3. Study Chapter 6 (Real-World Scenarios) for practical applications
4. Build your own automation scripts

### For Advanced Users

1. Focus on Phase 4 specialized tools
2. Study Chapter 5 (Advanced Usage) for evasion and scaling
3. Read Chapter 7 (Detection & Defense) to understand blue team perspective
4. Build custom tools and workflows

---

## Contributing

This is a living documentation project. Contributions are welcome:

1. **Add new tools** — Follow the existing template structure
2. **Update existing tools** — Keep information current
3. **Add examples** — Real-world scenarios and scripts
4. **Fix errors** — Correct any inaccuracies

---

## Resources

### Practice Platforms

- [HackTheBox](https://www.hackthebox.com/) — CTF challenges
- [TryHackMe](https://tryhackme.com/) — Guided learning
- [VulnHub](https://www.vulnhub.com/) — Vulnerable VMs
- [OverTheWire](https://overthewire.org/) — Wargames

### Certifications

- **OSCP** — Offensive Security Certified Professional
- **CEH** — Certified Ethical Hacker
- **PNPT** — Practical Network Penetration Tester
- **GPEN** — GIAC Penetration Tester

### Books

- "The Web Application Hacker's Handbook" — Dafydd Stuttard
- "Penetration Testing" — Georgia Weidman
- "Hacking: The Art of Exploitation" — Jon Erickson

---

*Last updated: July 2026*
