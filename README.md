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
| 08 | [Credential Access](08_credential_access/) | 52 | **Complete** | Hydra, John, Hashcat, Responder |
| 09 | [Discovery](09_discovery/) | 88 | **Complete** | Wireshark, BloodHound, Ettercap |
| 10 | Lateral Movement | 3 | Coming | PsExec, WMI |
| 11 | Collection | 12 | Coming | Keyloggers, Screen capture |
| 12 | Command and Control | 30 | Coming | C2 frameworks |
| 13 | Exfiltration | 3 | Coming | Data exfil tools |
| 14 | Impact | 10 | Coming | Destructive tools |
| 15 | Forensics | 51 | Coming | Volatility, Autopsy |
| 16 | Services | 15 | Coming | Network services |

**159 tools done. 273 remaining. Contributions welcome.**

---

---


- **Real examples** — every command shows what it does, not just the flag
- **Zero fluff** — straight to the point, zero filler
- **CTF ready** — scenarios you'll actually use in competitions
- **OSCP aligned** — covers every tool in the certification
- **Blue team included** — detection sections for every tool
- **Updated weekly** — new tools added constantly

---

## Start Your Journey

### Beginner Path (Week 1-2)
```
Nmap -> Gobuster -> SQLMap -> Metasploit -> Hydra
```

### Intermediate Path (Week 3-6)
```
Burp Suite -> Nuclei -> Ghidra -> LinPEAS -> Impacket
```

### Advanced Path (Month 2+)
```
Custom exploitation -> Red team ops -> Binary analysis -> C2 frameworks
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
[x] Phase 08 - Credential Access (52 tools) DONE NEXT
[x] Phase 09 - Discovery (88 tools) DONE
[ ] Phase 10 - Lateral Movement (3 tools)
[ ] Phase 11 - Collection (12 tools)
[ ] Phase 12 - Command and Control (30 tools)
[ ] Phase 13 - Exfiltration (3 tools)
[ ] Phase 14 - Impact (10 tools)
[ ] Phase 15 - Forensics (51 tools)
[ ] Phase 16 - Services and Other (15 tools)
```

---

## All 159 Tools — Click to Open

Every documented tool with direct links. Sorted by documentation depth.


### Phase 01

