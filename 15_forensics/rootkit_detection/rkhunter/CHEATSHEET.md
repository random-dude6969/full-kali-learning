# rkhunter Cheatsheet — Rootkit Hunter

## Quick Command

```bash
rkhunter [options]
```

## Installation

```bash
sudo apt install rkhunter
```

## Most Common Commands

```bash
# Full system check
sudo rkhunter --check

# Skip keypress (for automation)
sudo rkhunter --check --sk

# Update database
sudo rkhunter --update

# Update file properties
sudo rkhunter --propupd

# Check for updates
sudo rkhunter --versioncheck

# Remove lock file
sudo rkhunter --unlock
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-c, --check` | Check the local system |
| `--update` | Update database |
| `--versioncheck` | Check for version updates |
| `--propupd` | Update file properties database |
| `--unlock` | Remove lock file |
| `--sk` | Skip keypress |
| `--cronjob` | Cron job mode |
| `--disable test` | Disable specific tests |
| `--enable test` | Enable specific tests |
| `--hash ALG` | Hash function (default SHA256) |
| `-l file` | Log file (default /var/log/rkhunter.log) |
| `--append-log` | Append to log |
| `--nocolors` | B&W output |
| `-q` | Quiet mode |

## Output Indicators

| Indicator | Meaning |
|-----------|---------|
| `[ OK ]` | Check passed |
| `[ WARNING ]` | Potential issue |
| `[ INFECTED ]` | Suspicious finding |
| `[ NOT FOUND ]` | Expected file missing |
| `[ SKIPPED ]` | Test skipped |

## Key Tests

| Test | Description |
|------|-------------|
| `suspscan` | Suspicious file scan |
| `hidden_procs` | Hidden processes |
| `deleted_files` | Recently deleted files |
| `group_files` | Group file changes |
| `network` | Network checks |
| `rootkits` | Known rootkit signatures |
| `ssh_config` | SSH configuration |
| `file_phishing` | Phishing files |

## Workflow

```bash
# 1. Update database
sudo rkhunter --update

# 2. Update file properties
sudo rkhunter --propupd

# 3. Run check
sudo rkhunter --check --sk

# 4. Review log
grep "WARNING\|INFECTED" /var/log/rkhunter.log

# 5. Save results
sudo rkhunter --check --sk -l /tmp/results.txt
```

## Integration

```bash
# rkhunter + chkrootkit (comprehensive)
sudo rkhunter --check --sk > rk.txt 2>&1
sudo chkrootkit > ck.txt 2>&1

# rkhunter + unhide
sudo rkhunter --check --sk
sudo unhide proc
sudo unhide sys

# rkhunter + AIDE (integrity)
sudo rkhunter --check --sk
sudo aide --check

# Automated daily scan
echo "0 2 * * * root /usr/bin/rkhunter --check --sk --append-log" | sudo tee /etc/cron.d/rkhunter
```

## Log Location

```
/var/log/rkhunter.log
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Lock file exists | `sudo rkhunter --unlock` |
| Database not updated | `sudo rkhunter --update` |
| False positives | Whitelist in /etc/rkhunter.conf |
| Command not found | `sudo apt install rkhunter` |
| Slow scan | Use `--disable suspscan` |
