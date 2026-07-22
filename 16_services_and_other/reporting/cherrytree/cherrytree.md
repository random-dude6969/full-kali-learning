# CherryTree - Hierarchical Note-Taking Application

## Complete Educational Guide for Cybersecurity Professionals

---

## 1. Introduction

CherryTree is a powerful hierarchical note-taking application designed for organizing complex information in a tree structure. For cybersecurity professionals, CherryTree serves as an indispensable tool for documenting penetration test findings, organizing reconnaissance data, managing vulnerability assessments, and creating structured security reports.

### 1.1 What is CherryTree?

CherryTree is a free, open-source hierarchical note-taking application that supports:
- Rich text formatting
- Syntax highlighting for code
- Image embedding
- Table creation
- Hyperlink management
- Import/Export in multiple formats
- Password protection and encryption

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Author | Giuseppe Penone & Evgenii Gurianov |
| License | GPL v3 |
| Version | 1.2.0 |
| Language | C++ (GTK) |
| Storage | SQLite (.ctb) or XML (.ctd) |
| Platform | Linux, Windows, macOS |

### 1.3 Why Use CherryTree for Cybersecurity?

- **Hierarchical Organization:** Structure findings by target, vulnerability type, or engagement phase
- **Rich Content:** Embed screenshots, code snippets, and command outputs
- **Searchability:** Quickly find specific findings across large documentation sets
- **Export Options:** Generate professional reports in multiple formats
- **Encryption:** Protect sensitive client data
- **Cross-Platform:** Work on any operating system

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install cherrytree

# Launch from terminal
cherrytree

# Or from application menu
Applications > Reporting > cherrytree
```

### 2.2 Ubuntu/Debian

```bash
# Add PPA for latest version
sudo add-apt-repository ppa:andrey-utkin/cherrytree
sudo apt update
sudo apt install cherrytree
```

### 2.3 From Source

```bash
# Install dependencies
sudo apt install build-essential cmake libgtkmm-3.0-dev \
    libsqlite3-dev libxml2-dev libgspell-1-dev libvte-2.91-dev

# Clone repository
git clone https://github.com/giuspen/CherryTree.git
cd CherryTree

# Build
mkdir build && cd build
cmake ..
make -j$(nproc)

# Install
sudo make install
```

### 2.4 Windows/macOS

Download the installer from the official website:
- https://www.giuspen.net/cherrytree/

---

## 3. Interface Overview

### 3.1 Main Window Components

```
+------------------------------------------------------------------+
|  Menu Bar  |  File  Edit  View  Tree  Nodes  Insert  Help        |
+------------------------------------------------------------------+
|  Toolbar   |  New  Open  Save  |  B  I  U  S  |  H1  H2  H3     |
+------------------------------------------------------------------+
|            |                                                    |
|   Tree     |              Document Area                         |
|   Panel    |                                                    |
|            |  Rich text content, code blocks,                   |
|  (Left)    |  images, tables, etc.                              |
|            |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Status Bar  |  Line: 1  |  Column: 1  |  Words: 0             |
+------------------------------------------------------------------+
```

### 3.2 Tree Panel

The tree panel (left side) displays the hierarchical structure:

- **Nodes:** Individual pages or sections
- **Sub-nodes:** Child elements under parent nodes
- **Node Types:** Plain text, Rich text, Code, Executable, etc.
- **Drag & Drop:** Reorganize nodes by dragging

### 3.3 Document Area

The document area (right side) displays the content of the selected node:

- **Rich Text:** Formatting, colors, fonts
- **Code Blocks:** Syntax-highlighted code
- **Images:** Embedded screenshots and diagrams
- **Tables:** Structured data
- **Links:** Internal and external hyperlinks

---

## 4. Creating Pentest Documentation

### 4.1 Organizing by Engagement

```
Client: Example Corp
├── 1. Reconnaissance
│   ├── 1.1 Passive Recon
│   │   ├── DNS Enumeration
│   │   ├── WHOIS Lookup
│   │   ├── Social Media
│   │   └── Google Dorking
│   └── 1.2 Active Recon
│       ├── Nmap Scans
│       ├── Service Enumeration
│       └── Web Application Scanning
├── 2. Vulnerability Assessment
│   ├── 2.1 Web Application
│   │   ├── SQL Injection
│   │   ├── XSS
│   │   └── CSRF
│   └── 2.2 Network Services
│       ├── SMB
│       ├── SSH
│       └── RDP
├── 3. Exploitation
│   ├── 3.1 Initial Access
│   ├── 3.2 Privilege Escalation
│   └── 3.3 Lateral Movement
├── 4. Post-Exploitation
│   ├── 4.1 Data Exfiltration
│   ├── 4.2 Persistence
│   └── 4.3 Covering Tracks
├── 5. Findings Summary
│   ├── 5.1 Critical
│   ├── 5.2 High
│   ├── 5.3 Medium
│   └── 5.4 Low
└── 6. Recommendations
    ├── 6.1 Immediate Actions
    ├── 6.2 Short-term
    └── 6.3 Long-term
