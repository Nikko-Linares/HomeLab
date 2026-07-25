# Cloud Server (Nextcloud & Vaultwarden)

## Goals
- Create a self-hosted cloud server to maintain full ownership of personal data
- Access files securely from multiple devices without relying on third-party services
- Expand the server to include a Git server, VPN gateway, and internal DNS
- Learn containerization and reverse proxy configuration through hands-on deployment

---

## Tools Used
| Tool | Role |
|---|---|
| Nextcloud | Self-hosted cloud storage and file sync |
| Vaultwarden | Self-hosted password manager (Bitwarden-compatible) |
| Docker & Docker Compose | Container runtime and service orchestration |
| Nginx | Reverse proxy for routing traffic to services |
| VirtualBox | Hosts the Ubuntu Server VM running all services |

---

## How It Works
Both Nextcloud and Vaultwarden run as Docker containers on the Ubuntu Server VM. Nginx sits in front of both services as a reverse proxy, handling HTTPS termination via a self-signed certificate and forwarding requests to the appropriate container.

```
Browser (HTTPS) → Nginx → Nextcloud   (localhost:8080)
                        → Vaultwarden (<VM_IP>:8081)
```

---

## Nextcloud

### Overview
Nextcloud is a self-hosted cloud storage platform — think Google Drive or Dropbox, but running entirely on your own hardware. All data stays private and under your control.

### Step 1 — Create the Project Directory
```bash
mkdir ~/cloud
cd ~/cloud
```

### Step 2 — Create docker-compose.yml
```bash
nano docker-compose.yml
```

Paste the following:
```yaml
version: "3"
services:
  nextcloud:
    image: nextcloud:latest
    container_name: cloud-nextcloud-1
    restart: always
    ports:
      - "8080:80"
    volumes:
      - nextcloud_data:/var/www/html

volumes:
  nextcloud_data:
```

> **Common Mistake:** The key is `image` not `images` — Docker Compose will throw an error about "additional properties not allowed" if misspelled.

### Step 3 — Start Nextcloud
```bash
docker compose up -d
```

Verify it's running:
```bash
sudo docker ps
```

### Step 4 — Initial Setup
Open a browser and navigate to:
```
http://<VM_IP>:8080
```

Create your admin account and complete the setup wizard.

---

## Setting Up Nginx Reverse Proxy for Nextcloud

### Create Nginx Config
```bash
sudo nano /etc/nginx/sites-available/nextcloud
```

Paste the following:
```nginx
server {
    listen 80;
    server_name <VM_IP>;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

> **Common Mistake:** The syntax is `listen 80;` not `listen: 80;` — the colon causes Nginx to fail with a "could not listen" error.

### Enable the Config
```bash
# Create symlink to enable the site
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/

# Remove the default Nginx page
sudo rm /etc/nginx/sites-enabled/default

# Test config for syntax errors
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

---

## HTTPS via Self-Signed Certificate

Nginx is configured to serve all services over HTTPS using a self-signed SSL certificate:

| Service | HTTPS Port |
|---|---|
| Nextcloud | 8444 |
| Vaultwarden | 8445 |

See [reverse-proxy.md](../Containers/reverse-proxy.md) for full SSL setup details.

---

## Nextcloud User Management

```bash
# List all users
sudo docker exec -it --user www-data cloud-nextcloud-1 php occ user:list

# Reset a user password (Nextcloud v33+)
sudo docker exec -it --user www-data \
  -e OC_PASS="newpassword123" \
  cloud-nextcloud-1 php occ user:resetpassword \
  --password-from-env <username>

# Check Nextcloud version
sudo docker exec -it --user www-data cloud-nextcloud-1 php occ -V

# View all available occ commands
sudo docker exec -it --user www-data cloud-nextcloud-1 php occ list
```

> **Note:** The `--user www-data` and `--password-from-env` flags are required for `occ` commands to work correctly in Docker.

---

## Vaultwarden

### Overview
Vaultwarden is a lightweight self-hosted implementation of Bitwarden — a fully featured password manager. It is compatible with all official Bitwarden apps and browser extensions.

### Deployment
```bash
sudo docker run -d \
  --name vaultwarden \
  --restart always \
  -p <VM_IP>:8081:80 \
  -e SIGNUPS_ALLOWED=false \
  -v vaultwarden_data:/data \
  vaultwarden/server:latest
```

> **Note:** Vaultwarden requires the VM's real IP in the port binding — using `localhost` or `0.0.0.0` causes a 502 Bad Gateway error in Nginx.

### Enabling Registration
Signups are disabled by default. To temporarily allow account creation:
```bash
sudo docker stop vaultwarden
sudo docker rm vaultwarden

sudo docker run -d \
  --name vaultwarden \
  --restart always \
  -p <VM_IP>:8081:80 \
  -e SIGNUPS_ALLOWED=true \
  -v vaultwarden_data:/data \
  vaultwarden/server:latest
```

After creating your account, disable signups again by redeploying with `SIGNUPS_ALLOWED=false`.

---

## Port Reference

| Service | HTTP Port | HTTPS Port (Nginx) |
|---|---|---|
| Nextcloud | 8080 | 8444 |
| Vaultwarden | 8081 | 8445 |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| `images` property error in compose | Typo — `images` instead of `image` | Fix spelling in `docker-compose.yml` |
| Nginx "could not listen" error | `listen: 80;` instead of `listen 80;` | Remove the colon from the listen directive |
| Can't reach VM from browser | VirtualBox set to NAT mode | Change network adapter to Bridged Adapter |
| Vaultwarden 502 Bad Gateway | `proxy_pass` using `localhost` | Use VM's real IP in `proxy_pass` |
| `occ` password reset fails | Missing `--user www-data` flag | Always include `--user www-data` in exec command |
| Nextcloud user not found | Wrong username in `occ` command | Run `occ user:list` first to confirm exact username |

---

## Future Plans
- Add a self-hosted **Git server** (Gitea) for private code repositories
- Configure **internal DNS** via PiHole for friendly domain names (e.g. `nextcloud.home`)
- Set up **Nextcloud mobile app** for iPhone file sync over VPN
- Migrate everything to a **Raspberry Pi** for 24/7 uptime

---

## Lessons Learned
- Typos in `docker-compose.yml` (`images` vs `image`) cause cryptic validation errors — read error messages carefully
- Nginx config syntax is strict — `listen 80;` and `listen: 80;` are completely different and the colon breaks everything
- VirtualBox must be in **Bridged Adapter** mode for the VM to be reachable from a browser on another device — NAT blocks all incoming connections
- Vaultwarden's port binding requires the VM's real IP, not localhost
- `occ` commands in Nextcloud Docker must be run as `www-data` user — omitting this causes permission errors
- Self-hosting gives full data ownership and privacy — no third-party services have access to your files or passwords
