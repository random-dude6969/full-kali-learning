---
tool_name: spike
display_name: SPIKE - Network Protocol Fuzzer
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: fuzzing
install_command: "sudo apt install spike"
version: "4.x"
github_repo: "https://github.com/spikefuzz/spike"
kali_tools_page: "https://www.kali.org/tools/spike/"
difficulty_level: intermediate
requires_root: false
gui_available: false
tags: [fuzzing, protocol-fuzzing, network-protocol, spiike, dionysus]
related_tools: [sfuzz, bed, afl++, boofuzz]
---

# SPIKE - Network Protocol Fuzzer

## 1. Introduction & Overview

SPIKE is one of the original network protocol fuzzers, created by Dave Aitel (Immunity Inc.). It provides a C-based scripting framework for building protocol fuzzers, with pre-built templates for common protocols. SPIKE pioneered the concept of protocol-aware fuzzing and remains a foundational tool in vulnerability research.

**Created:** 2002 by Dave Aitel / Immunity Inc.
**License:** Custom (free for non-commercial use)

### What It Does

SPIKE uses "SPIKES" — C scripts that define protocol structures and mutation points. The fuzzer sends mutated protocol messages to a target and monitors for crashes. It includes pre-built SPIKES for HTTP, SMB, DNS, RDP, and many other protocols.

### When to Use

- **Protocol vulnerability research**: Discover buffer overflows and parsing bugs in network services
- **Custom protocol testing**: Write SPIKE scripts for proprietary protocols
- **Penetration testing**: Find exploitable vulnerabilities in discovered services
- **Protocol education**: Learn how protocol fuzzing works at a fundamental level

### When NOT to Use

- You need source-code-level fuzzing (use AFL++)
- Rapid, simple fuzzing (use sfuzz or bed instead)
- You prefer Python-based tools (consider boofuzz, the Python successor)

### Legal & Ethical

SPIKE is a professional security testing tool. Authorized use only. The tool's aggressive mutation strategies may cause service crashes and data corruption.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Spike script** | C program defining protocol structure and fuzz points |
| **s_string** | A regular string sent to the target |
| **s_binary** | Binary data that gets mutated with each iteration |
| **s_string_variable** | String that gets replaced with fuzz values |
| **s_xmit** | Sends the accumulated buffer to the target |
| **s_read** | Reads response from the target |

## 2. Installation & Setup

### System Requirements

- Linux (x86_64)
- GCC compiler for building SPIKE scripts
- Network access to target

### Installation

```bash
sudo apt install spike
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  spike
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 245 kB of archives.
After this operation, 1024 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 spike 4.x-0kali1 [245 kB]
Fetched 245 kB in 0s (1.2 MB/s)
Selecting previously unselected package spike.
Preparing to unpack .../spike_4.x-0kali1_amd64.deb ...
Unpacking spike (4.x-0kali1) ...
Setting up spike (4.x-0kali1) ...
```

### From Source (Alternative)

```bash
git clone https://github.com/spikefuzz/spike.git
cd spike
./build.sh
```

### Verification

```bash
ls /usr/share/spike/
```

**Expected output:**
```
webserver.spk  dns.spk  smb.spk  http.spk  ...
```

## 3. Basic Usage

### Using Pre-built SPIKE Scripts

```bash
generic_send_tcp 192.168.1.50 8080 /usr/share/spike/webserver.spk 0 0
```

**Explanation:** Sends HTTP fuzzing payloads to port 8080 using the webserver SPIKE script.

**Example output:**
```
Fuzzing with level=0 line=1
Fuzzing with level=0 line=2
...
Fuzzing with level=1 line=1
...
```

### SPIKE Script Anatomy

```c
// basic_http.spike
s_init_fuzzing();
s_set_variable("target_host", "192.168.1.50");

s_string("GET ");
s_string_variable("/");           // Fuzzed: long URIs, ../, etc.
s_string(" HTTP/1.1\r\n");
s_string("Host: ");
s_string_variable("192.168.1.50");  // Fuzzed: long hostnames
s_string("\r\n");
s_string("User-Agent: ");
s_string_variable("Mozilla/5.0");    // Fuzzed: long user-agents
s_string("\r\n");
s_string("\r\n");
s_xmit();
```

**Explanation:** Defines an HTTP GET request with four fuzz injection points (URI, Host, User-Agent).

### Flags Reference

| Command | Description |
|---------|-------------|
| `generic_send_tcp` | Send SPIKE against TCP target |
| `generic_send_udp` | Send SPIKE against UDP target |
| `generic_send_telnet` | Telnet-aware sending |
| `generic_chunked_send_tcp` | Chunked TCP sending |

| Argument | Description |
|----------|-------------|
| Host | Target IP |
| Port | Target port |
| Spike file | Path to .spk file |
| Variable | Which variable to fuzz (0 = first) |
| Limit | Maximum fuzz level |

## 4. Intermediate Usage

### Writing Custom SPIKE Scripts

```c
// custom_ftp.spike
s_init_fuzzing();

// FTP Banner
s_read(100);  // Read server banner

// USER command
s_string("USER ");
s_string_variable("anonymous");
s_string("\r\n");
s_xmit();
s_read(100);

// PASS command
s_string("PASS ");
s_string_variable("test@test.com");
s_string("\r\n");
s_xmit();
s_read(100);

// LIST command
s_string("LIST ");
s_string_variable("/");
s_string("\r\n");
s_xmit();
s_read(100);

// RETR command
s_string("RETR ");
s_string_variable("/etc/passwd");
s_string("\r\n");
s_xmit();
s_read(100);
```

**Explanation:** Defines an FTP protocol session with multiple fuzz points across different commands.

