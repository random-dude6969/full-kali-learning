---
title: "netmask - IP Subnet Mask Calculator"
description: "IP subnet mask calculator supporting CIDR, wildcard, hex, decimal conversions for network planning and configuration analysis"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "system_network_configuration_discovery"
categories:
  - "network"
  - "subnetting"
  - "calculator"
  - "network-planning"
tags:
  - "subnet"
  - "cidr"
  - "wildcard"
  - "calculator"
  - "network"
install_command: "sudo apt install netmask"
version: "2.3.12"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/netmask/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
related_tools:
  - "ipcalc"
  - "sipcalc"
  - "subnetcalc"
---

# netmask - IP Subnet Mask Calculator

## 1. Introduction

**netmask** is a versatile IP subnet mask calculator that supports multiple output formats including CIDR, wildcard, hexadecimal, and decimal representations. The tool is essential for network planning, subnetting analysis, and configuration verification.

### Key Capabilities

**Multiple Output Formats**: Generates CIDR, wildcard, hex, and decimal mask representations.

**Range Calculation**: Calculates IP address ranges for given subnets.

**Subnet Analysis**: Analyzes network configurations and identifies subnet boundaries.

**Batch Processing**: Handles multiple input formats and can process lists of addresses.

### Use Cases

- **Network Planning**: Design and verify subnet allocations.
- **Security Assessments**: Identify network ranges and potential attack surfaces.
- **Configuration Verification**: Validate subnet mask configurations.
- **Penetration Testing**: Calculate target ranges for scanning.
- **Network Documentation**: Generate accurate network documentation.

### How netmask Works

netmask parses IP address and subnet mask inputs, then calculates various representations of the network configuration. It outputs CIDR notation, wildcard masks, hexadecimal representations, and decimal formats as requested.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install netmask
```

### Verify Installation

```bash
netmask --version
```

### Dependencies

netmask has minimal dependencies and works with standard C libraries.

## 3. Core Concepts

### Subnet Mask Representations

**CIDR (Classless Inter-Domain Routing)**: Slash notation (e.g., /24) indicating prefix length.

**Wildcard Mask**: Inverse of subnet mask, used in access control lists (e.g., 0.0.255.255).

**Hexadecimal**: Hex representation of mask (e.g., 0xFFFFFF00).

**Decimal**: Standard dotted-decimal notation (e.g., 255.255.255.0).

### IP Address Classes

**Class A**: 1.0.0.0 - 126.255.255.255 (/8 default).

**Class B**: 128.0.0.0 - 191.255.255.255 (/16 default).

**Class C**: 192.0.0.0 - 223.255.255.255 (/24 default).

**Private Ranges**: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16.

### Subnetting Fundamentals

**Network Address**: First address in subnet (all host bits zero).

**Broadcast Address**: Last address in subnet (all host bits one).

**Host Range**: Usable addresses between network and broadcast.

**Subnet Size**: 2^(32-prefix_length) total addresses.

## 4. Command Reference

### Basic Syntax

```bash
netmask [OPTIONS] ADDRESS/MASK
```

### Core Flags

**-d**: Output decimal mask format.

**-c**: Output CIDR notation.

**-w**: Output wildcard mask format.

**-h**: Output hexadecimal mask format.

**-r**: Output range format (network - broadcast).

### Advanced Flags

**-s**: Short output format.

**-l**: Long output format with all details.

**-i**: Interactive mode for manual input.

**-e**: Exact match mode for ACL calculations.

## 5. Examples

### Basic CIDR Conversion

```bash
netmask 192.168.1.0/24
```

**Result**: Displays all mask representations for /24 network.

### Decimal Output

```bash
netmask -d 192.168.1.0/24
```

**Result**: Shows decimal mask (255.255.255.0).

### Wildcard Mask

```bash
netmask -w 192.168.1.0/24
```

**Result**: Shows wildcard mask (0.0.0.255).

### Hexadecimal Output

```bash
netmask -h 192.168.1.0/24
```

**Result**: Shows hex mask (0xFFFFFF00).

### Network Range

```bash
netmask -r 192.168.1.0/24
```

**Result**: Shows network and broadcast addresses.

### Custom Subnet

```bash
netmask 192.168.1.0/26
```

**Result**: Calculates /26 subnet details.

### Large Network

```bash
netmask 10.0.0.0/8
```

**Result**: Shows /8 network details.

### Small Subnet

```bash
netmask 192.168.1.0/30
```

**Result**: Shows /30 subnet (4 addresses, 2 usable).

### Multiple Formats

```bash
netmask -d -w -h 192.168.1.0/24
```

**Result**: Shows decimal, wildcard, and hex formats.

### All Formats

```bash
netmask -l 192.168.1.0/24
```

**Result**: Shows all available representations.

## 6. Advanced Techniques

### Subnet Planning

```bash
#!/bin/bash
NETWORK="192.168.1.0"
PREFIX=24

