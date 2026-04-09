# Homelab

Personal homelab built on a Dell OptiPlex 7070 Micro running Proxmox VE. Used for learning, self-hosting services, and hands-on practice for CompTIA Network+.

## Hardware

| Component | Details |
|-----------|---------|
| Host | Dell OptiPlex 7070 Micro |
| CPU | Intel i5-9500T |
| RAM | 16GB DDR4 |
| Storage (OS) | 256GB NVMe SSD |
| Storage (Data) | 2TB Crucial SSD (SATA) |
| Hypervisor | Proxmox VE (bare metal) |

## Services Running

| Service | Platform | Purpose |
|---------|----------|---------|
| Ubuntu Server VM | Proxmox VM | Base for all Docker services |
| Pi-hole | Docker | Network-wide DNS + ad blocking |
| Jellyfin | Docker | Media server (1.7TB NFS storage) |
| Nginx Proxy Manager | Docker | Reverse proxy / local domain routing |
| Portainer | Docker | Docker management UI |
| ntfy | Docker | Self-hosted push notification server |
| Uptime Kuma | Docker | Service monitoring + alerting |
| pfSense | Proxmox VM | Edge firewall, VLANs, network segmentation, WireGuard VPN |
| Tailscale | Host + VM | Remote access VPN mesh |
| Samba | Ubuntu VM | NAS-style network share (SMB) |

## Network Architecture

```
Internet
    |
Deco BE67 Mesh Router (ISP Gateway)
    |
    ├── Proxmox Host (accessible via Tailscale)
    |       ├── 2TB SSD mounted at /mnt/data
    |       |     └── NFS export → Ubuntu VM (/mnt/media)
    |       └── pfSense VM
    |               ├── WAN — gets IP from ISP router via DHCP
    |               └── LAN — 172.16.0.0/24 (pfSense manages this subnet)
    |                       └── Ubuntu Server VM (172.16.0.52)
    |                               ├── Pi-hole (port 8888)
    |                               ├── Jellyfin (port 8096)
    |                               ├── Nginx Proxy Manager (ports 80/443/81)
    |                               ├── Portainer (port 9000)
    |                               ├── ntfy (port 8090)
    |                               ├── Uptime Kuma (port 3001)
    |                               └── Samba (port 445 — SMB share)
    |
    └── IoT VLAN 10 — 192.168.10.0/24 (isolated via pfSense)
```

### Proxmox Bridge Layout

| Bridge | Role | Connected To |
|--------|------|-------------|
| vmbr0 | WAN bridge | Physical NIC (nic0) → ISP router |
| vmbr1 | LAN bridge | Internal only → pfSense LAN → homelab VMs |

### pfSense Port Forwards (WAN → LAN)

| Service | External Port | Internal Target |
|---------|--------------|----------------|
| Portainer | 9000 | 172.16.0.52:9000 |
| Jellyfin | 8096 | 172.16.0.52:8096 |
| Pi-hole | 8888 | 172.16.0.52:8888 |
| NPM Admin | 81 | 172.16.0.52:81 |
| WireGuard VPN | 51820/UDP | pfSense WireGuard tunnel |

## Projects

### 1. Proxmox VE Setup
- Installed Proxmox VE bare metal on OptiPlex 7070
- Configured VM templates, storage pools, and networking bridges
- Disabled sleep/hibernation for 24/7 uptime
- Installed Tailscale on Proxmox host for secure remote management

### 2. Ubuntu Server VM
- Deployed Ubuntu Server 22.04 LTS as Proxmox VM
- Configured static IP via Netplan on `ens18` interface
- Serves as Docker host for all containerized services
- Migrated between network subnets as homelab architecture evolved

### 3. Pi-hole DNS Server
- Network-wide ad and tracker blocking via DNS filtering
- Moved from default port 80 to port 8888 to free ports for NPM
- Custom local DNS records for clean homelab domain resolution
- Set as primary DNS on router; Google DNS as secondary fallback

### 4. Docker Service Stack
- Deployed Jellyfin, Nginx Proxy Manager, and Portainer via Docker
- Resolved port conflicts between Pi-hole (HTTP) and NPM (80/443)
- NPM reverse proxy routes `jellyfin.home` to Jellyfin container

### 5. pfSense Firewall + Network Segmentation
- pfSense CE installed as Proxmox VM
- Promoted to true edge firewall with dedicated WAN/LAN bridge separation
  - `vmbr0` — WAN bridge connected to physical NIC (ISP-facing)
  - `vmbr1` — LAN bridge (internal only, pfSense manages subnet)
- All homelab VMs moved behind pfSense LAN (172.16.0.0/24)
- Firewall rules controlling traffic between zones
- Disabled "Block Private Networks" on WAN (required for double-NAT behind ISP mesh router)

