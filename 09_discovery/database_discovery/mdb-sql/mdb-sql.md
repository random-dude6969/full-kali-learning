---
title: mdb-sql
category: discovery
subcategory: database_discovery
phase: 09
mitre_attack: TA0007
tags:
  - database
  - microsoft_access
  - mdb
  - sql
  - extraction
difficulty: beginner
os: linux
install: sudo apt install mdbtools
github: https://github.com/mdbtools/mdbtools
related:
  - sqlitebrowser
  - mysql
  - oscanner
---

# mdb-sql

## Chapter 1: Overview

**mdb-sql** is a tool for querying Microsoft Access (.mdb/.accdb) databases on Linux. It provides SQL query capabilities and data extraction from Access database files.

### Key Features

- **SQL queries**: Run SQL against Access databases
- **Data extraction**: Export tables and queries
- **Schema inspection**: List tables, columns, relationships
- **CSV export**: Export data to CSV format
- **Cross-platform**: Access databases on Linux

### Supported Operations

| Operation | Description |
|-----------|-------------|
| SQL Query | Execute SELECT statements |
| Table List | Show all tables |
| Schema | Show table structure |
| Export | Export to CSV/SQL |
| Import | Import CSV data |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install mdbtools
```

### Verify Installation

```bash
mdb-ver --help
mdb-tables --help
mdb-sql --help
```

---

## Chapter 3: Basic Usage

### List Tables

```bash
mdb-tables database.mdb
```

### Show Table Schema

```bash
mdb-schema database.mdb
```

### Run SQL Query

```bash
mdb-sql database.mdb
```

### Export Table to CSV

```bash
mdb-export database.mdb TableName > output.csv
```

---

## Chapter 4: Interactive SQL

### Start Interactive Mode

```bash
mdb-sql database.mdb
```

### SQL Commands

```sql
-- List tables
tables;

-- Show schema
schema TableName;

-- Query data
SELECT * FROM TableName;

-- Filter data
SELECT * FROM TableName WHERE Column = 'value';
```

### Exit

```sql
quit;
```

---

## Chapter 5: Batch Operations

### Export All Tables

```bash
for table in $(mdb-tables database.mdb); do
  mdb-export database.mdb $table > "${table}.csv"
done
```

### Export with Headers

```bash
mdb-export -H database.mdb TableName > output.csv
```

### SQL to CSV

```bash
mdb-sql database.mdb -p "SELECT * FROM Users" > users.csv
```

---

## Chapter 6: Advanced Features

### Show Relationships

```bash
mdb-schema database.mdb --relationships
```

### Show Indexes

```bash
mdb-indexes database.mdb
```

### Show Statistics

```bash
mdb-stat database.mdb
```

### Verbose Output

```bash
mdb-tables -v database.mdb
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Database Analysis

```bash
# List tables
mdb-tables database.mdb

# Show schema
mdb-schema database.mdb

# Export important tables
mdb-export database.mdb Users > users.csv
mdb-export database.mdb Orders > orders.csv
```

### Scenario 2: Data Extraction

```bash
# Extract all data
for table in $(mdb-tables database.mdb); do
  mdb-export database.mdb $table > "${table}.csv"
  echo "Exported: $table"
done
```

### Scenario 3: SQL Analysis

```bash
# Find sensitive data
mdb-sql database.mdb -p "SELECT * FROM Users WHERE Password IS NOT NULL"
```

---

## Chapter 8: Mitigation & Defense

**For administrators:**

- Restrict Access database access
- Implement access controls
- Encrypt sensitive databases
- Regular backups
- Monitor database access
- Migrate to enterprise databases

---

## Resources

- [mdbtools Documentation](https://github.com/mdbtools/mdbtools)
- [Microsoft Access Security](https://support.microsoft.com/en-us/office/encrypt-a-database)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
