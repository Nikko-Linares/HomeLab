# VPN Setup (WireGuard)

## Goal
- Build a local VPN for security and privacy
- Connect laptop and iPhone to the home lab server securely
- Route all client traffic through the VM with PiHole handling DNS

---

## Tools Used
| Tool | Role |
|---|---|
| VirtualBox | Hosts the Ubuntu Server VM |
| Ubuntu Server | WireGuard VPN server |
| WireGuard | VPN protocol |
| SSH Keys | Secure remote access |
| PiHole | DNS-based ad blocking over VPN |
| qrencode | Generate QR codes for mobile clients |

---

## Network Topology

```
Internet
    │
    ▼
Router (LAN)
    │
    ▼
Ubuntu VM (enp0s3) ──── WireGuard Server (10.0.0.1/24)
                              │
                    ┌─────────┴──────────┐
                    ▼                    ▼
            Laptop (10.0.0.2)    iPhone (10.0.0.3)
```

---

## WireGuard Address Reference

| Device | Tunnel IP | Role |
|---|---|---|
| Ubuntu VM | `10.0.0.1/24` | Server |
| Laptop | `10.0.0.2/32` | Client |
| iPhone | `10.0.0.3/32` | Client |

---

## Installation

```bash
sudo apt install wireguard qrencode -y
```

Verify WireGuard is installed:
```bash
which wg
wg --version
```

---

## Server Setup (Ubuntu VM)

### Create WireGuard Directory
```bash
sudo mkdir -p /etc/wireguard
sudo chmod 700 /etc/wireguard
```

### Generate Server Keys
```bash
wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
sudo chmod 600 /etc/wireguard/server_private.key
```

Verify keys:
```bash
sudo cat /etc/wireguard/server_private.key
sudo cat /etc/wireguard/server_public.key
```

> **Note:** A common issue during key generation is Linux's `sudo` redirection problem — the `tee` command is required instead of `>` when writing to protected directories. Using `wg genkey | sudo tee /path/key` ensures the redirect runs with elevated permissions.

### Server Configuration
Location: `/etc/wireguard/wg0.conf`

```bash
sudo nano /etc/wireguard/wg0.conf
```

```ini
[Interface]
PrivateKey = <server_private_key>
Address = 10.0.0.1/24
ListenPort = 51820
MTU = 1420
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE

# Laptop
[Peer]
PublicKey = <laptop_public_key>
AllowedIPs = 10.0.0.2/32

# iPhone
[Peer]
PublicKey = <iphone_public_key>
AllowedIPs = 10.0.0.3/32
```

> **Note:** Replace `enp0s3` with your actual network interface name found via `ip a`.

### Enable IP Forwarding
Required for WireGuard to route client traffic:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Verify:
```bash
cat /proc/sys/net/ipv4/ip_forward
# Should return: 1
```

### Allow WireGuard Through Firewall
```bash
sudo ufw allow 51820/udp
sudo ufw reload
```

### Start & Enable WireGuard
```bash
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
sudo systemctl status wg-quick@wg0
```

---

## Laptop Client Setup (Linux Mint)

### Generate Laptop Keys
```bash
sudo sh -c 'wg genkey | tee /etc/wireguard/laptop_private.key | wg pubkey > /etc/wireguard/laptop_public.key'
sudo cat /etc/wireguard/laptop_private.key
sudo cat /etc/wireguard/laptop_public.key
```

### Laptop Configuration
Location: `/etc/wireguard/wg0.conf`

```ini
[Interface]
PrivateKey = <laptop_private_key>
Address = 10.0.0.2/24
DNS = 10.0.0.1
MTU = 1420
Table = off

[Peer]
PublicKey = <server_public_key>
Endpoint = <VM_real_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```

### Add Laptop as Peer on Server
```bash
sudo wg set wg0 peer <laptop_public_key> allowed-ips 10.0.0.2/32
sudo wg-quick save wg0
```

### Connect
```bash
sudo wg-quick up wg0
sudo wg show
```

---

## iPhone Client Setup (iOS)

### Generate Phone Keys on VM
```bash
sudo sh -c 'wg genkey | tee /etc/wireguard/phone_private.key | wg pubkey > /etc/wireguard/phone_public.key'
```

### Phone Configuration File
Location: `/etc/wireguard/phone.conf`

```ini
[Interface]
PrivateKey = <phone_private_key>
Address = 10.0.0.3/24
DNS = 10.0.0.1
MTU = 1280

[Peer]
PublicKey = <server_public_key>
Endpoint = <VM_real_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

### Generate QR Code for iPhone
```bash
sudo cat /etc/wireguard/phone.conf | sudo qrencode -t ansiutf8
```

Open the **WireGuard app** on iPhone → tap **+** → **Create from QR code** → scan the code.

### Add Phone as Peer on Server
```bash
sudo wg set wg0 peer <phone_public_key> allowed-ips 10.0.0.3/32
sudo wg-quick save wg0
```

---

## Verification

```bash
# Check VPN status and connected peers
sudo wg show

# Test tunnel connectivity
ping -c 5 10.0.0.1

# Verify all traffic routes through VPN
curl ifconfig.me   # Should return VM's public IP

# Test DNS through PiHole
nslookup google.com 10.0.0.1

# Monitor incoming WireGuard packets
sudo tcpdump -i enp0s3 udp port 51820
```

---

## MTU Explanation
WireGuard adds ~60 bytes of overhead per packet. Setting MTU to 1420 (1500 − 80) prevents packet fragmentation, significantly improving performance especially on mobile connections.

| Device | MTU | Reason |
|---|---|---|
| Laptop | 1420 | Standard WireGuard optimization |
| iPhone | 1280 | Extra headroom for mobile networks |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| `tee` permission denied | sudo doesn't apply to redirection | Use `sudo sh -c 'wg genkey \| tee /path'` |
| Empty public key file | Typo in directory name (`wiregurad`) | Double-check path with `ls /etc/wireguard` |
| wg0 service fails to start | Private key stored incorrectly | Verify key content, not folder path in config |
| No handshake | Wrong public key in peer config | Derive public key fresh: `cat private.key \| wg pubkey` |
| Packets arrive but not decrypted | Key mismatch between peers | Compare keys character by character on both sides |
| Slow speeds on phone | Packet fragmentation | Set `MTU = 1280` in phone config |
| Docker blocking VPN traffic | Docker iptables interference | Add explicit FORWARD rules for wg0 interface |
| VM unreachable | VirtualBox in NAT mode | Switch to Bridged Adapter in VirtualBox settings |
| VPN down when laptop sleeps | Host sleep pauses VM | Set power settings to never sleep, or migrate to Raspberry Pi |

---

## Lessons Learned
- The `sudo` redirection issue (`sudo echo > file` fails) is solved by using `tee` or `sudo sh -c`
- A single character typo in a directory name (`wiregurad`) can waste hours of debugging
- Storing a folder path instead of the actual private key content in the config causes silent failures
- Public and private keys must never be swapped — the peer config needs the **public** key of the other device
- `SaveConfig = true` can overwrite manual edits to `wg0.conf` on shutdown — use with caution
- Docker's iptables rules can silently block WireGuard forwarding even when everything else looks correct
