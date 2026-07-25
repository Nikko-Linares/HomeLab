# Firewall (UFW)

## Goals
- Practice better security through firewall configuration
- Control incoming and outgoing traffic to the Ubuntu Server VM
- Protect home lab services from unauthorized access

---

## Tools Used
| Tool | Role |
|---|---|
| UFW (Uncomplicated Firewall) | Simple iptables-based firewall management |
| iptables | Advanced firewall rules (NAT, Docker, WireGuard) |
| Terminal | All configuration done via command line |

---

## What is UFW?
UFW stands for **Uncomplicated Firewall** — it is a frontend for `iptables` that makes managing firewall rules much simpler. By default, UFW denies all incoming traffic and allows all outgoing traffic, which is a secure starting point for any server.

---

## Installation & Initial Setup

```bash
# Install UFW
sudo apt install ufw -y

# Set default policies
sudo ufw default deny incoming    # Block all unsolicited incoming traffic
sudo ufw default allow outgoing   # Allow all outgoing traffic
```

---

## Allowed Ports

```bash
# SSH — remote terminal access
sudo ufw allow 22/tcp

# HTTP — standard web traffic
sudo ufw allow 80/tcp

# HTTPS — encrypted web traffic
sudo ufw allow 443/tcp

# WireGuard VPN
sudo ufw allow 51820/udp

# Nginx HTTPS (PiHole)
sudo ufw allow 8443/tcp

# Nginx HTTPS (Nextcloud)
sudo ufw allow 8444/tcp

# Nginx HTTPS (Vaultwarden)
sudo ufw allow 8445/tcp
```

### Port Reference

| Port | Protocol | Service | Purpose |
|---|---|---|---|
| 22 | TCP | SSH | Secure remote terminal access |
| 80 | TCP | HTTP | Standard web traffic |
| 443 | TCP | HTTPS | Encrypted web traffic |
| 51820 | UDP | WireGuard | VPN tunnel |
| 8443 | TCP | Nginx | HTTPS access to PiHole |
| 8444 | TCP | Nginx | HTTPS access to Nextcloud |
| 8445 | TCP | Nginx | HTTPS access to Vaultwarden |

---

## Enable & Verify

```bash
# Enable the firewall
sudo ufw enable

# Check status and all active rules
sudo ufw status verbose
```

---

## Managing Rules

```bash
# Allow a port
sudo ufw allow <port>/<protocol>

# Deny a specific port
sudo ufw deny <port>/<protocol>

# Delete a rule
sudo ufw delete allow <port>/<protocol>

# Reload rules after changes
sudo ufw reload

# Disable firewall temporarily
sudo ufw disable

# Reset all rules (start fresh)
sudo ufw reset
```

---

## Advanced: iptables Rules for WireGuard & Docker

UFW handles basic port rules, but WireGuard and Docker require additional `iptables` rules for proper traffic forwarding and NAT:

### WireGuard NAT (configured via wg0.conf PostUp/PostDown)
```bash
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```

### Docker Compatibility
```bash
sudo iptables -I DOCKER-USER -i wg0 -j ACCEPT
sudo iptables -I DOCKER-USER -o wg0 -j ACCEPT
```

### Persist iptables Rules
```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

---

## Checking What's Listening

```bash
# View all listening TCP ports
sudo ss -tlnp

# View all listening UDP ports
sudo ss -ulnp

# Check a specific port
sudo ss -tlnp | grep 22

# Find what process is using a port
sudo lsof -i :<port>
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| `ufw` command not found | UFW not installed | `sudo apt install ufw -y` |
| SSH locked out after enabling UFW | Port 22 not allowed before enabling | `sudo ufw allow 22/tcp` before `sudo ufw enable` |
| WireGuard not working after UFW enabled | Port 51820 not allowed | `sudo ufw allow 51820/udp` |
| Docker containers unreachable | Docker iptables conflict | Add explicit DOCKER-USER FORWARD rules |
| Rules not taking effect | UFW not reloaded | `sudo ufw reload` |

> **⚠️ Important:** Always allow port 22 (SSH) before enabling UFW — locking yourself out of SSH on a remote server means you lose access entirely.

---

## Lessons Learned
- The three foundational ports to allow are 22 (SSH), 80 (HTTP), and 443 (HTTPS) — they protect remote access and web traffic respectively
- Setting `default deny incoming` first ensures nothing slips through before rules are applied
- UFW simplifies iptables but Docker and WireGuard sometimes require direct iptables rules that bypass UFW
- Always run `sudo ufw status verbose` after making changes to confirm rules are applied correctly
- iptables rules added manually are lost on reboot — use `netfilter-persistent` to save them
