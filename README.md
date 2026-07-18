<div align="center">

<img src="headeroftherepo.png" width="100%" alt="Kali Linux penetration testing tools documentation - complete tutorial and cheat sheet">

<br>

# Kali Linux Tools Documentation

### Complete Penetration Testing Guide with Tutorials and Commands

<br>

![GitHub Stars](https://img.shields.io/github/stars/random-dude6969/full-kali-learning?style=social&label=Stars)
![GitHub Forks](https://img.shields.io/github/forks/random-dude6969/full-kali-learning?style=social&label=Forks)
![GitHub Watchers](https://img.shields.io/github/watchers/random-dude6969/full-kali-learning?style=social&label=Watchers)

![Tools](https://img.shields.io/badge/Tools-299-red?style=flat-square&logo=kali-linux&logoColor=white)
![Lines](https://img.shields.io/badge/Lines-200K%2B-green?style=flat-square)
![Phases](https://img.shields.io/badge/Phases-9%2F16-blue?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)
![PRs](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=flat-square)

---

**The most comprehensive Kali Linux tool documentation on GitHub.**

299 penetration testing tools documented with step-by-step tutorials, command references, and real-world scenarios.

From network scanning to exploitation to post-exploitation — complete guides for ethical hacking, CTF competitions, bug bounty, and security certifications.

<br>

<img src="star.png" width="500" alt="Star this Kali Linux tools repository">

</div>

---

## Table of Contents

- [What Is This Project](#what-is-this-project)
- [Featured Tools](#featured-tools)
- [Complete Phase Overview](#complete-phase-overview)
- [All Documented Tools](#all-documented-tools)
- [Learning Path](#learning-path)
- [OSCP Certification Guide](#oscp-certification-guide)
- [CTF Competition Tools](#ctf-competition-tools)
- [Bug Bounty Toolkit](#bug-bounty-toolkit)
- [Web Application Security](#web-application-security)
- [Password Cracking Guide](#password-cracking-guide)
- [Reverse Engineering Tools](#reverse-engineering-tools)
- [How Documentation Is Structured](#how-documentation-is-structured)
- [Who Uses This](#who-uses-this)
- [Contributing](#contributing)
- [License](#license)

---

## What Is This Project

This repository contains documentation for **299 Kali Linux penetration testing tools**. Each tool is documented with:

- **Installation guides** for APT, source, and Docker
- **Command references** with explanations for every flag
- **Real-world scenarios** for security audits, CTFs, and bug bounties
- **Detection and defense** sections for blue team defenders
- **Cheat sheets** for quick reference

The documentation follows the **MITRE ATT&CK framework** — the same framework used by security professionals worldwide.

### Why This Matters

| Other Resources | This Repository |
|-----------------|-----------------|
| Tool download links | Full documentation |
| Basic usage | Advanced techniques |
| No examples | Real-world scenarios |
| No defense info | Blue team sections |
| Scattered across sites | One organized collection |

---

## Featured Tools

The most popular penetration testing tools with direct links to full documentation:

### Network Scanning and Enumeration

| Tool | Lines | Description |
|------|-------|-------------|
| **[Nmap](01_reconnaissance/03_network_information/nmap/nmap.md)** | 1,048 | Port scanning, service detection, OS fingerprinting |
| **[Masscan](09_discovery/network_service_discovery/masscan/masscan.md)** | 340 | Fastest port scanner |
| **[Amass](01_reconnaissance/03_network_information/amass/amass.md)** | 2,121 | Attack surface mapping |
| **[Autorecon](01_reconnaissance/03_network_information/autorecon/autorecon.md)** | 3,631 | Automated network reconnaissance |

### Web Application Testing

| Tool | Lines | Description |
|------|-------|-------------|
| **[Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md)** | 1,079 | Web application security testing |
| **[SQLMap](03_initial_access/sqlmap/sqlmap.md)** | 891 | Automated SQL injection |
| **[Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md)** | 1,182 | Directory brute forcing |
| **[Nuclei](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md)** | 1,249 | Template-based vulnerability scanner |
| **[FFuf](01_reconnaissance/05_web_scanning/ffuf/ffuf.md)** | 1,143 | Fast web fuzzing |

### Exploitation Frameworks

| Tool | Lines | Description |
|------|-------|-------------|
| **[Metasploit](03_initial_access/metasploit-framework/metasploit-framework.md)** | 632 | Penetration testing framework |
| **[MSFvenom](02_resource_development/shellcode_payload/msfvenom/msfvenom.md)** | 907 | Payload generation |
| **[SearchSploit](02_resource_development/exploit_resources/searchsploit/searchsploit.md)** | 442 | Exploit database search |

### Password Attacks

| Tool | Lines | Description |
|------|-------|-------------|
| **[Hashcat](08_credential_access/password_cracking/hashcat/hashcat.md)** | 800+ | GPU-accelerated hash cracking |
| **[John the Ripper](08_credential_access/password_cracking/john/john.md)** | 700+ | Versatile password cracker |
| **[Hydra](08_credential_access/brute_force/hydra/hydra.md)** | 600+ | Online brute force attacks |
| **[Responder](08_credential_access/credential_harvesting/responder/responder.md)** | 600+ | Network credential harvesting |

### Post-Exploitation

| Tool | Lines | Description |
|------|-------|-------------|
| **[Mimikatz](07_defense_evasion/pass_the_hash/mimikatz/mimikatz.md)** | 564 | Windows credential extraction |
| **[LinPEAS](06_privilege_escalation/linpeas/linpeas.md)** | 574 | Linux privilege escalation |
| **[BloodHound](09_discovery/active_directory/bloodhound/bloodhound.md)** | 700+ | Active Directory attack paths |
| **[Impacket](07_defense_evasion/pass_the_hash/impacket-scripts/impacket-scripts.md)** | 837 | Network protocol exploitation |

### Reverse Engineering

| Tool | Lines | Description |
|------|-------|-------------|
| **[Ghidra](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md)** | 675 | NSA reverse engineering suite |
| **[GDB](02_resource_development/disassemblers_debuggers/gdb/gdb.md)** | 816 | GNU Debugger |
| **[Radare2](02_resource_development/disassemblers_debuggers/radare2/radare2.md)** | 534 | Reverse engineering framework |

---

## Complete Phase Overview

The documentation follows the MITRE ATT&CK framework — 16 phases of a cyber attack:

| Phase | Name | Tools | Status | Key Tools |
|-------|------|-------|--------|-----------|
| 01 | [Reconnaissance](01_reconnaissance/) | 80 | DONE | Nmap, Gobuster, Burp Suite, Nuclei |
| 02 | [Resource Development](02_resource_development/) | 34 | DONE | Ghidra, MSFvenom, AFL, Radare2 |
| 03 | [Initial Access](03_initial_access/) | 10 | DONE | SQLMap, Metasploit, GoPhish |
| 04 | [Execution](04_execution/) | 6 | DONE | BeEF, XSSer, PowerSploit |
| 05 | [Persistence](05_persistence/) | 7 | DONE | Weevely, Laudanum, PHPGGC |
| 06 | [Privilege Escalation](06_privilege_escalation/) | 6 | DONE | LinPEAS, WinPEAS, Lynis |
| 07 | [Defense Evasion](07_defense_evasion/) | 16 | DONE | Mimikatz, Impacket, Veil |
| 08 | [Credential Access](08_credential_access/) | 52 | DONE | Hydra, John, Hashcat, Responder |
| 09 | [Discovery](09_discovery/) | 88 | DONE | Wireshark, BloodHound, Ettercap |
| 10 | Lateral Movement | 3 | Coming | PsExec, WMI |
| 11 | Collection | 12 | Coming | Keyloggers, Screen capture |
| 12 | Command and Control | 30 | Coming | C2 frameworks |
| 13 | Exfiltration | 3 | Coming | Data exfil tools |
| 14 | Impact | 10 | Coming | DDoS, Destructive tools |
| 15 | Forensics | 51 | Coming | Volatility, Autopsy |
| 16 | Services | 15 | Coming | Network services |

---

## All Documented Tools

Every tool in this repository with direct links. Click any tool to open its full documentation.

<details>
<summary><b>Phase 01 — Reconnaissance (80 tools)</b></summary>

### Host Information
| Tool | Lines | Link |
|------|-------|------|
| Metagoofil | 2,076 | [Docs](01_reconnaissance/01_host_information/metagoofil/metagoofil.md) |
| SpiderFoot | 629 | [Docs](01_reconnaissance/01_host_information/spiderfoot/spiderfoot.md) |

### Identity Information
| Tool | Lines | Link |
|------|-------|------|
| EmailHarvester | 1,893 | [Docs](01_reconnaissance/02_identity_information/emailharvester/emailharvester.md) |
| Instaloader | 1,795 | [Docs](01_reconnaissance/02_identity_information/instaloader/instaloader.md) |
| Sherlock | 613 | [Docs](01_reconnaissance/02_identity_information/sherlock/sherlock.md) |
| Photon | 510 | [Docs](01_reconnaissance/02_identity_information/photon/photon.md) |
| Tookie-OSINT | 546 | [Docs](01_reconnaissance/02_identity_information/tookie-osint/tookie-osint.md) |
| LinkedIn2Username | 501 | [Docs](01_reconnaissance/02_identity_information/linkedin2username/linkedin2username.md) |

### Network Information
| Tool | Lines | Link |
|------|-------|------|
| Autorecon | 3,632 | [Docs](01_reconnaissance/03_network_information/autorecon/autorecon.md) |
| Amass | 2,121 | [Docs](01_reconnaissance/03_network_information/amass/amass.md) |
| Nmap | 1,048 | [Docs](01_reconnaissance/03_network_information/nmap/nmap.md) |
| TheHarvester | 886 | [Docs](01_reconnaissance/03_network_information/theharvester/theharvester.md) |
| Unicornscan | 802 | [Docs](01_reconnaissance/03_network_information/unicornscan/unicornscan.md) |
| Zenmap | 598 | [Docs](01_reconnaissance/03_network_information/zenmap/zenmap.md) |
| Dmitry | 535 | [Docs](01_reconnaissance/03_network_information/dmitry/dmitry.md) |
| Legion | 523 | [Docs](01_reconnaissance/03_network_information/legion/legion.md) |

### DNS Reconnaissance
| Tool | Lines | Link |
|------|-------|------|
| DNSRecon | 1,282 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnsrecon/dnsrecon.md) |
| DNSEnum | 1,241 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnsenum/dnsenum.md) |
| DNSMap | 1,059 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnsmap/dnsmap.md) |
| MassDNS | 1,136 | [Docs](01_reconnaissance/04_dns_reconnaissance/massdns/massdns.md) |
| DNSWalk | 1,185 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnswalk/dnswalk.md) |
| DNSTracer | 1,028 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnstracer/dnstracer.md) |

### Web Scanning
| Tool | Lines | Link |
|------|-------|------|
| Gobuster | 1,182 | [Docs](01_reconnaissance/05_web_scanning/gobuster/gobuster.md) |
| FFuf | 1,143 | [Docs](01_reconnaissance/05_web_scanning/ffuf/ffuf.md) |
| Feroxbuster | 1,128 | [Docs](01_reconnaissance/05_web_scanning/feroxbuster/feroxbuster.md) |
| Subfinder | 1,122 | [Docs](01_reconnaissance/05_web_scanning/subfinder/subfinder.md) |
| GoSpider | 1,065 | [Docs](01_reconnaissance/05_web_scanning/gospider/gospider.md) |
| FinalRecon | 1,033 | [Docs](01_reconnaissance/05_web_scanning/finalrecon/finalrecon.md) |
| LBD | 1,014 | [Docs](01_reconnaissance/05_web_scanning/lbd/lbd.md) |
| Findomain | 1,008 | [Docs](01_reconnaissance/05_web_scanning/findomain/findomain.md) |
| Recon-ng | 988 | [Docs](01_reconnaissance/05_web_scanning/recon-ng/recon-ng.md) |
| Arjun | 944 | [Docs](01_reconnaissance/05_web_scanning/arjun/arjun.md) |
| Wfuzz | 910 | [Docs](01_reconnaissance/05_web_scanning/wfuzz/wfuzz.md) |
| Sublist3r | 902 | [Docs](01_reconnaissance/05_web_scanning/sublist3r/sublist3r.md) |
| DirSearch | 842 | [Docs](01_reconnaissance/05_web_scanning/dirsearch/dirsearch.md) |
| Dirb | 783 | [Docs](01_reconnaissance/05_web_scanning/dirb/dirb.md) |
| Assetfinder | 758 | [Docs](01_reconnaissance/05_web_scanning/assetfinder/assetfinder.md) |
| WPProbe | 751 | [Docs](01_reconnaissance/05_web_scanning/wpprobe/wpprobe.md) |
| Parsero | 746 | [Docs](01_reconnaissance/05_web_scanning/parsero/parsero.md) |
| URLCrazy | 642 | [Docs](01_reconnaissance/05_web_scanning/urlcrazy/urlcrazy.md) |
| DirBuster | 616 | [Docs](01_reconnaissance/05_web_scanning/dirbuster/dirbuster.md) |
| Uro | 526 | [Docs](01_reconnaissance/05_web_scanning/uro/uro.md) |

### Vulnerability Scanning
| Tool | Lines | Link |
|------|-------|------|
| Heartleech | 819 | [Docs](01_reconnaissance/06_vulnerability_scanning/heartleech/heartleech.md) |
| GVM | 519 | [Docs](01_reconnaissance/06_vulnerability_scanning/gvm/gvm.md) |

### Web Vulnerability Scanning
| Tool | Lines | Link |
|------|-------|------|
| Nuclei | 1,249 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md) |
| Nikto | 1,169 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/nikto/nikto.md) |
| DavTest | 1,148 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/davtest/davtest.md) |
| JoomScan | 1,114 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/joomscan/joomscan.md) |
| Burp Suite | 1,079 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md) |
| Caido | 1,079 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/caido/caido.md) |
| CRLFuzz | 1,045 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/crlfuzz/crlfuzz.md) |
| WPScan | 1,011 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/wpscan/wpscan.md) |
| Skipfish | 879 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/skipfish/skipfish.md) |
| Wapiti | 840 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/wapiti/wapiti.md) |
| SSTImap | 823 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/sstimap/sstimap.md) |
| Tinja | 822 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/tinja/tinja.md) |
| Subjack | 813 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/subjack/subjack.md) |
| WhatWeb | 772 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/whatweb/whatweb.md) |
| Paros | 706 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/paros/paros.md) |
| WCVS | 689 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/wcvs/wcvs.md) |
| WebScarab | 641 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/webscarab/webscarab.md) |
| Watobo | 628 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/watobo/watobo.md) |
| ZAProxy | 517 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/zaproxy/zaproxy.md) |

