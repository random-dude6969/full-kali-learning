# Obsidian - Knowledge Base and Note-Taking

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

Obsidian is a powerful knowledge base and note-taking application that works on local Markdown files. For cybersecurity professionals, Obsidian provides an excellent platform for organizing security research, documenting penetration test methodologies, creating personal knowledge bases, and managing complex security projects with its unique bidirectional linking and graph visualization capabilities.

### 1.1 What is Obsidian?

Obsidian is a private and flexible writing app that:
- Stores notes locally as Markdown files
- Supports bidirectional linking between notes
- Provides graph visualization of connections
- Offers hundreds of plugins and themes
- Works offline without internet access
- Uses open, non-proprietary file formats

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Developer | Obsidian.md |
| License | Proprietary (free for personal use) |
| Version | 1.7.7 |
| Language | TypeScript/Electron |
| Storage | Local Markdown files |
| Platform | Linux, Windows, macOS |
| Website | obsidian.md |

### 1.3 Obsidian vs Other Note-Taking Tools

| Feature | Obsidian | Notion | Evernote | CherryTree |
|---------|----------|--------|----------|------------|
| Local Storage | Yes | No | No | Yes |
| Markdown | Native | No | No | Partial |
| Bidirectional Links | Yes | No | No | No |
| Graph View | Yes | No | No | No |
| Plugins | 1000+ | Limited | Limited | No |
| Offline | Yes | No | Yes | Yes |
| Pricing | Free* | Freemium | Freemium | Free |

*Free for personal use, commercial license required for business use

### 1.4 Why Use Obsidian for Cybersecurity?

- **Knowledge Management:** Build a personal security knowledge base
- **Methodology Documentation:** Create step-by-step testing guides
- **Research Organization:** Link related security concepts
- **Project Management:** Track penetration test progress
- **Learning:** Document techniques and findings
- **Collaboration:** Share vaults with team members

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install obsidian

# Launch
obsidian
```

### 2.2 AppImage (All Linux)

```bash
# Download AppImage from website
wget https://github.com/obsidianmd/obsidian-releases/releases/download/v1.7.7/Obsidian-1.7.7.AppImage

# Make executable
chmod +x Obsidian-1.7.7.AppImage

# Run
./Obsidian-1.7.7.AppImage
```

### 2.3 Snap Package

```bash
# Install via snap
sudo snap install obsidian
```

### 2.4 First-Time Setup

1. Launch Obsidian
2. Create a new vault (folder for notes)
3. Choose location (e.g., `~/Documents/SecurityVault`)
4. Start creating notes

---

## 3. Interface Overview

### 3.1 Main Window Components

```
+------------------------------------------------------------------+
|  Vault Name  |  Search  |  Graph View  |  Settings               |
+------------------------------------------------------------------+
|            |                                                    |
|  Sidebar   |              Editor Area                           |
|            |                                                    |
|  - Files   |  Markdown editor with live preview                |
|  - Search  |                                                    |
|  - Graph   |  Write notes with Markdown syntax                 |
|  - Tags    |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Status Bar  |  Word count  |  Sync status  |  Mode             |
+------------------------------------------------------------------+
```

### 3.2 Key Components

- **File Explorer:** Browse and manage notes
- **Editor:** Write and edit Markdown content
- **Graph View:** Visualize note connections
- **Search:** Find content across all notes
- **Backlinks:** See what links to current note
- **Tags Pane:** Browse notes by tags

---

## 4. Creating a Security Knowledge Base

### 4.1 Recommended Structure

```
SecurityVault/
├── 00-Inbox/
│   └── Quick notes and ideas
├── 01-Methodologies/
│   ├── Penetration Testing
│   ├── Web Application Testing
│   ├── Network Assessment
│   └── Social Engineering
├── 02-Techniques/
│   ├── SQL Injection
│   ├── XSS
│   ├── Privilege Escalation
│   └── Lateral Movement
├── 03-Tools/
│   ├── Nmap
│   ├── Burp Suite
│   ├── Metasploit
│   └── SQLMap
├── 04-Findings/
│   ├── Project A
│   ├── Project B
│   └── Project C
├── 05-References/
│   ├── OWASP
│   ├── CVE Database
│   └── MITRE ATT&CK
├── 06-Templates/
│   ├── Finding Template
│   ├── Project Template
│   └── Methodology Template
└── 07-Archive/
    └── Completed projects
