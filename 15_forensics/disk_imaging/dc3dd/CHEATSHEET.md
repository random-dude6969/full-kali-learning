# dc3dd Cheatsheet — Enhanced dd with Hashing

## Quick Command

```bash
dc3dd [options]
```

## Installation

```bash
sudo apt install dc3dd
```

## Most Common Commands

```bash
# Basic forensic image with SHA-256
sudo dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Multiple hash algorithms
sudo dc3dd if=/dev/sda of=/evidence/disk.img hash=md5 hash=sha256 log=/evidence/imaging.log

# Split output into 1GB files
sudo dc3dd if=/dev/sda of=/evidence/disk.img ofsz=1G hash=sha256 log=/evidence/imaging.log

# Verify image
dc3dd if=/evidence/disk.img hash=sha256

# Pattern wipe
sudo dc3dd if=/dev/zero of=/dev/sdb wipe=on
```

## Options Reference

| Option | Description |
|--------|-------------|
| `if=FILE` | Input file/device |
| `of=FILE` | Output file/device |
| `hof=FILE` | Output with hash verification |
| `ofsz=BYTES` | Max output file size |
| `hash=ALG` | Hash algorithm (md5, sha1, sha256, sha512) |
| `log=FILE` | Log I/O stats and hashes |
| `hlog=FILE` | Log hashes only |
| `mlog=FILE` | Machine-readable hash log |
| `rec=off` | Stop on read errors |
| `wipe=DEVICE` | Wipe device |
| `hwipe=DEVICE` | Wipe and verify |
| `pat=HEX` | Hex pattern for wipe |
| `cnt=N` | Read N sectors only |
| `iskip=N` | Skip N input sectors |
| `oskip=N` | Skip N output sectors |
| `app=on` | Append to output |
| `ssz=BYTES` | Sector size |
| `bufsz=BYTES` | Buffer size |
| `verb=on` | Verbose output |

## Unit Suffixes

| Suffix | Value |
|--------|-------|
| `c` | 1 byte |
| `w` | 2 bytes |
| `b` | 512 bytes |
| `kB` | 1000 bytes |
| `K` | 1024 bytes |
| `MB` | 1000² bytes |
| `M` | 1024² bytes |
| `GB` | 1000³ bytes |
| `G` | 1024³ bytes |

## Forensic Workflow

```bash
# 1. Identify source disk
lsblk

# 2. Write-protect source (hardware write blocker preferred)
sudo blockdev --setro /dev/sdb

# 3. Create image
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# 4. Verify image
dc3dd if=/evidence/disk.img hash=sha256

# 5. Document
echo "Source: /dev/sdb" > /evidence/custody.txt
echo "Image: disk.img" >> /evidence/custody.txt
echo "Hash: $(sha256sum /evidence/disk.img)" >> /evidence/custody.txt
```

## Integration

```bash
# dc3dd + Sleuth Kit
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log
mmls /evidence/disk.img
fls -r -m / /evidence/disk.img > body.txt

# dc3dd + Autopsy
sudo dc3dd if=/dev/sdb of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log
# Import into Autopsy

# dc3dd + Hash verification
sha256sum /evidence/disk.img
grep "sha256" /evidence/imaging.log
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No space left | Check `df -h`, use `ofsz=` for splitting |
| Permission denied | Use `sudo` |
| Hash mismatch | Retry with different `bs=` |
| Slow performance | Increase `bufsz=` |
| Read error | Check log, use `rec=off` to stop |