### Bluetooth
| Tool | Lines | Link |
|------|-------|------|
| Bettercap | 801 | [Docs](01_reconnaissance/08_bluetooth/bettercap/bettercap.md) |
| Ubertooth-Util | 847 | [Docs](01_reconnaissance/08_bluetooth/ubertooth-util/ubertooth-util.md) |
| Bluesnarfer | 732 | [Docs](01_reconnaissance/08_bluetooth/bluesnarfer/bluesnarfer.md) |
| BTScanner | 716 | [Docs](01_reconnaissance/08_bluetooth/btscanner/btscanner.md) |
| Spooftooph | 667 | [Docs](01_reconnaissance/08_bluetooth/spooftooph/spooftooph.md) |
| BlueLog | 665 | [Docs](01_reconnaissance/08_bluetooth/bluelog/bluelog.md) |
| RedFang | 617 | [Docs](01_reconnaissance/08_bluetooth/redfang/redfang.md) |
| BlueRanger | 603 | [Docs](01_reconnaissance/08_bluetooth/blueranger/blueranger.md) |

### WiFi
| Tool | Lines | Link |
|------|-------|------|
| Kismet | 999 | [Docs](01_reconnaissance/09_wifi/kismet/kismet.md) |
| Wash | 845 | [Docs](01_reconnaissance/09_wifi/wash/wash.md) |
| Sparrow-WiFi | 796 | [Docs](01_reconnaissance/09_wifi/sparrow-wifi/sparrow-wifi.md) |
| Asleap | 617 | [Docs](01_reconnaissance/09_wifi/asleap/asleap.md) |

