# mactime Cheatsheet — Timeline Analysis from MAC Times

## Quick Command

```bash
mactime [options] [date_range]
```

## Installation

```bash
sudo apt install sleuthkit
```

## Most Common Commands

```bash
# Generate body file then create timeline
fls -r -m / disk.img > body.txt
mactime -b body.txt

# CSV output for spreadsheet analysis
mactime -b body.txt -d > timeline.csv

# One-liner (pipe from fls)
fls -r -m / disk.img | mactime -

# ISO 8601 date format
mactime -b body.txt -y

# With timezone adjustment
mactime -b body.txt -z EST5EDT
```

## Date Filtering

```bash
# Date range
mactime -b body.txt 2024-01-01..2024-01-31

# Single day
mactime -b body.txt 2024-01-15

# Date with time
mactime -b body.txt 2024-01-15T10:00:00

# Time range on a day
mactime -b body.txt 2024-01-15T08:00..2024-01-15T18:00

# Open-ended ranges
mactime -b body.txt 2024-01-15..
mactime -b body.txt ..2024-01-15
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-b file` | Body file path (or `-` for stdin) |
| `-d` | CSV (comma-delimited) output |
| `-h` | Display header with session info |
| `-y` | ISO 8601 date format |
| `-m` | Months as numbers (not with `-y`) |
| `-z TZ` | Timezone of original data |
| `-g file` | Group name file (maps GIDs) |
| `-p file` | Password file (maps UIDs) |
| `-i day\|hour` | Create summary index file |
| `-V` | Print version |

## Body File Format

```
MD5|inode|mode|uid|gid|size|atime|mtime|ctime|crtime|filename
```

| Field | Example |
|-------|---------|
| MD5 | `d41d8cd98f00b204` or `-` |
| inode | `12345` |
| mode | `r/rrwxr-xr-x` |
| uid | `1000` |
| gid | `1000` |
| size | `4096` |
| atime/mtime/ctime/crtime | Unix epoch `1704067200` |
| filename | `/home/user/doc.txt` |

## Generating Body Files

```bash
# From NTFS partition (offset 63 sectors)
fls -f ntfs -o 63 -r -m / disk.img > body.txt

# From ext4 partition (offset 2048 sectors)
fls -f ext4 -o 2048 -r -m / disk.img > body.txt

# Include deleted files
fls -d -r -m / disk.img > body_deleted.txt

# From entire image
tsk_gettimes disk.img > body.txt

# Combine multiple partitions
cat body_p1.txt body_p2.txt > combined.txt
```

## Common Timezone Strings

| TZ | Meaning |
|----|---------|
| `GMT` | UTC |
| `EST5EDT` | US Eastern |
| `CST6CDT` | US Central |
| `PST8PDT` | US Pacific |
| `CET-1CEST` | Central Europe |
| `JST-9` | Japan |

## Output Formats

### Default (Human-Readable)

```
2024-01-15 10:30:00 EST 1000  r/rrwxr-xr-x  4096  /home/user
2024-01-15 10:30:15 EST 1000  r/rrw-r--r--  512   /home/user/doc.txt
```

### CSV (`-d`)

```csv
Date/Time,Source,Type,User,Group,Size,Mode,File
2024-01-15 10:30:00,1000,FILE,user,user,4096,r/rrwxr-xr-x,/home/user
```

## Practical Examples

```bash
# After-hours activity (potential intrusion)
mactime -b body.txt 2024-01-15T18:00..2024-01-16T06:00

# Specific user activity (UID 1000)
grep "|1000|" body.txt | mactime -

# Root activity
grep "|0|" body.txt | mactime -

# Find executable files
grep "\.exe" body.txt | mactime -

# Temp directory activity
grep -i "/tmp\|/temp" body.txt | mactime -
```

## MAC Time Reference

| Time | What It Tracks | User-Settable? |
|------|---------------|----------------|
| **mtime** | File content changes | Yes (`touch -m`) |
| **atime** | File reads/access | Yes (`touch -a`) |
| **ctime** | Metadata changes | **No** (kernel only) |
| **crtime** | File creation | Varies by FS |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No output | Check `wc -l body.txt` — file may be empty |
| Wrong dates | Try `-z GMT` or `-z EST5EDT` |
| Ambiguous times | Use `-y` for ISO 8601 format |
| Need CSV | Add `-d` flag |
| Slow with large files | Pipe through `grep` to filter first |
