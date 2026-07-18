# mysql CHEATSHEET

## Quick Reference

### Connect
```bash
mysql -u root -p                                    # Local
mysql -h 192.168.1.1 -u root -p                     # Remote
mysql -h 192.168.1.1 -P 3307 -u root -p             # Custom port
```

### Execute Query
```bash
mysql -h 192.168.1.1 -u root -p -e "SELECT VERSION();"
```

### SQL Commands
```sql
SHOW DATABASES;                # List databases
USE database_name;             # Select database
SHOW TABLES;                   # List tables
DESCRIBE table_name;           # Show columns
SELECT * FROM users;           # Query data
SELECT user, host FROM mysql.user;  # List users
SHOW GRANTS;                   # Show privileges
```

### Dump Database
```bash
mysqldump -h 192.168.1.1 -u root -p --all-databases > full_dump.sql
mysqldump -h 192.168.1.1 -u root -p database_name > db_dump.sql
```

### Default Credentials
```
root: (empty)
root: root
root: password
admin: admin
mysql: mysql
```

---

## Target: 192.168.1.x

```bash
mysql -h 192.168.1.1 -u root -p
SHOW DATABASES;
USE target_db;
SHOW TABLES;
SELECT * FROM users;
```