### Compiling and Running Custom Scripts

```bash
# Compile
spikeL compile custom_ftp.spike

# Run
generic_send_tcp 192.168.1.50 21 custom_ftp 0 0
```

### Fuzzing with Block Size Limits

```bash
generic_send_tcp 192.168.1.50 8080 /usr/share/spike/webserver.spk 0 50000
```

**Explanation:** Limits maximum fuzzed block size to 50,000 bytes, preventing overly large payloads.

### Parallel Fuzzing

```bash
# Fuzz different protocols simultaneously
generic_send_tcp 192.168.1.50 80 http.spk 0 0 &
generic_send_tcp 192.168.1.50 21 ftp.spk 0 0 &
generic_send_tcp 192.168.1.50 25 smtp.spk 0 0 &
wait
```

## 5. Advanced Usage

### Complex Protocol State Machines

```c
// complex_protocol.spike
s_init_fuzzing();

// Phase 1: Connection handshake
s_string("HELLO ");
s_string_variable("client");
s_string("\r\n");
s_xmit();
s_read(100);

// Phase 2: Authentication
s_string("AUTH ");
s_string_variable("admin");
s_string(" ");
s_string_variable("password");
s_string("\r\n");
s_xmit();
s_read(100);

// Phase 3: Data transfer
s_string("TRANSFER ");
s_string_variable("file.bin");
s_string(" ");
s_binary("\x00\x00\x00\x00");  // Fuzzed length field
s_string("\r\n");
s_xmit();
```

**Explanation:** Models a multi-phase protocol where each phase depends on the previous one succeeding.

### Binary Protocol Fuzzing

```c
// binary_protocol.spike
s_init_fuzzing();

// Header
s_binary("\xDE\xAD\xBE\xEF");  // Magic bytes (not fuzzed)
s_binary_variable(4);            // Length field (fuzzed)
s_binary_variable(1);            // Command byte (fuzzed)

// Payload
s_string_variable("data");       // Payload content (fuzzed)

s_xmit();
```

**Explanation:** Fuzzes binary protocol fields using `s_binary_variable()` for raw byte mutations.

### Integration with Crash Analysis

```bash
#!/bin/bash
# Run SPIKE and watch for crashes
generic_send_tcp 192.168.1.50 8080 http.spk 0 0 &
SPIKE_PID=$!

# Monitor target
while kill -0 $SPIKE_PID 2>/dev/null; do
    if ! nc -z 192.168.1.50 8080 2>/dev/null; then
        echo "CRASH DETECTED at $(date)"
        echo "Last fuzzed level: check spike output"
        break
    fi
    sleep 5
done
```

## 6. Real-World Scenarios

### Scenario 1: Custom Embedded Protocol

**Target:** Proprietary command server on port 7777

```bash
cat > custom.spike << 'EOF'
s_init_fuzzing();
s_read(100);

s_string("CMD:");
s_string_variable("GET_STATUS");
s_string("\n");
s_xmit();
s_read(100);

s_string("CMD:");
s_string_variable("SET_CONFIG");
s_string(" ");
s_string_variable("param=value");
s_string("\n");
s_xmit();
EOF

spikeL compile custom.spike
generic_send_tcp 192.168.1.200 7777 custom 0 0
```

### Scenario 2: SMB Fuzzing

**Target:** Windows file share

```bash
generic_send_tcp 192.168.1.100 445 /usr/share/spike/smb.spk 0 0
```

### Scenario 3: DNS Fuzzing

**Target:** Internal DNS server

```bash
generic_send_udp 192.168.1.1 53 /usr/share/spike/dns.spk 0 0
```

## 7. Detection & Defense

### How Defenders Detect

- **Protocol anomaly detection**: IDS systems flag malformed protocol messages
- **Connection patterns**: Rapid sequential connections with varying payload sizes
- **Error responses**: Unusual HTTP 500/400 responses in bursts

### Mitigation

- Deploy protocol-aware firewalls
- Implement input validation at the application layer
- Use protocol conformance testing in development
- Monitor service health with automated restarts

### Blue Team Response

```bash
# Capture traffic during suspected fuzzing
tcpdump -i eth0 port 8080 -w spike_capture.pcap -c 1000
# Analyze for fuzzing patterns in Wireshark
```

## 8. Troubleshooting

### Common Errors

**"generic_send_tcp: command not found":**
Ensure SPIKE is installed: `sudo apt install spike`

**Target doesn't crash:**
SPIKE's pre-built scripts may not cover your protocol's vulnerability surface. Write a custom SPIKE script targeting specific fields.

**Connection refused:**
Verify target is running: `nc -zv 192.168.1.50 8080`

### Performance Optimization

- Use block size limits to prevent overly large payloads
- Focus SPIKE scripts on the most complex protocol fields
- Run against a local test target first to verify script correctness

### FAQ

**Q: SPIKE vs boofuzz?**
A: boofuzz is the Python-based spiritual successor to SPIKE with better reporting and session management. Use boofuzz for new projects; SPIKE for legacy scripts.

**Q: Is SPIKE still maintained?**
A: The original SPIKE is largely unmaintained. The open-source fork on GitHub receives occasional updates. Consider boofuzz for actively maintained protocol fuzzing.

---

## Resources

- **SPIKE GitHub**: https://github.com/spikefuzz/spike
- **Boofuzz (successor)**: https://github.com/fortra/boofuzz
- **Immunity Inc.**: https://immunityinc.com/
- **SPIKE Tutorial**: `/usr/share/spike/docs/` (if available)
- **Protocol Fuzzing Guide**: https://resources.infosecinstitute.com/topic/protocol-fuzzing/