### Radio Frequency
| Tool | Lines | Link |
|------|-------|------|
| GnuRadio | 1,258 | [Docs](01_reconnaissance/10_radio_frequency/gnuradio/gnuradio.md) |
| RFcat | 1,019 | [Docs](01_reconnaissance/10_radio_frequency/rfcat/rfcat.md) |
| Chirp | 839 | [Docs](01_reconnaissance/10_radio_frequency/chirp/chirp.md) |
| HackRF | 716 | [Docs](01_reconnaissance/10_radio_frequency/hackrf/hackrf.md) |
| Gqrx | 766 | [Docs](01_reconnaissance/10_radio_frequency/gqrx/gqrx.md) |

</details>

<details>
<summary><b>Phase 02 — Resource Development (34 tools)</b></summary>

### Disassemblers and Debuggers
| Tool | Lines | Link |
|------|-------|------|
| GDB | 816 | [Docs](02_resource_development/disassemblers_debuggers/gdb/gdb.md) |
| Ghidra | 675 | [Docs](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md) |
| Rizin | 589 | [Docs](02_resource_development/disassemblers_debuggers/rizin/rizin.md) |
| GEF | 578 | [Docs](02_resource_development/disassemblers_debuggers/gef/gef.md) |
| Capstone | 547 | [Docs](02_resource_development/disassemblers_debuggers/capstone/capstone.md) |
| Radare2 | 534 | [Docs](02_resource_development/disassemblers_debuggers/radare2/radare2.md) |
| Cutter | 444 | [Docs](02_resource_development/disassemblers_debuggers/cutter/cutter.md) |
| EDB-Debugger | 415 | [Docs](02_resource_development/disassemblers_debuggers/edb-debugger/edb-debugger.md) |
| RecStudio | 376 | [Docs](02_resource_development/disassemblers_debuggers/recstudio/recstudio.md) |
| OllyDbg | 349 | [Docs](02_resource_development/disassemblers_debuggers/ollydbg/ollydbg.md) |

