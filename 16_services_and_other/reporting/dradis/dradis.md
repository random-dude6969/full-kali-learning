# Dradis - Collaboration Framework for Penetration Testing

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

Dradis is an open-source collaboration framework designed specifically for penetration testing teams. It serves as a central hub where security professionals can aggregate evidence from multiple tools, manage findings, collaborate in real-time, and generate comprehensive reports. Dradis eliminates the chaos of managing multiple spreadsheets, text files, and tool outputs by providing a unified platform for all penetration testing documentation.

### 1.1 What is Dradis?

Dradis is a web-based application that provides:
- Centralized evidence management
- Real-time team collaboration
- Import from popular security tools
- Template-based reporting
- Methodology tracking
- Quality assurance checks

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Type | Collaboration Framework |
| Version | 5.0.0 (Community Edition) |
| License | GNU GPL v3 |
| Architecture | Ruby on Rails + React |
| Database | PostgreSQL / SQLite |
| Port | 3000 (default) |
| GitHub | Dradis/dradis-ce |

### 1.3 Dradis CE vs Pro

| Feature | Community Edition | Pro |
|---------|-------------------|-----|
| Price | Free | Paid |
| Import | Basic plugins | Advanced plugins |
| Reporting | Basic templates | Advanced templates |
| Support | Community | Professional |
| Methodologies | Basic | Advanced |
| QA Checks | Basic | Advanced |

### 1.4 Why Use Dradis for Penetration Testing?

- **Centralized Data:** All evidence in one place
- **Real-time Collaboration:** Multiple testers work simultaneously
- **Tool Integration:** Import from Nmap, Nessus, Burp, Metasploit
- **Consistent Reporting:** Template-based output ensures quality
- **Methodology Tracking:** Follow established testing frameworks
- **Audit Trail:** Track all changes and contributions

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install dradis

# Start the service
sudo dradis-start

# Access at http://127.0.0.1:3000

# Stop the service
sudo dradis-stop
```

### 2.2 Docker

```bash
# Clone the repository
git clone https://github.com/dradis/dradis-ce.git
cd dradis-ce

# Start with Docker Compose
docker compose up

# Access at http://localhost:3000
```

### 2.3 From Source

```bash
# Install dependencies
sudo apt install ruby ruby-dev build-essential libsqlite3-dev

# Clone repository
git clone https://github.com/dradis/dradis-ce.git
cd dradis-ce

# Install gems
bundle install

# Setup database
bin/rake db:setup

# Start server
bin/rails server

# Access at http://localhost:3000
```

### 2.4 First-Time Setup

1. Navigate to `http://localhost:3000`
2. Create an admin account
3. Set a password
4. Start adding evidence and findings

---

## 3. Interface Overview

### 3.1 Main Dashboard

```
+------------------------------------------------------------------+
|  Dradis CE  |  Dashboard  |  Project  |  Team  |  Help           |
+------------------------------------------------------------------+
|            |                                                    |
|  Project   |              Main Content Area                    |
|  Tree      |                                                    |
|            |  Evidence  |  Findings  |  Notes  |  Export         |
|  (Left)    |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Methodology  |  QA  |  Log  |  Comments                      |
+------------------------------------------------------------------+
```

### 3.2 Key Components

- **Project Tree:** Hierarchical organization of evidence and findings
- **Content Area:** Display and edit content
- **Methodology Panel:** Track testing methodology
- **QA Panel:** Quality assurance checks
- **Export Panel:** Generate reports
- **Activity Log:** Track team activity

---

## 4. Project Structure

### 4.1 Nodes and Notes

Dradis organizes data into a tree structure:

```
Project
├── Evidence
│   ├── Host 192.168.1.1
│   │   ├── Nmap Scan Results
│   │   ├── Service Enumeration
│   │   └── Vulnerability Findings
│   └── Host 192.168.1.2
│       └── ...
├── Findings
│   ├── SQL Injection
│   ├── XSS Vulnerability
│   └── Weak Password Policy
├── Notes
│   ├── Meeting Notes
│   └── Client Communications
└── Methodologies
    ├── OWASP Testing Guide
    └── PTES
```

