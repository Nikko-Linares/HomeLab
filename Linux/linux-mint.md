# Linux Mint (Laptop)

## Goals
- Learn a new Linux-based operating system hands-on
- Practice using the terminal for real-world tasks
- Use the laptop as a WireGuard VPN client and home lab management workstation

---

## Tools Used
| Tool | Role |
|---|---|
| USB Drive (16 GB) | Bootable Linux Mint installer |
| Desktop Computer | Used to create the bootable USB |
| Laptop | Primary device running Linux Mint |
| WireGuard | VPN client connecting to home lab |
| Firefox | Browser for accessing self-signed cert services |

---

## Installation

### Step 1 — Download Linux Mint
Download the latest ISO from the official site:
```
https://linuxmint.com/download.php
```
> **Note:** The ISO is around 2–3 GB — a USB drive of at least 5 GB is required.

### Step 2 — Create Bootable USB
Use **Balena Etcher** or **Rufus** on your desktop to flash the ISO to the USB drive:
```
https://etcher.balena.io
```

### Step 3 — Boot from USB
1. Insert the USB into the laptop
2. Restart and enter BIOS/UEFI (usually `F2`, `F12`, or `Del` on boot)
3. Set USB as the first boot device
4. Boot into the Linux Mint live environment
5. Double-click **Install Linux Mint** and follow the setup wizard

---

## Role in Home Lab
- WireGuard VPN client (`10.0.0.2`)
- SSH access to the Ubuntu Server VM
- Browser access to home lab services over the VPN tunnel
- Primary workstation for managing all home lab services

---

## Network Interfaces
| Interface | Description |
|---|---|
| `lo` | Loopback |
| `wlp2s0` | WiFi (primary network interface) |
| `wg0` | WireGuard VPN tunnel |

---

## WireGuard VPN Client Setup

### Installation
```bash
sudo apt install wireguard wireguard-tools -y
```

### Key Generation
```bash
sudo sh -c 'wg genkey | tee /etc/wireguard/laptop_private.key | wg pubkey > /etc/wireguard/laptop_public.key'
sudo cat /etc/wireguard/laptop_private.key
sudo cat /etc/wireguard/laptop_public.key
```

### Client Configuration
Location: `/etc/wireguard/wg0.conf`
```ini
[Interface]
PrivateKey = <laptop_private_key>
Address = 10.0.0.2/24
DNS = 10.0.0.1
MTU = 1420
Table = off

[Peer]
PublicKey = <vm_public_key>
Endpoint = <VM_real_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```

### Connect / Disconnect
```bash
sudo wg-quick up wg0      # Connect to VPN
sudo wg-quick down wg0    # Disconnect from VPN
sudo wg show              # Check connection status
```

---

## SSH Access to VM

```bash
ssh username@<VM_real_IP>
```

### SSH Key Setup
```bash
ssh-keygen -t ed25519 -C "homelab-laptop"
ssh-copy-id username@<VM_real_IP>
```

---

## Useful Terminal Commands

```bash
# Check network interfaces and IPs
ip a

# Check routing table
ip route

# Test connectivity
ping -c 5 10.0.0.1

# Check open ports
sudo ss -tulpn

# Check running processes
ps aux

# Update system
sudo apt update && sudo apt upgrade -y
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| WireGuard no handshake | Wrong public key in config | Re-derive with `cat private.key \| wg pubkey` and update config |
| VPN traffic not routing | `Table = off` missing | Add `Table = off` under `[Interface]` in `wg0.conf` |
| Slow VPN speeds | Packet fragmentation | Set `MTU = 1420` in `wg0.conf` |
| Self-signed cert blocked | Opera GX strict cert policy | Use Firefox instead — allows manual cert exceptions |
| SSH connection refused | VM not running or firewall blocking | Check VM is on and port 22 is allowed in UFW |

---

## Lessons Learned
- Linux Mint is an excellent starting point for learning Linux — familiar desktop environment with full terminal access
- The terminal is essential for managing a home lab — basic commands like `ip a`, `ping`, `ssh`, and `sudo` become second nature quickly
- A USB drive of at least 5 GB is needed for the Linux Mint ISO
- Firefox is more reliable than Opera GX when working with self-signed SSL certificates
- `Table = off` in the WireGuard config prevents routing conflicts when the laptop is on WiFi (`wlp2s0`)
- The laptop being on WiFi (`wlp2s0`) rather than ethernet doesn't affect VPN functionality but requires the correct interface to be referenced in routing
