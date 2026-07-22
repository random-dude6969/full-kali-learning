# TestDisk — Quick Reference Cheatsheet

## Essential Commands

```bash
# Start TestDisk on a disk device
sudo testdisk /dev/sda

# Start TestDisk on a partition
sudo testdisk /dev/sda1

# Start TestDisk on a disk image
sudo testdisk /path/to/image.raw

# Check version
testdisk --version

# Create disk image first (IMPORTANT)
sudo dd if=/dev/sda of=/tmp/image.raw bs=4M status=progress

# Backup partition table (MBR)
sudo sfdisk -d /dev/sda > /backup/mbr_backup.txt

# Restore partition table
sudo sfdisk /dev/sda < /backup/mbr_backup.txt

# Backup GPT header
sudo sgdisk --backup=/backup/gpt_backup.bin /dev/sda

# Restore GPT header
sudo sgdisk --load-backup=/backup/gpt_backup.bin /dev/sda
```

## Navigation Keys

| Key | Action |
|-----|--------|
| Arrow keys | Navigate menus |
| Enter | Select/Confirm |
| Right arrow | Next step / Proceed |
| Left arrow | Previous step |
| `Q` | Quit (safe exit) |

## Interactive Workflow

### Partition Recovery

```
1. sudo testdisk /dev/sda
2. Select disk → Enter
3. Select partition table type (Intel/EFI GPT) → Enter
4. Analyse → Quick Search
5. Review found partitions
6. [Deeper Search] if needed
7. Select lost partition → mark as [P]rimary
8. [Write] → [Yes] to save changes
9. Reboot
```

### Boot Sector Repair

```
1. sudo testdisk /dev/sda
2. Select disk → Select partition table type
3. Advanced → Select partition
4. Boot → Restore from backup
```

### File System Repair (NTFS/FAT/ext)

```
1. sudo testdisk /dev/sda
2. Advanced → Select damaged partition
3. Boot → Compare/Repair
```

## Partition Table Types

| Type | Use When |
|------|----------|
| Intel/PC | MBR disks, < 2TB |
| EFI GPT | Modern disks, > 2TB |
| Mac | Apple HFS+ |

## Partition Flags

| Flag | Meaning |
|------|---------|
| `P` | Primary partition |
| `*` | Bootable |
| `D` | Deleted |
| `L` | Logical |
| `E` | Extended |

## Common Recovery Scenarios

### Lost Partition After Reinstall
```bash
sudo dd if=/dev/sda of=/tmp/evidence.raw bs=4M status=progress
sudo testdisk /tmp/evidence.raw
# Quick Search → find deleted partition → Write
```

### Corrupted MBR
```bash
sudo testdisk /dev/sda
# Intel → Analyse → find backup → Write
```

### Corrupted GPT
```bash
sudo testdisk /dev/sda
# EFI GPT → Analyse → find backup header → Write
```

## Integration with Other Tools

```bash
# After recovery, verify with fdisk
sudo fdisk -l /dev/sda

# Mount and inspect recovered partition
sudo mount -o ro /dev/sda2 /mnt/recovered
ls -la /mnt/recovered

# Use PhotoRec for file-level recovery
sudo photorec /dev/sda1

# Use The Sleuth Kit for analysis
fls -r -d /dev/sda2
```

## Safety Rules

1. **Always work on images** — Never run on original evidence
2. **Backup first** — `sfdisk -d` or `sgdisk --backup` before any changes
3. **Hash evidence** — `md5sum` and `sha256sum` before and after
4. **Write blockers** — Use hardware/software write blockers
5. **Document everything** — Screenshot every step

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No partitions found | Try Deeper Search |
| Read error | Use `ddrescue` first |
| Hangs/crashes | Run on specific partition, not whole disk |
| Won't mount after recovery | `fsck` or `ntfsfix` the partition |
| Permission denied | Use `sudo` |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
