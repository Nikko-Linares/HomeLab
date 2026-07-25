# VirtualBox

## Goals
- Run a personal Virtual Machine to practice building and managing servers
- Host all home lab services (WireGuard, PiHole, Nextcloud, Vaultwarden) inside the VM
- Learn virtualization concepts in a safe, isolated environment

---

## Tools Used
| Tool | Role |
|---|---|
| VirtualBox | Virtualization software |
| Desktop Computer | Host machine running VirtualBox |
| Ubuntu Server 22.04 LTS | Guest OS installed inside the VM |

---

## Installation

### Step 1 — Download VirtualBox
Download the installer for your host OS from the official site:
```
https://www.virtualbox.org/wiki/Downloads
```

### Step 2 — Download Ubuntu Server 22.04 LTS
```
https://ubuntu.com/download/server
```
> **Tip:** Always use an LTS (Long Term Support) release for a server — it receives security updates for 5 years and is far more stable than non-LTS releases.

### Step 3 — Create a New VM
1. Open VirtualBox and click **New**
2. Set the following:

| Setting | Value |
|---|---|
| Name | Ubuntu Server (or any name) |
| Type | Linux |
| Version | Ubuntu (64-bit) |
| Memory | 4096 MB (4 GB) |
| CPU Cores | 2 |
| Storage | 20 GB minimum (dynamically allocated) |

3. Click **Create**

### Step 4 — Attach the Ubuntu ISO
1. Go to **Settings → Storage**
2. Click the empty optical drive
3. Click the disc icon → **Choose a disk file**
4. Select the downloaded Ubuntu Server ISO

### Step 5 — Configure Network (Bridged Adapter)
1. Go to **Settings → Network → Adapter 1**
2. Set **Attached to:** `Bridged Adapter`
3. Select your host machine's active network interface

> **Important:** Bridged mode gives the VM its own IP address on your local network. This is required for WireGuard VPN and SSH access from other devices. NAT mode will NOT work for this home lab setup.

### Step 6 — Boot and Install Ubuntu Server
1. Start the VM
2. Follow the Ubuntu Server installation wizard
3. When prompted, set up:
   - Username and password
   - Enable **OpenSSH server** during installation
4. Complete installation and reboot

---

## VM Specifications

| Component | Value |
|---|---|
| Memory | 4096 MB (4 GB) |
| CPU Cores | 2 |
| Network Mode | Bridged Adapter |
| Network Interface | `enp0s3` |
| Guest OS | Ubuntu Server 22.04 LTS |
| WireGuard Tunnel IP | `10.0.0.1/24` |

---

## First Boot Setup

```bash
# Update and upgrade all packages
sudo apt update && sudo apt upgrade -y
# -y flag automatically confirms all prompts

# Check IP address assigned to VM
ip a show enp0s3

# Verify internet connectivity
ping -c 3 google.com
```

---

## Auto-Start VM on Boot

Configure VirtualBox to start the VM headlessly (no window) when the host boots:

```bash
# Enable autostart for your VM
VBoxManage modifyvm "YourVMName" --autostart-enabled on
```

Start the VM manually in headless mode:
```bash
VBoxManage startvm "YourVMName" --type headless
```

---

## Useful VBoxManage Commands

```bash
# List all VMs
VBoxManage list vms

# List running VMs
VBoxManage list runningvms

# Start VM in headless mode
VBoxManage startvm "YourVMName" --type headless

# Gracefully shut down VM
VBoxManage controlvm "YourVMName" acpipowerbutton

# Take a snapshot
VBoxManage snapshot "YourVMName" take "before-changes"

# Restore a snapshot
VBoxManage snapshot "YourVMName" restore "before-changes"
```

> **Tip:** Take a snapshot before making major changes — it lets you roll back instantly if something breaks.

---

## Services Running Inside the VM

| Service | Type | Port |
|---|---|---|
| WireGuard | systemd | 51820/udp |
| Docker | systemd | — |
| PiHole | Docker | 53, 8082 |
| Nextcloud | Docker | 8080 |
| Vaultwarden | Docker | 8081 |
| Nginx | systemd | 8443, 8444, 8445 |
| Fail2ban | systemd | — |
| SSH | systemd | 22 |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| VM can't be reached from other devices | Network set to NAT | Switch to Bridged Adapter in network settings |
| VM pauses when laptop sleeps | Host sleep suspends VM | Set host power settings to never sleep when plugged in |
| No internet inside VM | Bridged adapter on wrong interface | Select the correct host network interface in settings |
| VM running slowly | Not enough resources | Increase RAM or CPU in settings (VM must be off) |
| SSH connection refused | OpenSSH not installed | `sudo apt install openssh-server -y` |

---

## Limitations & Future Plans
- The VM depends on the host machine staying on — if the host sleeps, all services go offline
- Performance is limited by the host machine's hardware and ISP upload speed
- Planned migration to a **Raspberry Pi 4** for dedicated 24/7 uptime with lower power consumption (~5W)

---

## Lessons Learned
- Bridged Adapter is essential for a home lab VM — NAT blocks incoming connections needed for WireGuard and SSH
- Ubuntu Server 22.04 LTS is the ideal choice for stability and long-term support
- Allocating 4 GB RAM and 2 CPU cores provides enough headroom to run Docker, WireGuard, PiHole, Nextcloud, and Vaultwarden simultaneously
- `sudo apt update && sudo apt upgrade -y` should always be the first command run on a fresh install
- Snapshots are invaluable — take one before any major configuration change
