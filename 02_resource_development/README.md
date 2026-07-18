# Kali Linux Resource Development — Phase 02

## Welcome

This is comprehensive documentation covering **11 tools** across the **Shellcode & Payload** subcategory of Phase 02: Resource Development. Each tool has detailed documentation with installation, usage, automation scripts, and real-world scenarios.

---

## What Is Resource Development?

MITRE ATT&CK defines Resource Development (TA0042) as the phase where adversaries create, purchase, or compromise resources to support operations. In Kali Linux, this translates to generating payloads, developing shellcode, creating backdoors, and preparing the tools needed for exploitation.

This phase bridges reconnaissance (knowing what to attack) and initial access (actually getting in). Without properly developed resources — functional payloads, evasive shellcode, injected binaries — the attack chain stalls.

---

## Subcategories Overview

| Subcategory | Tool Count | Description |
|-------------|------------|-------------|
| **Shellcode & Payload** | 11 | Payload generation, shellcode writing, binary infection |
| **Digital Certificates** | 4 | SSL/TLS certificate creation and management |
| **Code Signing Certificates** | 3 | Windows code signing for binary trust |
| **Malware** | 2 | C2 frameworks and malware development |
| **Stage Capabilities** | 1 | Infrastructure staging tools |

This documentation covers the **Shellcode & Payload** subcategory — the largest and most critical subcategory in Phase 02.

---

## Tool Catalog — Shellcode & Payload

### Core Payload Generators

| Tool | Description | Difficulty | Lines |
|------|-------------|------------|-------|
| [MSFvenom](shellcode_payload/msfvenom/) | Metasploit payload generator — the industry standard | Beginner–Advanced | 800+ |
| [MSFPC](shellcode_payload/msfpc/) | MSFvenom wrapper for simplified payload creation | Beginner | 500+ |

### Shellcode Development

| Tool | Description | Difficulty | Lines |
|------|-------------|------------|-------|
| [ShellNoob](shellcode_payload/shellnoob/) | Shellcode writing toolkit with format conversion | Beginner–Intermediate | 500+ |
| [Sickle](shellcode_payload/sickle-pdk/) | Shellcode development kit with Shell-Storm integration | Intermediate | 500+ |
| [MSF nasm_shell](shellcode_payload/msf-nasm_shell/) | Interactive assembly ↔ opcode translator | Beginner | 500+ |

### Binary Infection & Injection

| Tool | Description | Difficulty | Lines |
|------|-------------|------------|-------|
| [Shellter](shellcode_payload/shellter/) | Dynamic PE infector for Windows 32-bit binaries | Intermediate | 600+ |
| [Backdoor Factory](shellcode_payload/backdoor-factory/) | PE/ELF/Mach-O patcher with code cave injection | Intermediate | 500+ |
| [Cymothoa](shellcode_payload/cymothoa/) | Linux process injection via ptrace | Intermediate | 500+ |

### Advanced Payload Tools

| Tool | Description | Difficulty | Lines |
|------|-------------|------------|-------|
| [Donut](shellcode_payload/donut/) | Position-independent shellcode for .NET assemblies | Intermediate–Advanced | 600+ |
| [Veil](shellcode_payload/veil/) | AV evasion payload generator | Intermediate | 600+ |

### Utility Tools

| Tool | Description | Difficulty | Lines |
|------|-------------|------------|-------|
| [exe2hex](shellcode_payload/exe2hex/) | EXE to hex array converter | Beginner | 500+ |

---

## Learning Roadmap

### Phase 1: Foundations (Week 1–2)

Master the core payload generation tools.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Must** | MSFvenom | Payload types, encoders, output formats |
| **Must** | MSFPC | Quick payload generation workflow |
| **Must** | MSF nasm_shell | Assembly ↔ opcode conversion |

**Milestone:** Generate payloads for Windows, Linux, and web targets. Set up handlers.

### Phase 2: Shellcode Development (Week 3–4)

Learn to write, analyze, and convert shellcode.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Must** | ShellNoob | Format conversion, interactive mode, debugging |
| **Should** | Sickle | Shell-Storm integration, bad character analysis |
| **Should** | exe2hex | Script-based payload delivery |

**Milestone:** Write custom shellcode from scratch. Convert between all formats. Debug with gdb/strace.

### Phase 3: Binary Infection (Week 5–6)

Learn to inject shellcode into legitimate binaries.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Must** | Shellter | PE injection, MSFvenom integration |
| **Should** | Backdoor Factory | Code cave injection, ELF patching |
| **Should** | Cymothoa | Linux process injection |

**Milestone:** Infect PE and ELF binaries. Maintain original program functionality. Understand AV evasion implications.

### Phase 4: Advanced Evasion (Week 7–8)

Master advanced payload techniques for red team operations.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Must** | Veil | AV evasion, multiple language payloads |
| **Should** | Donut | .NET assembly execution, AMSI bypass |
| **Optional** | All tools | Integration, automation, custom workflows |

**Milestone:** Generate AV-evasive payloads. Execute .NET assemblies from memory. Build automated payload pipelines.

---

## Difficulty Progression