| Tool | Lines | Link |
|------|-------|------|
| Autorecon | 3632 | [Docs](01_reconnaissance/03_network_information/autorecon/autorecon.md) |
| Amass | 2121 | [Docs](01_reconnaissance/03_network_information/amass/amass.md) |
| Metagoofil | 2076 | [Docs](01_reconnaissance/01_host_information/metagoofil/metagoofil.md) |
| Emailharvester | 1893 | [Docs](01_reconnaissance/02_identity_information/emailharvester/emailharvester.md) |
| Instaloader | 1795 | [Docs](01_reconnaissance/02_identity_information/instaloader/instaloader.md) |
| Dnsenum | 1282 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnsenum/dnsenum.md) |
| Gnuradio | 1258 | [Docs](01_reconnaissance/10_radio_frequency/gnuradio/gnuradio.md) |
| Nuclei | 1249 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/nuclei/nuclei.md) |
| Dnsrecon | 1241 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnsrecon/dnsrecon.md) |
| Dnswalk | 1185 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnswalk/dnswalk.md) |
| Gobuster | 1182 | [Docs](01_reconnaissance/05_web_scanning/gobuster/gobuster.md) |
| Nikto | 1169 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/nikto/nikto.md) |
| Davtest | 1148 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/davtest/davtest.md) |
| Ffuf | 1143 | [Docs](01_reconnaissance/05_web_scanning/ffuf/ffuf.md) |
| Massdns | 1136 | [Docs](01_reconnaissance/04_dns_reconnaissance/massdns/massdns.md) |
| Feroxbuster | 1128 | [Docs](01_reconnaissance/05_web_scanning/feroxbuster/feroxbuster.md) |
| Subfinder | 1122 | [Docs](01_reconnaissance/05_web_scanning/subfinder/subfinder.md) |
| Joomscan | 1114 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/joomscan/joomscan.md) |
| Caido | 1079 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/caido/caido.md) |
| Burpsuite | 1079 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/burpsuite/burpsuite.md) |
| Gospider | 1065 | [Docs](01_reconnaissance/05_web_scanning/gospider/gospider.md) |
| Dnsmap | 1059 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnsmap/dnsmap.md) |
| Nmap | 1048 | [Docs](01_reconnaissance/03_network_information/nmap/nmap.md) |
| Crlfuzz | 1045 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/crlfuzz/crlfuzz.md) |
| Finalrecon | 1033 | [Docs](01_reconnaissance/05_web_scanning/finalrecon/finalrecon.md) |
| Dnstracer | 1028 | [Docs](01_reconnaissance/04_dns_reconnaissance/dnstracer/dnstracer.md) |
| Rfcat | 1019 | [Docs](01_reconnaissance/10_radio_frequency/rfcat/rfcat.md) |
| Lbd | 1014 | [Docs](01_reconnaissance/05_web_scanning/lbd/lbd.md) |
| Wpscan | 1011 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/wpscan/wpscan.md) |
| Findomain | 1008 | [Docs](01_reconnaissance/05_web_scanning/findomain/findomain.md) |
| Kismet | 999 | [Docs](01_reconnaissance/09_wifi/kismet/kismet.md) |
| Recon Ng | 988 | [Docs](01_reconnaissance/05_web_scanning/recon-ng/recon-ng.md) |
| Arjun | 944 | [Docs](01_reconnaissance/05_web_scanning/arjun/arjun.md) |
| Wfuzz | 910 | [Docs](01_reconnaissance/05_web_scanning/wfuzz/wfuzz.md) |
| Sublist3R | 902 | [Docs](01_reconnaissance/05_web_scanning/sublist3r/sublist3r.md) |
| Theharvester | 886 | [Docs](01_reconnaissance/03_network_information/theharvester/theharvester.md) |
| Skipfish | 879 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/skipfish/skipfish.md) |
| Ubertooth Util | 847 | [Docs](01_reconnaissance/08_bluetooth/ubertooth-util/ubertooth-util.md) |
| Wash | 845 | [Docs](01_reconnaissance/09_wifi/wash/wash.md) |
| Dirsearch | 842 | [Docs](01_reconnaissance/05_web_scanning/dirsearch/dirsearch.md) |
| Wapiti | 840 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/wapiti/wapiti.md) |
| Chirp | 839 | [Docs](01_reconnaissance/10_radio_frequency/chirp/chirp.md) |
| Sstimap | 823 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/sstimap/sstimap.md) |
| Tinja | 822 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/tinja/tinja.md) |
| Heartleech | 819 | [Docs](01_reconnaissance/06_vulnerability_scanning/heartleech/heartleech.md) |
| Subjack | 813 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/subjack/subjack.md) |
| Unicornscan | 802 | [Docs](01_reconnaissance/03_network_information/unicornscan/unicornscan.md) |
| Bettercap | 801 | [Docs](01_reconnaissance/08_bluetooth/bettercap/bettercap.md) |
| Sparrow Wifi | 796 | [Docs](01_reconnaissance/09_wifi/sparrow-wifi/sparrow-wifi.md) |
| Dirb | 783 | [Docs](01_reconnaissance/05_web_scanning/dirb/dirb.md) |
| Whatweb | 772 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/whatweb/whatweb.md) |
| Gqrx | 766 | [Docs](01_reconnaissance/10_radio_frequency/gqrx/gqrx.md) |
| Assetfinder | 758 | [Docs](01_reconnaissance/05_web_scanning/assetfinder/assetfinder.md) |
| Wpprobe | 751 | [Docs](01_reconnaissance/05_web_scanning/wpprobe/wpprobe.md) |
| Parsero | 746 | [Docs](01_reconnaissance/05_web_scanning/parsero/parsero.md) |
| Bluesnarfer | 732 | [Docs](01_reconnaissance/08_bluetooth/bluesnarfer/bluesnarfer.md) |
| Btscanner | 716 | [Docs](01_reconnaissance/08_bluetooth/btscanner/btscanner.md) |
| Hackrf | 716 | [Docs](01_reconnaissance/10_radio_frequency/hackrf/hackrf.md) |
| Paros | 706 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/paros/paros.md) |
| Wcvs | 689 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/wcvs/wcvs.md) |
| Spooftooph | 667 | [Docs](01_reconnaissance/08_bluetooth/spooftooph/spooftooph.md) |
| Bluelog | 665 | [Docs](01_reconnaissance/08_bluetooth/bluelog/bluelog.md) |
| Urlcrazy | 642 | [Docs](01_reconnaissance/05_web_scanning/urlcrazy/urlcrazy.md) |
| Webscarab | 641 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/webscarab/webscarab.md) |
| Spiderfoot | 629 | [Docs](01_reconnaissance/01_host_information/spiderfoot/spiderfoot.md) |
| Watobo | 628 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/watobo/watobo.md) |
| Redfang | 617 | [Docs](01_reconnaissance/08_bluetooth/redfang/redfang.md) |
| Asleap | 617 | [Docs](01_reconnaissance/09_wifi/asleap/asleap.md) |
| Dirbuster | 616 | [Docs](01_reconnaissance/05_web_scanning/dirbuster/dirbuster.md) |
| Sherlock | 613 | [Docs](01_reconnaissance/02_identity_information/sherlock/sherlock.md) |
| Blueranger | 603 | [Docs](01_reconnaissance/08_bluetooth/blueranger/blueranger.md) |
| Zenmap | 598 | [Docs](01_reconnaissance/03_network_information/zenmap/zenmap.md) |
| Tookie Osint | 546 | [Docs](01_reconnaissance/02_identity_information/tookie-osint/tookie-osint.md) |
| Dmitry | 535 | [Docs](01_reconnaissance/03_network_information/dmitry/dmitry.md) |
| Uro | 526 | [Docs](01_reconnaissance/05_web_scanning/uro/uro.md) |
| Legion | 523 | [Docs](01_reconnaissance/03_network_information/legion/legion.md) |
| Gvm | 519 | [Docs](01_reconnaissance/06_vulnerability_scanning/gvm/gvm.md) |
| Zaproxy | 517 | [Docs](01_reconnaissance/07_web_vulnerability_scanning/zaproxy/zaproxy.md) |
| Photon | 510 | [Docs](01_reconnaissance/02_identity_information/photon/photon.md) |
| Linkedin2Username | 501 | [Docs](01_reconnaissance/02_identity_information/linkedin2username/linkedin2username.md) |

