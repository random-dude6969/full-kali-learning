# chkrootkit Cheatsheet — Check for Rootkits

## Quick Command

```bash
sudo chkrootkit [options] [tests]
```

## Installation

```bash
sudo apt install chkrootkit
```

## Most Common Commands

```bash
# Full system scan
sudo chkrootkit

# Quiet mode (only suspicious output)
sudo chkrootkit -q

# Expert mode (thorough)
sudo chkrootkit -x

# Specific tests
sudo chkrootkit suspscan netscan

# Save results
sudo chkrootkit > results.txt 2>&1

# Show available tests
chkrootkit -l
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-h` | Help |
| `-V` | Version |
| `-l` | List available tests |
| `-d` | Debug mode |
| `-q` | Quiet mode |
| `-x` | Expert mode |
| `-e 'FILES'` | Exclude files/dirs |
| `-r DIR` | Use DIR as root |
| `-p PATH` | Path for external commands |
| `-n` | Skip NFS mounts |

## Key Tests

| Test | Description |
|------|-------------|
| `suspscan` | Suspicious files |
| `bindshell` | Backdoor shells |
| `sniffer` | Network sniffers |
| `netscan` | Suspicious connections |
| `shellshock` | Shellshock vulnerability |
| `chklastlog` | lastlog anomalies |
| `chkwtmp` | wtmp anomalies |
| `ps` | Modified ps binary |
| `ls` | Modified ls binary |
| `netstat` | Modified netstat binary |
| `find` | Modified find binary |
| `lsof` | Modified lsof binary |

## Output Indicators

| Output | Meaning |
|--------|---------|
| `not found` | Binary not installed |
| `not infected` | Binary is clean |
| `INFECTED` | Suspicious modification |
| `unknown` | Cannot determine |

## Quick Workflow

```bash
# 1. Quick scan
sudo chkrootkit -q

# 2. If suspicious, run expert mode
sudo chkrootkit -x

# 3. Check specific tests
sudo chkrootkit suspscan netscan bindshell

# 4. Save for documentation
sudo chkrootkit -x > full_scan.txt 2>&1

# 5. Cross-reference with rkhunter
sudo rkhunter --check --sk
```

## Integration

```bash
# chkrootkit + rkhunter (comprehensive)
sudo chkrootkit > chk.txt 2>&1
sudo rkhunter --check --sk > rk.txt 2>&1

# chkrootkit + unhide (hidden processes)
sudo chkrootkit
sudo unhide proc
sudo unhide sys

# chkrootkit + ClamAV (malware)
sudo chkrootkit
sudo clamscan -r / --infected

# Automated daily scan
echo "0 2 * * * root /usr/sbin/chkrootkit -q > /var/log/chkrootkit.log 2>&1" | sudo tee /etc/cron.d/chkrootkit
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Permission denied | Use `sudo` |
| False positives | Check README.FALSE-POSITIVES |
| Slow scan | Run specific tests only |
| No output | Try `-d` for debug |
| Binary not found | `sudo apt install chkrootkit` |
