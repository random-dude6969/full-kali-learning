# Phase 08 — Credential Access

52 tools for discovering and extracting credentials from target systems.

## Subcategories

| Category | Tools | Description |
|----------|-------|-------------|
| Brute Force | 8 | Online password attacks (Hydra, Medusa, Patator) |
| Credential Dumping | 4 | Extract hashes from memory/registry (Mimikatz, creddump7) |
| Credential Harvesting | 1 | Network sniffing for creds (Responder) |
| Hash Identification | 2 | Identify hash types (hashid, hash-identifier) |
| Kerberos | 2 | Kerberos attack tools (Kerberoast, krbrelayx) |
| NFC Tools | 5 | Near-field communication attacks |
| Password Cracking | 10 | Offline hash cracking (Hashcat, John) |
| Password Wordlists | 7 | Wordlist generation (Crunch, CeWL, CUPP) |
| WiFi Credential | 11 | Wireless password attacks (Aircrack-ng, Reaver) |

## Learning Path

1. Start with **hashid** to identify unknown hashes
2. Learn **Hydra** for online brute force attacks
3. Master **Hashcat** and **John** for offline cracking
4. Study **Responder** for network credential harvesting
5. Explore **Mimikatz** for Windows credential dumping
6. Practice WiFi attacks with **Aircrack-ng**

## Key Tools

| Tool | Lines | Use Case |
|------|-------|----------|
| Hashcat | 800+ | GPU-accelerated hash cracking |
| John the Ripper | 700+ | Versatile password cracker |
| Hydra | 600+ | Fast online brute force |
| Responder | 600+ | LLMNR/NBT-NS poisoning |
| Mimikatz | 700+ | Windows credential extraction |
| Aircrack-ng | 700+ | WiFi handshake cracking |

## MITRE ATT&CK Mapping

| Technique | ID | Tools |
|-----------|-----|-------|
| Brute Force | T1110 | Hydra, Medusa, Ncrack |
| Credential Dumping | T1003 | Mimikatz, creddump7 |
| LLMNR Poisoning | T1557 | Responder |
| Kerberoasting | T1558 | Kerberoast |
| WiFi Password Guessing | T1110.001 | Aircrack-ng, Reaver |