### Phase 02

| Tool | Lines | Link |
|------|-------|------|
| Msfvenom | 907 | [Docs](02_resource_development/shellcode_payload/msfvenom/msfvenom.md) |
| Gdb | 816 | [Docs](02_resource_development/disassemblers_debuggers/gdb/gdb.md) |
| Ghidra | 675 | [Docs](02_resource_development/disassemblers_debuggers/ghidra/ghidra.md) |
| Veil | 603 | [Docs](02_resource_development/shellcode_payload/veil/veil.md) |
| Afl Fuzz | 602 | [Docs](02_resource_development/fuzzing/afl-fuzz/afl-fuzz.md) |
| Shellter | 598 | [Docs](02_resource_development/shellcode_payload/shellter/shellter.md) |
| Rizin | 589 | [Docs](02_resource_development/disassemblers_debuggers/rizin/rizin.md) |
| Donut | 580 | [Docs](02_resource_development/shellcode_payload/donut/donut.md) |
| Gef | 578 | [Docs](02_resource_development/disassemblers_debuggers/gef/gef.md) |
| Capstone | 547 | [Docs](02_resource_development/disassemblers_debuggers/capstone/capstone.md) |
| Radare2 | 534 | [Docs](02_resource_development/disassemblers_debuggers/radare2/radare2.md) |
| Jadx | 520 | [Docs](02_resource_development/java_android_reversing/jadx/jadx.md) |
| Backdoor Factory | 514 | [Docs](02_resource_development/shellcode_payload/backdoor-factory/backdoor-factory.md) |
| Shellnoob | 506 | [Docs](02_resource_development/shellcode_payload/shellnoob/shellnoob.md) |
| Apktool | 491 | [Docs](02_resource_development/java_android_reversing/apktool/apktool.md) |
| Cymothoa | 485 | [Docs](02_resource_development/shellcode_payload/cymothoa/cymothoa.md) |
| Cutter | 444 | [Docs](02_resource_development/disassemblers_debuggers/cutter/cutter.md) |
| Searchsploit | 442 | [Docs](02_resource_development/exploit_resources/searchsploit/searchsploit.md) |
| Msf Nasm_Shell | 436 | [Docs](02_resource_development/shellcode_payload/msf-nasm_shell/msf-nasm_shell.md) |
| Bytecode Viewer | 421 | [Docs](02_resource_development/java_android_reversing/bytecode-viewer/bytecode-viewer.md) |
| Edb Debugger | 415 | [Docs](02_resource_development/disassemblers_debuggers/edb-debugger/edb-debugger.md) |
| Spike | 414 | [Docs](02_resource_development/fuzzing/spike/spike.md) |
| Exploitdb | 413 | [Docs](02_resource_development/exploit_resources/exploitdb/exploitdb.md) |
| Exe2Hex | 401 | [Docs](02_resource_development/shellcode_payload/exe2hex/exe2hex.md) |
| Sickle Pdk | 396 | [Docs](02_resource_development/shellcode_payload/sickle-pdk/sickle-pdk.md) |
| Dex2Jar | 393 | [Docs](02_resource_development/java_android_reversing/dex2jar/dex2jar.md) |
| Msfpc | 389 | [Docs](02_resource_development/shellcode_payload/msfpc/msfpc.md) |
| Recstudio | 376 | [Docs](02_resource_development/disassemblers_debuggers/recstudio/recstudio.md) |
| Sfuzz | 373 | [Docs](02_resource_development/fuzzing/sfuzz/sfuzz.md) |
| Jd Gui | 352 | [Docs](02_resource_development/java_android_reversing/jd-gui/jd-gui.md) |
| Bed | 351 | [Docs](02_resource_development/fuzzing/bed/bed.md) |
| Ollydbg | 349 | [Docs](02_resource_development/disassemblers_debuggers/ollydbg/ollydbg.md) |
| Pompem | 330 | [Docs](02_resource_development/exploit_resources/pompem/pompem.md) |
| Javasnoop | 301 | [Docs](02_resource_development/java_android_reversing/javasnoop/javasnoop.md) |

