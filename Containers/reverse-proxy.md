# Reverse Proxy (Nginx)

## Goals
- Increase security for all home lab web applications
- Provide HTTPS encryption via self-signed SSL certificates
- Centralize access control for PiHole, Nextcloud, and Vaultwarden under a single proxy

---

## Tools Used
| Tool | Role |
|---|---|
| Nginx | Reverse proxy and web server |
| OpenSSL | Self-signed SSL certificate generation |
| UFW | Firewall rules for HTTPS ports |

---

## How It Works
Instead of exposing each service directly on its own port over plain HTTP, Nginx sits in front of all services and handles HTTPS termination. Clients connect to Nginx over an encrypted connection, and Nginx forwards requests internally to the appropriate container.

```
Browser (HTTPS) → Nginx → PiHole     (localhost:8082)
                        → Nextcloud   (localhost:8080)
                        → Vaultwarden (<VM_IP>:8081)
```

---

## Installation

```bash
sudo apt install nginx openssl -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Verify Nginx is installed and running:
```bash
which nginx
dpkg -l | grep nginx
dpkg -L nginx
sudo systemctl status nginx
```

> **Note:** If Nginx appears broken or misconfigured, check if the binary exists first with `which nginx` — it installs to `/usr/sbin/nginx`, not `/usr/nginx`. A missing `sites-available` config (like `myapp`) will cause `nginx -t` to fail even if Nginx itself is installed correctly.

---

## SSL Certificate Generation

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/homelab.key \
  -out /etc/nginx/ssl/homelab.crt
```

### Certificate Prompts
| Field | Value |
|---|---|
| Country | US |
| State | California |
| City | Anaheim |
| Organization | Home Lab |
| Common Name | VM's real IP address |

Verify files were created:
```bash
ls /etc/nginx/ssl/
# Should show: homelab.crt  homelab.key
```

---

## Nginx Configuration

Create the config file:
```bash
sudo nano /etc/nginx/sites-available/homelab
```

Paste the following:
```nginx
# PiHole
server {
    listen 8443 ssl;
    ssl_certificate /etc/nginx/ssl/homelab.crt;
    ssl_certificate_key /etc/nginx/ssl/homelab.key;

    location / {
        proxy_pass http://localhost:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Nextcloud
server {
    listen 8444 ssl;
    ssl_certificate /etc/nginx/ssl/homelab.crt;
    ssl_certificate_key /etc/nginx/ssl/homelab.key;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Vaultwarden
server {
    listen 8445 ssl;
    ssl_certificate /etc/nginx/ssl/homelab.crt;
    ssl_certificate_key /etc/nginx/ssl/homelab.key;

    location / {
        proxy_pass http://<VM_IP>:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

> **Note:** Vaultwarden requires the VM's real IP in `proxy_pass` instead of `localhost` due to how it binds its network port. Using `localhost` causes a 502 Bad Gateway error.

---

## Enable Configuration

```bash
# Create symlink to enable the site
sudo ln -s /etc/nginx/sites-available/homelab /etc/nginx/sites-enabled/

# Test configuration for syntax errors
sudo nginx -t

# Restart Nginx to apply changes
sudo systemctl restart nginx
```

---

## Firewall Rules

```bash
sudo ufw allow 8443/tcp
sudo ufw allow 8444/tcp
sudo ufw allow 8445/tcp
sudo ufw reload
```

---

## HTTPS Access URLs

| Service | HTTPS URL |
|---|---|
| PiHole | `https://<VM_IP>:8443/admin/` |
| Nextcloud | `https://<VM_IP>:8444` |
| Vaultwarden | `https://<VM_IP>:8445` |

---

## Verification Commands

```bash
# Check Nginx is running
sudo systemctl status nginx

# Test config before restarting
sudo nginx -t

# Check what is listening on a port
sudo ss -tulpn | grep 8443

# Test a service locally
curl http://localhost:8082/admin/

# Test with verbose output
curl -v http://localhost:8443/admin/
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| `nginx -t` fails | Config file missing or has syntax error | Verify file exists in `sites-available` and check for typos |
| 502 Bad Gateway | Nginx can't reach the backend service | Check the backend container is running with `sudo docker ps` |
| Can't connect to IP/port | Nginx not running | Check `sudo systemctl status nginx` and restart if needed |
| Browser blocks self-signed cert | Certificate not trusted by browser | Use Firefox → Advanced → Accept the Risk |
| `sites-available/myapp` not found | Config file was never created | Create the correct config file first before symlinking |
| Vaultwarden 502 error | `proxy_pass` using `localhost` | Use VM's real IP instead: `proxy_pass http://<VM_IP>:8081` |

---

## Lessons Learned
- A server failing doesn't always mean the config is wrong — the service itself may not be running. Always check `systemctl status` first
- Nginx installs to `/usr/sbin/nginx`, not `/usr/nginx` — verify with `which nginx` before assuming it's missing
- The symlink in `sites-enabled` must point to a file that actually exists in `sites-available`
- Always run `sudo nginx -t` before restarting — it catches syntax errors without taking the server down
- Vaultwarden's port binding requires the VM's real IP in `proxy_pass`, not `localhost`
- Firefox handles self-signed certificate overrides more reliably than Opera GX