```

### 4.2 Organizing by Vulnerability Type

```
Project Findings
├── Injection Vulnerabilities
│   ├── SQL Injection
│   │   ├── Finding 1: Login Bypass
│   │   └── Finding 2: Data Extraction
│   └── Command Injection
│       └── Finding 1: OS Command Execution
├── Authentication Issues
│   ├── Weak Password Policy
│   ├── Missing Account Lockout
│   └── Session Management Flaws
├── Access Control
│   ├── IDOR Vulnerabilities
│   ├── Privilege Escalation
│   └── Missing Authorization
└── Information Disclosure
    ├── Verbose Error Messages
    ├── Source Code Disclosure
    └── Sensitive Data Exposure
```

### 4.3 Organizing by Phase

```
Penetration Test - Acme Corp
├── Phase 1: Planning & Scoping
│   ├── Rules of Engagement
│   ├── Target Systems
│   └── Timeline
├── Phase 2: Reconnaissance
│   └── [Reconnaissance nodes]
├── Phase 3: Scanning
│   └── [Scanning nodes]
├── Phase 4: Exploitation
│   └── [Exploitation nodes]
├── Phase 5: Post-Exploitation
│   └── [Post-exploitation nodes]
└── Phase 6: Reporting
    ├── Executive Summary
    ├── Technical Findings
    └── Recommendations
```

---

## 5. Rich Content Support

### 5.1 Code Blocks

CherryTree supports syntax highlighting for multiple languages:

```python
# Python example
import requests

def test_sql_injection(url):
    payload = "' OR 1=1--"
    response = requests.get(url, params={'id': payload})
    if 'admin' in response.text:
        return True
    return False