### Exploit Resources
| Tool | Lines | Link |
|------|-------|------|
| SearchSploit | 442 | [Docs](02_resource_development/exploit_resources/searchsploit/searchsploit.md) |
| ExploitDB | 413 | [Docs](02_resource_development/exploit_resources/exploitdb/exploitdb.md) |
| Pompem | 330 | [Docs](02_resource_development/exploit_resources/pompem/pompem.md) |

### Fuzzing
| Tool | Lines | Link |
|------|-------|------|
| AFL | 602 | [Docs](02_resource_development/fuzzing/afl-fuzz/afl-fuzz.md) |
| Spike | 414 | [Docs](02_resource_development/fuzzing/spike/spike.md) |
| SFuzz | 373 | [Docs](02_resource_development/fuzzing/sfuzz/sfuzz.md) |
| BED | 351 | [Docs](02_resource_development/fuzzing/bed/bed.md) |

### Java and Android Reversing
| Tool | Lines | Link |
|------|-------|------|
| JADX | 520 | [Docs](02_resource_development/java_android_reversing/jadx/jadx.md) |
| Apktool | 491 | [Docs](02_resource_development/java_android_reversing/apktool/apktool.md) |
| Bytecode Viewer | 421 | [Docs](02_resource_development/java_android_reversing/bytecode-viewer/bytecode-viewer.md) |
| DEX2JAR | 393 | [Docs](02_resource_development/java_android_reversing/dex2jar/dex2jar.md) |
| JD-GUI | 352 | [Docs](02_resource_development/java_android_reversing/jd-gui/jd-gui.md) |
| JavaSnoop | 301 | [Docs](02_resource_development/java_android_reversing/javasnoop/javasnoop.md) |

