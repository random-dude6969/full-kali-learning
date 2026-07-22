# Siege - HTTP Load Testing and Benchmarking Tool

## Overview

Siege is a mature, comprehensive HTTP regression testing and benchmarking utility that has been actively developed since the early 2000s. Unlike simple DoS tools, Siege is designed for legitimate load testing, performance benchmarking, and regression testing of web applications. It can stress test a single URL with simulated users or read multiple URLs from a file and test them simultaneously.

**Author**: Jeffrey Fulmer (JoeDog)
**License**: GPLv2
**Version**: 4.1.7
**Category**: Impact / Load Testing

### Key Features

- HTTP/1.0 and HTTP/1.1 protocol support
- GET and POST request methods
- Cookie handling and session management
- Basic authentication support
- Transaction logging to file
- JSON output for automation
- Configurable delays and concurrency
- URL file support for batch testing
- Benchmark mode (no delays)

### How It Differs from DoS Tools

Siege is primarily a **testing tool**, not an attack tool. Key differences:

| Feature | Siege | DoS Tools |
|---------|-------|-----------|
| Purpose | Performance testing | Service disruption |
| Reporting | Detailed metrics | Minimal output |
| Delays | Configurable | None |
| Concurrency | Controlled | Maximum |
| Logging | Full transaction logs | None |
| Legitimacy | Industry standard | Malicious use |

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install siege
```

### From Source

```bash
git clone https://github.com/JoeDog/siege.git
cd siege
./utils/configure
make
sudo make install
```

### Dependencies

- libc6
- libssl3t64
- zlib1g

## Usage

### Basic Syntax

```bash
siege [options] [URL]
siege [options] -f <url-file>
siege -g URL  # Debug mode - show headers
```

### Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-c, --concurrent=NUM` | Number of concurrent users | 10 |
| `-r, --reps=NUM` | Number of times to run test | - |
| `-t, --time=NUMm` | Timed test (S/M/H suffix) | - |
| `-d, --delay=NUM` | Random delay before each request | - |
| `-b, --benchmark` | No delays between requests | - |
| `-i, --internet` | Internet user simulation | - |
| `-f, --file=FILE` | URL file to test | - |
| `-l, --log[=FILE]` | Log to file | /var/log/siege.log |
| `-g, --get` | Pull HTTP headers | - |
| `-p, --print` | Print entire page | - |
| `-v, --verbose` | Verbose output | - |
| `-q, --quiet` | Suppress output | - |
| `-j, --json-output` | JSON output | - |
| `-H, --header="text"` | Add request header | - |
| `-A, --user-agent="text"` | Set User-Agent | - |
| `-T, --content-type="text"` | Set Content-Type | - |
| `--no-parser` | Disable HTML parser | - |
| `--no-follow` | Don't follow redirects | - |

## Detailed Usage Examples

### Basic Load Test

```bash
# Test single URL with 10 concurrent users (default)
siege http://example.com

# Test with 50 concurrent users
siege -c 50 http://example.com

# Run test for 1 hour
siege -c 25 -t 1H http://example.com
```

### Timed Testing

```bash
# 5 minutes test
siege -c 20 -t 5M http://example.com

# 30 seconds burst test
siege -c 100 -t 30S http://example.com

# 2 hour endurance test
siege -c 10 -t 2H http://example.com
```

### Repetition Testing

```bash
# Run 1000 iterations with 20 users
siege -c 20 -r 1000 http://example.com

# 500 repetitions with logging
siege -c 50 -r 500 -l http://example.com
```

### URL File Testing

```bash
# Create urls.txt
cat > urls.txt << EOF
http://example.com/
http://example.com/about
http://example.com/contact
http://example.com/products
EOF

# Test all URLs
siege -f urls.txt -c 25

# Internet simulation (random URL selection)
siege -f urls.txt -i -c 10
```

### Benchmark Mode

```bash
# Maximum throughput test (no delays)
siege -b -c 100 http://example.com

# Compare with normal mode
siege -c 100 http://example.com
siege -b -c 100 http://example.com
```

### POST Request Testing

```bash
# Test login endpoint
siege -c 25 http://example.com/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=test&password=pass"

# JSON API testing
siege -c 10 http://api.example.com/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### Debug Mode

```bash
# View response headers
siege -g http://example.com