```

```bash
# Bash commands
nmap -sV -sC -p- target.com
nikto -h target.com
sqlmap -u "http://target.com/page?id=1" --dbs
```

```sql
-- SQL queries
SELECT * FROM users WHERE id = '1' OR '1'='1';
UNION SELECT username, password FROM admin_users;
```

### 5.2 Tables

| Severity | Count | Status |
|----------|-------|--------|
| Critical | 3 | Open |
| High | 7 | Open |
| Medium | 12 | Remediated |
| Low | 5 | Accepted |

### 5.3 Images

- Drag and drop images directly into nodes
- Supports PNG, JPEG, GIF, BMP
- Images are embedded in the database file
- Useful for screenshots of vulnerabilities

### 5.4 Hyperlinks

```markdown
Internal link: [[Node Name]]
External link: https://owasp.org
File link: file:///path/to/evidence.pdf
```

---

## 6. Syntax Highlighting

### 6.1 Supported Languages

CherryTree supports syntax highlighting for:

- **Programming:** Python, JavaScript, PHP, Ruby, Java, C/C++, C#
- **Shell:** Bash, PowerShell, Zsh
- **Web:** HTML, CSS, XML, JSON, YAML
- **Database:** SQL
- **Config:** INI, TOML, Apache, Nginx
- **Security:** Metasploit RC, Nmap XML

### 6.2 Code Block Features

```python
# Auto-completion
# Line numbers (toggle)
# Highlight current line
# Show spaces and tabs
# Word wrap
```

---

## 7. Node Types

### 7.1 Node Type Options

| Type | Description | Use Case |
|------|-------------|----------|
| Rich Text | Full formatting | Reports, documentation |
| Plain Text | No formatting | Quick notes, logs |
| Code | Syntax highlighted | Commands, scripts |
| Executable | Runnable code | Automation scripts |
| Diff | Diff highlighting | Comparing configs |
| Image | Image viewer | Screenshots |
| Without Text | Container only | Organization nodes |

### 7.2 Node Properties

- **Name:** Node title
- **Type:** Content type
- **Tags:** Custom labels
- **State:** Custom states (e.g., TODO, IN PROGRESS, DONE)
- **Icon:** Custom node icon
- **Color:** Node color coding
- **Read-only:** Prevent editing
- **Protection:** Password protection per node

---

## 8. Password Protection and Encryption

### 8.1 Database Password

```
File > Password Protection > Set Password
```

### 8.2 Per-Node Password

Right-click node > Node Properties > Protection

### 8.3 Encryption Methods

| Method | Security | Description |
|--------|----------|-------------|
| Standard | Medium | Default encryption |
| Password-based | High | Derived from password |

---

## 9. Import/Export Capabilities

### 9.1 Import Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| CherryTree XML | .ctd | Native XML format |
| CherryTree SQLite | .ctb | Native SQLite format |
| Gnote | XML | Gnote notes |
| Tomboy | XML | Tomboy notes |
| KeepNote | HTML | KeepNote format |
| Basket | HTML | Basket format |
| Knowdown | MD | Markdown files |
| Plain Text | .txt | Text files |

### 9.2 Export Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| CherryTree XML | .ctd | Full fidelity |
| HTML | .html | Web-ready |
| PDF | .pdf | Document sharing |
| Plain Text | .txt | Universal |
| Markdown | .md | Documentation |
| PNG | .png | Images |
| SVG | .svg | Vector images |
| Multipage PDF | .pdf | Multi-page document |

### 9.3 Export Commands

```bash
# Export all nodes to HTML
cherrytree /path/to/file.ctb --export-to-html-dir /path/to/html/

# Export to plain text
cherrytree /path/to/file.ctb --export-to-txt-dir /path/to/txt/

# Export to PDF
cherrytree /path/to/file.ctb --export-to-pdf /path/to/output.pdf
```

---

## 10. Search and Filtering

### 10.1 Search Features

- **Full-text search:** Find text across all nodes
- **Regex search:** Pattern matching
- **Case-sensitive search:** Exact case matching
- **Search in node names:** Find by title
- **Search in code:** Search within code blocks
- **Replace:** Find and replace text

### 10.2 Keyboard Shortcuts for Search

```
Ctrl + F          Find
Ctrl + H          Find and Replace
Ctrl + Shift + F  Find in all nodes
F3                Find next
Shift + F3        Find previous
```

---

## 11. Syncing and Backup Strategies

### 11.1 Backup Options

```bash
# Manual backup
cp -r ~/.local/share/cherrytree/ /backup/cherrytree/

# Automated backup script
#!/bin/bash
BACKUP_DIR="/backup/cherrytree"
SOURCE_DIR="$HOME/.local/share/cherrytree"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
cp -r "$SOURCE_DIR" "$BACKUP_DIR/cherrytree_$DATE"
```

### 11.2 Sync with Cloud Storage

- Save .ctb files in Dropbox/Google Drive/OneDrive
- Use Syncthing for P2P sync
- Keep Git repository of documentation

### 11.3 Version Control

```bash
# Initialize Git repository in CherryTree directory
cd /path/to/notes/
git init
git add .
git commit -m "Backup $(date +%Y-%m-%d)"
```

---

## 12. Templates for Pentest Reports

### 12.1 Executive Summary Template

```markdown
# Executive Summary

## Engagement Overview
- **Client:** [Client Name]
- **Date Range:** [Start Date] - [End Date]
- **Scope:** [Systems in scope]
- **Methodology:** [Standards followed]

## Key Findings
- Total vulnerabilities: [X]
- Critical: [X]
- High: [X]
- Medium: [X]
- Low: [X]

## Risk Assessment
[Overall risk rating and brief explanation]

## Recommendations
[Top 3-5 critical recommendations]
```

### 12.2 Vulnerability Finding Template

```markdown
# [Vulnerability Title]

## Summary
Brief description of the vulnerability.

## Affected System(s)
- Host: [IP/Hostname]
- Port: [Port Number]
- Service: [Service Name]
- URL: [Web URL]

