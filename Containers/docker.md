# Docker

## Overview
Docker is used to containerize all core home lab services, ensuring isolated, reproducible, and easily manageable deployments on an Ubuntu Server VM.

---

## Installation

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

---

## Running Containers

### PiHole (Ad Blocker)
```bash
sudo docker run -d \
  --name pihole \
  --restart always \
  -p 53:53/tcp \
  -p 53:53/udp \
  -p 8082:80/tcp \
  -e TZ="America/Los_Angeles" \
  -e WEBPASSWORD="yourpassword" \
  -v pihole_data:/etc/pihole \
  -v dnsmasq_data:/etc/dnsmasq.d \
  pihole/pihole:latest
```

### Nextcloud (Cloud Storage)
```bash
sudo docker run -d \
  --name cloud-nextcloud-1 \
  --restart always \
  -p 8080:80 \
  -v nextcloud_data:/var/www/html \
  nextcloud:latest
```

### Vaultwarden (Password Manager)
```bash
sudo docker run -d \
  --name vaultwarden \
  --restart always \
  -p <VM_IP>:8081:80 \
  -e SIGNUPS_ALLOWED=false \
  -v vaultwarden_data:/data \
  vaultwarden/server:latest
```

---

## Useful Commands

| Command | Description |
|---|---|
| `sudo docker ps` | List running containers |
| `sudo docker ps -a` | List all containers including stopped |
| `sudo docker start <name>` | Start a container |
| `sudo docker stop <name>` | Stop a container |
| `sudo docker restart <name>` | Restart a container |
| `sudo docker rm <name>` | Remove a container |
| `sudo docker logs <name>` | View container logs |
| `sudo docker exec -it <name> bash` | Enter container shell |
| `sudo docker inspect <name>` | View container details |

---

## Auto-Restart on Boot
All containers are set to restart automatically:
```bash
sudo docker update --restart always <container_name>
```

Verify restart policy:
```bash
sudo docker inspect <name> | grep -A 3 RestartPolicy
```

---

## Port Reference

| Service | Host Port | Container Port |
|---|---|---|
| PiHole Web UI | 8082 | 80 |
| PiHole DNS | 53 | 53 |
| Nextcloud | 8080 | 80 |
| Vaultwarden | 8081 | 80 |

---

## Notes
- Vaultwarden requires the VM's real IP in the port binding (`-p <VM_IP>:8081:80`) rather than localhost due to how it handles network binding
- Docker's iptables rules can interfere with WireGuard — ensure forwarding rules are explicitly set
- Enable Docker on boot: `sudo systemctl enable docker`