# View full response
siege -p http://example.com

# Debug with verbose output
siege -v -g http://example.com
```

### Custom Headers and Auth

```bash
# Add authentication header
siege -c 10 -H "Authorization: Bearer TOKEN" http://api.example.com

# Custom User-Agent
siege -c 25 -A "LoadTestBot/1.0" http://example.com

# Multiple headers
siege -c 10 \
  -H "Authorization: Bearer TOKEN" \
  -H "X-API-Key: 12345" \
  http://api.example.com
```

## Configuration File

### Generating Config

```bash
siege.config
# Creates ~/.siege/siege.conf
```

### Key Configuration Options

```bash
# ~/.siege/siege.conf

# Verbose setting (0=quiet, 1=normal, 2=verbose)
verbose = false

# Logging
log FILE = /var/log/siege.log

# Concurrency default
concurrent = 10

# Delay between requests
delay = 1

# Connection timeout
timeout = 30

# Failures allowable before aborting
failures = 50

# Reuse connections
reps = 1
```

## Output Analysis

### Sample Output

```
** siege 4.1.7
** Preparing 25 concurrent users for battle.
** The server is now under siege...
HTTP/1.1 200   0.14 secs:    2878 bytes ==> GET /
HTTP/1.1 200   0.09 secs:    1532 bytes ==> GET /about
HTTP/1.1 200   0.11 secs:    2156 bytes ==> GET /products
...
Lifting the server siege...
Transactions:              4523 hits
Availability:             99.87 %
Elapsed time:             300.00 secs
Data transferred:        12.45 MB
Response time:            0.08 secs
Transaction rate:        15.08 trans/sec
Throughput:              42.56 KB/sec
Concurrency:              9.87
Successful transactions:  4523
Failed transactions:         6
Longest transaction:      2.34 secs
Shortest transaction:     0.02 secs
```

### Metrics Explained

| Metric | Description |
|--------|-------------|
| Transactions | Total HTTP requests completed |
| Availability | Percentage of successful requests |
| Elapsed time | Total test duration |
| Data transferred | Total bytes received |
| Response time | Average response time |
| Transaction rate | Requests per second |
| Throughput | Data per second |
| Concurrency | Average simultaneous connections |
| Failed transactions | HTTP errors or timeouts |
| Longest/Shortest | Response time extremes |

## Advanced Scenarios

### Regression Testing

```bash
# Before deployment
siege -c 50 -r 100 -l http://staging.example.com

# After deployment
siege -c 50 -r 100 -l http://staging.example.com

# Compare logs
diff /var/log/siege_before.log /var/log/siege_after.log
```

### API Load Testing

```bash
# Test REST API endpoints
siege -c 25 -t 5M \
  -H "Authorization: Bearer TOKEN" \
  -H "Accept: application/json" \
  http://api.example.com/v1/users
```

### E-Commerce Load Test

```bash
# Create comprehensive test file
cat > ecommerce.txt << EOF
http://example.com/
http://example.com/products
http://example.com/cart
http://example.com/checkout
EOF

# Simulate shopping traffic
siege -f ecommerce.txt -i -c 100 -t 30M
```

### Content Management System

```bash
# Test WordPress under load
siege -c 50 -t 10M -f wordpress_urls.txt

# Include admin pages (auth required)
siege -c 10 -H "Authorization: Basic YWRtaW46cGFzc3dvcmQ=" \
  http://example.com/wp-admin/
```

## Automation Integration

### Shell Script for CI/CD

```bash
#!/bin/bash
# load_test.sh

TARGET=$1
USERS=$2
DURATION=$3

echo "Starting load test against $TARGET"
siege -c $USERS -t $DURATION -j $TARGET > results.json

# Parse results
AVG_RESPONSE=$(jq '.response_time' results.json)
ERROR_RATE=$(jq '.failed_transactions' results.json)

if [ $ERROR_RATE -gt 5 ]; then
    echo "FAIL: Error rate too high"
    exit 1
fi

echo "PASS: Average response ${AVG_RESPONSE}s"
```

### Python Integration

```python
import subprocess
import json

