# regripper Cheatsheet — Windows Registry Parser

## Quick Command

```bash
rip [options] -r <hive_file>
```

## Installation

```bash
sudo apt install regripper
```

## Most Common Commands

```bash
# Auto-analyze a hive
rip -r /evidence/SOFTWARE -a

# Run specific plugin
rip -r /evidence/NTUSER.DAT -p userassist

# Timeline output
rip -r /evidence/NTUSER.DAT -aT -s "WORKSTATION" -u "jsmith"

# List all plugins
rip -l

# Guess hive type
rip -r /evidence/file -g

# Check if hive is dirty
rip -r /evidence/SOFTWARE -d
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-r file` | Registry hive file |
| `-f profile` | Use profile (software, system, sam, etc.) |
| `-p plugin` | Run specific plugin |
| `-a` | Auto-run hive-specific plugins |
| `-aT` | Auto-run TLN plugins |
| `-d` | Check if hive is dirty |
| `-g` | Guess hive type |
| `-l` | List all plugins |
| `-c` | CSV plugin list |
| `-s name` | System name (TLN) |
| `-u user` | Username (TLN) |
| `-h` | Help |

## Registry Hive Files

| File | Location | Contents |
|------|----------|----------|
| SAM | `System32\config\SAM` | User accounts |
| SECURITY | `System32\config\SECURITY` | Security policies |
| SOFTWARE | `System32\config\SOFTWARE` | Installed software |
| SYSTEM | `System32\config\SYSTEM` | Hardware, boot |
| NTUSER.DAT | `%UserProfile%\NTUSER.DAT` | User settings |

## Profiles

| Profile | Use For |
|---------|---------|
| `software` | Installed software |
| `system` | System configuration |
| `sam` | User accounts |
| `security` | Security policies |
| `ntuser` | User activity |

## Key Plugins

| Plugin | Description |
|--------|-------------|
| `userassist` | Recently executed programs |
| `shimcache` | App compatibility cache |
| `amcache` | App execution history |
| `usb` | USB device history |
| `network` | Network config |
| `services` | Windows services |
| `run` | Startup programs |
| `uninstall` | Installed software |
| `typedurls` | Browser typed URLs |
| `mrulist` | Most recently used |

## Workflow

```bash
# 1. Identify hive type
rip -r /evidence/file -g

# 2. Auto-analyze
rip -r /evidence/SOFTWARE -a > software.txt

# 3. Specific analysis
rip -r /evidence/NTUSER.DAT -p userassist > userassist.txt

# 4. Timeline
rip -r /evidence/NTUSER.DAT -aT -s "PC01" -u "jsmith" > timeline.txt
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Cannot open hive | Check permissions, verify file type |
| Plugin not found | Use `rip -l` to list available plugins |
| Dirty hive | Process transaction logs first |
| No output | Try `-g` to verify hive type |
