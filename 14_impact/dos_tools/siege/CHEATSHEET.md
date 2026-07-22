# Siege Quick Reference

## Installation

```bash
sudo apt install siege
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `siege <url>` | Basic load test (10 users) |
| `siege -c 50 <url>` | 50 concurrent users |
| `siege -c 25 -t 5M <url>` | 5 min timed test |
| `siege -b <url>` | Benchmark mode (no delays) |
| `siege -f urls.txt` | Test URL file |
| `siege -i -f urls.txt` | Internet simulation |
| `siege -j <url>` | JSON output |

## Options Quick Reference

```
-c NUM        # Concurrent users (default: 10)
-r NUM        # Repetitions
-t NUMm       # Timed test (S/M/H)
-d NUM        # Delay between requests (seconds)
-b            # Benchmark mode (no delays)
-f FILE       # URL file
-l            # Log output
-j            # JSON output
-g            # Debug headers
-v            # Verbose
-q            # Quiet
-H "Header"   # Add header
-A "Agent"    # Custom User-Agent
```

## Common Scenarios

### Quick Benchmark
```bash
siege -c 50 -b http://example.com
```

### Endurance Test
```bash
siege -c 10 -t 2H http://example.com
```

### API Load Test
```bash
siege -c 25 -H "Authorization: Bearer TOKEN" http://api.example.com
```

### Regression Test
```bash
siege -c 50 -r 100 -j http://test.example.com > results.json
```

## URL File Format

```bash
# urls.txt - one URL per line
http://example.com/
http://example.com/about
http://example.com/products
http://example.com/api/data
```

## Output Metrics

| Metric | Meaning |
|--------|---------|
| Transactions | Total requests |
| Availability | Success % |
| Response time | Average (secs) |
| Transaction rate | Requests/sec |
| Throughput | KB/sec |
| Failed | Error count |

## Configuration

```bash
# Generate config file
siege.config

# Edit ~/.siege/siege.conf
verbose = false
concurrent = 10
delay = 1
timeout = 30
```

## CI/CD Integration

```bash
# Basic pass/fail test
siege -c 50 -r 100 -j http://staging.example.com > results.json
ERRORS=$(jq '.failed_transactions' results.json)
[ "$ERRORS" -gt 5 ] && exit 1
```

## Monitoring During Test

```bash
# Watch server connections
watch -n 1 'netstat -an | grep :80 | wc -l'

# Monitor response times
tail -f /var/log/siege.log
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Too many failures | Reduce `-c` or add `-d` delay |
| Connection refused | Verify target is reachable |
| Memory issues | Reduce concurrency, use URL file |
| SSL errors | Check certificates, use `-T` header |
