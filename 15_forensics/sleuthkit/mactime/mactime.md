# mactime — Timeline Analysis from MAC Times

## Table of Contents

1. [Overview](#overview)
2. [Understanding MAC Times](#understanding-mac-times)
3. [The Body File Format](#the-body-file-format)
4. [Installation](#installation)
5. [Command Syntax](#command-syntax)
6. [Options Reference](#options-reference)
7. [Generating Body Files](#generating-body-files)
8. [Creating Timelines](#creating-timelines)
9. [Date Filtering](#date-filtering)
10. [Output Formats](#output-formats)
11. [Working with Time Zones](#working-with-time-zones)
12. [Practical Lab Exercises](#practical-lab-exercises)
13. [Advanced Timeline Analysis](#advanced-timeline-analysis)
14. [Integration with Other Tools](#integration-with-other-tools)
15. [Common Forensic Scenarios](#common-forensic-scenarios)
16. [Troubleshooting](#troubleshooting)
17. [Best Practices](#best-practices)
18. [Summary](#summary)

---

## Overview

`mactime` is a command-line tool that is part of **The Sleuth Kit (TSK)**, an open-source digital forensics toolkit. Its purpose is to create a **chronological timeline of file system activity** based on MAC (Modified, Accessed, Changed) timestamps found in file metadata.

Timeline analysis is one of the most powerful techniques in digital forensics. By arranging file system events in chronological order, investigators can reconstruct what happened on a system, identify suspicious activity, establish alibis or timelines of events, and find correlations between different types of evidence.

`mactime` takes a **body file** (a structured text file containing file metadata from `fls` or `tsk_gettimes`) and produces a sorted timeline showing when files were created, modified, accessed, and changed.

### Why Timeline Analysis Matters

Consider a forensic investigation where you need to answer:

- When did the suspect create that document?
- What files were modified during the timeframe of the incident?
- Were any files deleted to cover tracks?
- What was the user doing at a specific time?
- How do file system events correlate with network logs or registry changes?

A timeline answers all of these questions by presenting file system events in chronological order. `mactime` is the tool that transforms raw file metadata into this actionable format.

### The Timeline Analysis Workflow

```
Disk Image
    ↓
mmls (identify partitions)
    ↓
fls -m / (generate body file)
    ↓
mactime -b body.txt (create timeline)
    ↓
Investigator analyzes timeline
```

---

## Understanding MAC Times

Every file and directory on a filesystem has timestamps associated with it. The most commonly used timestamps are:

### Modified Time (mtime)

The last time the **contents** of the file were changed. For example:
- Editing a text file and saving it
- Compiling a source code file
- Modifying a configuration file

**Important:** Changing file permissions or ownership does NOT update the modified time.

### Accessed Time (atime)

The last time the file was **read** or accessed. This includes:
- Opening the file in an application
- Copying the file
- Listing the file's contents with `cat`, `less`, etc.
- Some operations like `ls` may update atime

**Note:** On modern Linux systems with `noatime` or `relatime` mount options, atime may not be updated with every access.

### Changed Time (ctime)

The last time the file's **metadata** (inode) was changed. This includes:
- Changes to permissions, ownership, or link count
- The file being modified (mtime change also triggers ctime change)
- The file being moved or renamed

**Key insight:** ctime cannot be set directly by the user (unlike mtime and atime). It is always updated by the kernel when the inode changes. This makes ctime particularly valuable in forensics.

### Birth/Created Time (crtime)

Available on NTFS, HFS+, ext4 (with `crtime` feature), and APFS. This is the time the file was first created on the filesystem.

- On NTFS:称为 `$STANDARD_INFORMATION` creation time
- On ext4: stored in the inode as `crtime`
- On FAT: not available (FAT only has a "create" date, not a full timestamp)

### Timestamps and Forensics

| Timestamp | Can Be Altered By User? | Forensic Value |
|-----------|------------------------|----------------|
| mtime | Yes (touch command) | Useful but can be manipulated |
| atime | Yes (touch command) | Useful but can be manipulated |
| ctime | No (only kernel) | **High** — reliable indicator |
| crtime | Varies by OS | **High** — creation time |

---

## The Body File Format

The body file is the input format for `mactime`. It is a pipe-delimited (`|`) text file with the following fields:

### Format Specification

```
MD5|inode|mode|uid|gid|size|atime|mtime|ctime|crtime|filename
```

### Field Definitions

| Field | Description | Example |
|-------|-------------|---------|
| MD5 | MD5 hash of the file content (or `-` if not calculated) | `d41d8cd98f00b204e9800998ecf8427e` |
| inode | Inode number (or NTFS MFT entry number) | `12345` |
| mode | File type and permissions (in octal or TSK notation) | `r/rrwxr-xr-x` |
| uid | User ID of the owner | `0` |
| gid | Group ID of the owner | `0` |
| size | File size in bytes | `4096` |
| atime | Access time (Unix epoch) | `1704067200` |
| mtime | Modified time (Unix epoch) | `1704067200` |
| ctime | Changed time (Unix epoch) | `1704067200` |
| crtime | Birth/created time (Unix epoch) | `1704067200` |
| filename | Full path of the file | `/home/user/document.txt` |

### Example Body File

```
0|12|r/rrwxr-xr-x|0|0|4096|1704067200|1704067200|1704067200|1704067200|/home/user
0|13|r/rrw-r--r--|1000|1000|512|1704067300|1704067300|1704067300|1704067200|/home/user/document.txt
0|14|r/rrw-r--r--|1000|1000|1024|1704067400|1704067400|1704067400|1704067200|/home/user/notes.txt
-|15|r/rrw-------|1000|1000|256|1704067500|1704067500|1704067500|0|/home/user/deleted_file.dat
```

### Mode Field Values

| TSK Mode | Meaning |
|----------|---------|
| r/rrwxrwxrwx | Regular file with permissions |
| d/drwxrwxrwx | Directory |
| l/lrwxrwxrwx | Symbolic link |
| c/drwxrwxrwx | Character device |
| b/drwxrwxrwx | Block device |
| p/drwxrwxrwx | Named pipe (FIFO) |
| s/drwxrwxrwx | Socket |
| `----------` | Deleted/unallocated entry |

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install sleuthkit
```

### Verify Installation

```bash
mactime -V
```

---

## Command Syntax

```
mactime [options] [date_range]
```

### Basic Usage

```bash
# Read body file from stdin
fls -r -m / disk.img | mactime -

# Read body file from a specific file
mactime -b body.txt

# Display full timeline
mactime -b body.txt
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-b body_file` | Path to the body file (required unless using stdin) |
| `-d` | Output in comma-delimited (CSV) format |
| `-h` | Display a header with session information |
| `-y` | Display dates in ISO 8601 format |
| `-m` | Display months as numbers instead of names (does not work with `-y`) |
| `-z TIME_ZONE` | Specify the timezone of the original data |
| `-g group_file` | Path to a group name file (maps GIDs to names) |
| `-p password_file` | Path to a password file (maps UIDs to usernames) |
| `-i day` | Create a daily summary index file |
| `-i hour` | Create an hourly summary index file |
| `-i file` | Use specified index file |
| `-V` | Print version information |

### The `-d` (CSV) Flag

The CSV output is extremely useful for importing timelines into spreadsheet applications:

```bash
mactime -b body.txt -d > timeline.csv
```

### The `-y` (ISO 8601) Flag

ISO 8601 format provides unambiguous date/time representation:

```bash
# Without -y: Jan 15 10:30:00 2024
# With -y:    2024-01-15T10:30:00
mactime -b body.txt -y
```

### The `-h` (Header) Flag

Adds a header line with information about the timeline session:

```bash
mactime -b body.txt -h
```

### The `-z` (Timezone) Flag

Adjust the output timezone:

```bash
# Convert to Eastern Time
mactime -b body.txt -z EST5EDT

# Convert to UTC
mactime -b body.txt -z GMT
```

---

## Generating Body Files

### Using `fls` (File Listing)

```bash
# Generate body file from an NTFS partition
fls -f ntfs -o 63 -r -m / disk.img > body.txt

# Generate body file from an ext4 partition
fls -f ext4 -o 2048 -r -m / disk.img > body.txt

# Include deleted files (unallocated entries)
fls -f ntfs -o 63 -r -d -m / disk.img > body_with_deleted.txt
```

### Using `tsk_gettimes`

`tsk_gettimes` generates a body file from the entire image:

```bash
# Generate body file from entire image
tsk_gettimes disk.img > body.txt

# With MD5 hashing (slower but more complete)
tsk_gettimes -m disk.img > body.txt

# From a specific partition
tsk_gettimes -f ntfs -o 63 disk.img > body.txt
```

### Combining Multiple Body Files

```bash
# Merge body files from multiple partitions
cat body_p1.txt body_p2.txt body_p3.txt > combined_body.txt
mactime -b combined_body.txt

# Or merge during generation
(fls -r -m / -f ntfs -o 63 disk.img;
 fls -r -m / -f ext4 -o 1048576 disk.img) > body.txt
```

### Using Body File with Other Tools

The body file format is also used by tools like:
- **Plaso/log2timeline** — timeline analysis framework
- **Log2timeline** — creates body files from various sources
- **SIFT Workstation** — includes body file utilities

---

## Creating Timelines

### Full Timeline

```bash
# Display all file system events sorted by time
mactime -b body.txt
```

### Filtered Timeline (Date Range)

```bash
# Only show events from January 2024
mactime -b body.txt 2024-01-01..2024-01-31

# Events on a specific day
mactime -b body.txt 2024-01-15

# Events in a specific time window
mactime -b body.txt 2024-01-15T08:00:00..2024-01-15T18:00:00

# Events from a specific hour
mactime -b body.txt 2024-01-15T10:00..2024-01-15T11:00
```

### Timeline for a Specific User

```bash
# Use the password file to map UIDs to usernames
mactime -b body.txt -p /etc/passwd

# Or filter the body file first
grep "|1000|" body.txt | mactime -

# For root activity (UID 0)
grep "|0|" body.txt | mactime -
```

### Timeline with Group Information

```bash
# Use the group file to map GIDs to group names
mactime -b body.txt -g /etc/group
```

---

## Date Filtering

### Supported Date Formats

```bash
# Full date
mactime -b body.txt 2024-01-15

# Date range
mactime -b body.txt 2024-01-01..2024-01-31

# Date with time (ISO 8601)
mactime -b body.txt 2024-01-15T10:00:00

# Range with times
mactime -b body.txt 2024-01-15T08:00:00..2024-01-15T17:00:00

# Open-ended ranges
mactime -b body.txt 2024-01-15..
mactime -b body.txt ..2024-01-15
```

### Filtering for Specific Time Windows

This is useful when correlating file system activity with known events:

```bash
# During business hours
mactime -b body.txt 2024-01-15T09:00:00..2024-01-15T17:00:00

# After hours (potential unauthorized access)
mactime -b body.txt 2024-01-15T18:00:00..2024-01-16T06:00:00

# Weekend activity
mactime -b body.txt 2024-01-13..2024-01-14
```

---

## Output Formats

### Default (Human-Readable) Format

```
Date/Time               Source           Destination/Description
2024-01-15 10:30:00 EST 1000  r/rrwxr-xr-x 4096  /home/user
2024-01-15 10:30:15 EST 1000  r/rrw-r--r-- 512   /home/user/document.txt
2024-01-15 10:31:00 EST 1000  r/rrw-r--r-- 1024  /home/user/notes.txt
2024-01-15 10:32:00 EST 1000  r/rrw------- 256   /home/user/deleted.dat
```

### CSV Format (`-d`)

```csv
Date/Time,Source Type,Source Ref,Type,User,Group,Size,Mode,File Path
2024-01-15 10:30:00,---------,1000,FILE,root,root,4096,r/rrwxr-xr-x,/home/user
2024-01-15 10:30:15,---------,1000,FILE,user,user,512,r/rrw-r--r--,/home/user/document.txt
```

The CSV format can be imported into:
- Microsoft Excel
- LibreOffice Calc
- Google Sheets
- ELK Stack (Elasticsearch)
- SIEM tools

### ISO 8601 Format (`-y`)

```
2024-01-15T10:30:00-05:00  ...
2024-01-15T10:30:15-05:00  ...
```

---

## Working with Time Zones

### Understanding Time Zone Issues

Disk images often come from systems in different time zones. The timestamps in the body file are stored as Unix epoch values (seconds since 1970-01-01 UTC), but `mactime` needs to know the **original timezone** of the system to display dates correctly.

### Setting the Original Timezone

```bash
# Original system was in Eastern Time
mactime -b body.txt -z EST5EDT

# Original system was in Pacific Time
mactime -b body.txt -z PST8PDT

# Original system was in UTC
mactime -b body.txt -z GMT

# Original system was in Central European Time
mactime -b body.txt -z CET-1CEST
```

### Common Timezone Strings

| Timezone | String |
|----------|--------|
| UTC/GMT | GMT |
| US Eastern | EST5EDT |
| US Central | CST6CDT |
| US Mountain | MST7MDT |
| US Pacific | PST8PDT |
| UK/Ireland | GMT0BST |
| Central Europe | CET-1CEST |
| Japan | JST-9 |

### How to Determine the Original Timezone

1. **Check the system registry** (Windows): `HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`
2. **Check `/etc/localtime`** (Linux): Read the symlink target
3. **Check `/etc/timezone`** (Linux): Contains the timezone string
4. **Check browser history**: Look at timezone-sensitive web requests
5. **Correlate with other evidence**: Compare timestamps with known events

---

## Practical Lab Exercises

### Lab 1: Basic Timeline Generation

```bash
# Create a test disk image
dd if=/dev/zero of=test.img bs=1M count=100
mkfs.ext4 test.img

# Mount and create some files
mkdir /tmp/mnt
sudo mount test.img /tmp/mnt
echo "Created at $(date)" > /tmp/mnt/file1.txt
sleep 2
echo "Modified at $(date)" > /tmp/mnt/file1.txt
sleep 2
touch /tmp/mnt/file2.txt
sleep 2
rm /tmp/mnt/file2.txt
sudo umount /tmp/mnt

# Generate body file
fls -r -m / test.img > body.txt

# View the timeline
mactime -b body.txt
```

### Lab 2: CSV Export and Spreadsheet Analysis

```bash
# Generate CSV timeline
mactime -b body.txt -d > timeline.csv

# Import into LibreOffice or Excel
libreoffice --calc timeline.csv
```

### Lab 3: Date Range Filtering

```bash
# Create files at different times
# (Assume disk was created on 2024-01-15)

# Full timeline
mactime -b body.txt

# Only morning activity
mactime -b body.txt 2024-01-15T00:00..2024-01-15T12:00

# Only afternoon activity
mactime -b body.txt 2024-01-15T12:00..2024-01-15T23:59
```

### Lab 4: User-Specific Timeline

```bash
# Create body file with UID/GID information
fls -r -m / test.img > body.txt

# Filter for specific user (UID 1000)
grep "|1000|" body.txt | mactime -

# Filter for root (UID 0)
grep "|0|" body.txt | mactime -
```

### Lab 5: Timezone Adjustment

```bash
# Timeline in UTC
mactime -b body.txt -z GMT

# Timeline in Eastern Time
mactime -b body.txt -z EST5EDT

# Compare the two
mactime -b body.txt -z GMT > utc_timeline.txt
mactime -b body.txt -z EST5EDT > eastern_timeline.txt
diff utc_timeline.txt eastern_timeline.txt
```

---

## Advanced Timeline Analysis

### Correlating with External Logs

```bash
# Generate CSV timeline
mactime -b body.txt -d > filesystem_timeline.csv

# Combine with:
# - Windows Event Logs
# - Browser history
# - Registry timestamps
# - Network connection logs
# - USB device history

# Use tools like Plaso/log2timeline for multi-source timelines
```

### Finding Anomalous Activity

```bash
# Look for files modified at unusual hours (e.g., 2-4 AM)
mactime -b body.txt 2024-01-15T02:00..2024-01-15T04:00

# Look for mass file deletions
mactime -b body.txt | grep "----------"

# Look for executable files created recently
grep "\.exe" body.txt | mactime -
```

### Detecting Timestamp Manipulation

```bash
# Compare mtime and ctime for discrepancies
# If mtime is older than ctime, the file may have been modified
# If mtime is much newer than crtime, check for touch usage

# Look for files with all identical timestamps (suspicious)
mactime -b body.txt -d | awk -F',' '{print $2}' | sort | uniq -c | sort -rn
```

### Timeline Visualization

```bash
# Generate data for visualization tools
mactime -b body.txt -d > timeline.csv

# Use with timeline visualization tools:
# - Plaso/log2timeline + Timesketch
# - Elasticsearch + Kibana
# - Apache Spot
# - Custom Python scripts with matplotlib
```

---

## Integration with Other Tools

### With `fls`

```bash
# Standard workflow: fls → body file → mactime
fls -r -m / disk.img > body.txt
mactime -b body.txt
```

### With `tsk_gettimes`

```bash
# Alternative: tsk_gettimes generates body file directly
tsk_gettimes disk.img > body.txt
mactime -b body.txt
```

### With `icat`

```bash
# After identifying suspicious files in timeline
mactime -b body.txt -d > timeline.csv
# Review timeline, note inode numbers
icat -f ntfs -o 63 disk.img [inode] > suspicious_file.exe
```

### With `ils`

```bash
# Find deleted files, then generate timeline of deleted activity
ils -e -m -f ntfs -o 63 disk.img > deleted_body.txt
mactime -b deleted_body.txt
```

### With Plaso/log2timeline

```bash
# Plaso can ingest body files and combine with other sources
log2timeline.py --storage-file timeline.plaso body.txt
psort.py -o L2tcsv timeline.plaso > full_timeline.csv
```

---

## Common Forensic Scenarios

### Scenario 1: Data Exfiltration

```bash
# Look for large files created or modified around the incident time
mactime -b body.txt 2024-01-15T08:00..2024-01-15T18:00 -d > workday.csv
# Sort by size in spreadsheet to find large files
```

### Scenario 2: Ransomware Attack

```bash
# Look for mass file modifications (all files changed within minutes)
mactime -b body.txt 2024-01-15T14:00..2024-01-15T15:00

# Look for the ransomware executable creation
mactime -b body.txt | grep "\.exe"
```

### Scenario 3: Insider Threat

```bash
# Look for after-hours activity
mactime -b body.txt 2024-01-15T18:00..2024-01-16T06:00

# Look for USB activity (files with removable media paths)
grep -i "media\|usb\|mnt" body.txt | mactime -
```

### Scenario 4: Malware Analysis

```bash
# Look for files created by suspicious process
# (correlate with process timeline from event logs)
mactime -b body.txt 2024-01-15T10:00..2024-01-15T10:30

# Look for executables in temp directories
grep -i "/tmp\|/temp\|appdata" body.txt | mactime -
```

---

## Troubleshooting

### No Output

```bash
# Check if body file has content
wc -l body.txt

# Check if timestamps are valid
head -5 body.txt

# Ensure mactime is reading the file
cat body.txt | mactime -b -
```

### Dates Look Wrong

```bash
# Check the timezone
mactime -b body.txt -z GMT    # Try UTC
mactime -b body.txt -z EST5EDT # Try Eastern

# Check if timestamps are Unix epoch
# (should be 10-digit numbers)
head -1 body.txt | cut -d'|' -f7-10
```

### "Cannot open body file"

```bash
# Check file permissions
ls -la body.txt

# Check if file is empty
cat body.txt

# Try reading from stdin
cat body.txt | mactime -
```

### Sorting Issues

```bash
# Ensure timestamps are numeric (not text)
# Body file should have Unix epoch timestamps, not text dates
```

---

## Best Practices

1. **Always use `-d` for CSV output** — import into spreadsheets for easier analysis
2. **Record the timezone** of the original system before generating the timeline
3. **Use `-y` for ISO 8601 dates** — eliminates AM/PM ambiguity
4. **Combine body files** from all partitions for a complete picture
5. **Document your date filters** — note which time range you analyzed and why
6. **Correlate with external evidence** — timestamps from files alone don't tell the full story
7. **Check ctime vs mtime** — discrepancies may indicate timestamp manipulation
8. **Include deleted files** — use `fls -d` to capture unallocated entries
9. **Save raw timeline** — always keep the unfiltered timeline for later reference
10. **Use Plaso for production cases** — `mactime` is great for quick analysis; Plaso is better for complex multi-source timelines

---

## Summary

`mactime` is an essential tool for forensic timeline analysis. Key capabilities:

- Creates chronological timelines from MAC timestamps
- Supports date filtering for focused analysis
- Outputs CSV for spreadsheet analysis
- Works with body files from `fls` and `tsk_gettimes`
- Handles time zone adjustments
- Supports UID/GID mapping with password and group files

The timeline is often the most powerful artifact in a forensic investigation — it shows you the story of what happened on the system, in the order it happened.

### Quick Reference

```bash
# Basic timeline
mactime -b body.txt

# CSV output
mactime -b body.txt -d > timeline.csv

# Date filtered
mactime -b body.txt 2024-01-15..2024-01-31

# With timezone
mactime -b body.txt -z EST5EDT

# ISO 8601 dates
mactime -b body.txt -y

# With user mapping
mactime -b body.txt -p /etc/passwd

# Generate body file and timeline in one command
fls -r -m / disk.img | mactime -
```

---

## References

- The Sleuth Kit Documentation: https://www.sleuthkit.org/sleuthkit/man/
- Brian Carrier, "File System Forensic Analysis," Addison-Wesley, 2005.
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response.
- Plaso Project: https://github.com/log2timeline/plaso
- Timesketch: https://timesketch.org/
