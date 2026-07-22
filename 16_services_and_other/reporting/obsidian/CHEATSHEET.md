# Obsidian - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Kali Linux
sudo apt install obsidian
obsidian

# Create new vault
# Open folder as vault
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Version | 1.7.7 |
| Storage | Local Markdown files |
| Format | Markdown |
| Sync | Optional (paid) |
| Plugins | 1000+ available |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Bidirectional Links | Connect notes with [[links]] |
| Graph View | Visualize note connections |
| Plugins | Extend functionality |
| Templates | Reusable note structures |
| Tags | Categorize notes |
| Search | Full-text search |

---

## Keyboard Shortcuts

### Navigation
| Shortcut | Action |
|----------|--------|
| Ctrl + O | Quick switcher |
| Ctrl + P | Command palette |
| Ctrl + E | Toggle edit/preview |
| Ctrl + F | Search in file |
| Ctrl + Shift + F | Search in vault |

### Editing
| Shortcut | Action |
|----------|--------|
| Ctrl + B | Bold |
| Ctrl + I | Italic |
| Ctrl + K | Insert link |
| Ctrl + Shift + K | Insert embed |
| Ctrl + Enter | Create new note |

### Views
| Shortcut | Action |
|----------|--------|
| Ctrl + G | Toggle graph view |
| Ctrl + Shift + L | Toggle backlinks |
| Ctrl + Shift + T | Toggle tags |

---

## Markdown Syntax

### Links
```markdown
# Link to note
[[Note Name]]

# Link with alias
[[Note Name|Display Text]]

# Embed note
![[Note Name]]

# Embed section
![[Note Name#Section Name]]
```

### Code Blocks
```markdown
```python
# Python code
```

```bash
# Bash commands
```

```sql
-- SQL queries
```
```

### Callouts
```markdown
> [!info]
> Information callout

> [!warning]
> Warning callout

> [!tip]
> Tip callout

> [!danger]
> Danger callout
```

### Tables
```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data 1   | Data 2   | Data 3   |
```

---

## Security Vault Structure

```
SecurityVault/
├── 00-Inbox/
├── 01-Methodologies/
├── 02-Techniques/
├── 03-Tools/
├── 04-Findings/
├── 05-References/
├── 06-Templates/
└── 07-Archive/
```

---

## Essential Plugins

| Plugin | Purpose |
|--------|---------|
| Templater | Advanced templates |
| Dataview | Query notes like database |
| Calendar | Visual calendar |
| Kanban | Task management |
| Excalidraw | Diagrams |
| Git | Version control |
| Advanced Tables | Table editing |

---

## Finding Template

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

---

## Useful Dataview Queries

```dataview
TABLE severity, status, date
FROM #finding
SORT date DESC
```

```dataview
LIST
FROM "04-Findings"
WHERE severity = "critical"
```

```dataview
TABLE length(rows) AS "Count"
FROM #finding
GROUP BY severity
```

---

## Version Control

```bash
# Initialize Git
cd ~/Documents/SecurityVault
git init
git add .
git commit -m "Initial vault"

# Add remote
git remote add origin https://github.com/user/vault.git
git push -u origin main
```

---

## Troubleshooting

```bash
# Check vault permissions
ls -la ~/Documents/SecurityVault

# Open from terminal
obsidian ~/Documents/SecurityVault

# Reset settings
rm -rf ~/.config/obsidian/
```
