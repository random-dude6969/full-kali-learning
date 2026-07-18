# Kali Linux Initial Access — Phase 03

## Welcome

This is comprehensive documentation covering **10 tools** across the **Exploitation & Phishing** subcategory of Phase 03: Initial Access. Each tool has detailed documentation with installation, usage, automation scripts, and real-world scenarios.

---

## What Is Initial Access?

MITRE ATT&CK defines Initial Access (TA0001) as the phase where adversaries gain a foothold in a target network. In Kali Linux, this translates to exploiting vulnerabilities, executing SQL and command injection attacks, launching phishing campaigns, and abusing misconfigured services to establish initial control.

This phase bridges resource development (having payloads ready) and persistence (maintaining access). Without successfully executing initial access — exploiting a web application, injecting malicious commands, or tricking a user into clicking a payload — the attack chain never begins.

---

## Subcategories Overview

| Subcategory | Tool Count | Description |
|-------------|------------|-------------|
| **SQL Injection** | 4 | Automated SQL injection detection and exploitation |
| **Command Injection** | 1 | Automated command injection exploitation |
| **Exploitation Framework** | 1 | Multi-protocol exploitation framework |
| **Social Engineering & Phishing** | 2 | Phishing simulation and social engineering |
| **Server Exploitation** | 1 | JBoss application server exploitation |
| **Network Attack** | 1 | DNS rebinding attack tool |

This documentation covers the **Exploitation & Phishing** subcategory — the most direct and commonly used tools for gaining initial access.

---

## Tool Catalog — Initial Access

### SQL Injection

| Tool | Description | Difficulty | Status |
|------|-------------|------------|--------|
| [sqlmap](sqlmap/) | Automated SQL injection and database takeover | Beginner–Advanced | Complete |
| [jsql](jsql/) | Visual SQL injection tool with GUI | Beginner | Developing |
| [sqlninja](sqlninja/) | SQL injection for MSSQL/MSAccess environments | Intermediate | Developing |
| [sqlsus](sqlsus/) | MySQL injection and takeover tool | Intermediate | Developing |

### Command Injection

| Tool | Description | Difficulty | Status |
|------|-------------|------------|--------|
| [commix](commix/) | Automated command injection exploitation | Beginner–Intermediate | Developing |

### Exploitation Framework

| Tool | Description | Difficulty | Status |
|------|-------------|------------|--------|
| [metasploit-framework](metasploit-framework/) | World's most used penetration testing framework | All Levels | Complete |

### Social Engineering & Phishing

| Tool | Description | Difficulty | Status |
|------|-------------|------------|--------|
| [setoolkit](setoolkit/) | Social Engineering Toolkit for phishing attacks | Beginner–Intermediate | Developing |
| [gophish](gophish/) | Phishing simulation and awareness training | Beginner | Developing |

### Server Exploitation

| Tool | Description | Difficulty | Status |
|------|-------------|------------|--------|
| [jboss-autopwn](jboss-autopwn/) | Automated JBoss server exploitation | Intermediate | Developing |

### Network Attack

| Tool | Description | Difficulty | Status |
|------|-------------|------------|--------|
| [dns-rebind](dns-rebind/) | DNS rebinding attack tool | Advanced | Developing |

---

## Learning Roadmap

### Phase 1: Foundations (Week 1–2)

Master the core SQL injection and exploitation tools.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Must** | sqlmap | Target enumeration, injection techniques, database enumeration, data exfiltration |
| **Must** | metasploit-framework | Module selection, payload configuration, session management |
| **Should** | commix | Command injection identification and command execution |

**Milestone:** Exploit a vulnerable web app via SQL injection. Establish a Meterpreter session through Metasploit. Execute commands via command injection.

### Phase 2: Phishing & Social Engineering (Week 3–4)

Learn the social engineering attack vector.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Must** | setoolkit | Credential harvesting, website cloner, mass mailer |
| **Should** | gophish | Campaign creation, email templates, landing pages, reporting |
| **Should** | sqlninja | MSSQL-specific injection and OS shell retrieval |

**Milestone:** Build a phishing campaign with gophish. Clone a login page with setoolkit. Harvest credentials. Analyze campaign results.

### Phase 3: Server & Web Exploitation (Week 5–6)

Exploit misconfigured services and advanced injection.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Should** | jboss-autopwn | JBoss deployment scanner, WAR deployment exploitation |
| **Should** | jsql | GUI-based SQL injection with automated payload generation |
| **Should** | sqlsus | MySQL-specific injection and file system access |

**Milestone:** Exploit JBoss servers. Use jsql for visual injection workflows. Chain SQL injection with Metasploit for privilege escalation.

### Phase 4: Advanced Attacks (Week 7–8)

Master advanced initial access techniques and integration.

| Priority | Tool | Focus Area |
|----------|------|------------|
| **Should** | dns-rebind | DNS rebinding bypass of same-origin policy |
| **Optional** | All tools | Integration into full attack chains |
| **Optional** | Custom scripts | Automation and evasion |

**Milestone:** Execute DNS rebinding attacks against internal services. Build automated initial access pipelines combining multiple tools.

---

## Difficulty Progression

```
Beginner                    Intermediate                Advanced
────────────────────────────────────────────────────────────────
sqlmap (basic inject)       sqlmap (tamper scripts)     sqlmap (custom UDF)
metasploit (exploit/unix)   metasploit (post modules)   metasploit (custom modules)
commix (basic cmd)          sqlninja (MSSQL shells)     dns-rebind (policy bypass)
gophish (template)          jsql (automated payloads)   sqlmap (time-based blind)
setoolkit (cloner)          jboss-autopwn (WAR deploy)  Full attack chain automation
jsql (GUI scan)             sqlsus (file read/write)    Phishing → SQLi → Meterpreter
```

