# Homelab

Personal homelab built on a Dell OptiPlex 7070 Micro running Proxmox VE. Used for learning, self-hosting services, and hands-on practice for CompTIA Network+.

## Hardware

| Component | Details |
|-----------|---------|
| Host | Dell OptiPlex 7070 Micro |
| CPU | Intel i5-9500T |
| RAM | 16GB |
| Storage | 256GB NVMe SSD |
| Hypervisor | Proxmox VE (bare metal) |

## Services Running

| Service | Platform | Purpose |
|---------|----------|---------|
| Ubuntu Server VM | Proxmox VM | Base for all Docker services |
| Pi-hole | Docker | Network-wide DNS + ad blocking |
| Jellyfin | Docker | Media server |
| Nginx Proxy Manager | Docker | Reverse proxy / local domain routing |
| Portainer | Docker | Docker management UI |
| pfSense | Proxmox VM | Firewall, VLANs, network segmentation |
| Tailscale | Host + VM | Remote access VPN |

## Network Architecture

```
Internet
    |
TP-Link Router (192.168.68.1)
    |
    ├── Proxmox Host (192.168.68.100) [Tailscale: 100.74.48.44]
    |       └── pfSense VM
    |               ├── LAN (192.168.68.x)
    |               └── IoT VLAN 10 (192.168.10.0/24)
    |
    └── Ubuntu Server VM (192.168.68.52) [Tailscale: 100.115.128.31]
            ├── Pi-hole (port 8888)
            ├── Jellyfin (port 8096) → jellyfin.home
            ├── Nginx Proxy Manager (ports 80/443/81)
            └── Portainer (port 9000)
```

## Projects

### Proxmox VE Setup
- Installed Proxmox VE bare metal on OptiPlex 7070
- Configured VM templates, storage pools, networking
- Installed Tailscale for remote Proxmox access from anywhere
- Disabled sleep for 24/7 uptime

### Ubuntu Server + Static IP
- Deployed Ubuntu Server VM with static IP via Netplan
- Serves as the Docker host for all containerized services

### Pi-hole DNS Server
- Network-wide ad blocking via DNS filtering
- Custom local DNS records for clean homelab URLs
- Moved from port 80 to 8888 to allow Nginx Proxy Manager

### Docker Service Stack
- Deployed Jellyfin, Nginx Proxy Manager, and Portainer via Docker
- Resolved port conflicts between Pi-hole and NPM
- Clean local domain access via Pi-hole DNS + NPM reverse proxy

### Tailscale Remote Access
- Zero-config VPN across all homelab devices
- Access Proxmox, Jellyfin, Portainer from anywhere without port forwarding

### pfSense Firewall + VLAN Segmentation
- pfSense VM with WAN/LAN interfaces
- IoT VLAN (VLAN 10) isolating smart devices from main LAN
- Firewall rules: Block IoT-to-LAN, Allow IoT-to-Internet
- DHCP server for each VLAN

### Nginx Proxy Manager
- Reverse proxy routing clean URLs to backend services
- Local DNS records in Pi-hole for homelab domain resolution
- Example: http://jellyfin.home routes to Jellyfin on port 8096

## Skills Demonstrated

- **Virtualization**: Proxmox VE, VM deployment, snapshots
- **Networking**: VLANs, subnetting, DHCP, DNS, NAT, firewall rules
- **Linux**: Ubuntu Server, Netplan, systemd, package management
- **Containerization**: Docker, docker-compose, Portainer
- **Security**: Network segmentation, firewall rules, Pi-hole DNS filtering
- **Remote Access**: Tailscale VPN, SSH

## Certifications & Education

- CompTIA Network+ (in progress, target April 2026)
- Long Beach City College — COSN 205 (Unix/Linux), COSN 299 (Networking Capstone), COSS 71 (Network Security)
