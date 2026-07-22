# unhide Cheatsheet — Detect Hidden Processes

## Quick Commands

```bash
# Process detection
unhide [options] test_list

# Port detection
unhide-tcp [options]
```

## Installation

```bash
sudo apt install unhide
```

## Most Common Commands

```bash
# Basic process scan
sudo unhide proc sys

# Quick scan (faster)
sudo unhide quick

# Comprehensive scan
sudo unhide -m -d proc procfs sys brute reverse

# Port detection
sudo unhide-tcp

# Verbose output
sudo unhide -v proc sys
```

## Options Reference

### unhide

| Option | Description |
|--------|-------------|
| `-V` | Version |
| `-v` | Verbose |
| `-h` | Help |
| `-m` | More checks |
| `-r` | Alternate sysinfo test |
| `-f` | Log to file |
| `-d` | Double check brute test |

### unhide-tcp

| Option | Description |
|--------|-------------|
| `-V` | Version |
| `-v` | Verbose |
| `-h` | Help |
| `-f` | Show fuser output |
| `-l` | Show lsof output |
| `-o` | Log to file |
| `-s` | Quick server mode |
| `-n` | Use netstat instead of ss |

## Tests

| Test | Description |
|------|-------------|
| `proc` | Compare /proc vs /bin/ps |
| `procfs` | Walk procfs directly |
| `sys` | Compare ps with syscalls |
| `brute` | PID bruteforcing |
| `reverse` | Verify all threads |
| `quick` | Fast combined test |
| `procall` | proc + procfs combined |

## Detection Techniques

| Technique | Method | Speed |
|-----------|--------|-------|
| proc | /proc vs ps | Fast |
| procfs | Walk procfs | Medium |
| sys | Syscall scanning | Medium |
| brute | Try all PIDs | Slow |
| reverse | Verify ps output | Fast |
| quick | Combined | Fast |

## Output Indicators

| Output | Meaning |
|--------|---------|
| `No hidden processes` | Clean system |
| `Hidden process found` | Suspicious process detected |
| Exit code 0 | No hidden found |
| Exit code 1 | Hidden found |

## Workflow

```bash
# 1. Quick scan
sudo unhide quick

# 2. If suspicious, comprehensive scan
sudo unhide -m -d proc procfs sys brute reverse

# 3. Check ports
sudo unhide-tcp -v

# 4. Save results
sudo unhide proc sys > results.txt 2>&1
```

## Integration

```bash
# unhide + chkrootkit
sudo chkrootkit
sudo unhide proc sys
sudo unhide-tcp

# unhide + rkhunter
sudo rkhunter --check --sk
sudo unhide proc sys

# unhide + monitoring script
if sudo unhide proc sys 2>&1 | grep -q "Hidden"; then
    echo "ALERT: Hidden processes detected!"
fi
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Permission denied | Use `sudo` |
| False positives | Use multiple tests to confirm |
| Slow scan | Use `quick` test |
| Command not found | `sudo apt install unhide` |
| Port issues | Try `-n` (netstat) or `-s` (quick) |