### Shellcode and Payload Generation
| Tool | Lines | Link |
|------|-------|------|
| MSFvenom | 907 | [Docs](02_resource_development/shellcode_payload/msfvenom/msfvenom.md) |
| Veil | 603 | [Docs](02_resource_development/shellcode_payload/veil/veil.md) |
| Shellter | 598 | [Docs](02_resource_development/shellcode_payload/shellter/shellter.md) |
| Donut | 580 | [Docs](02_resource_development/shellcode_payload/donut/donut.md) |
| Backdoor-Factory | 514 | [Docs](02_resource_development/shellcode_payload/backdoor-factory/backdoor-factory.md) |
| ShellNoob | 506 | [Docs](02_resource_development/shellcode_payload/shellnoob/shellnoob.md) |
| Cymothoa | 485 | [Docs](02_resource_development/shellcode_payload/cymothoa/cymothoa.md) |
| MSF-NASM | 436 | [Docs](02_resource_development/shellcode_payload/msf-nasm_shell/msf-nasm_shell.md) |
| EXE2HEX | 401 | [Docs](02_resource_development/shellcode_payload/exe2hex/exe2hex.md) |
| Sickle-PDK | 396 | [Docs](02_resource_development/shellcode_payload/sickle-pdk/sickle-pdk.md) |
| MSFPC | 389 | [Docs](02_resource_development/shellcode_payload/msfpc/msfpc.md) |

