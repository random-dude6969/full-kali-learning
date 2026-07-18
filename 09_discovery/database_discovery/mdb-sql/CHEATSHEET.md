# mdb-sql CHEATSHEET

## Quick Reference

### List Tables
```bash
mdb-tables database.mdb
```

### Show Schema
```bash
mdb-schema database.mdb
```

### Export to CSV
```bash
mdb-export database.mdb TableName > output.csv
```

### Interactive SQL
```bash
mdb-sql database.mdb
```

### SQL Commands
```sql
tables;                          # List tables
schema TableName;                # Show schema
SELECT * FROM TableName;         # Query data
SELECT * FROM TableName WHERE Column = 'value';
quit;                            # Exit
```

### Batch Export
```bash
for table in $(mdb-tables database.mdb); do
  mdb-export database.mdb $table > "${table}.csv"
done
```

### Options
```bash
-H           # Include headers
-v           # Verbose
```

---

## Target: database.mdb

```bash
mdb-tables database.mdb
mdb-schema database.mdb
mdb-export database.mdb Users > users.csv
```
