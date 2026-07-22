# ils Cheatsheet — List Inode Entries

## Quick Command

```bash
ils [options] image [inum[-end]]
```

## Installation

```bash
sudo apt install sleuthkit
```

## Most Common Commands

```bash
# List all inodes
ils -e -f ntfs -o 63 disk.img

# Only unallocated (deleted) inodes
ils -A -f ntfs -o 63 disk.img

# Deleted files with non-zero size (most interesting)
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0'

# Orphan inodes (no filename)
ils -p -f ntfs -o 63 disk.img

# Mactime body file format
ils -m -f ntfs -o 63 disk.img

# Inode range query
ils -f ntfs -o 63 disk.img 32-64
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-e` | Display all inodes |
| `-a` | Allocated inodes only |
| `-A` | Unallocated inodes only |
| `-l` | Linked inodes only |
| `-L` | Unlinked inodes only |
| `-z` | Unused inodes only |
| `-Z` | Used inodes only |
| `-p` | Orphan inodes (no dir entry) |
| `-O` | Unallocated but still open (ext only) |
| `-m` | Output in mactime body format |
| `-f fstype` | Filesystem type |
| `-o OFFSET` | Offset to filesystem in sectors |
| `-s SECS` | Time skew in seconds |
| `-v` | Verbose output |
| `-V` | Print version |

## Output Columns

| Column | Meaning |
|--------|---------|
| Meta | Internal metadata address |
| Num | Inode number |
| Alloc | `a` = allocated, `u` = unallocated |
| Use | `u` = used, `z` = unused |
| Fmt | Format type (`fsl` = filesystem) |
| Fsi | File type (`DIR`, `---`, etc.) |
| Size | File size in bytes |
| uid/gid | Owner/group IDs |
| Mode | Unix permissions |
| crtime | Creation time |
| amtime | Access time |
| ctime | Changed time |
| dtime | Deleted time (`0000-00-00` = not deleted) |

## `ils` vs `fls` vs `istat`

| Tool | Scope | Key Use |
|------|-------|---------|
| `ils` | All inodes | Find deleted/orphan files |
| `fls` | Filenames only | Browse directory structure |
| `istat` | Single inode | Detailed metadata examination |

## Workflow

```bash
# 1. Find deleted files
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0'

# 2. Examine specific inode
istat -f ntfs -o 63 disk.img <inode>

# 3. Extract file
icat -f ntfs -o 63 disk.img <inode> > recovered.dat

# 4. Generate timeline
ils -m -f ntfs -o 63 disk.img | mactime -
```

## Inode States

| State | Meaning | Forensic Value |
|-------|---------|----------------|
| Allocated | Active file/directory | Normal analysis |
| Unallocated | Previously used, now free | **Deleted file recovery** |
| Linked | Has directory entry | Normal file |
| Unlinked | No dir entry, possibly open | **Active malware hiding** |
| Orphan | No filename reference | **Lost data recovery** |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No results | Check `-f` type and `-o` offset |
| Too many results | Filter with `-A` or `-a` |
| No orphan inodes | Normal on clean filesystems |
| Need different format | Use `-m` for mactime body format |