### Phase 03

| Tool | Lines | Link |
|------|-------|------|
| Sqlsus | 1256 | [Docs](03_initial_access/sqlsus/sqlsus.md) |
| Gophish | 959 | [Docs](03_initial_access/gophish/gophish.md) |
| Sqlninja | 952 | [Docs](03_initial_access/sqlninja/sqlninja.md) |
| Sqlmap | 891 | [Docs](03_initial_access/sqlmap/sqlmap.md) |
| Commix | 759 | [Docs](03_initial_access/commix/commix.md) |
| Dns Rebind | 690 | [Docs](03_initial_access/dns-rebind/dns-rebind.md) |
| Jsql | 642 | [Docs](03_initial_access/jsql/jsql.md) |
| Metasploit Framework | 632 | [Docs](03_initial_access/metasploit-framework/metasploit-framework.md) |
| Jboss Autopwn | 612 | [Docs](03_initial_access/jboss-autopwn/jboss-autopwn.md) |
| Setoolkit | 511 | [Docs](03_initial_access/setoolkit/setoolkit.md) |

### Phase 04

| Tool | Lines | Link |
|------|-------|------|
| Xsser | 828 | [Docs](04_execution/xsser/xsser.md) |
| Nishang | 738 | [Docs](04_execution/nishang/nishang.md) |
| Powersploit | 738 | [Docs](04_execution/powersploit/powersploit.md) |
| Beef Xss | 723 | [Docs](04_execution/beef-xss/beef-xss.md) |
| Evilgrade | 708 | [Docs](04_execution/evilgrade/evilgrade.md) |
| Armitage | 611 | [Docs](04_execution/armitage/armitage.md) |

### Phase 05