### 6. VLAN Segmentation (IoT Isolation)
- Created VLAN 10 for IoT device isolation
- pfSense OPT1 interface: `192.168.10.1/24`
- DHCP scope: `192.168.10.100–200`
- Firewall rules:
  - Block IoT → LAN (prevents IoT devices accessing homelab)
  - Allow IoT → Internet (IoT devices maintain internet access)
  - Block LAN → IoT (no uninitiated access from LAN to IoT)

### 7. WireGuard VPN
- WireGuard tunnel configured in pfSense (HomeVPN)
- Listen port 51820 UDP
- VPN subnet: `10.0.0.0/24`
- Port forwarded through ISP router to pfSense WAN
- Enables encrypted remote access to homelab from any network

### 8. Nginx Proxy Manager
- Reverse proxy for clean local domain routing
- `jellyfin.home` → Jellyfin container (port 8096)
- Works alongside Pi-hole DNS for local name resolution

### 9. Tailscale Remote Access
- Zero-config mesh VPN deployed on Proxmox host and Ubuntu VM
- Encrypted remote access without port forwarding
- Survives network restructuring — operates independently of local subnet changes
- Used as fallback remote access when WireGuard port forwarding unavailable

### 10. Packet Analysis with Wireshark
- Captured and analyzed live network traffic on Windows
- Identified DNS query/response cycles, TTLs, and record types
- Recognized CDN traffic from Akamai, Fastly, and Cloudflare
- Identified Tailscale DNS resolver in local traffic
- Applied security-focused filters: NXDOMAIN detection, non-standard DNS servers, TCP RST analysis
- Followed TCP streams; compared plaintext HTTP vs encrypted HTTPS
- Demonstrated why HTTPS is required — HTTP streams visible in plaintext

### 11. NAS Storage Build (2TB SSD)
- Installed 2TB Crucial SATA SSD into OptiPlex 7070 second bay
- Partitioned and formatted with ext4; mounted at `/mnt/data` on Proxmox host
- Registered as Proxmox storage pool (`ssd-storage`) for VM disks, backups, and ISOs
- Set up NFS server on Proxmox; exported `/mnt/data/media` to 172.16.0.0/24 subnet
- Ubuntu VM auto-mounts NFS share at `/mnt/media` on boot via `/etc/fstab` with `_netdev`
- Proxmox host assigned static IP `172.16.0.2` on vmbr1 for internal NFS routing
- Jellyfin Docker container updated to use NFS-backed `/mnt/media` for library storage
- Result: 1.7TB available for media storage, persistent across reboots

### 12. Samba NAS Share (Windows Network Drive)
- Installed Samba on Ubuntu VM; configured `/mnt/media` as SMB share
- Share accessible from Windows via `\\<tailscale-ip>` in File Explorer
- Mapped as permanent network drive (M:) on Windows laptop
- Enables drag-and-drop file transfers directly to homelab storage
- Jellyfin auto-detects new media dropped into `/mnt/media/movies`, `/tv`, `/music`

### 13. Monitoring & Alerting (Uptime Kuma + ntfy)
- Deployed ntfy (self-hosted push notification server) on Docker port 8090
- Deployed Uptime Kuma (uptime monitoring) on Docker port 3001
- Uptime Kuma monitors: Jellyfin, Portainer, Pi-hole, Nginx Proxy Manager, Proxmox
- ntfy integrated as Uptime Kuma notification channel
- Push notifications delivered to phone via ntfy app over Tailscale
- Full alerting pipeline: service down → Uptime Kuma detects → ntfy → phone notification

## Skills Demonstrated

- **Virtualization**: Proxmox VE, VM lifecycle management, bridge networking (vmbr0/vmbr1)
- **Networking**: VLANs, subnetting, DHCP, DNS, NAT, firewall rules, port forwarding, double-NAT troubleshooting, NFS, SMB/Samba
- **Linux**: Ubuntu Server, Netplan static IP config, systemd, package management, tcpdump, NFS exports, fstab
- **Containerization**: Docker, docker-compose, Portainer, container networking
- **Storage**: SSD installation, partition management, ext4 formatting, NFS server/client, Samba file sharing
- **Security**: Network segmentation, VLAN isolation, firewall rules, Pi-hole DNS filtering, WireGuard VPN, packet analysis
- **Remote Access**: Tailscale mesh VPN, WireGuard, SSH, SMB over Tailscale
- **Monitoring**: Uptime Kuma, ntfy push notifications, service health checks
- **Troubleshooting**: ARP debugging, bridge interface issues, DHCP conflicts, pfSense firewall rule ordering, NFS routing, fstab boot hang recovery

## Certifications & Education

- CompTIA Network+ (in progress, target April 2026)
- Long Beach City College — COSN 205 (Unix/Linux Fundamentals), COSN 299 (Networking & Security Capstone), COSS 71 (Network Security Fundamentals)