```
Beginner                  Intermediate              Advanced
─────────────────────────────────────────────────────────────
MSFvenom (basic)          Shellter                  MSFvenom (encoding)
MSFPC                     Backdoor Factory          Veil (custom modules)
MSF nasm_shell            Cymothoa                  Donut (CLR hosting)
exe2hex                   Sickle                    Backdoor Factory (IAT)
ShellNoob (conversion)    Veil (Ordnance)           Process injection chains
ShellNoob (interactive)   Donut (basic)             Custom encoder dev
```

---

## Prerequisites

### System Requirements

- **Operating System:** Kali Linux (recommended) or any Linux distribution
- **RAM:** 4GB minimum (8GB recommended)
- **Disk Space:** 20GB+ free space for tool installations and payload storage
- **Network:** Internet access for tool updates, Shell-Storm downloads, and payload staging

### Required Knowledge

Before diving into these tools, ensure you understand:

- **Linux command line**: File operations, permissions, pipes, redirection
- **Basic networking**: TCP/UDP, ports, IP addressing, firewall concepts
- **Assembly fundamentals**: Registers, instructions, memory addressing (x86)
- **C programming basics**: Pointers, memory management, compilation

### Recommended Preparation

| Resource | Purpose |
|----------|---------|
| [OverTheWire Bandit](https://overthewire.org/wargames/bandit/) | Linux command line fluency |
| [x86 Assembly Guide](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html) | Assembly language basics |
| [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) | MSF framework fundamentals |
| [Phase 01: Reconnaissance](../01_reconnaissance/) | Prior phase prerequisites |

---

## How to Use This Documentation

### For Beginners

1. Start with MSFvenom (the most documented and essential tool)
2. Follow the learning roadmap sequentially
3. Practice each tool on legal targets (HackTheBox, TryHackMe)
4. Read Chapter 1 of each tool to understand its purpose
5. Complete Chapter 3 (Basic Usage) exercises before moving on

### For Intermediate Users

1. Focus on Binary Infection tools (Shellter, Backdoor Factory, Cymothoa)
2. Study Chapter 4 (Intermediate Usage) for automation scripts
3. Practice tool chaining: MSFvenom → Shellter, MSFvenom → Veil
4. Begin building custom payload pipelines

### For Advanced Users

1. Study Chapter 5 (Advanced Usage) for evasion techniques
2. Build custom encoders and injectors
3. Develop automated red team workflows
4. Contribute detection rules in Chapter 7 (Detection & Defense)

---

## Tool Relationship Map

```
                        ┌─────────────┐
                        │  MSFvenom   │
                        │  (Core Gen) │
                        └──────┬──────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │    Veil      │ │   Shellter   │ │    Donut     │
      │  (AV Bypass) │ │ (PE Inject)  │ │ (.NET Load)  │
      └──────────────┘ └──────────────┘ └──────────────┘
              │                │                │
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │ BDF          │ │ Cymothoa     │ │ exe2hex      │
      │ (Binary Pkg) │ │ (Ptrace)     │ │ (Hex Array)  │
      └──────────────┘ └──────────────┘ └──────────────┘

Supporting Tools:
  MSF nasm_shell ── Assembly ↔ Opcode (development aid)
  ShellNoob ─────── Format conversion & debugging
  Sickle ─────────── Shell-Storm integration & conversion
  MSFPC ──────────── MSFvenom simplification wrapper
```

---

## Legal Disclaimer

**IMPORTANT:** This documentation is for educational and authorized security testing purposes only.

- **Always obtain written permission** before testing any system you don't own
- **Understand your scope** — know exactly what you're authorized to test
- **Follow responsible disclosure** — report vulnerabilities properly
- **Respect privacy laws** — don't collect more data than necessary
- **Document everything** — keep records of your authorization and activities

Payload generation and binary infection tools can be used maliciously. Using them without authorization is illegal in most jurisdictions.

---

## Resources

### Practice Platforms

- [HackTheBox](https://www.hackthebox.com/) — CTF challenges with exploitation focus
- [TryHackMe](https://tryhackme.com/) — Guided learning paths
- [VulnHub](https://www.vulnhub.com/) — Vulnerable VMs for offline practice
- [PentesterLab](https://pentesterlab.com/) — Hands-on exploitation exercises

### Certifications

- **OSCP** — Offensive Security Certified Professional (exploitation focus)
- **CEH** — Certified Ethical Hacker
- **PNPT** — Practical Network Penetration Tester
- **GPEN** — GIAC Penetration Tester

### Books

- "Metasploit: The Penetration Tester's Guide" — David Kennedy et al.
- "Hacking: The Art of Exploitation" — Jon Erickson
- "The Shellcoder's Handbook" — Chris Anley et al.
- "Penetration Testing" — Georgia Weidman

### References

- [MITRE ATT&CK TA0042](https://attack.mitre.org/techniques/TA0042/) — Resource Development
- [Shell-Storm Database](http://shell-storm.org/shellcode/) — Community shellcode
- [Exploit Database](https://www.exploit-db.com/) — Exploit archive

---

*Last updated: July 2026*
