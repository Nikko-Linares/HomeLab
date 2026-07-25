# Ubuntu Server

## Overview
Ubuntu Server (Debian-based) is the core operating system running inside a VirtualBox VM. It hosts all home lab services including WireGuard, PiHole, Nextcloud, Vaultwarden, Nginx, and Fail2ban.

---

## Role in Home Lab
- WireGuard VPN server (`10.0.0.1/24`)
- Docker host for all containerized services
- Nginx reverse proxy for HTTPS
- DNS server via PiHole
- SSH server with key-based authentication

---

## System Information
| Component | Details |
|---|---|
| OS | Ubuntu Server (Debian-based) |
| Virtualization | VirtualBox (Bridged Networking) |
| Network Interface | `enp0s3` |
| WireGuard IP | `10.0.0.1/24` |

---

## Initial Setup

### Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Essential Tools
```bash
sudo apt install curl wget git net-tools ufw fail2ban -y
```

### Enable IP Forwarding
Required for WireGuard to route traffic:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Verify:
```bash
sudo sysctl net.ipv4.ip_forward
# Should return: net.ipv4.ip_forward = 1
```

---

## Services Running

| Service | Type | Port |
|---|---|---|
| WireGuard | systemd | 51820/udp |
| Docker | systemd | - |
| PiHole | Docker | 53, 8082 |
| Nextcloud | Docker | 8080 |
| Vaultwarden | Docker | 8081 |
| Nginx | systemd | 8443, 8444, 8445 |
| Fail2ban | systemd | - |
| SSH | systemd | 22 |

---

## Auto-Start Services on Boot
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl enable docker
sudo systemctl enable nginx
sudo systemctl enable fail2ban
```

Verify all enabled:
```bash
sudo systemctl is-enabled wg-quick@wg0
sudo systemctl is-enabled docker
sudo systemctl is-enabled nginx
sudo systemctl is-enabled fail2ban
```

---

## Useful Commands

```bash
# Check all running services
sudo systemctl list-units --type=service --state=running

# Check system resources
top
free -h
df -h

# Check open ports
sudo ss -tlnp
sudo ss -ulnp

# Check logs
sudo journalctl -u <service> -n 50
```

---

## Notes
- IP forwarding must be enabled for WireGuard to route client traffic
- Docker's iptables rules can interfere with WireGuard forwarding — add explicit FORWARD rules if needed
- VirtualBox must be set to **Bridged Adapter** mode for the VM to get a real network IP
