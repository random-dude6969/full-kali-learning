# DC3DD CHEATSHEET

## Quick Reference

```bash
# Basic image with MD5 hash
sudo dc3dd if=/dev/sda of=/evidence/image.dd hash=md5 log=/evidence/image.log

# Image with multiple hashes
sudo dc3dd if=/dev/sda of=/evidence/image.dd hash=md5,sha1,sha256 log=/evidence/image.log

# Image with error tolerance
sudo dc3dd if=/dev/sdb of=/evidence/image.dd conv=noerror,sync hash=md5 log=/evidence/image.log

# Split image into 4GB chunks
sudo dc3dd if=/dev/sda of=/evidence/image_%f.dd ofsz=4294967296 hash=md5 log=/evidence/image.log

# Image specific partition (skip sectors)
sudo dc3dd if=/dev/sda of=/evidence/partition.dd skip=2048 count=1000000 hash=md5 log=/evidence/partition.log

# Search pattern while imaging
sudo dc3dd if=/dev/sda of=/evidence/image.dd hash=md5 pattern="50415353574f5244" log=/evidence/image.log

# Wipe with DoD 5220.22-M
sudo dc3dd if=/dev/zero of=/dev/sdb ofsz=1048576 hash=md5 wmode=dod522022m log=/evidence/wipe.log

# Verify image against source
dc3dd if=/dev/sda of=/evidence/image.dd rhlog=/evidence/src.md5 whlog=/evidence/dst.md5 log=/evidence/verify.log
```

## Parameters

| Parameter | Description |
|---|---|
| `if=` | Input file/device |
| `of=` | Output file/device |
| `bs=` | Block size (default: 512) |
| `count=` | Number of blocks to copy |
| `skip=` | Skip N blocks at input start |
| `seek=` | Skip N blocks at output start |
| `hash=` | Hash algorithm(s) |
| `log=` | Log file path |
| `pattern=` | Hex pattern to search |
| `conv=` | Conversion flags (noerror,sync) |
| `ofsz=` | Output file split size |
| `wmode=` | Wipe mode |
| `rhlog=` | Read hash log |
| `whlog=` | Write hash log |
| `hwlog=` | Hash window log |
| `hashwindow=` | Hash every N blocks |
| `trim=` | Trim zero blocks |

## Hash Algorithms

| Algorithm | Flag | Notes |
|---|---|---|
| MD5 | `hash=md5` | Fast, use for speed |
| SHA-1 | `hash=sha1` | Moderate security |
| SHA-256 | `hash=sha256` | Recommended |
| SHA-512 | `hash=sha512` | High security |
| Multiple | `hash=md5,sha1,sha256` | Comma-separated |

## Wipe Modes

| Mode | Description | Passes |
|---|---|---|
| `zero` | All zeros | 1 |
| `one` | All ones | 1 |
| `dod522022m` | DoD 5220.22-M | 3 |
| `dodshort` | DoD Short | 3 |
| `gutmann` | Gutmann method | 35 |
| `prng` | Pseudorandom | 1 |
| `bsi` | BSI/TL-03423 | 2 |
| `oneszero` | Ones then zeros | 2 |
| `custom` | User-defined | N |

## Common Patterns

```bash
# Hash only (no imaging)
dc3dd if=/dev/sda hash=md5 log=/dev/null

# Resume interrupted image
sudo dc3dd if=/dev/sda of=/evidence/image.dd skip=12345678 seek=12345678 hash=md5 log=/evidence/resume.log

# Quick verification
md5sum /evidence/image.dd
sha256sum /evidence/image.dd
```

## Error Handling

```bash
# Continue past errors, pad with zeros
sudo dc3dd if=/dev/sdb of=/evidence/image.dd conv=noerror,sync hash=md5 log=/evidence/image.log

# Count errors
grep -c "Input/output error" /evidence/image.log
```

## Automation

```bash
# Background imaging with nohup
nohup sudo dc3dd if=/dev/sda of=/evidence/image.dd hash=md5 log=/evidence/image.log &

# Monitor progress
tail -f /evidence/image.log

# Kill if needed
pkill -f dc3dd
```

## Forensic Workflow

1. Write-block the source device
2. Document chain of custody
3. Image with multiple hashes: `hash=md5,sha1,sha256`
4. Generate hash logs: `rhlog=` and `whlog=`
5. Verify image integrity
6. Create independent hash verification
7. Store evidence securely
8. Document all steps
