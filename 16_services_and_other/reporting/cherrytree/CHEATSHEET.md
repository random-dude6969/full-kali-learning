# CherryTree - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Launch CherryTree
cherrytree

# Launch with file
cherrytree /path/to/file.ctb

# Create new database
cherrytree --new_tab
```

---

## Keyboard Shortcuts

### File Operations
| Shortcut | Action |
|----------|--------|
| Ctrl + N | New node |
| Ctrl + Shift + N | New child node |
| Ctrl + Enter | New sibling node |
| Ctrl + O | Open file |
| Ctrl + S | Save |
| Ctrl + Shift + S | Save as |
| Ctrl + P | Print |

### Text Formatting
| Shortcut | Action |
|----------|--------|
| Ctrl + B | Bold |
| Ctrl + I | Italic |
| Ctrl + U | Underline |
| Ctrl + Shift + S | Strikethrough |
| Ctrl + Shift + H | Highlight |
| Ctrl + Shift + C | Code block |
| Ctrl + Shift + L | Lowercase |
| Ctrl + Shift + U | Uppercase |

### Navigation
| Shortcut | Action |
|----------|--------|
| Ctrl + F | Find |
| Ctrl + H | Find and Replace |
| F3 | Find next |
| Shift + F3 | Find previous |
| Ctrl + D | Bookmark |
| Tab | Indent |
| Shift + Tab | Unindent |

### Node Operations
| Shortcut | Action |
|----------|--------|
| F2 | Rename node |
| Delete | Delete node |
| Ctrl + M | Move node |
| Ctrl + Up | Move node up |
| Ctrl + Down | Move node down |

---

## Export Commands

```bash
# Export to HTML directory
cherrytree file.ctb --export-to-html-dir /path/to/html/

# Export to plain text
cherrytree file.ctb --export-to-txt-dir /path/to/txt/

# Export to PDF
cherrytree file.ctb --export-to-pdf /path/to/output.pdf

# Export single node
cherrytree file.ctb -n "Node Name" --export-to-html-dir /path/to/html/
```

---

## Node Types

| Type | Extension | Use Case |
|------|-----------|----------|
| Rich Text | .ctb | Reports, documentation |
| Plain Text | .txt | Quick notes |
| Code | .ctb | Commands, scripts |
| Without Text | .ctb | Container nodes |

---

## Backup Script

```bash
#!/bin/bash
BACKUP_DIR="/backup/cherrytree"
SOURCE_DIR="$HOME/.local/share/cherrytree"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"
cp -r "$SOURCE_DIR" "$BACKUP_DIR/cherrytree_$DATE"
echo "Backup completed: $BACKUP_DIR/cherrytree_$DATE"
```

---

## Pentest Template Structure

```
Project Name
├── 1. Reconnaissance
│   ├── Passive Recon
│   └── Active Recon
├── 2. Scanning
│   ├── Network Scans
│   └── Service Enumeration
├── 3. Exploitation
│   ├── Initial Access
│   └── Privilege Escalation
├── 4. Findings
│   ├── Critical
│   ├── High
│   ├── Medium
│   └── Low
└── 5. Report
    ├── Executive Summary
    └── Technical Details
```

---

## Useful SQL Queries

```sql
-- List all nodes
SELECT * FROM node;

-- Search node content
SELECT * FROM node WHERE txt LIKE '%keyword%';

-- Count nodes
SELECT COUNT(*) FROM node;

-- Export node content
SELECT txt FROM node WHERE name = 'Node Name';
```

---

## Troubleshooting

```bash
# Kill stuck CherryTree
killall cherrytree

# Reset settings
rm -rf ~/.config/cherrytree/

# Check SQLite file integrity
sqlite3 file.ctb "PRAGMA integrity_check;"

# View CherryTree logs
journalctl -u cherrytree
```

---

## Cloud Sync Options

- **Dropbox:** Save .ctb in Dropbox folder
- **Google Drive:** Sync via Google Drive
- **Syncthing:** P2P sync between devices
- **Git:** Version control for documentation
