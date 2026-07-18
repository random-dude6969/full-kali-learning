---
title: sqlitebrowser
category: discovery
subcategory: database_discovery
phase: 09
mitre_attack: TA0007
tags:
  - database
  - sqlite
  - gui
  - browsing
  - extraction
difficulty: beginner
os: linux
install: sudo apt install sqlitebrowser
github: https://github.com/sqlitebrowser/sqlitebrowser
related:
  - mdb-sql
  - mysql
  - sqlite3
---

# sqlitebrowser

## Chapter 1: Overview

**sqlitebrowser** (DB Browser for SQLite) is a GUI tool for creating, designing, and editing SQLite database files. It's used in penetration testing to examine SQLite databases found on systems.

### Key Features

- **GUI interface**: Visual database browsing
- **SQL editor**: Execute custom queries
- **Data browsing**: View and edit table data
- **Schema inspection**: Visual table structure
- **Export/Import**: CSV, SQL formats
- **Cross-platform**: Works on multiple OS

### Common Uses

| Use Case | Description |
|----------|-------------|
| Browser databases | Examine Firefox/Chrome data |
| Application databases | Analyze app data |
| Mobile forensics | SQLite from Android/iOS |
| Config files | Many apps use SQLite |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install sqlitebrowser
```

### Verify Installation

```bash
sqlitebrowser --version
```

### Launch

```bash
sqlitebrowser
```

---

## Chapter 3: Basic Usage

### Open Database

```bash
sqlitebrowser database.db
```

### Create New Database

```bash
sqlitebrowser --new database.db
```

### Command Line Queries

```bash
sqlite3 database.db "SELECT * FROM users;"
```

---

## Chapter 4: GUI Features

### Browse Data

1. Open database file
2. Click "Browse Data" tab
3. Select table from dropdown
4. View and filter data

### Execute SQL

1. Click "Execute SQL" tab
2. Enter SQL query
3. Click "Execute" button
4. View results

### Edit Data

1. Browse to table
2. Double-click cell to edit
3. Click "Write Changes" to save

---

## Chapter 5: SQL Operations

### List Tables

```sql
SELECT name FROM sqlite_master WHERE type='table';
```

### Show Schema

```sql
SELECT sql FROM sqlite_master WHERE name='table_name';
```

### Query Data

```sql
SELECT * FROM users;
SELECT * FROM users WHERE username = 'admin';
SELECT COUNT(*) FROM users;
```

### Export Data

```sql
.mode csv
.headers on
.output users.csv
SELECT * FROM users;
.quit
```

---

## Chapter 6: Common SQLite Locations

### Browser Databases

```bash
# Firefox
~/.mozilla/firefox/*.default/places.sqlite

# Chrome
~/.config/google-chrome/Default/History
```

### Application Databases

```bash
# WhatsApp
~/Android/data/com.whatsapp/databases/msgstore.db

# Telegram
~/.local/share/TelegramDesktop/tdata/ ...
```

### System Databases

```bash
# Various system databases
/var/lib/dpkg/status
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Browser Forensics

```bash
# Open Firefox database
sqlitebrowser ~/.mozilla/firefox/*.default/places.sqlite

# Browse history
# Execute: SELECT url, title, visit_count FROM moz_places;
```

### Scenario 2: Application Analysis

```bash
# Find SQLite databases
find / -name "*.db" -o -name "*.sqlite" 2>/dev/null

# Open and examine
sqlitebrowser found_database.db
```

### Scenario 3: Data Extraction

```bash
# Extract data via command line
sqlite3 database.db <<EOF
.mode csv
.headers on
.output export.csv
SELECT * FROM sensitive_table;
.quit
EOF
```

---

## Chapter 8: Mitigation & Defense

**For administrators:**

- Encrypt sensitive SQLite databases
- Restrict file permissions
- Implement access controls
- Regular security audits
- Secure application development

### SQLite Encryption

```bash
# Using SQLCipher for encryption
sqlcipher encrypted.db
PRAGMA key = 'secret_key';
 ATTIBUTE DATABASE;
CREATE TABLE sensitive (data TEXT);
```

---

## Resources

- [sqlitebrowser Website](https://sqlitebrowser.org/)
- [SQLite Documentation](https://www.sqlite.org/docs.html)
- [SQLCipher](https://www.zetetic.net/sqlcipher/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