</details>

<details>
<summary><b>Phase 03 — Initial Access (10 tools)</b></summary>

| Tool | Lines | Link |
|------|-------|------|
| SQLSus | 1,256 | [Docs](03_initial_access/sqlsus/sqlsus.md) |
| GoPhish | 959 | [Docs](03_initial_access/gophish/gophish.md) |
| SQLNinja | 952 | [Docs](03_initial_access/sqlninja/sqlninja.md) |
| SQLMap | 891 | [Docs](03_initial_access/sqlmap/sqlmap.md) |
| Commix | 759 | [Docs](03_initial_access/commix/commix.md) |
| DNS-Rebind | 690 | [Docs](03_initial_access/dns-rebind/dns-rebind.md) |
| jSQL | 642 | [Docs](03_initial_access/jsql/jsql.md) |
| Metasploit | 632 | [Docs](03_initial_access/metasploit-framework/metasploit-framework.md) |
| JBoss-Autopwn | 612 | [Docs](03_initial_access/jboss-autopwn/jboss-autopwn.md) |
| SET Toolkit | 511 | [Docs](03_initial_access/setoolkit/setoolkit.md) |

</details>

<details>
<summary><b>Phase 04 — Execution (6 tools)</b></summary>

| Tool | Lines | Link |
|------|-------|------|
| XSSer | 828 | [Docs](04_execution/xsser/xsser.md) |
| Nishang | 738 | [Docs](04_execution/nishang/nishang.md) |
| PowerSploit | 738 | [Docs](04_execution/powersploit/powersploit.md) |
| BeEF | 723 | [Docs](04_execution/beef-xss/beef-xss.md) |
| Evilgrade | 708 | [Docs](04_execution/evilgrade/evilgrade.md) |
| Armitage | 611 | [Docs](04_execution/armitage/armitage.md) |

</details>

<details>
<summary><b>Phase 05 — Persistence (7 tools)</b></summary>

| Tool | Lines | Link |
|------|-------|------|
| Laudanum | 689 | [Docs](05_persistence/laudanum/laudanum.md) |
| Weevely | 630 | [Docs](05_persistence/weevely/weevely.md) |
| Cymothoa | 614 | [Docs](05_persistence/cymothoa/cymothoa.md) |
| WeBaCoo | 588 | [Docs](05_persistence/webacoo/webacoo.md) |
| Backdoor-Factory | 550 | [Docs](05_persistence/backdoor-factory/backdoor-factory.md) |
| Webshells | 517 | [Docs](05_persistence/webshells/webshells.md) |
| PHPGGC | 510 | [Docs](05_persistence/phpggc/phpggc.md) |

</details>

<details>
<summary><b>Phase 06 — Privilege Escalation (6 tools)</b></summary>

| Tool | Lines | Link |
|------|-------|------|
| BloodyAD | 746 | [Docs](06_privilege_escalation/bloodyad/bloodyad.md) |
| Lynis | 644 | [Docs](06_privilege_escalation/lynis/lynis.md) |
| Unix-Privesc-Check | 616 | [Docs](06_privilege_escalation/unix-privesc-check/unix-privesc-check.md) |
| LinPEAS | 574 | [Docs](06_privilege_escalation/linpeas/linpeas.md) |
| PEASS | 508 | [Docs](06_privilege_escalation/peass/peass.md) |
| WinPEAS | 505 | [Docs](06_privilege_escalation/winpeas/winpeas.md) |

</details>

<details>
<summary><b>Phase 07 — Defense Evasion (16 tools)</b></summary>

