---
title: merge-router-config
category: discovery
subcategory: cisco_tools
phase: 09
mitre_attack: TA0007
tags:
  - cisco
  - configuration
  - merging
  - automation
  - network
difficulty: intermediate
os: linux
install: sudo apt install merge-router-config
github: https://github.com/jeelabs/torch
related:
  - copy-router-config
  - cisco-torch
---

# merge-router-config

## Chapter 1: Overview

**merge-router-config** merges multiple Cisco router and switch configurations. It combines configuration files from multiple devices or backup copies, useful for configuration management and auditing.

### Key Features

- **Configuration merging**: Combine multiple configs
- **Diff analysis**: Compare configurations
- **Template support**: Apply templates to configs
- **Batch processing**: Multiple file support
- **Output formatting**: Various output formats

### Use Cases

| Scenario | Description |
|----------|-------------|
| Config comparison | Find differences between backups |
| Template application | Apply standard templates |
| Config aggregation | Combine multiple devices |
| Version tracking | Track configuration changes |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install merge-router-config
```

### Verify Installation

```bash
merge-router-config --help
```

---

## Chapter 3: Basic Usage

### Merge Two Configs

```bash
merge-router-config -a config1.txt -b config2.txt -o merged.txt
```

### Compare Configs

```bash
merge-router-config -a config1.txt -b config2.txt --diff
```

### Merge Directory

```bash
merge-router-config -d /configs/ -o merged.txt
```

---

## Chapter 4: Comparison Options

### Unified Diff

```bash
merge-router-config -a config1.txt -b config2.txt --diff --unified
```

### Side-by-Side

```bash
merge-router-config -a config1.txt -b config2.txt --diff --side-by-side
```

### Context Diff

```bash
merge-router-config -a config1.txt -b config2.txt --diff --context
```

---

## Chapter 5: Advanced Features

### Ignore Comments

```bash
merge-router-config -a config1.txt -b config2.txt --diff --ignore-comments
```

### Ignore Whitespace

```bash
merge-router-config -a config1.txt -b config2.txt --diff --ignore-whitespace
```

### Apply Template

```bash
merge-router-config -a base.txt -t template.txt -o output.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Configuration Audit

```bash
# Compare current vs backup
merge-router-config -a current.txt -b backup.txt --diff
```

### Scenario 2: Standardization

```bash
# Apply template to configs
for config in /configs/*.txt; do
  merge-router-config -a $config -t template.txt -o "/standardized/$(basename $config)"
done
```

### Scenario 3: Change Tracking

```bash
# Compare versions
merge-router-config -a v1.txt -b v2.txt --diff --unified
```

---

## Chapter 7: Mitigation & Defense

**For network administrators:**

- Implement configuration version control
- Regular configuration backups
- Change management procedures
- Configuration auditing
- Access controls on config files

---

## Resources

- [merge-router-config Documentation](https://www pentest-tools.com)
- [Cisco Configuration Management](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SW_Merge.html)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
