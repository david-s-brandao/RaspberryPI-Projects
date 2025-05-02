<p align="center"> 
  <img src="../Media/pi-packet-capture-wide.png" alt="Raspberry Pi Packet Sniffer" width="600" style="border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
</p>

<h1 align="center" style="text-shadow: 1px 1px 2px #00000020;">Raspberry Pi Dockerized Packet Sniffer</h1>
<p align="center"><strong>Transform Your Raspberry Pi into a Comprehensive Network Security and Monitoring Tool</strong></p>

<p align="center">
  This project provides a containerized solution for network monitoring and security using PiHole and a Packet Sniffer. Manage network traffic efficiently with enhanced control and flexibility.
</p>

<div align="center">
  <img src="https://img.shields.io/badge/Raspberry%20Pi-4B%2F5-red" alt="Supported Pi Versions">
  <img src="https://img.shields.io/badge/Storage-32GB+%20microSD-blue" alt="Minimum Storage">
  <img src="https://img.shields.io/badge/Containers-PiHole%2FPacket%20Sniffer-green" alt="Required Containers">
  <img src="https://img.shields.io/badge/Power%20Usage-~5W-yellow" alt="Approx Power Consumption">
</div>

---

## Features

- **Network Security and Monitoring**: Block advertisements and malicious domains using PiHole while monitoring network traffic with a custom Packet Sniffer.
- **Containerized Deployment**: Leverage Docker containers for portability, scalability, and ease of use.
- **Customizable and Flexible**: Tailor configurations to meet specific requirements.
- **Energy Efficient**: Designed to run on Raspberry Pi hardware with minimal power consumption.

---

## Table of Contents

- [Requirements](#requirements)
- [Setup Guide](#setup-guide)
- [PiHole Configuration](#pihole-configuration)
- [Packet Sniffer Configuration](#packet-sniffer-configuration)
- [Security Implementation](#security-implementation)
- [Results and Analysis](#results-and-analysis)
- [Advanced Features](#advanced-features)
- [Performance and Maintenance](#performance-and-maintenance)
- [Troubleshooting](#troubleshooting)

---

## Requirements

### Core Components

| Component          | Specification         | Purpose                              |
|--------------------|-----------------------|--------------------------------------|
| Raspberry Pi       | 4B (2GB+ RAM)         | System processor                     |
| Storage            | 32GB+ microSD         | Base system                          |
| External Drive     | SSD/HDD               | Optional, for additional storage     |
| Operating System   | Raspberry Pi OS Lite  | Lightweight operating system         |
| Network Connection | Ethernet cable        | Stable network connectivity          |

---

## Setup Guide

### Step 1: Install the Operating System

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
2. Select **"Raspberry Pi OS Lite (64-bit)"**.
3. Enable SSH during installation for remote management.

### Step 2: Configure the System

Update and upgrade the system packages:

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3: Install Docker

Install Docker using the following command:

```bash
curl -sSL https://get.docker.com | sh
```

For convenience, add your user to the Docker group (optional):

```bash
sudo usermod -aG docker $USER
```

### Step 4: Install Docker Compose

Install Docker Compose to manage multi-container setups:

```bash
sudo apt install docker-compose
```

---

## PiHole Configuration

### Step 1: Define the PiHole Service

Add the following configuration to the `docker-compose.yml` file:

```yaml
pihole:
  container_name: pihole
  image: pihole/pihole:latest
  environment:
    TZ: 'Europe/Lisbon'
    WEBPASSWORD: '${PIHOLE_WEBPASSWORD}'
  volumes:
    - './pihole/etc-pihole:/etc/pihole'
    - './pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
  ports:
    - "53:53/tcp"
    - "53:53/udp"
    - "80:80"
  restart: unless-stopped
  cap_add:
    - NET_ADMIN
```

### Step 2: Update the PiHole Password

You can set or reset the PiHole admin password using:

```bash
docker exec -it pihole pihole -a setpassword "newsecurepassword" # This is ony available for the current session, if the rapsberry pi or docker is rebooted, the password will be deleted
```

To make the password `non volatile`, you must create a .env file, this file is encrypted and will keep you password no matter what happens:

```bash
# Create a new .env file with this content
PIHOLE_WEBPASSWORD=newsecurepassword
```

---

## Packet Sniffer Configuration

### Step 1: Define the Packet Sniffer Service

Add the following configuration to the `docker-compose.yml` file:

```yaml
packet_capture:
  container_name: packet-sniffer
  image: packet-sniffer
  network_mode: "host"
  volumes:
    - './packet-sniffer/pcap:/data'
  restart: unless-stopped
```

### Step 2: Create the Dockerfile for the Packet Sniffer

Save the following Dockerfile in the `packet-sniffer/` directory:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y tcpdump
WORKDIR /data
CMD ["tcpdump", "-i", "eth0", "-w", "/data/capture.pcap"]
```

---

## Security Implementation

### Restrict PiHole Interface Access

To restrict access to the PiHole web interface, configure a firewall using UFW:

```bash
sudo apt install ufw
sudo ufw allow from 192.168.0.0/24 to any port 80 proto tcp
sudo ufw enable
```

Disable external exposure by avoiding wide port mappings, such as `0.0.0.0:80`.

### Avoid Host Network Mode

Where possible, use a dedicated Docker bridge network for increased isolation:

```bash
docker network create --driver bridge secure_net
```

Connect containers to the bridge network as required.

### Encrypt Captured Data

If `.pcap` files contain sensitive information, encrypt them using GPG:

```bash
gpg -c ./packet-sniffer/pcap/capture.pcap
```

### Automate Container Updates

Use Watchtower to automate updates for your containers:

```yaml
watchtower:
  image: containrrr/watchtower
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  restart: unless-stopped
```

---

## Results and Analysis

### Viewing Captured Packets

The captured packets are saved in the `packet-sniffer/pcap/capture.pcap` file. You can analyze these using:

1. **Graphical Interface**: Transfer the file to your PC and open it with [Wireshark](https://www.wireshark.org/).
2. **Command Line**: Install TShark on the Raspberry Pi:

   ```bash
   sudo apt update
   sudo apt install tshark
   tshark -r ./packet-sniffer/pcap/capture.pcap
   ```

---

## Advanced Features

- **Scheduled Captures**: Automate packet captures using cron jobs:

  ```bash
  @hourly docker exec packet-sniffer tcpdump -i eth0 -G 3600 -w "/data/capture-%Y%m%d%H%M.pcap"
  ```

- **Custom Filters**: Capture specific traffic types by adding filters:

  ```bash
  tcpdump -i eth0 port 80 or port 53
  ```

- **External Storage**: Mount an external SSD for persistent storage:

  ```bash
  sudo mount /dev/sda1 /mnt/external
  ```

---

## Performance and Maintenance

- Use `raspi-config` to enable safe overclocking and monitor system temperatures.
- Clean up unused Docker images and containers:

  ```bash
  docker system prune -a
  ```

- Update all containers with a single command:

  ```bash
  docker-compose pull && docker-compose up -d
  ```

---

## Troubleshooting

- If port 53 is in conflict, ensure no other DNS service (e.g., `systemd-resolved`) is using the port.
- If permissions are denied while capturing packets, confirm the container has `CAP_NET_ADMIN` or is running as root.
- If the interface `eth0` does not exist, use `ip a` to identify the correct interface name (e.g., `enp0s3`).
- If the PiHole web interface is inaccessible, check firewall rules, Docker port mappings, and the Raspberry Pi's IP address.

---

<p align="center">
  <strong>Leverage the power of your Raspberry Pi to enhance network security and monitoring.</strong>
</p>