### 4.2 Creating Nodes

```
Right-click in Project Tree > New Node
```

Or use keyboard shortcut:
```
Ctrl + N (New Node)
Ctrl + Shift + N (New Child Node)
```

---

## 5. Working with Findings and Evidence

### 5.1 Adding Evidence

1. Right-click on a node > "Add Evidence"
2. Select a template (Nmap, Nessus, etc.)
3. Paste or import the evidence
4. Save the node

### 5.2 Creating Findings

1. Navigate to the Findings section
2. Click "New Finding"
3. Fill in the template:
   - **Title:** Finding name
   - **Severity:** Critical/High/Medium/Low/Info
   - **CVSS Score:** If applicable
   - **Description:** Detailed explanation
   - **Impact:** Business impact
   - **Remediation:** How to fix
   - **References:** Links to CVEs, OWASP, etc.

### 5.3 Finding Templates

```markdown
# Finding Title

## Severity
High (CVSS: 8.5)

## Description
[Detailed description of the vulnerability]

## Impact
[Business and technical impact]

## Steps to Reproduce
1. Step one
2. Step two
3. Step three

## Evidence
[Screenshots, command output, etc.]

## Remediation
[How to fix the vulnerability]

## References
- [CVE Link]
- [OWASP Reference]
```

---

## 6. Importing from External Tools

### 6.1 Supported Tools

| Tool | Import Method |
|------|---------------|
| Nmap | XML output |
| Nessus | XML export |
| Burp Suite | XML export |
| Metasploit | XML output |
| Nikto | HTML/XML output |
| SQLMap | HTML output |
| OpenVAS | XML export |
| Qualys | CSV export |

### 6.2 Nmap Import

```bash
# Generate Nmap XML output
nmap -sV -sC -oX nmap_results.xml target.com

# In Dradis:
# 1. Click "Import" in the toolbar
# 2. Select "Nmap XML"
# 3. Upload nmap_results.xml
# 4. Review imported hosts and services
```

### 6.3 Nessus Import

```bash
# Export from Nessus:
# 1. Go to My Scans
# 2. Select scan
# 3. Export > Nessus (.nessus)

# In Dradis:
# 1. Click "Import"
# 2. Select "Nessus"
# 3. Upload .nessus file
```

### 6.4 Burp Suite Import

```bash
# Export from Burp:
# 1. Go to Project options
# 2. Click "Export"
# 3. Select "XML"

# In Dradis:
# 1. Click "Import"
# 2. Select "Burp Suite"
# 3. Upload XML file
```

### 6.5 Metasploit Import

```bash
# Generate Metasploit XML
msfconsole -q
db_export -f xml /path/to/export.xml

# In Dradis:
# 1. Click "Import"
# 2. Select "Metasploit"
# 3. Upload XML file
```

---

## 7. Export Capabilities

### 7.1 Export Formats

| Format | Use Case |
|--------|----------|
| HTML | Web-ready report |
| PDF | Formal document |
| Word (DOCX) | Client delivery |
| Text | Plain text |
| CSV | Data analysis |

### 7.2 Generating Reports

1. Click "Export" in the toolbar
2. Select format (HTML, PDF, Word)
3. Choose template
4. Configure options
5. Generate and download

### 7.3 Report Templates

Dradis includes several built-in templates:
- Executive Summary
- Technical Report
- Vulnerability Assessment
- Penetration Test Report

---

## 8. Team Collaboration Features

### 8.1 Real-Time Updates

Multiple team members can work on the same project simultaneously:
- Changes appear in real-time
- Conflict resolution built-in
- Activity log tracks all changes

### 8.2 User Management

```
Team > Manage Users > Add User
```

