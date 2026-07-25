# Kali Linux

## Goals
- Learn offensive security tools and techniques in a safe, controlled environment
- Perform penetration testing against the home lab to identify vulnerabilities
- Practice network scanning, enumeration, and password testing
- Build foundational cybersecurity skills

---

## Tools Used
| Tool | Role |
|---|---|
| VirtualBox | Hosts the Kali Linux VM |
| Kali Linux | Penetration testing operating system |
| Nmap | Network scanning and enumeration |
| WireGuard | VPN access to home lab network |

---

## Installation

### Step 1 — Download Kali Linux
Download the VirtualBox image (pre-built) from the official site:
```
https://www.kali.org/get-kali/#kali-virtual-machines
```
> **Tip:** Download the pre-built VirtualBox image (`.ova`) instead of the ISO — it saves setup time and comes pre-configured.

### Step 2 — Import into VirtualBox
1. Open VirtualBox
2. Go to **File → Import Appliance**
3. Select the downloaded `.ova` file
4. Click **Import** and wait for it to finish

### Step 3 — Configure Network
Set the network adapter to **Bridged Adapter** so Kali gets its own IP on your local network:
- **Settings → Network → Adapter 1 → Bridged Adapter**

### Step 4 — First Boot
Default credentials for the pre-built image:
| Field | Value |
|---|---|
| Username | `kali` |
| Password | `kali` |

> **Important:** Change the default password immediately after first login:
> ```bash
> passwd
> ```

---

## Network Scanning (Nmap)

Nmap is the industry-standard tool for discovering hosts and services on a network.

### Basic Commands

```bash
# Scan a single host
nmap <target_IP>

# Scan your entire home lab network
nmap 10.0.0.0/24

# Detect OS and service versions
nmap -sV -O <target_IP>

# Aggressive scan (OS, versions, scripts, traceroute)
nmap -A <target_IP>

# Scan specific ports
nmap -p 22,80,443,8080,8443 <target_IP>

# Scan all 65535 ports
nmap -p- <target_IP>

# UDP scan
nmap -sU <target_IP>

# Fast scan (top 100 ports)
nmap -F <target_IP>

# Save results to a file
nmap -oN scan_results.txt <target_IP>
```

### Home Lab Scan Example
```bash
# Discover all devices on the WireGuard network
nmap -sV 10.0.0.0/24

# Full scan of the Ubuntu VM
nmap -A 10.0.0.1
```

---

## Password Testing

> **⚠️ Important:** Only perform password testing against systems you own or have explicit permission to test. All testing below is against your own home lab.

### Hydra (Online Password Testing)
```bash
# SSH brute force test
hydra -l <username> -P /usr/share/wordlists/rockyou.txt ssh://<target_IP>

# Test a web login form
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target_IP> http-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

### Common Wordlists
```bash
# Extract rockyou wordlist (comes with Kali)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# List available wordlists
ls /usr/share/wordlists/
```

### John the Ripper (Offline Password Testing)
```bash
# Crack a hash file
john --wordlist=/usr/share/wordlists/rockyou.txt hashfile.txt

# Show cracked passwords
john --show hashfile.txt
```

---

## Penetration Testing the Home Lab

### Reconnaissance
```bash
# Discover open ports on VM
nmap -sV -p- 10.0.0.1

# Check for common vulnerabilities
nmap --script vuln 10.0.0.1

# Banner grabbing
nmap -sV --version-intensity 5 10.0.0.1
```

### Testing SSH Security
```bash
# Check SSH configuration weaknesses
nmap -p 22 --script ssh-auth-methods 10.0.0.1

# Verify Fail2ban is working by attempting failed logins
# (Watch fail2ban logs on the VM during this test)
ssh wronguser@10.0.0.1
```

### Testing Web Services
```bash
# Check for open web ports
nmap -p 8080,8081,8082,8443,8444,8445 10.0.0.1

# Basic web enumeration
nikto -h http://10.0.0.1:8082
```

---

## Useful Kali Tools Reference

| Tool | Category | Description |
|---|---|---|
| `nmap` | Scanning | Network discovery and port scanning |
| `hydra` | Password | Online brute force attacks |
| `john` | Password | Offline hash cracking |
| `nikto` | Web | Web server vulnerability scanner |
| `wireshark` | Traffic Analysis | Packet capture and analysis |
| `netcat` | Networking | TCP/UDP connection tool |
| `gobuster` | Web | Directory and file brute forcing |
| `metasploit` | Exploitation | Vulnerability exploitation framework |

---

## General Terminal Commands

```bash
# Update Kali
sudo apt update && sudo apt upgrade -y

# Check network interfaces
ip a

# Check routing table
ip route

# Monitor network traffic
sudo wireshark

# Check open connections
sudo ss -tulpn

# View running processes
ps aux

# Check disk space
df -h
```

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Can't reach home lab network | Kali not on same network | Set VirtualBox to Bridged Adapter |
| Nmap shows no open ports | Firewall blocking scans | Try `nmap -Pn` to skip host discovery |
| Hydra running very slowly | Too many threads | Add `-t 4` to limit threads |
| Can't access WireGuard tunnel | WireGuard not installed on Kali | `sudo apt install wireguard -y` |
| Permission denied on tools | Not running as root/sudo | Prefix commands with `sudo` |

---

## Lessons Learned
- Always test only against systems you own — the home lab is a perfect safe environment for this
- Nmap is an essential first step in any penetration test — understanding what's exposed is key
- Fail2ban on the Ubuntu VM actively blocks repeated failed login attempts — visible proof that defensive tools work
- Running Kali in a VirtualBox VM keeps it isolated from the main system while still giving full access to the home lab network
- The pre-built VirtualBox `.ova` image is the fastest way to get Kali running
- Change default credentials immediately — `kali/kali` is the first thing an attacker would try

---

## ⚠️ Legal & Ethical Reminder
All penetration testing must only be performed on systems you own or have written permission to test. Unauthorized scanning or testing of external systems is illegal. This home lab setup provides a safe, legal environment to develop these skills.
