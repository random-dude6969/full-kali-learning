<div align="center">

<img src="headeroftherepo.png" width="100%" alt="Kali Linux Tools - Complete Educational Documentation">

<img src="star.png" width="600" alt="Give a Star for this repo">

# Kali Linux Tools - Complete Documentation & Tutorial

### The Ultimate Kali Linux Commands Cheat Sheet and Penetration Testing Guide

[![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-16%20Tactics-blue?style=for-the-badge&logo=mitreattack&logoColor=white)](https://attack.mitre.org/)
[![Tools](https://img.shields.io/badge/Tools-432%2B-red?style=for-the-badge&logo=kali-linux&logoColor=white)](#full-tool-list)
[![Lines](https://img.shields.io/badge/Lines-115K%2B-green?style=for-the-badge)](#)
[![Phase](https://img.shields.io/badge/Phase%2001-Complete-brightgreen?style=for-the-badge)](#phase-01--reconnaissance-80-tools)
[![Phase](https://img.shields.io/badge/Phase%2002-Complete-brightgreen?style=for-the-badge)](#phase-02--resource-development-34-tools)
[![Phase](https://img.shields.io/badge/Phase%2003-Complete-brightgreen?style=for-the-badge)](#phase-03--initial-access-10-tools)
[![WIP](https://img.shields.io/badge/Status-Active%20Development-yellow?style=for-the-badge)](#roadmap)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)
 [![Stars](https://img.shields.io/github/stars/random-dude6969/full-kali-learning?style=for-the-badge[![PRs]logo=github[![PRs]logoColor=white)](https://github.com/random-dude6969/full-kali-learning/stargazers)
[![PRs](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=for-the-badge)](CONTRIBUTING.md)

---

**432+ Kali Linux tools documented with tutorials, commands, and examples.**

The most complete penetration testing tools list with step-by-step guides.
From nmap scanning to metasploit exploitation — everything you need to learn ethical hacking.

**Star this repo** to save it for later. New tools added weekly.

</div>

---

## Table of Contents

- [What Is This Repo?](#what-is-this-repo)
- [Why Star This Repo?](#why-star-this-repo)
- [Quick Start](#quick-start)
- [Phase 01 - Reconnaissance (80 Tools)](#phase-01--reconnaissance-80-tools)
- [Phase 02 - Resource Development (34 Tools)](#phase-02--resource-development-34-tools)
- [Phase 03 - Initial Access (10 Tools)](#phase-03--initial-access-10-tools)
- [All Phases Overview](#all-phases-overview)
- [Kali Linux Commands Cheat Sheet](#kali-linux-commands-cheat-sheet)
- [Best Kali Tools for Beginners](#best-kali-tools-for-beginners)
- [OSCP Study Guide Tools](#oscp-study-guide-tools)
- [CTF Tools](#ctf-tools)
- [Bug Bounty Tools](#bug-bounty-tools)
- [Web Application Security Tools](#web-application-security-tools)
- [How Every Tool Is Documented](#how-every-tool-is-documented)
- [Who Is This For?](#who-is-this-for)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Contributing](#contributing)
- [License](#license)

---

## What Is This Repo?

This is a complete collection of **Kali Linux penetration testing tools** with documentation, tutorials, and command references. Every tool is documented from beginner to advanced level.

Whether you are preparing for **OSCP**, **CEH**, **GPEN**, or just learning **ethical hacking**, this repo has you covered. Each tool includes:

- Installation guide (APT, source, Docker)
- Basic to advanced commands with explanations
- Real-world penetration testing scenarios
- CTF and bug bounty walkthroughs
- Detection and defense for blue teams

---

## Why Star This Repo?

- **432+ tools** — the largest Kali Linux tool documentation collection on GitHub
- **115,000+ lines** of tutorials and examples
- **8-chapter format** per tool — not just a man page reprint
- **MITRE ATT&CK organized** — follow the same framework used by professionals
- **Beginner friendly** — starts simple, builds to advanced
- **Regular updates** — new tools added every week

---

## Quick Start

### For Beginners
Start with these essential Kali Linux tools:

1. **Nmap** — [Network scanning tutorial](01_reconnaissance/03_network_information/nmap/nmap.md)
2. **Gobuster** — [Directory brute forcing](01_reconnaissance/05_web_scanning/gobuster/gobuster.md)
3. **SQLMap** — [SQL injection automation](03_initial_access/sqlmap/sqlmap.md)
4. **Metasploit** — [Exploitation framework](03_initial_access/metasploit-framework/metasploit-framework.md)
5. **Hydra** — [Password brute forcing](08_credential_access/)

### For CTF Players
See [CTF Tools](#ctf-tools) section below.

### For Bug Bounty
See [Bug Bounty Tools](#bug-bounty-tools) section below.

---

## Phase 01 - Reconnaissance (80 Tools)

The first phase of any penetration test. Learn to discover hosts, enumerate services, and map attack surfaces.

### Network Scanning Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Nmap**](01_reconnaissance/03_network_information/nmap/nmap.md) | 1,048 | Port scanning, service detection, OS fingerprinting |
| [**Autorecon**](01_reconnaissance/03_network_information/autorecon/autorecon.md) | 3,631 | Automated network reconnaissance |
| [**Amass**](01_reconnaissance/03_network_information/amass/amass.md) | 2,121 | Attack surface mapping and DNS enumeration |
| [**Unicornscan**](01_reconnaissance/03_network_information/unicornscan/unicornscan.md) | 802 | Asynchronous TCP/UDP scanner |

### Web Vulnerability Scanning Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Burp Suite**](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md) | 1,079 | Web application security testing |
| [**Nuclei**](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md) | 1,249 | Template-based vulnerability scanner |
| [**Nikto**](01_reconnaissance/07_web_vulnerability_scanning/nikto/nikto.md) | - | Web server scanner |
| [**ZAP**](01_reconnaissance/07_web_vulnerability_scanning/zaproxy/zaproxy.md) | 517 | OWASP Zed Attack Proxy |

### Directory and Subdomain Brute Forcing

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Gobuster**](01_reconnaissance/05_web_scanning/gobuster/gobuster.md) | 1,182 | Directory, DNS, and vhost brute forcing |
| [**FFuf**](01_reconnaissance/05_web_scanning/ffuf/ffuf.md) | 1,143 | Fast web fuzzer |
| [**Feroxbuster**](01_reconnaissance/05_web_scanning/feroxbuster/feroxbuster.md) | 1,128 | Recursive content discovery |
| [**Dirsearch**](01_reconnaissance/05_web_scanning/dirsearch/dirsearch.md) | 842 | Web path scanner |

### OSINT Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Sherlock**](01_reconnaissance/02_identity_information/sherlock/sherlock.md) | 613 | Username search across 400+ social networks |
| [**TheHarvester**](01_reconnaissance/03_network_information/theharvester/theharvester.md) | 886 | Email and subdomain enumeration |
| [**SpiderFoot**](01_reconnaissance/01_host_information/spiderfoot/spiderfoot.md) | 629 | OSINT automation |
| [**Metagoofil**](01_reconnaissance/01_host_information/metagoofil/metagoofil.md) | 2,076 | Public document metadata extraction |

### DNS Enumeration Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**DNSRecon**](01_reconnaissance/04_dns_reconnaissance/dnsrecon/dnsrecon.md) | 1,282 | DNS enumeration and zone walking |
| [**DNSEnum**](01_reconnaissance/04_dns_reconnaissance/dnsenum/dnsenum.md) | 1,241 | DNS enumeration script |
| [**DNSMap**](01_reconnaissance/04_dns_reconnaissance/dnsmap/dnsmap.md) | 1,059 | DNS bruteforcing and subdomain scanning |

### Bluetooth and WiFi Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Bettercap**](01_reconnaissance/08_bluetooth/bettercap/bettercap.md) | 801 | Network attack and monitoring |
| [**Kismet**](01_reconnaissance/09_wifi/kismet/kismet.md) | 999 | Wireless network detector |
| [**Aircrack-ng**](01_reconnaissance/09_wifi/) | - | WiFi security assessment |

### Radio Frequency Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**HackRF**](01_reconnaissance/10_radio_frequency/hackrf/hackrf.md) | 716 | SDR transceiver |
| [**GnuRadio**](01_reconnaissance/10_radio_frequency/gnuradio/gnuradio.md) | 1,258 | Signal processing framework |

---

## Phase 02 - Resource Development (34 Tools)

Build your exploitation toolkit. Learn reverse engineering, fuzzing, and payload generation.

### Reverse Engineering Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Ghidra**](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md) | 675 | NSA reverse engineering suite |
| [**GDB**](02_resource_development/disassemblers_debuggers/gdb/gdb.md) | 816 | GNU Debugger |
| [**Radare2**](02_resource_development/disassemblers_debuggers/radare2/radare2.md) | 534 | Reverse engineering framework |
| [**Rizin**](02_resource_development/disassemblers_debuggers/rizin/rizin.md) | 589 | Radare2 fork |
| [**GEF**](02_resource_development/disassemblers_debuggers/gef/gef.md) | 578 | GDB Enhanced Features |

### Payload Generation Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**MSFvenom**](02_resource_development/shellcode_payload/msfvenom/msfvenom.md) | 907 | Metasploit payload generator |
| [**Veil**](02_resource_development/shellcode_payload/veil/veil.md) | 603 | AV evasion payload generator |
| [**Shellter**](02_resource_development/shellcode_payload/shellter/shellter.md) | 598 | Shellcode injection into PE files |
| [**Donut**](02_resource_development/shellcode_payload/donut/donut.md) | 580 | .NET shellcode generator |

### Fuzzing Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**AFL**](02_resource_development/fuzzing/afl-fuzz/afl-fuzz.md) | 602 | Coverage-guided fuzzer |
| [**Spike**](02_resource_development/fuzzing/spike/spike.md) | 414 | Network protocol fuzzer |

### Android and Java Reverse Engineering

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**JADX**](02_resource_development/java_android_reversing/jadx/jadx.md) | 520 | APK to Java source decompiler |
| [**Apktool**](02_resource_development/java_android_reversing/apktool/apktool.md) | 491 | APK reverse engineering |

---

## Phase 03 - Initial Access (10 Tools)

Gain initial access to target systems through exploitation and social engineering.

### SQL Injection Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**SQLMap**](03_initial_access/sqlmap/sqlmap.md) | 891 | Automated SQL injection and database takeover |
| [**SQLNinja**](03_initial_access/sqlninja/sqlninja.md) | 952 | SQL injection for MSSQL |
| [**SQLSus**](03_initial_access/sqlsus/sqlsus.md) | 1,256 | MySQL injection tool |
| [**jSQL**](03_initial_access/jsql/jsql.md) | 642 | Visual SQL injection |
| [**Commix**](03_initial_access/commix/commix.md) | 759 | Command injection exploitation |

### Exploitation Frameworks

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**Metasploit**](03_initial_access/metasploit-framework/metasploit-framework.md) | 632 | Penetration testing framework |
| [**JBoss Autopwn**](03_initial_access/jboss-autopwn/jboss-autopwn.md) | 612 | JBoss server exploitation |

### Social Engineering Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**SET Toolkit**](03_initial_access/setoolkit/setoolkit.md) | 511 | Social Engineering Toolkit |
| [**GoPhish**](03_initial_access/gophish/gophish.md) | 959 | Phishing simulation platform |

### DNS Attack Tools

| Tool | Lines | What It Does |
|------|-------|--------------|
| [**DNS-Rebind**](03_initial_access/dns-rebind/dns-rebind.md) | 690 | DNS rebinding attack tool |

---

## All Phases Overview

| Phase | Name | Tools | Status |
|-------|------|-------|--------|
| 01 | [Reconnaissance](01_reconnaissance/) | 80 | **Complete** |
| 02 | [Resource Development](02_resource_development/) | 34 | **Complete** |
| 03 | [Initial Access](03_initial_access/) | 10 | **Complete** |
| 04 | Execution | 6 | Developing |
| 05 | Persistence | 7 | Developing |
| 06 | Privilege Escalation | 6 | Developing |
| 07 | Defense Evasion | 16 | Developing |
| 08 | Credential Access | 52 | Developing |
| 09 | Discovery | 88 | Developing |
| 10 | Lateral Movement | 3 | Developing |
| 11 | Collection | 12 | Developing |
| 12 | Command and Control | 30 | Developing |
| 13 | Exfiltration | 3 | Developing |
| 14 | Impact | 10 | Developing |
| 15 | Forensics | 51 | Developing |
| 16 | Services and Other | 15 | Developing |

---

## Kali Linux Commands Cheat Sheet

Quick reference for the most used Kali Linux commands:

### Network Scanning
```bash
nmap -sV -sC target           # Service version + default scripts
nmap -p- target               # Scan all 65535 ports
nmap -A target                # Aggressive scan (OS, version, scripts, traceroute)
```

### Directory Brute Forcing
```bash
gobuster dir -u URL -w wordlist.txt           # Basic directory scan
ffuf -u URL/FUZZ -w wordlist.txt              # Fast fuzzing
feroxbuster -u URL -w wordlist.txt            # Recursive scan
```

### SQL Injection
```bash
sqlmap -u "URL?id=1" --dbs                    # Enumerate databases
sqlmap -u "URL?id=1" -D dbname --tables       # Enumerate tables
sqlmap -u "URL?id=1" -D dbname -T table --dump # Dump table data
```

### Exploitation
```bash
msfconsole                                   # Start Metasploit
msfvenom -p payload LHOST=IP LPORT=PORT -f exe  # Generate payload
searchsploit apache 2.4                       # Search ExploitDB
```

### Password Attacks
```bash
hydra -l user -P wordlist.txt ssh://target     # SSH brute force
john --wordlist=wordlist.txt hash.txt          # Crack hashes
hashcat -m 0 hash.txt wordlist.txt             # GPU hash cracking
```

---

## Best Kali Tools for Beginners

If you are just starting with Kali Linux and penetration testing, learn these tools first:

1. **Nmap** — Every penetration test starts with network scanning
2. **Gobuster** — Find hidden directories and files on web servers
3. **SQLMap** — Automate SQL injection attacks
4. **Metasploit** — The most popular exploitation framework
5. **Hydra** — Brute force login pages and services
6. **Nikto** — Scan web servers for vulnerabilities
7. **John the Ripper** — Crack password hashes
8. **Wireshark** — Analyze network traffic

See the [Beginner Track](#quick-start) for a structured learning path.

---

## OSCP Study Guide Tools

Preparing for the **Offensive Security Certified Professional (OSCP)** exam? These are the tools you need to master:

| Category | Tools |
|----------|-------|
| **Reconnaissance** | Nmap, Amass, Gobuster, FFuf |
| **Web Attacks** | Burp Suite, SQLMap, Nikto |
| **Exploitation** | Metasploit, MSFvenom, SearchSploit |
| **Password Attacks** | Hydra, John, Hashcat |
| **Privilege Escalation** | LinPEAS, WinPEAS, Linux Smart Enumeration |
| **Post Exploitation** | Metasploit Meterpreter, Impacket |
| **Reporting** | Documentation templates |

Each tool above links to its full documentation in this repo.

---

## CTF Tools

Capture The Flag competitions require quick access to the right tools. Here are the most useful ones:

| CTF Category | Tools |
|--------------|-------|
| **Web** | SQLMap, Gobuster, FFuf, Burp Suite, Nikto |
| **Crypto** | CyberChef, John, Hashcat |
| **Forensics** | Binwalk, Steghide, Volatility |
| **Reverse Engineering** | GDB, GEF, Ghidra, Radare2, Rizin |
| **Pwn** | GDB, GEF, MSFvenom, ROPgadget |
| **OSINT** | Sherlock, TheHarvester, SpiderFoot |

---

## Bug Bounty Tools

Top tools for bug bounty programs:

| Category | Tools |
|----------|-------|
| **Recon** | Amass, Subfinder, Assetfinder, LinkFinder |
| **Scanning** | Nuclei, Nmap, Masscan |
| **Web** | Burp Suite, FFuf, Gobuster, Arjun |
| **API** | Kiterunner, Arjun |
| **Vuln Scanning** | Nuclei, Nikto, Wapiti |

---

## Web Application Security Tools

Complete list of tools for testing web applications:

| Tool | Use Case |
|------|----------|
| **Burp Suite** | Intercepting proxy, vulnerability scanning |
| **SQLMap** | SQL injection detection and exploitation |
| **Nikto** | Web server vulnerability scanning |
| **Gobuster** | Directory and file discovery |
| **FFuf** | Web fuzzing |
| **Nuclei** | Template-based vulnerability scanning |
| **Commix** | Command injection |
| **XSSer** | Cross-site scripting detection |
| **WPScan** | WordPress vulnerability scanning |

---

## How Every Tool Is Documented

Each tool follows a consistent 8-chapter structure:

```
tool-name.md
+-- Chapter 1: Introduction & Overview
|   +-- What is it?
|   +-- History & Background
|   +-- When to Use / When NOT to Use
|   +-- Key Concepts
+-- Chapter 2: Installation & Setup
|   +-- System Requirements
|   +-- Installation Methods (APT, Source, Docker)
|   +-- Verification
+-- Chapter 3: Basic Usage
|   +-- First Run (with sample output)
|   +-- Essential Commands (5-10+)
|   +-- COMPLETE Flag Reference Tables
+-- Chapter 4: Intermediate Usage
|   +-- Bash Scripting & Automation
|   +-- Python Scripting
|   +-- Tool Chaining
+-- Chapter 5: Advanced Usage
|   +-- Custom Techniques
|   +-- Evasion Techniques
|   +-- Pentest Workflow Integration
+-- Chapter 6: Real-World Scenarios
|   +-- Security Audit walkthrough
|   +-- CTF Challenge
|   +-- Bug Bounty
+-- Chapter 7: Detection & Defense
|   +-- How Defenders Detect It
|   +-- IDS/IPS Signatures
|   +-- Blue Team Response
+-- Chapter 8: Troubleshooting
    +-- Common Errors & Solutions
    +-- FAQ
```

Plus a **CHEATSHEET.md** in every tool directory for quick field reference.

---

## Who Is This For?

- **Students** learning cybersecurity and penetration testing
- **CTF players** looking for tool references during competitions
- **Bug bounty hunters** wanting comprehensive tool documentation
- **Pentesters** needing a quick reference for tools they don't use daily
- **Blue teamers** wanting to understand attacker tools for detection
- **OSCP/GPEN/CEH candidates** preparing for certifications
- **Anyone** who wants to learn Kali Linux tools from scratch

---

## Frequently Asked Questions

### What is Kali Linux?
Kali Linux is a Debian-based Linux distribution designed for digital forensics and penetration testing. It comes with hundreds of security tools pre-installed.

### What are the best Kali Linux tools for beginners?
Start with Nmap, Gobuster, SQLMap, Metasploit, and Hydra. These cover the core skills needed for penetration testing.

### Is this repo free?
Yes. This repo is completely free and open source under the MIT License.

### How often is this repo updated?
New tools and documentation are added regularly. Check the [Roadmap](#roadmap) for what is coming next.

### Can I use these tools legally?
Only on systems you own or have written authorization to test. Unauthorized access to computer systems is illegal.

### What is the difference between this repo and other Kali tool lists?
Most lists just link to tools. This repo provides full documentation with tutorials, commands, examples, and real-world scenarios for every tool.

### How do I contribute?
See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Contributing

This project is actively developing. Contributions are welcome:

1. Pick a tool that needs documentation
2. Follow the 8-chapter template
3. Submit a pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## License

MIT License - see [LICENSE](LICENSE) for details.

Educational purposes only. Always get authorization before testing systems you don't own.

---

<div align="center">

### Star This Repo

<img src="star.png" width="500" alt="Give a Star">

If this helps your learning, give it a star. It motivates more content.

**Phase 01: Done** | **Phase 02: Done** | **Phase 03: Done** | **Phase 04: Coming Next**

[Back to Top](#kali-linux-tools---complete-documentation--tutorial)

</div>