User roles:
- **Admin:** Full access
- **Editor:** Create/edit content
- **Viewer:** Read-only access

### 8.3 Comments and Notes

Add comments to any node:
```
Right-click > Add Comment
```

---

## 9. Integrations with Other Tools

### 9.1 Plugin System

Dradis uses plugins to extend functionality:

```bash
# List installed plugins
ls plugins/

# Install a plugin
cd plugins/plugin-name
bundle install
```

### 9.2 Built-in Plugins

- **Nmap Import:** Parse Nmap XML
- **Nessus Import:** Parse Nessus output
- **Burp Import:** Parse Burp Suite XML
- **Metasploit Import:** Parse MSF output
- **HTML Export:** Generate HTML reports
- **Word Export:** Generate Word documents

### 9.3 Custom Plugins

Create custom plugins for specific tools:

```ruby
# plugins/my_tool/lib/my_tool/importer.rb
module MyTool
  class Importer < ActiveSupport::Concern
    include Dradis::Plugin::Importer
    
    def import(params={})
      # Parse your tool's output
      # Create notes and findings
    end
  end
end
```

---

## 10. Methodology Tracking

### 10.1 Built-in Methodologies

- OWASP Testing Guide v4
- PTES (Penetration Testing Execution Standard)
- OSSTMM
- Custom methodologies

### 10.2 Using Methodologies

1. Click "Methodology" in the toolbar
2. Select a methodology
3. Check off completed steps
4. Track progress

### 10.3 Custom Methodologies

Create your own methodology:

1. Go to "Methodologies"
2. Click "New Methodology"
3. Add phases and steps
4. Save and use

---

## 11. Quality Assurance Checks

### 11.1 Running QA Checks

```
QA > Run Checks
```

### 11.2 QA Rule Examples

- Evidence without findings
- Findings without evidence
- Missing severity ratings
- Incomplete descriptions
- Missing remediation

### 11.3 Custom QA Rules

Create custom rules for your team's standards.

---

## 12. API Usage

### 12.1 API Endpoints

```bash
# List nodes
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:3000/api/nodes

# Create a node
curl -X POST -H "Authorization: Token YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"node":{"label":"New Host","type_id":1}}' \
    http://localhost:3000/api/nodes

# Update a node
curl -X PUT -H "Authorization: Token YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"node":{"label":"Updated Host"}}' \
    http://localhost:3000/api/nodes/1
```

### 12.2 API Authentication

Generate an API token in the Dradis UI:
```
Team > API Tokens > Generate New Token
```

---

## 13. Best Practices

### 13.1 Workflow

1. **Setup:** Create project and methodology
2. **Import:** Bring in tool outputs early
3. **Document:** Add evidence as you go
4. **Review:** Use QA checks regularly
5. **Report:** Generate final report

### 13.2 Organization

- Use consistent naming conventions
- Create templates for common findings
- Tag nodes with severity/phase
- Use the methodology panel

### 13.3 Collaboration

- Assign clear roles
- Use comments for discussions
- Review each other's work
- Run QA checks before reporting

---

## 14. Troubleshooting

### Common Issues

**Problem:** Service won't start

```bash
# Check service status
sudo systemctl status dradis

# Start manually
cd /usr/share/dradis-ce
bundle exec rails server
```

**Problem:** Import errors

```bash
# Check plugin compatibility
# Ensure correct file format
# Check Dradis logs
tail -f /var/log/dradis/production.log
```

**Problem:** Slow performance

```bash
# Clear cache
bundle exec rake tmp:clear

# Restart service
sudo systemctl restart dradis
```

---

## References

- [Dradis Official Website](https://dradis.com/)
- [Dradis GitHub](https://github.com/dradis/dradis-ce)
- [Dradis Documentation](https://docs.dradisframework.com/)
- [Kali Linux Dradis Package](https://www.kali.org/tools/dradis/)