echo "Planning subnets for $NETWORK/$PREFIX"
for i in $(seq 1 10); do
  SUBNET="192.168.1.$((i * 32))"
  echo "Subnet $i: $SUBNET/27"
  netmask -r "$SUBNET/27"
done
```

### ACL Calculation

```bash
# Calculate wildcard masks for access lists
netmask -w 192.168.1.0/24
# Result: 0.0.0.255 (for ACL: access-list 10 permit 192.168.1.0 0.0.0.255)
```

### Subnet Comparison

```bash
# Compare two subnets
echo "=== Subnet 1 ==="
netmask -l 192.168.1.0/24

echo "=== Subnet 2 ==="
netmask -l 192.168.2.0/24
```

### Network Documentation

```bash
#!/bin/bash
NETWORKS="networks.txt"
OUTPUT="network_doc_$(date +%Y%m%d).txt"

while IFS= read -r network; do
  echo "=== $network ===" >> "$OUTPUT"
  netmask -l "$network" >> "$OUTPUT"
  echo "" >> "$OUTPUT"
done < "$NETWORKS"

echo "Documentation saved to $OUTPUT"
```

### Subnet Verification

```bash
# Verify subnet configuration
EXPECTED="255.255.255.0"
ACTUAL=$(netmask -d 192.168.1.0/24)

if [ "$EXPECTED" = "$ACTUAL" ]; then
  echo "Subnet mask correct"
else
  echo "Subnet mask mismatch: expected $EXPECTED, got $ACTUAL"
fi
```

### CIDR Calculator Script

```bash
#!/bin/bash
# Interactive CIDR calculator
echo "Enter IP address (e.g., 192.168.1.0/24):"
read INPUT

echo "=== Results ==="
netmask -l "$INPUT"
```

### Batch Processing

```bash
# Process multiple networks
cat > networks.txt << 'EOF'
192.168.1.0/24
192.168.2.0/24
10.0.0.0/8
EOF

while read network; do
  echo "=== $network ==="
  netmask -d -w "$network"
done < networks.txt
```

## 7. Detection & Defense

### Network Detection

**Subnet Scanning**: netmask helps identify network ranges for scanning activities.

**Address Enumeration**: Calculator outputs reveal target address ranges.

**Configuration Analysis**: subnet information can expose network topology.

### Defense Strategies

**Network Segmentation**: Use subnet calculations to design secure network segments.

**ACL Planning**: Calculate wildcard masks for firewall access control lists.

**IP Address Management**: Implement proper IP address allocation.

**Monitoring**: Monitor for subnet scanning activities.

**Documentation**: Maintain accurate network documentation.

## 8. Troubleshooting

### Invalid Input Format

**Correct Syntax**: Use proper IP/CIDR format.

```bash
netmask 192.168.1.0/24
```

**Avoid Common Errors**: Don't use spaces or invalid characters.

### No Output

**Check Input**: Verify input format is correct.

```bash
netmask 192.168.1.0/24
```

**Try Different Flags**: Use -l for comprehensive output.

```bash
netmask -l 192.168.1.0/24
```

### Unexpected Results

**Verify Subnet Mask**: Ensure CIDR prefix is correct.

```bash
netmask -d 192.168.1.0/24
```

**Check Network Class**: Some calculations vary by IP class.

### Permission Issues

**User Permissions**: netmask typically doesn't require root.

```bash
netmask 192.168.1.0/24
```

### Format Confusion

**Understand Output**: Review netmask documentation for format explanations.

**Use -l Flag**: Long format shows all representations.

```bash
netmask -l 192.168.1.0/24
```

## Resources

- **netmask Kali Documentation**: https://www.kali.org/tools/netmask/
- **Subnetting Tutorial**: Various networking training resources
- **CIDR Notation Guide**: Network engineering documentation
- **IP Subnet Calculator Reference**: Online subnet calculators for verification

## Related Tools

**ipcalc**: IP address calculator with subnet and broadcast calculations.

**sipcalc**: Advanced subnet calculator with multiple output formats.

**subnetcalc**: Subnet calculator with detailed network information.

**subnetting practice tools**: Online tools for learning subnetting.