| Tool | Lines | Link |
|------|-------|------|
| Impacket | 837 | [Docs](07_defense_evasion/pass_the_hash/impacket-scripts/impacket-scripts.md) |
| Veil | 790 | [Docs](07_defense_evasion/av_evasion/veil/veil.md) |
| Shellter | 743 | [Docs](07_defense_evasion/av_evasion/shellter/shellter.md) |
| Evil-WinRM | 729 | [Docs](07_defense_evasion/pass_the_hash/evil-winrm/evil-winrm.md) |
| SniffJoke | 697 | [Docs](07_defense_evasion/av_evasion/sniffjoke/sniffjoke.md) |
| Stegsnow | 655 | [Docs](07_defense_evasion/steganography/stegsnow/stegsnow.md) |
| FreeRDP | 649 | [Docs](07_defense_evasion/pass_the_hash/freerdp/freerdp.md) |
| OutGuess | 612 | [Docs](07_defense_evasion/steganography/outguess/outguess.md) |
| SMBMap | 611 | [Docs](07_defense_evasion/pass_the_hash/smbmap/smbmap.md) |
| Stegosuite | 600 | [Docs](07_defense_evasion/steganography/stegosuite/stegosuite.md) |
| Rubeus | 583 | [Docs](07_defense_evasion/pass_the_hash/rubeus/rubeus.md) |
| Mimikatz | 564 | [Docs](07_defense_evasion/pass_the_hash/mimikatz/mimikatz.md) |
| Passing-the-Hash | 542 | [Docs](07_defense_evasion/pass_the_hash/passing-the-hash/passing-the-hash.md) |
| CrackMapExec | 536 | [Docs](07_defense_evasion/pass_the_hash/crackmapexec/crackmapexec.md) |
| Steghide | 536 | [Docs](07_defense_evasion/steganography/steghide/steghide.md) |
| NetExec | 532 | [Docs](07_defense_evasion/pass_the_hash/netexec/netexec.md) |

</details>

---

## Learning Path

### Beginner (Week 1-2)
Start with these foundational tools:
1. **Nmap** — Network scanning basics
2. **Gobuster** — Directory discovery
3. **SQLMap** — SQL injection
4. **Metasploit** — Exploitation framework
5. **Hydra** — Brute force attacks

### Intermediate (Week 3-6)
Build on your foundation:
1. **Burp Suite** — Web application testing
2. **Nuclei** — Vulnerability scanning
3. **Hashcat** — Password cracking
4. **BloodHound** — Active Directory
5. **Wireshark** — Network analysis

### Advanced (Month 2+)
Master advanced techniques:
1. **Ghidra** — Reverse engineering
2. **MSFvenom** — Payload creation
3. **Mimikatz** — Credential extraction
4. **Impacket** — Network exploitation
5. **Custom tooling** — Build your own

---

## OSCP Certification Guide

The Offensive Security Certified Professional (OSCP) exam requires mastery of specific tools. This repository covers every tool in the OSCP syllabus:

| Category | Tools in This Repo |
|----------|-------------------|
| Enumeration | [Nmap](01_reconnaissance/03_network_information/nmap/nmap.md), [Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md), [FFuf](01_reconnaissance/05_web_scanning/ffuf/ffuf.md) |
| Web Attacks | [SQLMap](03_initial_access/sqlmap/sqlmap.md), [Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md), [Nikto](01_reconnaissance/07_web_vulnerability_scanning/nikto/nikto.md) |
| Exploitation | [Metasploit](03_initial_access/metasploit-framework/metasploit-framework.md), [MSFvenom](02_resource_development/shellcode_payload/msfvenom/msfvenom.md), [SearchSploit](02_resource_development/exploit_resources/searchsploit/searchsploit.md) |
| Password Attacks | [Hydra](08_credential_access/brute_force/hydra/hydra.md), [John](08_credential_access/password_cracking/john/john.md), [Hashcat](08_credential_access/password_cracking/hashcat/hashcat.md) |
| Privilege Escalation | [LinPEAS](06_privilege_escalation/linpeas/linpeas.md), [WinPEAS](06_privilege_escalation/winpeas/winpeas.md) |
| Post Exploitation | [Mimikatz](07_defense_evasion/pass_the_hash/mimikatz/mimikatz.md), [Impacket](07_defense_evasion/pass_the_hash/impacket-scripts/impacket-scripts.md) |

---

## CTF Competition Tools

Capture The Flag competitions require quick access to specific tools:

