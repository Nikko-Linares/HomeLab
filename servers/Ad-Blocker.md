# Ad Blocker (PiHole)

## Goals
- Block ads network-wide to enhance browsing speed and privacy
- Filter malicious domains and trackers for all devices on the VPN
- Learn DNS-based filtering and network traffic management

---

## Tools Used
| Tool | Role |
|---|---|
| PiHole | DNS sinkhole and ad blocker |
| Docker | Runs PiHole as an isolated container |
| docker-compose | Defines and manages the PiHole container configuration |

---

## How PiHole Works
PiHole acts as a **DNS sinkhole** — instead of your devices using your ISP's DNS server, they send all DNS queries to PiHole first. PiHole checks each query against its blocklists and either resolves it normally or returns an empty response for blocked domains, effectively removing ads before they ever load.

```
Device → PiHole DNS (10.0.0.1) → Blocked? → Return empty response (ad blocked)
                                → Allowed? → Forward to upstream DNS (1.1.1.1)
```

---

## Installation via Docker Compose

### Step 1 — Create the docker-compose.yml
```bash
nano docker-compose.yml
```

Paste the following configuration:
```yaml
version: "3"
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: always
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8082:80/tcp"
    environment:
      TZ: "America/Los_Angeles"
      WEBPASSWORD: "yourpassword"
    volumes:
      - pihole_data:/etc/pihole
      - dnsmasq_data:/etc/dnsmasq.d

volumes:
  pihole_data:
  dnsmasq_data:
```

### Step 2 — Start PiHole
```bash
sudo docker compose up -d
```

### Step 3 — Verify it's running
```bash
sudo docker ps
sudo ss -tulpn | grep :53
```

---

## Fixing Port 53 Conflict

Port 53 is reserved for DNS. On Ubuntu, `systemd-resolved` occupies port 53 by default, which prevents PiHole from binding to it.

### Diagnose the conflict
```bash
sudo ss -tulpn | grep :53
```

If `systemd-resolved` is listed, free up port 53:

```bash
# Stop and disable systemd-resolved
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# Remove the existing resolv.conf symlink
sudo rm /etc/resolv.conf

# Set a fallback DNS so the VM keeps internet access
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
```

Then restart PiHole:
```bash
sudo docker compose down
sudo docker compose up -d
sudo docker ps
sudo ss -tulpn | grep :53
```

> **Note:** PiHole must run on port 53 — switching to port 54 will not work because DNS traffic is hardcoded to port 53 by the DNS protocol. The fix is always to free up port 53 on the host.

---

## Password Setup (PiHole v6+)

PiHole v6 changed the password reset command. If the web UI password isn't working:

```bash
# PiHole v6+
sudo docker exec -it pihole pihole setpassword "newpassword"

# Older versions
sudo docker exec -it pihole pihole -a -p
```

---

## Fixing DNS Origin Issues

If PiHole is running but not blocking ads, it may be rejecting DNS queries from clients. To allow queries from all origins:

1. Open the PiHole admin dashboard
2. Go to **Settings → DNS**
3. Enable **"Permit all origins"**
4. Save settings

> This was the final fix that got PiHole working correctly — without it, PiHole silently ignores queries from VPN clients.

---

## Access

| Method | URL |
|---|---|
| HTTP (local) | `http://<VM_IP>:8082/admin/` |
| HTTPS (via Nginx) | `https://<VM_IP>:8443/admin/` |

---

## Adding Blocklists (PiHole v6)

Navigate to: **Lists → Add a new subscribed list**

| List | URL | Description |
|---|---|---|
| StevenBlack | `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` | General ads & tracking |
| Malware Filter | `https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-hosts.txt` | Malware & phishing |
| Social Tracking | `https://raw.githubusercontent.com/StevenBlack/hosts/master/alternates/social/hosts` | Social media trackers |

After adding lists, update gravity to pull in new domains:
```bash
sudo docker exec pihole pihole -g
```

Current blocklist size: **106,419+ domains blocked**

---

## Useful Commands

```bash
# Check PiHole container status
sudo docker ps | grep pihole

# View PiHole logs
sudo docker logs pihole --tail 50

# Update gravity (refresh blocklists)
sudo docker exec pihole pihole -g

# Restart PiHole
sudo docker restart pihole

# Check DNS is working
nslookup google.com 10.0.0.1
dig google.com @10.0.0.1
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Port 53 already in use | `systemd-resolved` occupying port 53 | Stop and disable `systemd-resolved`, set fallback DNS |
| Switched to port 54 but DNS not working | DNS protocol requires port 53 | Always use port 53 — fix the conflict instead |
| Password not working on web UI | PiHole v6 changed password command | Use `pihole setpassword` instead of `pihole -a -p` |
| PiHole running but ads not blocked | Queries rejected from VPN clients | Enable "Permit all origins" in Settings → DNS |
| Web UI not loading | Port 80 not mapped in container | Verify `8082:80` port mapping with `sudo docker inspect pihole` |
| Container keeps stopping | Startup error | Check `sudo docker logs pihole` for the cause |

---

## Lessons Learned
- PiHole **must** run on port 53 — switching to port 54 breaks DNS entirely since the protocol is hardcoded to port 53
- `systemd-resolved` occupies port 53 on Ubuntu by default and must be disabled before PiHole can bind to it
- After disabling `systemd-resolved`, manually set a fallback DNS (`1.1.1.1`) so the VM keeps internet access during setup
- PiHole v6 changed several CLI commands — `pihole setpassword` replaces the old `pihole -a -p`
- Enabling **"Permit all origins"** in DNS settings is required for VPN clients to use PiHole — without it queries are silently dropped
- Persistence pays off — stepping away from a problem and returning fresh led to finding the origin settings fix
