# sqlitebrowser CHEATSHEET

## Quick Reference

### Open GUI
```bash
sqlitebrowser database.db
```

### Command Line Queries
```bash
sqlite3 database.db "SELECT * FROM users;"
sqlite3 database.db ".tables"                           # List tables
sqlite3 database.db ".schema table_name"               # Show schema
sqlite3 database.db "SELECT * FROM users WHERE id=1;"  # Query
```

### Export to CSV
```bash
sqlite3 database.db <<EOF
.mode csv
.headers on
.output export.csv
SELECT * FROM users;
.quit
EOF
```

### Common SQLite Locations
```bash
# Firefox
~/.mozilla/firefox/*.default/places.sqlite

# Chrome
~/.config/google-chrome/Default/History
```

### SQL Commands
```sql
SELECT name FROM sqlite_master WHERE type='table';  # List tables
SELECT sql FROM sqlite_master WHERE name='table';   # Schema
SELECT * FROM table;                                 # Query data
```

---

## Target: database.db

```bash
sqlite3 database.db ".tables"
sqlite3 database.db "SELECT * FROM users;"
```