## Vulnerability Details
### Description
Detailed technical description.

### Steps to Reproduce
1. Step one
2. Step two
3. Step three

### Evidence
[Screenshots, command output, etc.]

## Impact
[Business and technical impact]

## Risk Rating
- Likelihood: [High/Medium/Low]
- Impact: [High/Medium/Low]
- Overall: [Critical/High/Medium/Low]

## Remediation
### Immediate Actions
[Steps to fix immediately]

### Long-term Solutions
[Strategic improvements]

## References
- [CVE Link]
- [OWASP Reference]
- [Vendor Advisory]
```

---

## 13. Best Practices

### 13.1 Documentation Workflow

1. **Create Structure First:** Set up the tree hierarchy before starting
2. **Document As You Go:** Don't wait until the end
3. **Use Consistent Naming:** Standardize node names and tags
4. **Include Evidence:** Screenshots, logs, command output
5. **Tag Findings:** Use tags for severity, status, type
6. **Regular Backups:** Export or backup daily

### 13.2 Organization Tips

- Use numbered prefixes for ordering (01_, 02_, etc.)
- Color-code nodes by severity or status
- Use tags consistently (Critical, High, Medium, Low)
- Create templates for common findings
- Keep a "Notes" node for quick capture

### 13.3 Collaboration

- Export to HTML for sharing with team
- Use PDF export for formal reports
- Keep source files in version control
- Standardize templates across team

---

## 14. Integration with Other Tools

### 14.1 Nmap Integration

```bash
# Import Nmap results
nmap -sV -sC -oX nmap_results.xml target.com

# Copy output to CherryTree node
cat nmap_results.xml | clipboard
```

### 14.2 Metasploit Integration

```bash
# Document Metasploit sessions
msfconsole -q
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
sessions -l  # Copy session info to CherryTree
```

### 14.3 Burp Suite Integration

```bash
# Export Burp findings
Burp > Project options > Export
# Import screenshots and notes into CherryTree
```

---

## 15. Advanced Features

### 15.1 Bookmarks

- Add bookmarks to important nodes
- Quick navigation to key findings
- Ctrl + D to bookmark/unbookmark

### 15.2 Cross-References

```markdown
# Internal links
See [[SQL Injection - Finding 1]]

# Anchors
See [[#section-name]]
```

### 15.3 Tooltips

Add hover text to nodes for quick summaries:

```
Right-click > Node Properties > Tooltip
```

### 15.4 Node Summaries

Add a summary that appears in the tree view:

```
Right-click > Node Properties > Summary
```

---

## 16. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl + N | New node |
| Ctrl + Shift + N | New child node |
| Ctrl + Enter | New sibling node |
| Ctrl + S | Save |
| Ctrl + F | Find |
| Ctrl + H | Find and Replace |
| Ctrl + B | Bold |
| Ctrl + I | Italic |
| Ctrl + U | Underline |
| Ctrl + Shift + S | Strikethrough |
| Ctrl + Shift + H | Highlight |
| Tab | Indent |
| Shift + Tab | Unindent |
| Ctrl + Z | Undo |
| Ctrl + Y | Redo |
| Ctrl + D | Bookmark |
| Ctrl + L | Link |
| Ctrl + Shift + C | Code syntax highlighting |

---

## 17. Troubleshooting

### Common Issues

**Problem:** CherryTree won't start

```bash
# Check for running processes
ps aux | grep cherrytree

# Kill stuck process
killall cherrytree

# Reset settings
rm -rf ~/.config/cherrytree/
```

**Problem:** File won't open

```bash
# Check file permissions
ls -la file.ctb

# Check SQLite integrity
sqlite3 file.ctb "PRAGMA integrity_check;"
```

**Problem:** Slow performance with large databases

- Archive old nodes
- Export completed engagements
- Split into multiple .ctb files
- Use SQLite format instead of XML

---

## References

- [CherryTree Official Website](https://www.giuspen.net/cherrytree/)
- [CherryTree GitHub](https://github.com/giuspen/CherryTree)
- [Kali Linux CherryTree Package](https://www.kali.org/tools/cherrytree/)
- [CherryTree Manual](https://www.giuspen.net/cherrytree-manual/)