| CTF Category | Tools |
|--------------|-------|
| **Web** | [SQLMap](03_initial_access/sqlmap/sqlmap.md), [Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md), [FFuf](01_reconnaissance/05_web_scanning/ffuf/ffuf.md), [Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md) |
| **Crypto** | [John](08_credential_access/password_cracking/john/john.md), [Hashcat](08_credential_access/password_cracking/hashcat/hashcat.md) |
| **Forensics** | [Steghide](07_defense_evasion/steganography/steghide/steghide.md), [Binwalk](15_forensics/) |
| **Reverse Engineering** | [GDB](02_resource_development/disassemblers_debuggers/gdb/gdb.md), [Ghidra](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md), [Radare2](02_resource_development/disassemblers_debuggers/radare2/radare2.md) |
| **Pwn** | [GDB](02_resource_development/disassemblers_debuggers/gdb/gdb.md), [MSFvenom](02_resource_development/shellcode_payload/msfvenom/msfvenom.md) |
| **OSINT** | [Sherlock](01_reconnaissance/02_identity_information/sherlock/sherlock.md), [TheHarvester](01_reconnaissance/03_network_information/theharvester/theharvester.md) |

---

## Bug Bounty Toolkit

Essential tools for bug bounty programs:

| Phase | Tools |
|-------|-------|
| Recon | [Amass](01_reconnaissance/03_network_information/amass/amass.md), [Subfinder](01_reconnaissance/05_web_scanning/subfinder/subfinder.md), [Assetfinder](01_reconnaissance/05_web_scanning/assetfinder/assetfinder.md) |
| Scanning | [Nuclei](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md), [Nmap](01_reconnaissance/03_network_information/nmap/nmap.md) |
| Web | [Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md), [FFuf](01_reconnaissance/05_web_scanning/ffuf/ffuf.md), [Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md) |

---

## Web Application Security

Complete documentation for web application penetration testing:

| Tool | Purpose |
|------|---------|
| [Burp Suite](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md) | Intercepting proxy, vulnerability scanning |
| [SQLMap](03_initial_access/sqlmap/sqlmap.md) | SQL injection detection and exploitation |
| [Nikto](01_reconnaissance/07_web_vulnerability_scanning/nikto/nikto.md) | Web server vulnerability scanning |
| [Gobuster](01_reconnaissance/05_web_scanning/gobuster/gobuster.md) | Directory and file discovery |
| [FFuf](01_reconnaissance/05_web_scanning/ffuf/ffuf.md) | Web fuzzing |
| [Nuclei](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md) | Template-based vulnerability scanning |
| [Commix](03_initial_access/commix/commix.md) | Command injection |
| [XSSer](04_execution/xsser/xsser.md) | Cross-site scripting detection |

---

## Password Cracking Guide

Offline and online password attack tools:

| Tool | Type | Use Case |
|------|------|----------|
| [Hashcat](08_credential_access/password_cracking/hashcat/hashcat.md) | GPU cracking | Fastest hash cracking with GPU |
| [John the Ripper](08_credential_access/password_cracking/john/john.md) | CPU cracking | Versatile password cracker |
| [Hydra](08_credential_access/brute_force/hydra/hydra.md) | Online brute force | Login page brute forcing |
| [Medusa](08_credential_access/brute_force/medusa/medusa.md) | Online brute force | Parallel network login brute forcer |
| [Responder](08_credential_access/credential_harvesting/responder/responder.md) | Network poisoning | LLMNR/NBT-NS credential harvesting |

---

## Reverse Engineering Tools

Binary analysis and reverse engineering documentation:

| Tool | Purpose |
|------|---------|
| [Ghidra](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md) | NSA reverse engineering suite with decompiler |
| [GDB](02_resource_development/disassemblers_debuggers/gdb/gdb.md) | GNU Debugger for Linux |
| [GEF](02_resource_development/disassemblers_debuggers/gef/gef.md) | GDB Enhanced Features for exploit development |
| [Radare2](02_resource_development/disassemblers_debuggers/radare2/radare2.md) | Reverse engineering framework |
| [Rizin](02_resource_development/disassemblers_debuggers/rizin/rizin.md) | Radare2 fork |
| [Capstone](02_resource_development/disassemblers_debuggers/capstone/capstone.md) | Disassembly framework |

---

## How Documentation Is Structured

Every tool follows an 8-chapter template:

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

## Who Uses This

- **Penetration testers** documenting their methodology
- **CTF players** needing quick tool references
- **Bug bounty hunters** wanting comprehensive guides
- **Students** learning cybersecurity and ethical hacking
- **OSCP/CEH/GPEN candidates** preparing for certifications
- **Blue teamers** understanding attacker tools for defense

---

## Contributing

Want to add documentation for a tool? See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT License. Educational purposes only.

Always obtain proper authorization before testing systems you do not own.

---

<div align="center">

**299 tools documented. 200,000+ lines. Updated weekly.**

</div>