---

## Prerequisites

### System Requirements

- **Operating System:** Kali Linux (recommended) or any Linux distribution
- **RAM:** 4GB minimum (8GB recommended)
- **Disk Space:** 20GB+ free space for tool installations and output storage
- **Network:** Internet access for tool updates, payload staging, and phishing infrastructure
- **Target Environment:** Legal vulnerable applications (DVWA, Juice Shop, HackTheBox, TryHackMe)

### Required Knowledge

Before diving into these tools, ensure you understand:

- **Linux command line**: File operations, permissions, pipes, redirection
- **Basic networking**: TCP/UDP, ports, IP addressing, HTTP/HTTPS fundamentals
- **SQL fundamentals**: SELECT, INSERT, WHERE clauses, UNION operations
- **Web application concepts**: Request/response cycle, sessions, cookies, POST data
- **Metasploit basics**: Modules, payloads, sessions, listeners

### Recommended Preparation

| Resource | Purpose |
|----------|---------|
| [DVWA](https://dvwa.co.uk/) | Practice target for SQL and command injection |
| [PortSwigger Web Academy](https://portswigger.net/web-security) | Web vulnerability fundamentals |
| [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) | MSF framework fundamentals |
| [Phase 01: Reconnaissance](../01_reconnaissance/) | Target identification prerequisites |
| [Phase 02: Resource Development](../02_resource_development/) | Payload generation prerequisites |

---

## Tool Relationship Map

```
                        ┌──────────────────┐
                        │  sqlmap          │
                        │  (SQL Injection) │
                        └────────┬─────────┘
                                 │
               ┌─────────────────┼─────────────────┐
               │                 │                 │
               ▼                 ▼                 ▼
       ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
       │  sqlninja    │ │    jsql      │ │    sqlsus    │
       │  (MSSQL)     │ │  (Visual)    │ │  (MySQL)     │
       └──────────────┘ └──────────────┘ └──────────────┘

       ┌──────────────────┐       ┌──────────────────┐
       │ metasploit       │       │ setoolkit        │
       │ (Exploitation)   │       │ (Social Eng)     │
       └────────┬─────────┘       └────────┬─────────┘
                │                          │
                ▼                          ▼
       ┌──────────────┐           ┌──────────────┐
       │ commix       │           │ gophish      │
       │ (Cmd Inject) │           │ (Phishing)   │
       └──────────────┘           └──────────────┘

Specialized:
  jboss-autopwn ── JBoss server exploitation
  dns-rebind ───── DNS rebinding attack
```

---

## MITRE ATT&CK Mapping

| Tool | ATT&CK Technique | Technique ID |
|------|------------------|--------------|
| sqlmap | Exploitation for Client Execution | T1203 |
| sqlmap | Data from Information Repositories | T1213 |
| sqlninja | Exploitation for Client Execution | T1203 |
| jsql | Exploitation for Client Execution | T1203 |
| sqlsus | Exploitation for Client Execution | T1203 |
| metasploit-framework | Exploitation for Client Execution | T1203 |
| metasploit-framework | Remote Services | T1021 |
| commix | Command and Scripting Interpreter | T1059 |
| setoolkit | Phishing | T1566 |
| setoolkit | User Execution: Malicious Link | T1204.001 |
| setoolkit | User Execution: Malicious File | T1204.002 |
| gophish | Phishing | T1566 |
| gophish | Spearphishing Link | T1566.002 |
| gophish | Spearphishing Attachment | T1566.001 |
| jboss-autopwn | Exploitation for Client Execution | T1203 |
| jboss-autopwn | Server Software Component | T1505 |
| dns-rebind | Drive-by Compromise | T1189 |
| dns-rebind | Exploitation for Client Execution | T1203 |

---

## Legal Disclaimer

**IMPORTANT:** This documentation is for educational and authorized security testing purposes only.

- **Always obtain written permission** before testing any system you don't own
- **Understand your scope** — know exactly what you're authorized to test
- **Follow responsible disclosure** — report vulnerabilities properly
- **Respect privacy laws** — don't collect more data than necessary
- **Document everything** — keep records of your authorization and activities

SQL injection, command injection, exploitation frameworks, and phishing tools can cause significant damage when misused. Unauthorized access to computer systems is illegal in most jurisdictions.

---

## Resources

### Practice Platforms

- [HackTheBox](https://www.hackthebox.com/) — CTF challenges with exploitation focus
- [TryHackMe](https://tryhackme.com/) — Guided learning paths
- [DVWA](https://dvwa.co.uk/) — Damn Vulnerable Web Application
- [Juice Shop](https://owasp.org/www-project-juice-shop/) — OWASP vulnerable web app
- [VulnHub](https://www.vulnhub.com/) — Vulnerable VMs for offline practice
- [PentesterLab](https://pentesterlab.com/) — Hands-on exploitation exercises

### Certifications

- **OSCP** — Offensive Security Certified Professional (exploitation focus)
- **CEH** — Certified Ethical Hacker
- **PNPT** — Practical Network Penetration Tester
- **GPEN** — GIAC Penetration Tester
- **GWAPT** — GIAC Web Application Penetration Tester

### Books & References

- "Metasploit: The Penetration Tester's Guide" — David Kennedy et al.
- "SQL Injection Attacks and Defense" — Justin Clarke
- "The Web Application Hacker's Handbook" — Dafydd Stuttard
- [MITRE ATT&CK TA0001](https://attack.mitre.org/tactics/TA0001/) — Initial Access
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) — SQLi reference

---

*Last updated: July 2026*
