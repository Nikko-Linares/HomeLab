# 🏠 Home Lab

A self-hosted home lab built from the ground up to learn Linux administration, networking, containerization, and security. All services run on a local Ubuntu Server VM with secure remote access via WireGuard VPN.

---

## 🖥️ Hardware

| Device | Role | OS |
|---|---|---|
| Desktop Computer | Hosts the Ubuntu Server VM (VirtualBox) | Windows |
| Laptop | VPN client and management workstation | Linux Mint |
| iPhone | Mobile VPN client | iOS |

---

## 📦 Services

| Service | Category | Description |
|---|---|---|
| WireGuard | Networking | Self-hosted VPN server for secure remote access |
| PiHole | Networking | Network-wide ad blocker and DNS sinkhole |
| Nextcloud | Cloud | Self-hosted file storage and cloud sync |
| Vaultwarden | Security | Self-hosted Bitwarden-compatible password manager |
| Nginx | Networking | Reverse proxy with HTTPS via self-signed certificates |
| Fail2ban | Security | Intrusion prevention — bans brute force IPs |
| UFW | Security | Firewall — controls incoming and outgoing traffic |
| Docker | Infrastructure | Containerizes all home lab services |
| SSH | Security | Key-based secure remote access to the VM |

---

## 🗺️ Network Topology

```
Internet
    │
    ▼
Router (LAN)
    │
    ▼
Desktop → VirtualBox VM (Ubuntu Server)
               │
               ├── WireGuard Server (10.0.0.1/24)
               │         │
               │   ┌──────┴──────────┐
               │   ▼                 ▼
               │ Laptop (10.0.0.2)  iPhone (10.0.0.3)
               │
               ├── PiHole (DNS: port 53, Web UI: port 8082)
               ├── Nextcloud (port 8080 → HTTPS 8444)
               ├── Vaultwarden (port 8081 → HTTPS 8445)
               └── Nginx Reverse Proxy (HTTPS: 8443, 8444, 8445)
```

---

## 🎯 Goals

- ✅ Learn Linux administration through hands-on server management
- ✅ Build and manage a secure VPN for remote access
- ✅ Practice containerization with Docker
- ✅ Self-host cloud services for full data ownership and privacy
- ✅ Implement network security with firewall rules and intrusion prevention
- ✅ Learn reverse proxy configuration and HTTPS/SSL
- ✅ Explore penetration testing with Kali Linux
- 🔄 Migrate from VirtualBox VM to a Raspberry Pi for 24/7 uptime
- 🔄 Add a self-hosted Git server (Gitea)
- 🔄 Configure internal DNS for friendly domain names

---

## 📁 Documentation

### Containers
| File | Description |
|---|---|
| [docker.md](Containers/docker.md) | Docker installation, container management, and all running services |
| [reverse-proxy.md](Containers/reverse-proxy.md) | Nginx reverse proxy setup with self-signed SSL certificates |

### Linux
| File | Description |
|---|---|
| [linux-mint.md](Linux/linux-mint.md) | Linux Mint laptop setup as a WireGuard VPN client |
| [ubuntu-server.md](Linux/ubuntu-server.md) | Ubuntu Server VM configuration and service management |
| [kali-linux.md](Linux/kali-linux.md) | Kali Linux VM for penetration testing and network scanning |

### Virtual OS
| File | Description |
|---|---|
| [virtualbox.md](VirtualOS/virtualbox.md) | VirtualBox VM setup, configuration, and management |

### Network
| File | Description |
|---|---|
| [vpn-setup.md](network/vpn-setup.md) | WireGuard VPN server and client configuration for laptop and iPhone |

### Security
| File | Description |
|---|---|
| [firewall.md](security/firewall.md) | UFW firewall rules and iptables configuration |
| [fail2ban.md](security/fail2ban.md) | Fail2ban intrusion prevention for SSH protection |

### Servers
| File | Description |
|---|---|
| [Ad-Blocker.md](servers/Ad-Blocker.md) | PiHole DNS ad blocker setup and blocklist management |
| [cloud-server.md](servers/cloud-server.md) | Nextcloud file storage and Vaultwarden password manager |

---

## 🔐 Security Highlights

- **WireGuard VPN** — all remote traffic encrypted through a self-hosted tunnel
- **PiHole** — 106,000+ ad and tracker domains blocked at the DNS level
- **Fail2ban** — automatic IP banning after repeated failed SSH login attempts
- **SSH key authentication** — password-based SSH login disabled
- **UFW firewall** — only essential ports open (22, 51820, 8443-8445)
- **HTTPS everywhere** — all web services served over SSL via Nginx
- **Self-hosted passwords** — Vaultwarden keeps credentials off third-party servers

---

## 🛠️ Tech Stack

![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat&logo=nginx&logoColor=white)
![WireGuard](https://img.shields.io/badge/WireGuard-88171A?style=flat&logo=wireguard&logoColor=white)
![Nextcloud](https://img.shields.io/badge/Nextcloud-0082C9?style=flat&logo=nextcloud&logoColor=white)
![Kali](https://img.shields.io/badge/Kali_Linux-557C94?style=flat&logo=kalilinux&logoColor=white)

---

## 📌 Key Lessons Learned

- Bridged Adapter in VirtualBox is essential — NAT blocks all incoming VPN and SSH connections
- Docker's iptables rules can silently conflict with WireGuard — explicit FORWARD rules are required
- Port 53 is hardcoded for DNS — PiHole must run on port 53, not an alternative
- Self-signed certificates work well for a local home lab — Firefox handles overrides better than Opera GX
- MTU tuning (1420) significantly improves WireGuard performance by preventing packet fragmentation
- `systemd-resolved` occupies port 53 on Ubuntu by default and must be disabled for PiHole
- A single character typo (wrong key, wrong port, wrong syntax) can cause hours of debugging — read error messages carefully
