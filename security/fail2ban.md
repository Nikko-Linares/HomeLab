# Fail2ban

## Overview
Fail2ban monitors system logs and automatically bans IP addresses that show malicious behavior such as repeated failed SSH login attempts. It acts as an intrusion prevention system protecting the Ubuntu Server VM from brute force attacks.

---

## Installation

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## Configuration

Fail2ban uses a jail system — each "jail" monitors a specific service. The main config file is `/etc/fail2ban/jail.conf` but you should create a local override:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

### SSH Jail Configuration
```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
findtime = 600
```

| Setting | Value | Description |
|---|---|---|
| `enabled` | true | Activate this jail |
| `maxretry` | 5 | Failed attempts before ban |
| `bantime` | 3600 | Ban duration in seconds (1 hour) |
| `findtime` | 600 | Time window to count failures (10 min) |

---

## Useful Commands

```bash
# Check Fail2ban status
sudo systemctl status fail2ban

# Check all active jails
sudo fail2ban-client status

# Check SSH jail specifically
sudo fail2ban-client status sshd

# Unban an IP address
sudo fail2ban-client set sshd unbanip <IP_ADDRESS>

# View banned IPs
sudo fail2ban-client get sshd banned

# Test a filter
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf
```

---

## Log Monitoring
```bash
# View Fail2ban logs
sudo tail -f /var/log/fail2ban.log

# View authentication logs
sudo tail -f /var/log/auth.log
```

---

## How It Works
1. A client repeatedly fails to authenticate via SSH
2. Fail2ban detects the failures in `/var/log/auth.log`
3. After `maxretry` failures within `findtime` seconds, the IP is banned
4. An iptables rule is added to drop all traffic from that IP
5. After `bantime` seconds, the ban is lifted automatically

---

## Notes
- Fail2ban is actively running and protecting SSH on the home lab VM
- Default SSH port 22 is monitored
- Combined with SSH key authentication, this provides strong protection against unauthorized access