def run_siege(url, concurrent=10, duration='1M'):
    cmd = [
        'siege', '-c', str(concurrent),
        '-t', duration, '-j', url
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return json.loads(result.stdout)

results = run_siege('http://example.com', concurrent=50, duration='5M')
print(f"Throughput: {results['throughput']}")
```

## Best Practices

### Test Planning

1. **Establish Baseline**: Run test during off-peak hours first
2. **Gradual Ramp-up**: Start with low concurrency, increase gradually
3. **Realistic Delays**: Use `-d` to simulate real user think time
4. **Multiple Scenarios**: Test different page types (static, dynamic, API)

### Resource Monitoring

```bash
# Monitor server during test
# Terminal 1: Run siege
siege -c 50 -t 5M http://example.com

# Terminal 2: Monitor server
top -d 1
iostat -x 1
sar -n DEV 1
```

### Result Validation

```bash
# Run test 3 times for consistency
for i in {1..3}; do
    echo "Run $i"
    siege -c 25 -r 100 -j http://example.com > "run_$i.json"
done

# Compare results
jq -s '.[].transaction_rate' run_*.json
```

## Limitations

- Single-machine testing may not represent distributed load
- HTTP/2 support is limited
- WebSocket testing not supported
- Complex user flows require URL file preparation
- SSL/TLS performance testing limited

## Troubleshooting

### Connection Refused

```bash
# Check target availability
curl -I http://example.com

# Increase timeout
siege -t 30S http://example.com
```

### High Failure Rate

```bash
# Reduce concurrency
siege -c 10 http://example.com

# Add delays
siege -c 25 -d 1 http://example.com

# Check server logs for errors
tail -f /var/log/apache2/error.log
```

### Memory Issues

```bash
# Reduce concurrent users
siege -c 10 -r 1000 http://example.com

# Use URL file instead of single URL
siege -f urls.txt -c 25
```

## System Requirements

### Minimum Requirements

```
- OS: Linux, macOS, Unix
- RAM: 256MB
- Network: 10 Mbps+
- Disk: 50MB for logs
- Privileges: Normal user (root for high concurrency)
```

### Recommended Setup

```
- OS: Linux (Ubuntu/CentOS)
- RAM: 2GB+
- Network: 100 Mbps+
- CPU: Multi-core
- Disk: SSD for log writes
```

### Scaling Considerations

```bash
# Check system limits
ulimit -n

# Increase for high concurrency
ulimit -n 65535

# Monitor system resources during test
htop
iftop
iostat -x 1
```

## Comparison with Other Load Testing Tools

| Feature | Siege | Apache Bench | wrk | JMeter |
|---------|-------|--------------|-----|--------|
| Language | C | C | C | Java |
| Concurrency Model | Multi-process | Single-process | Multi-thread | Multi-thread |
| Configuration | Config file | CLI only | CLI only | GUI/CLI |
| Scripting | Limited | No | Lua | Java/Groovy |
| Output | Text/JSON | Text | Text | XML/CSV |
| Best For | Quick tests | Simple benchmarks | High performance | Complex scenarios |

## Integration with CI/CD

### Jenkins Pipeline Example

```groovy
pipeline {
    agent any
    stages {
        stage('Load Test') {
            steps {
                sh 'siege -c 50 -t 5M -j http://staging.example.com > results.json'
            }
        }
        stage('Analyze Results') {
            steps {
                script {
                    def results = readJSON file: 'results.json'
                    if (results.failed_transactions > 10) {
                        error "Load test failed: too many errors"
                    }
                }
            }
        }
    }
}
```

### GitHub Actions Example

```yaml
name: Load Test
on: [push]
jobs:
  siege:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Siege
        run: sudo apt-get install -y siege
      - name: Run Load Test
        run: siege -c 25 -t 2M -j http://localhost:8080 > results.json
      - name: Check Results
        run: |
          ERRORS=$(jq '.failed_transactions' results.json)
          if [ "$ERRORS" -gt 5 ]; then
            echo "Load test failed"
            exit 1
          fi
```

## Advanced Metrics Interpretation

### Response Time Analysis

| Response Time | Interpretation |
|---------------|----------------|
| < 100ms | Excellent |
| 100-500ms | Good |
| 500ms-1s | Acceptable |
| 1-3s | Poor |
| > 3s | Critical |

### Throughput Benchmarks

| Application Type | Expected Throughput |
|------------------|---------------------|
| Static HTML | 1000+ req/sec |
| Dynamic PHP | 100-500 req/sec |
| API (JSON) | 500-2000 req/sec |
| Database-heavy | 50-200 req/sec |

### Availability Thresholds

| Availability | Downtime/Year | Target |
|--------------|---------------|--------|
| 99% | 3.65 days | Basic |
| 99.9% | 8.76 hours | Standard |
| 99.99% | 52.6 minutes | Premium |
| 99.999% | 5.26 minutes | Mission Critical |

## Real-World Case Studies

### Case Study 1: E-Commerce Platform

**Scenario**: Pre-Black Friday load testing

```bash
# Baseline test
siege -c 10 -r 100 http://shop.example.com > baseline.json

# Expected load (10x normal)
siege -c 100 -t 30M http://shop.example.com > blackfriday.json

# Compare results
jq -s '.[].transaction_rate' baseline.json blackfriday.json
```

**Findings**: Server degraded at 500 concurrent users. Recommended:
- Add CDN for static assets
- Implement connection pooling
- Upgrade database server

### Case Study 2: SaaS Application

**Scenario**: API performance validation

```bash
# Test API endpoints
siege -c 50 -t 5M -H "Authorization: Bearer TOKEN" \
  http://api.example.com/v1/data > api_test.json

# Test with different payload sizes
siege -c 25 -t 2M -d '{"small": "data"}' http://api.example.com/v1/small
siege -c 25 -t 2M -d '{"large": "...50KB..."}' http://api.example.com/v1/large
```

**Findings**: Large payloads caused timeout at 50 concurrent users. Recommended:
- Implement pagination
- Add response compression
- Use CDN for large responses

## Troubleshooting Guide

### Common Error Messages

**"Connection refused"**
```bash
# Verify target is accessible
curl -I http://example.com
# Check if port is open
nmap -p 80 example.com
```

**"Connection timed out"**
```bash
# Increase timeout in config
# ~/.siege/siege.conf
timeout = 60
# Or use -d flag for delays
siege -d 1 http://example.com
```

**"Too many open files"**
```bash
# Increase file descriptor limit
ulimit -n 65535
# Or reduce concurrency
siege -c 50 http://example.com
```

**"SSL certificate problem"**
```bash
# Disable SSL verification (not recommended for production)
# Or update CA certificates
sudo apt-get update && sudo apt-get install ca-certificates
```

### Performance Tuning

```bash
# Optimize for throughput
siege -b -c 100 http://example.com

# Optimize for accuracy
siege -c 25 -d 1 http://example.com

# Optimize for logging
siege -c 50 -l -j http://example.com
```

## Best Practices Summary

### Planning Phase

1. Define clear test objectives
2. Establish baseline metrics
3. Identify success/failure criteria
4. Plan test duration and concurrency
5. Prepare monitoring infrastructure

### Execution Phase

1. Start with low concurrency
2. Gradually increase load
3. Monitor both client and server
4. Document anomalies in real-time
5. Be ready to abort if needed

### Analysis Phase

1. Compare against baseline
2. Identify bottlenecks
3. Correlate metrics (response time vs throughput)
4. Document findings with evidence
5. Recommend specific improvements

### Reporting Phase

1. Include executive summary
2. Provide detailed metrics
3. Show graphs/charts
4. List specific recommendations
5. Prioritize improvements by impact

## Legal Notice

Siege is a legitimate load testing tool used by developers and system administrators to verify application performance. Always ensure you have proper authorization before testing any system. Unauthorized load testing may violate terms of service or local laws.

**Authorization Checklist**:
- [ ] Written permission from system owner
- [ ] Defined scope and time window
- [ ] Notification to operations team
- [ ] Rollback plan documented
- [ ] Emergency contacts identified
- [ ] Monitoring in place
- [ ] Sign-off from stakeholders

## References

- Siege Documentation: https://www.joedog.org/siege-home/
- GitHub: https://github.com/JoeDog/siege
- RFC 2616 - HTTP/1.1
- RFC 7230 - HTTP/1.1 Message Syntax
- Web Performance Testing Best Practices
- OWASP Performance Testing Guide
- Load Testing Methodology Guide