```

### 4.2 Creating Notes

```markdown
# SQL Injection

## Overview
SQL Injection is a code injection technique that exploits security vulnerabilities in an application's database layer.

## Types
- [[Classic SQL Injection]]
- [[Blind SQL Injection]]
- [[Union-Based SQL Injection]]
- [[Time-Based SQL Injection]]

## Tools
- [[SQLMap]]
- [[Burp Suite]]
- [[Manual Testing]]

## References
- [[OWASP SQL Injection]]
- [[CWE-89]]

## My Notes
- Found this technique useful in [[Project A]]
- Related to [[Command Injection]] in some cases
```

### 4.3 Bidirectional Links

```markdown
# Create a link to another note
See [[SQL Injection]] for more details.

# Link with alias
See [[SQL Injection|this technique]] for more details.

# Embed another note
![[SQL Injection]]

# Embed a section
![[SQL Injection#Overview]]
```

---

## 5. Security Research Organization

### 5.1 MITRE ATT&CK Integration

```markdown
# T1059 - Command and Scripting Interpreter

## MITRE ATT&CK
- **Technique ID:** T1059
- **Tactic:** Execution
- **Platform:** Windows, Linux, macOS

## Description
Adversaries may abuse command and script interpreters to execute commands, scripts, or binaries.

## My Implementation Notes
- Used this in [[Project A]] to execute [[Reverse Shell]]
- See [[PowerShell Execution]] for Windows variants

## Related Techniques
- [[T1059.001]] - PowerShell
- [[T1059.003]] - Windows Command Shell
- [[T1059.004]] - Unix Shell
```

### 5.2 Vulnerability Documentation

```markdown
# SQL Injection - Login Bypass

## Finding
- **Severity:** Critical
- **CVSS:** 9.8
- **Date Found:** 2024-01-15

## Description
The login form is vulnerable to SQL injection, allowing authentication bypass.

## Payload
```sql
admin' --
```

## Steps to Reproduce
1. Navigate to login page
2. Enter "admin' --" as username
3. Enter anything as password
4. Click Login

## Evidence
![[login_screenshot.png]]

## Remediation
Use parameterized queries (prepared statements).

## References
- [[SQL Injection]]
- [[OWASP Testing Guide]]
- [[CWE-89]]

## Tags
#vulnerability #critical #sql-injection #web
```

---

## 6. Graph View and Connections

### 6.1 Understanding Graph View

- **Nodes:** Individual notes
- **Edges:** Links between notes
- **Clusters:** Groups of related notes
- **Color Coding:** Tags or folders

### 6.2 Improving Graph Visualization

1. Use consistent naming conventions
2. Create hub notes for major topics
3. Link related concepts
4. Use tags for categorization
5. Create MOCs (Maps of Content)

### 6.3 Map of Content (MOC)

```markdown
# Security Testing MOC

## Methodologies
- [[Penetration Testing Methodology]]
- [[Web Application Testing]]
- [[Network Assessment]]

## Techniques
- [[SQL Injection]]
- [[XSS]]
- [[Command Injection]]
- [[Privilege Escalation]]

## Tools
- [[Nmap Guide]]
- [[Burp Suite Guide]]
- [[Metasploit Guide]]

## References
- [[OWASP Top Ten]]
- [[MITRE ATT&CK]]
```

---

## 7. Plugins for Security Professionals

### 7.1 Essential Plugins

| Plugin | Description |
|--------|-------------|
| Templater | Advanced templates |
| Dataview | Query notes like a database |
| Calendar | Visual calendar |
| Kanban | Task management |
| Excalidraw | Diagrams and drawings |
| Advanced Tables | Table editing |
| Git | Version control |
| Sync | Cross-device sync |

### 7.2 Installing Plugins

```
Settings > Community Plugins > Browse > Install
```

### 7.3 Security-Specific Plugin Ideas

- Custom transform for CVE lookup
- API integration for vulnerability databases
- Automated findings import
- Report generation templates

---

## 8. Markdown Syntax for Security Documentation

### 8.1 Code Blocks

```markdown
```python
# Python script for SQL injection testing
import requests

def test_sqli(url):
    payload = "' OR 1=1--"
    response = requests.get(url, params={'id': payload})
    return 'admin' in response.text
```

```bash
# Nmap scan
nmap -sV -sC -p- target.com
```

```sql
-- SQL injection payload
SELECT * FROM users WHERE id = '1' OR '1'='1';
```
```

### 8.2 Tables

```markdown
| Severity | Count | Status |
|----------|-------|--------|
| Critical | 3 | Open |
| High | 7 | Open |
| Medium | 12 | Remediated |
| Low | 5 | Accepted |
```

### 8.3 Callouts

```markdown
> [!warning]
> This is a critical vulnerability that needs immediate attention.

> [!tip]
> Use parameterized queries to prevent SQL injection.

> [!info]
> See [[OWASP Top Ten]] for more information.
```

---

## 9. Templates for Security Work

### 9.1 Finding Template

```markdown
# {{title}}

## Metadata
- **Date:** {{date}}
- **Severity:** 
- **CVSS:** 
- **Status:** New

## Description


## Impact


## Steps to Reproduce
1. 

## Evidence


## Remediation


## References
- 

## Tags
#finding #severity
```

### 9.2 Project Template

```markdown
# {{title}} - Penetration Test

## Project Details
- **Client:** 
- **Start Date:** {{date}}
- **End Date:** 
- **Scope:** 

## Methodology
- [[Penetration Testing Methodology]]

## Findings
- 

## Timeline
- 

## Notes
- 
```

---

## 10. Version Control with Git

### 10.1 Initializing Git

```bash
# Navigate to vault
cd ~/Documents/SecurityVault

# Initialize Git
git init
git add .
git commit -m "Initial vault setup"

# Add remote (optional)
git remote add origin https://github.com/username/SecurityVault.git
git push -u origin main
```

### 10.2 Obsidian Git Plugin

Install the Git plugin for integrated version control:
```
Settings > Community Plugins > Git > Install
```

---

## 11. Best Practices

### 11.1 Organization

- Use consistent folder structure
- Create MOCs for major topics
- Link related notes extensively
- Use tags for categorization
- Review and update regularly

### 11.2 Note-Taking

- Start with quick notes in Inbox
- Process and organize daily
- Link as you write
- Use templates for consistency
- Review graph view weekly

### 11.3 Security Considerations

- Encrypt sensitive vaults
- Use strong master password
- Back up regularly
- Be careful with cloud sync
- Don't store credentials in plain text

---

## 12. Troubleshooting

### Common Issues

**Problem:** Vault won't open

```bash
# Check vault folder permissions
ls -la ~/Documents/SecurityVault

# Try opening from terminal
obsidian ~/Documents/SecurityVault
```

**Problem:** Sync issues

```bash
# Check Git status
git status

# Resolve conflicts manually
git merge --abort
git pull
```

**Problem:** Plugin not working

```bash
# Disable and re-enable plugin
# Check plugin updates
# Review plugin settings
```

---

## References

- [Obsidian Official Website](https://obsidian.md/)
- [Obsidian Documentation](https://help.obsidian.md/)
- [Obsidian Forum](https://forum.obsidian.md/)
- [Kali Linux Obsidian Package](https://www.kali.org/tools/obsidian/)
- [Obsidian Plugins](https://obsidian.md/plugins)
