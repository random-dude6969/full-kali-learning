---
tool_name: sfuzz
display_name: sfuzz - Simple Fuzzer
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: fuzzing
install_command: "sudo apt install sfuzz"
version: "0.7.0"
github_repo: "https://gitlab.com/kalilinux/packages/sfuzz"
kali_tools_page: "https://www.kali.org/tools/sfuzz/"
difficulty_level: intermediate
requires_root: false
gui_available: false
tags: [fuzzing, blackbox-testing, protocol-fuzzing, tcp-fuzzing, udp-fuzzing]
related_tools: [spike, bed, afl-fuzz, wfuzz]
---

# sfuzz - Simple Fuzzer

## 1. Introduction & Overview

sfuzz (Simple Fuzzer) is a black-box protocol fuzzing tool designed for testing network services. It uses configuration files to define protocol templates and applies literal and sequence-based mutations to fuzz target services over TCP, UDP, or stdout.

**Created:** ~2005 by Aaron Conole
**License:** GPL v2
**Homepage:** http://aconole.brad-x.com/programs/sfuzz.html

### What It Does

sfuzz reads a configuration file that defines protocol commands with literal strings and variable sequences. It then systematically replaces protocol fields with fuzzed values (overflows, format strings, edge-case numbers) and sends them to the target, monitoring for crashes or unusual responses.

### When to Use

- **Protocol fuzzing**: HTTP, custom TCP/UDP services
- **Network service testing**: Quick vulnerability scanning of server applications
- **Black-box testing**: No source code or instrumentation required
- **Template-based fuzzing**: Write once, reuse across similar services

### When NOT to Use

