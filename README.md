# Home Network Lab with Samba, VPN, and Pi-hole

> **Note:** For security and privacy reasons, all IP addresses, MAC addresses, and hostnames in this documentation have been changed from their actual values. The network configuration principles remain accurate, but the specific addresses shown are examples only.

## Project Overview

This repository documents my home network lab, featuring a Samba file share, WireGuard VPN, and an existing Pi-hole ad blocker on a Raspberry Pi 5 running Ubuntu 24.04 LTS. I’ve configured a managed switch and router to create a secure, high-performance network, showcasing my skills in networking, Linux administration, and troubleshooting.

## System Specifications


*My Raspberry Pi 5, switch, and router setup*

- **Hardware**: Raspberry Pi 5 (4GB RAM)
- **Storage**: 64GB microSD card
- **Operating System**: Ubuntu 24.04 LTS (64-bit)
- **Network**: Static IP configuration on Ethernet (eth0)
- **Router**: AT&T Fiber router
- **Switch**: TP-Link TL-SG108E (8-port managed)

## Installation Steps

### 1. Update System
```yaml
sudo apt update && sudo apt upgrade -y
sudo apt install curl -y
```

### 2. Leverage Existing Pi-hole
Pi-hole was already installed and running in Docker, managing DHCP and DNS.

### 3. Install Samba
```yaml
sudo apt install samba samba-common-bin -y
sudo mkdir -p /home/user/shared
sudo chmod 777 /home/user/shared
```
- Configured guest access in `/etc/samba/smb.conf`.

### 4. Install WireGuard

```yaml
sudo apt install wireguard -y
wg genkey | sudo tee /etc/wireguard/privatekey
```
- Set up VPN server and clients.

pi hole dashboard

Pi-hole admin interface showing ad-blocking statistics

## Implementation Process

### 1. Network Configuration

I configured a static IP for the Raspberry Pi 5 using netplan by editing `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  ethernets:
    eth0:
      addresses: [10.0.0.10/24]
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

- Configured QoS on the TP-Link switch: gaming PCs (Ports 2-3, Priority 4), Pi 5 (Port 7, Priority 1).
- Disabled router DHCP to centralize control with Pi-hole.

### 2. Pi-hole Integration
- Utilized existing Pi-hole setup with DHCP range `10.0.0.100-199`.
- Integrated with WireGuard to extend ad-blocking to my iPhone over cellular data.

### 3. Samba Configuration
- Set up a file share at `/home/user/shared`, accessible at `\\10.0.0.10\shared`.
- Adjusted permissions for cross-platform access:

```yaml
sudo chmod 755 /home/user
```

### 4. WireGuard VPN
- Configured server on Pi 5 (`172.16.0.1/24`, port `51234`):

```yaml
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

- Set up iPhone client (`172.16.0.2`) with on-demand activation for cellular.

sambda file share

Samba file share accessed via iPhone over VPN

## Troubleshooting Challenges & Solutions

### 1. Samba Access Issues

### Challenge: iPhone and Windows couldn’t access the share due to permission errors.

### Solution:

Fixed parent directory permissions: `sudo chmod 755 /home/user`.
Disabled Windows SMB signing:

```yaml
Set-SmbClientConfiguration -RequireSecuritySignature $false -Force
```

## 2. Guest Wi-Fi Isolation

### Challenge: Router lacks VLAN support, breaking switch VLAN attempts.

### Solution:

- Disabled 802.1Q VLANs on the TP-Link switch to restore connectivity; planned for future router upgrade.

## 3. WireGuard Handshake Failure

### Challenge: iPhone VPN failed to connect.

### Solution:

Corrected key mismatch, verified with `wg` and `tcpdump`.

## Key Achievements

- Built a Samba share supporting all file types (e.g., 2.61 GB video), accessible locally and via VPN.
- Integrated an existing Pi-hole for network-wide ad-blocking, extended to my iPhone over VPN on cellular data.
- Configured QoS on a managed switch for optimized gaming performance.

## Skills Demonstrated
- Networking: Subnetting, QoS, DHCP/DNS, VLAN exploration, port forwarding.
- Linux Administration: Service management (`systemctl`), file permissions, netplan.
- Security: WireGuard VPN, ad-blocking over VPN, Samba setup.
- Troubleshooting: Resolved permission issues, SMB conflicts, VPN failures.
- Tools: Docker, Samba, WireGuard, FE File Explorer Pro, Termius.

## Future Improvements
- Upgrade to a VLAN-capable router (e.g., Ubiquiti EdgeRouter) for guest isolation.
- Add FileBrowser for web-based file access.
- Automate backups for Pi-hole and Samba configs.

## Resources
- https://docs.pi-hole.net/
- https://www.wireguard.com/
- https://www.samba.org/samba/docs/
