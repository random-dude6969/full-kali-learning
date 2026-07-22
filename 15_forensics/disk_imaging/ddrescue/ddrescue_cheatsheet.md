# ddrescue CHEATSHEET

## Quick Reference

```bash
# Basic image (healthy drive)
sudo ddrescue -d /dev/sda /evidence/image.dd /evidence/mapfile.log

# Fast first pass (no retries)
sudo ddrescue -d -n /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Retry bad sectors (3 passes)
sudo ddrescue -d -r3 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Unlimited retries with timeout
sudo ddrescue -d -r-1 -T30 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Image scratched optical disc
sudo ddrescue -n /dev/sr0 /evidence/dvd.iso /evidence/dvd_mapfile.log
sudo ddrescue -r3 /dev/sr0 /evidence/dvd.iso /evidence/dvd_mapfile.log

# Force overwrite
sudo ddrescue -f -d /dev/sda /evidence/image.dd /evidence/mapfile.log

# Wipe output before imaging
sudo ddrescue -w -d /dev/sda /evidence/image.dd /evidence/mapfile.log

# Generate mapfile only
sudo ddrescue -g -d /dev/sda /evidence/image.dd /evidence/mapfile.log

# View mapfile statistics
ddrescue --show-map /evidence/mapfile.log
```

## Parameters

| Parameter | Description |
|---|---|
| `-d` | Direct disc access (bypass OS cache) |
| `-r<n>` | Retry passes (-1 = unlimited) |
| `-b<n>` | Block size for retries (default: 512) |
| `-c<n>` | Cluster size (default: 128) |
| `-e<n>` | Error size for trimming (default: 256) |
| `-f` | Force overwrite |
| `-n` | No-scrape (skip scraping phase) |
| `-N` | No-trim (skip trimming phase) |
| `-T<n>` | Timeout per error area (seconds) |
| `-x<n>` | Max retry time per area (seconds) |
| `-w` | Wipe output before copy |
| `-W` | Wipe mapfile (start fresh) |
| `-Y` | Allocate output space upfront |
| `-g` | Generate mapfile only |
| `-G` | Mapfile only mode (assume output exists) |

## Recovery Phases

| Phase | Name | Description |
|---|---|---|
| 1 | Trimming | Fast scan to identify error boundaries |
| 2 | Scraping | Sector-by-sector read of bad areas |
| 3 | Retrying | Multiple retry strategies on failed areas |
| 4 | Splitting | Copy remaining fragments |

## Common Workflows

```bash
# Standard recovery (damaged drive)
# Step 1: Fast copy
sudo ddrescue -d -n /dev/sdb /evidence/image.dd /evidence/mapfile.log
# Step 2: Retry
sudo ddrescue -d -r3 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Aggressive recovery (severely damaged)
sudo ddrescue -d -r-1 -x300 -T10 /dev/sdb /evidence/image.dd /evidence/mapfile.log

# Image specific sector range
sudo ddrescue -i0 -s1000000 /dev/sdb /evidence/part.dd /evidence/part.log

# Resume interrupted recovery (just re-run same command)
sudo ddrescue -d /dev/sdb /evidence/image.dd /evidence/mapfile.log
```

## Cluster Size Tuning

| Cluster Size | Use Case |
|---|---|
| 16 | Severely damaged media |
| 64 | Moderately damaged |
| 128 | General purpose (default) |
| 512 | Light damage |
| 1024 | Healthy media |

## Post-Recovery

```bash
# Verify image
md5sum /evidence/image.dd
sha256sum /evidence/image.dd

# Check mapfile for remaining errors
ddrescue --show-map /evidence/mapfile.log

# Mount recovered image
sudo mount -o loop,ro /evidence/image.dd /mnt/recovered

# Carve files from image
foremost -i /evidence/image.dd -o /evidence/carved/
photorec /evidence/image.dd
```

## Tips

- Always use mapfile — enables resume after interruption
- Use `-d` (direct access) for failing drives
- Start with `-n` (no-scrape) for fast initial copy
- Mapfile must be on different filesystem than image
- Multiple passes improve recovery from damaged media
- ddrescue does not compute hashes — use md5sum/sha256sum afterward
