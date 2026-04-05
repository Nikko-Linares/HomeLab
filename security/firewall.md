## Firewall (UFW)

## Goal
- To practice better security and knowledge about firewalls

## Tools Used
- Terminal

## Commands
- sudo apt install ufw
- sudo ufw default deny incoming
- sudo ufw default allow outgoing
- sudo ufw allow 22/tcp   #SSH
- sudo ufw allow 80/tcp   #HTTP
- sudo ufw allow 443/tcp  #HTTPS
- sudo ufw enable
- sudo ufw status verbose

## Notes
- 22, 80, and 443 allow me to protect my ssh, http, and https all in that order.