- You need coverage-guided feedback (use AFL++)
- Target is a local file parser (not network-facing)
- Complex protocol state machines requiring session persistence (sfuzz doesn't track state)

### Legal & Ethical

sfuzz is a legitimate security testing tool. Use it only on systems you own or have authorization to test. Malformed packets may crash services.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Config file** | Template defining protocol commands and fuzz points |
| **Literal** | Fixed string values in the protocol |
| **Sequence** | Fuzzed values injected at specific positions |
| **Symbol** | User-defined variable substituted during fuzzing |
| **Req_del** | Delay between requests (in ms) |
| **Mseq_len** | Maximum sequence length |

## 2. Installation & Setup

### System Requirements

- Linux (x86_64)
- ~191 KB disk space
- Network access to target

### Installation

```bash
sudo apt install sfuzz
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  sfuzz
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 84.2 kB of archives.
After this operation, 191 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 sfuzz 0.7.0-1kali2+b1 [84.2 kB]
Fetched 84.2 kB in 0s (421 kB/s)
Selecting previously unselected package sfuzz.
Preparing to unpack .../sfuzz_0.7.0-1kali2+b1_amd64.deb ...
Unpacking sfuzz (0.7.0-1kali2+b1) ...
Setting up sfuzz (0.7.0-1kali2+b1) ...
```

### Sample Configuration Files

Located at `/usr/share/sfuzz/sfuzz-sample/`:

- `basic.http` — HTTP protocol template
- `basic.ftp` — FTP protocol template
- Various others

## 3. Basic Usage

### First Run — Fuzz an HTTP Service

```bash
sfuzz -S 192.168.1.50 -p 8080 -T -f /usr/share/sfuzz/sfuzz-sample/basic.http
```

**Explanation:** Fuzzes an HTTP service on 192.168.1.50:8080 using the built-in HTTP template over TCP.

**Example output:**
```
[12:53:47] dumping options:
    filename: </usr/share/sfuzz/sfuzz-sample/basic.http>
    state:    <8>
    lineno:   <56>
    literals:  [74]
    sequences: [34]
    symbols: [0]
    req_del:  <200>
    mseq_len: <10024>
    plugin: <none>
    s_syms: <0>
    literal[1] = [AREALLYBADSTRING]
```

### UDP Fuzzing

```bash
sfuzz -S 192.168.1.100 -p 53 -U -f /usr/share/sfuzz/sfuzz-sample/basic.dns
```

**Explanation:** Fuzzes a DNS service over UDP.

### Silent Mode

```bash
sfuzz -S 192.168.1.50 -p 8080 -T -q -f /usr/share/sfuzz/sfuzz-sample/basic.http
```

**Explanation:** Runs with minimal output, useful for scripting or long sessions.

### Flags Reference

| Flag | Description |
|------|-------------|
| `-S TARGET` | Remote host IP or hostname |
| `-p PORT` | Target port number |
| `-T` | TCP mode (default) |
| `-U` | UDP mode |
| `-O` | Output to stdout only |
| `-f FILE` | Configuration file to use |
| `-v` | Verbose output |
| `-q` | Quiet/silent mode |
| `-X` | Print output in hex |
| `-t SECONDS` | Socket read timeout |
| `-b NUM` | Begin fuzzing at specified test number |
| `-e` | End testing on first failure |
| `-R` | Don't close connections (leak them) |
| `-L FILE` | Log file path |
| `-n` | Create new logfile after each fuzz |
| `-r` | Trim trailing newline from input |
| `-D X=Y` | Define a symbol and value |
| `-l` | Literal fuzzing only |
| `-s` | Sequence fuzzing only |

## 4. Intermediate Usage

### Creating Custom Config Files

```bash
cat > custom_http.fuzz << 'EOF'
protocol: TCP
port: 80

req_del: 200
mseq_len: 10024

literal: GET
literal: HTTP/1.1
literal: Host:
literal: User-Agent:
literal: Content-Length:
literal: \r\n
sequence: 50       # Will be replaced with fuzzed values
literal: /
sequence: 1000
literal: \r\n
EOF
```

**Explanation:** Defines a custom HTTP GET request template with fuzz injection points.

### Targeted Field Fuzzing

```bash
sfuzz -S 192.168.1.50 -p 8080 -T -f custom_http.fuzz -l
```

**Explanation:** Fuzzes only literal values, skipping sequence mutations.

### Using Symbols for Dynamic Values

```bash
sfuzz -S 192.168.1.50 -p 8080 -T -f custom.fuzz -D target=192.168.1.50
```

**Explanation:** Substitutes the `$target` symbol in the config file with the specified value.

### Logging Results

```bash
sfuzz -S 192.168.1.50 -p 8080 -T -f basic.http -L sfuzz_results.log -v
```

**Explanation:** Logs all sent/received data to a file for later analysis.

### Hex Output for Protocol Analysis

```bash
sfuzz -S 192.168.1.50 -p 8080 -T -f basic.http -X
```

**Explanation:** Prints output in hexadecimal format, useful for comparing protocol deviations.

## 5. Advanced Usage

### Protocol Template Engineering

sfuzz config files support a mini-language for protocol definition:

```
# Comments start with #
protocol: TCP
port: 80
req_del: 200        # Delay between requests (ms)
mseq_len: 10024     # Max fuzzed sequence length

# Literals are sent verbatim
literal: GET

# Sequences define fuzz injection points
# The number specifies the maximum value
sequence: 100

# Symbols are replaced at runtime
literal: Host:
symbol: $target
literal: \r\n
```

### Fuzzing with Multiple Configs

```bash
#!/bin/bash
TARGET=192.168.1.50
for config in /usr/share/sfuzz/sfuzz-sample/*; do
    echo "=== Fuzzing with $config ==="
    sfuzz -S $TARGET -p 8080 -T -f "$config" -L "log_$(basename $config).log"
done
```

**Explanation:** Automatically tests a target with all available protocol templates.

### Parallel Fuzzing

```bash
# Run multiple sfuzz instances with different configs
sfuzz -S 192.168.1.50 -p 80 -T -f basic.http -L http.log &
sfuzz -S 192.168.1.50 -p 21 -T -f basic.ftp -L ftp.log &
sfuzz -S 192.168.1.50 -p 25 -T -f basic.smtp -L smtp.log &
wait
```

**Explanation:** Tests multiple services simultaneously on the same target.

## 6. Real-World Scenarios

### Scenario 1: Embedded Device Testing

**Target:** Router web interface on port 80

```bash
sfuzz -S 192.168.1.1 -p 80 -T -f /usr/share/sfuzz/sfuzz-sample/basic.http -L router_fuzz.log -v
```

### Scenario 2: Custom IoT Protocol

**Target:** Proprietary protocol on port 5000

```bash
cat > iot_custom.fuzz << 'EOF'
protocol: TCP
port: 5000
req_del: 500
literal: CONNECT
sequence: 256
literal: AUTH
sequence: 1024
literal: \r\n
EOF

sfuzz -S 192.168.1.200 -p 5000 -T -f iot_custom.fuzz -L iot_fuzz.log
```

### Scenario 3: VoIP SIP Fuzzing

**Target:** SIP proxy on port 5060

```bash
sfuzz -S 192.168.1.100 -p 5060 -U -f sip_template.fuzz -v
```

## 7. Detection & Defense

### How Defenders Detect

- **Pattern detection**: sfuzz sends repeated, malformed protocol messages
- **Connection rate monitoring**: High connection frequency from single source
- **Error logging**: Services generate unusual error patterns

### Mitigation

- Rate limit connections per source IP
- Deploy protocol-aware firewalls
- Monitor service health and auto-restart crashed services
- Use input validation libraries

### Blue Team Response

```bash
# Monitor for sfuzz-like activity
tcpdump -i eth0 port 8080 -w fuzz_capture.pcap
# Analyze captured traffic for fuzzing patterns
```

## 8. Troubleshooting

### Common Errors

**Connection timeout:**
```
[!] No response from target
```
**Fix:** Check target is reachable: `ping TARGET` and `nc -zv TARGET PORT`.

**Empty output:**
Verify the config file exists and is well-formed. Check path: `ls /usr/share/sfuzz/sfuzz-sample/`.

**Service crashes immediately:**
sfuzz may be sending overly large payloads. Reduce `mseq_len` in config.

### Performance Optimization

- Use `-t` to set appropriate socket timeouts
- Use `-R` for connection-oriented protocols that don't need fresh connections each time
- Use `-b` to resume from a specific test number after crashes

### FAQ

**Q: sfuzz vs spike?**
A: Both are protocol fuzzers. Spike is more feature-rich with a full scripting API. sfuzz is simpler and more template-driven. Start with sfuzz for quick tests, graduate to spike for complex protocols.

**Q: Can sfuzz fuzz TLS connections?**
A: Not natively. You'd need to terminate TLS (e.g., with socat or stunnel) and fuzz the plaintext backend.

---

## Resources

- **Kali Tools Page**: https://www.kali.org/tools/sfuzz/
- **Homepage**: http://aconole.brad-x.com/programs/sfuzz.html
- **Sample Configs**: `/usr/share/sfuzz/sfuzz-sample/`
- **Spike (alternative)**: https://github.com/spikefuzz/spike
