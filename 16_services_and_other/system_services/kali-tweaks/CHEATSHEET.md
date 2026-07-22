# Kali Tweaks - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Launch Kali Tweaks
sudo kalitweaks

# From application menu
Applications > Settings > kali-tweaks
```

---

## Menu Navigation

| Key | Action |
|-----|--------|
| Arrow keys | Navigate |
| Enter | Select |
| Escape | Go back |
| Tab | Switch panels |
| q | Quit |

---

## Main Menu Options

```
System    - System services and configuration
Desktop   - Desktop environment settings
Network   - Network configuration
Shell     - Shell and terminal options
Updates   - Update configuration
Security  - Security settings
Help      - Help and documentation
Quit      - Exit Kali Tweaks
```

---

## Common Service Management

```bash
# Enable service
sudo systemctl enable service_name
sudo systemctl start service_name

# Disable service
sudo systemctl disable service_name
sudo systemctl stop service_name

# Check status
sudo systemctl status service_name
```

---

## Services to Enable/Disable

### Enable for Desktop
```bash
sudo systemctl enable NetworkManager
sudo systemctl enable ssh
```

### Disable if Not Used
```bash
sudo systemctl disable bluetooth
sudo systemctl disable cups
sudo systemctl disable avahi-daemon
```

### Enable for Testing
```bash
sudo systemctl enable apache2
sudo systemctl enable postgresql
sudo systemctl enable docker
```

---

## Desktop Environments

| Environment | Command |
|-------------|---------|
| GNOME | Start GNOME session |
| XFCE | Start XFCE session |
| KDE | Start KDE session |
| LXDE | Start LXDE session |
| i3 | Start i3 session |

---

## Shell Configuration

```bash
# Edit Zsh config
nano ~/.zshrc

# Common aliases
alias ll='ls -la'
alias update='sudo apt update && sudo apt upgrade'
alias tools='cd /opt/tools'

# Add to PATH
export PATH=$PATH:/opt/tools
```

---

## Network Configuration

```bash
# Configure NetworkManager
sudo nmtui

# Edit connections
nmcli connection edit

# Show connections
nmcli connection show

# Set hostname
sudo hostnamectl set-hostname new-hostname
```

---

## Update Configuration

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade

# Full system upgrade
sudo apt full-upgrade

# Clean up
sudo apt autoremove
sudo apt autoclean
```

---

## Security Hardening

```bash
# Enable UFW firewall
sudo ufw enable
sudo ufw allow ssh

# Check firewall status
sudo ufw status

# Configure SSH
sudo nano /etc/ssh/sshd_config

# Restart SSH
sudo systemctl restart ssh
```

---

## Optimization Profiles

### Desktop Use
```bash
# Enable desktop services
sudo systemctl enable NetworkManager
sudo systemctl enable bluetooth  # if needed
```

### Headless/Server
```bash
# Disable unnecessary services
sudo systemctl disable bluetooth
sudo systemctl disable cups
sudo systemctl enable ssh
```

### Penetration Testing
```bash
# Enable testing services
sudo systemctl enable apache2
sudo systemctl enable postgresql
sudo systemctl enable ssh
```

---

## Troubleshooting

```bash
# Check Kali Tweaks installation
dpkg -l | grep kali-tweaks

# Reinstall if needed
sudo apt reinstall kali-tweaks

# Check service status
sudo systemctl status service_name

# Restart service
sudo systemctl restart service_name

# View logs
journalctl -u service_name
```

---

## Useful Commands

```bash
# List all services
systemctl list-unit-files

# List running services
systemctl list-units --type=service

# Check system info
uname -a
lsb_release -a

# Check disk usage
df -h

# Check memory
free -h
```
