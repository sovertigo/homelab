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
| pfSense | Proxmox VM | Edge firewall, VLANs, network segmentation, WireGuard VPN |
| Tailscale | Host + VM | Remote access VPN |

## Network Architecture

```
Internet
    |
Deco BE67 Mesh Router (192.168.68.1)
    |
    ├── Proxmox Host (192.168.68.100) [Tailscale: 100.74.48.44]
    |       └── pfSense VM (WAN: 192.168.68.54)
    |               └── pfSense LAN (172.16.0.1/24)
    |                       └── Ubuntu Server VM (172.16.0.52) [Tailscale: 100.115.128.31]
    |                               ├── Pi-hole (port 8888)
    |                               ├── Jellyfin (port 8096)
    |                               ├── Nginx Proxy Manager (ports 80/443/81)
    |                               └── Portainer (port 9000)
    |
    └── IoT VLAN 10 (192.168.10.0/24) — via pfSense
```

### Access Methods

| Service | Local (via pfSense port forward) | Remote (Tailscale) |
|---------|-----------------------------------|--------------------|
| pfSense WebUI | http://192.168.68.54 | N/A |
| Portainer | http://192.168.68.54:9000 | http://100.115.128.31:9000 |
| Jellyfin | http://192.168.68.54:8096 | http://100.115.128.31:8096 |
| Pi-hole | http://192.168.68.54:8888/admin | http://100.115.128.31:8888/admin |
| NPM Admin | http://192.168.68.54:81 | http://100.115.128.31:81 |

## Projects

### Proxmox VE Setup
- Installed Proxmox VE bare metal on OptiPlex 7070
- Configured VM templates, storage pools, networking
- Installed Tailscale for remote Proxmox access from anywhere
- Disabled sleep for 24/7 uptime

### Ubuntu Server + Static IP
- Deployed Ubuntu Server VM with static IP via Netplan
- Serves as the Docker host for all containerized services
- Migrated from direct LAN to pfSense LAN subnet (172.16.0.0/24)

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
- Survived network restructure — operates independently of local subnet changes

### pfSense Firewall + Network Segmentation
- pfSense VM promoted to true homelab edge firewall
- Separate vmbr0 (WAN bridge) and vmbr1 (LAN bridge) in Proxmox
- All homelab services routed through pfSense LAN (172.16.0.0/24)
- IoT VLAN (VLAN 10) isolating smart devices from main LAN
- Firewall rules: Block IoT-to-LAN, Allow IoT-to-Internet
- Port forwarding for all services through pfSense WAN
- WireGuard VPN tunnel configured (HomeVPN, port 51820)
- Blocked private network rule disabled (required for double-NAT behind ISP router)

### Nginx Proxy Manager
- Reverse proxy routing clean URLs to backend services
- Local DNS records in Pi-hole for homelab domain resolution
- Example: http://jellyfin.home routes to Jellyfin on port 8096

### Packet Analysis with Wireshark
- Captured and analyzed live network traffic on Windows laptop
- Identified DNS queries, TCP handshakes, CDN traffic (Akamai, Fastly, Cloudflare)
- Followed HTTP/HTTPS streams; demonstrated plaintext HTTP vs encrypted HTTPS
- Identified Tailscale DNS resolver (100.100.100.100) in traffic
- Applied security filters: NXDOMAIN detection, non-standard DNS servers, RST flood indicators

## Skills Demonstrated

- **Virtualization**: Proxmox VE, VM deployment, bridge networking (vmbr0/vmbr1)
- **Networking**: VLANs, subnetting, DHCP, DNS, NAT, firewall rules, port forwarding, double-NAT
- **Linux**: Ubuntu Server, Netplan, systemd, package management, tcpdump
- **Containerization**: Docker, docker-compose, Portainer
- **Security**: Network segmentation, firewall rules, Pi-hole DNS filtering, WireGuard VPN, packet analysis
- **Remote Access**: Tailscale VPN, SSH, WireGuard
- **Troubleshooting**: Network connectivity debugging, ARP analysis, bridge interface issues

## Certifications & Education

- CompTIA Network+ (in progress, target April 2026)
- Long Beach City College — COSN 205 (Unix/Linux), COSN 299 (Networking Capstone), COSS 71 (Network Security)
