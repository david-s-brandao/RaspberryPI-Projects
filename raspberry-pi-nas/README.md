<p align="center"> 
  <img src="../Media/Raspberry-pi-nas-wide.png" alt="Raspberry Pi NAS Setup" width="600" style="border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
</p>

<h1 align="center" style="text-shadow: 1px 1px 2px #00000020;">Raspberry Pi NAS</h1>
<p align="center"><strong>Transform Your Raspberry Pi into a Professional Network Storage Solution</strong></p>
<p align="center">
  A comprehensive guide to building a high-performance NAS system that delivers<br>
  enterprise-grade features while maintaining exceptional value and efficiency.
</p>

<div align="center">
  <img src="https://img.shields.io/badge/Raspberry%20Pi-4B%2F5-red" alt="Pi Version">
  <img src="https://img.shields.io/badge/Storage-1TB+-blue" alt="Storage">
  <img src="https://img.shields.io/badge/Protocols-SMB%2FSSH%2FRSYNC-green" alt="Protocols">
  <img src="https://img.shields.io/badge/Power%20Usage-~5W-yellow" alt="Power Usage">
</div>

---

## Table of Contents
- [Requirements](#requirements)
- [Setup Guide](#setup-guide)
- [File Sharing Configuration](#file-sharing)
- [Security Implementation](#security)
- [Advanced Features](#advanced-features)

---

## Requirements

### Core Components
| Component | Specification | Purpose |
|-----------|--------------|---------|
| Raspberry Pi | 4B (2GB+ RAM) | System processor |
| Storage | 32GB+ microSD | Boot drive |
| External Drive | SSD/HDD | Data storage |
| OS | Raspberry Pi OS Lite | Base system |
| Network | Ethernet cable | Connectivity |

---

## Setup Guide

### 1. System Installation
- Download and use [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- Select "Raspberry Pi OS Lite (64-bit)"
- Enable SSH during installation

### 2. Initial Configuration
```bash
# System update
sudo apt update && sudo apt upgrade -y

# Note: For Raspberry Pi 5 2GB, use only:
sudo apt update
```

### 3. Storage Setup
```bash
# Identify storage device
lsblk

# Create mount point
sudo mkdir -p /mnt/nas

# Mount drive
sudo mount /dev/sda1 /mnt/nas

# Format for optimal performance
sudo mkfs.ext4 /dev/sda1
```

### 4. Persistent Mount
```bash
# Configure auto-mount
sudo nano /etc/fstab

# Add this line:
/dev/sda1 /mnt/nas ext4 defaults,noatime 0 1
```

### 5. User Configuration
```bash
# Create dedicated user
sudo adduser nasadmin

# Set permissions
sudo chown -R nasadmin:nasadmin /mnt/nas
```

---

## File Sharing

### Samba Installation
```bash
# Install Samba
sudo apt install samba samba-common-bin -y
```

### Share Configuration
```bash
# Edit configuration
sudo nano /etc/samba/smb.conf
```

Add this optimized configuration:
```ini
[NAS]
path = /mnt/nas
browseable = yes
writeable = yes
only guest = no
create mask = 0777
directory mask = 0777
valid users = nasadmin
```

### Access Setup
```bash
# Set user password
sudo smbpasswd -a nasadmin

# Restart service
sudo systemctl restart smbd
```

### Access Methods
- Windows: `\\raspberrypi.local\NAS`
- Mac/Linux: `smb://raspberrypi.local/NAS`

---

## Security

### Essential Security Measures
1. Change default credentials
2. Implement SSH key authentication
3. Configure firewall:
```bash
sudo apt install ufw
sudo ufw allow OpenSSH
sudo ufw allow from 192.168.0.0/24 to any port 445 proto tcp
sudo ufw enable
```

### Advanced Protection
- Install and configure Fail2ban
- Set up VPN access for remote management
- Implement regular security updates

---

## Advanced Features

### Automated Backups
- Configure rsync with cron for scheduled backups
- Implement version control for critical data

### System Monitoring
- Install Cockpit or Netdata for system metrics
- Set up alerting for critical events

### Storage Encryption
- Implement LUKS for full-disk encryption
- Configure secure key management

### Network Optimization
```bash
# Configure static IP
sudo nano /etc/dhcpcd.conf
```

---

## Maintenance

### Best Practices
- Use self-powered drives for reliability
- Monitor drive health with hdparm
- Implement regular backup verification

### Troubleshooting
```bash
# View system logs
journalctl -xe

# Monitor system logs
sudo tail -f /var/log/syslog
```

---

<p align="center">
  <strong>Professional NAS Solution</strong><br>
  <em>Built with precision and reliability in mind</em>
</p>  