| Tool | Lines | Link |
|------|-------|------|
| Laudanum | 689 | [Docs](05_persistence/laudanum/laudanum.md) |
| Weevely | 630 | [Docs](05_persistence/weevely/weevely.md) |
| Cymothoa | 614 | [Docs](05_persistence/cymothoa/cymothoa.md) |
| Webacoo | 588 | [Docs](05_persistence/webacoo/webacoo.md) |
| Backdoor Factory | 550 | [Docs](05_persistence/backdoor-factory/backdoor-factory.md) |
| Webshells | 517 | [Docs](05_persistence/webshells/webshells.md) |
| Phpggc | 510 | [Docs](05_persistence/phpggc/phpggc.md) |

### Phase 06

| Tool | Lines | Link |
|------|-------|------|
| Bloodyad | 746 | [Docs](06_privilege_escalation/bloodyad/bloodyad.md) |
| Lynis | 644 | [Docs](06_privilege_escalation/lynis/lynis.md) |
| Unix Privesc Check | 616 | [Docs](06_privilege_escalation/unix-privesc-check/unix-privesc-check.md) |
| Linpeas | 574 | [Docs](06_privilege_escalation/linpeas/linpeas.md) |
| Peass | 508 | [Docs](06_privilege_escalation/peass/peass.md) |
| Winpeas | 505 | [Docs](06_privilege_escalation/winpeas/winpeas.md) |

### Phase 07

| Tool | Lines | Link |
|------|-------|------|
| Impacket Scripts | 837 | [Docs](07_defense_evasion/pass_the_hash/impacket-scripts/impacket-scripts.md) |
| Veil | 790 | [Docs](07_defense_evasion/av_evasion/veil/veil.md) |
| Shellter | 743 | [Docs](07_defense_evasion/av_evasion/shellter/shellter.md) |
| Evil Winrm | 729 | [Docs](07_defense_evasion/pass_the_hash/evil-winrm/evil-winrm.md) |
| Sniffjoke | 697 | [Docs](07_defense_evasion/av_evasion/sniffjoke/sniffjoke.md) |
| Stegsnow | 655 | [Docs](07_defense_evasion/steganography/stegsnow/stegsnow.md) |
| Freerdp | 649 | [Docs](07_defense_evasion/pass_the_hash/freerdp/freerdp.md) |
| Outguess | 612 | [Docs](07_defense_evasion/steganography/outguess/outguess.md) |
| Smbmap | 611 | [Docs](07_defense_evasion/pass_the_hash/smbmap/smbmap.md) |
| Stegosuite | 600 | [Docs](07_defense_evasion/steganography/stegosuite/stegosuite.md) |
| Rubeus | 583 | [Docs](07_defense_evasion/pass_the_hash/rubeus/rubeus.md) |
| Mimikatz | 564 | [Docs](07_defense_evasion/pass_the_hash/mimikatz/mimikatz.md) |
| Passing The Hash | 542 | [Docs](07_defense_evasion/pass_the_hash/passing-the-hash/passing-the-hash.md) |
| Crackmapexec | 536 | [Docs](07_defense_evasion/pass_the_hash/crackmapexec/crackmapexec.md) |
| Steghide | 536 | [Docs](07_defense_evasion/steganography/steghide/steghide.md) |
| Netexec | 532 | [Docs](07_defense_evasion/pass_the_hash/netexec/netexec.md) |

**Total: 159 tools**


---

## Contributing

Want to help document a tool? See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Help This Repo Grow

If this helps you learn, here's how you can help others find it:

**Star it** — tells GitHub this is useful, shows it to more people
**Share it** — post on Reddit, Twitter, Discord, or your study group
**Contribute** — add a tool you know well

### Share on Social Media

Copy and paste:

```
Just found the most comprehensive Kali Linux tools repo on GitHub:
159 tools documented with tutorials, commands, and real examples.
From Nmap to Mimikatz — everything for penetration testing.
Check it out: https://github.com/random-dude6969/full-kali-learning
```
---
## License

MIT License. Educational purposes only.

---

<div align="center">

### If this repo helps you, give it a star

It takes one click and motivates more content.

![Star History Chart](https://api.star-history.com/svg?repos=random-dude6969/full-kali-learning&type=Date)

</div>
