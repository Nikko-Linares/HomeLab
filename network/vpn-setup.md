# VPN Setup

## Goal
- Have a local VPN for security and safety.
- Connect my laptop to my cloud server securely.

## Tools Used
- VirtualBox
- Ubuntu Server
- WireGuard
- SSH Keys

## Commands
- sudo apt install wireguard qrencode
- wg genkey | sudo tee /etc/wireguard/server_private.key
- sudo cat /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
- sudo chmod 600 /etc/wireguard/server_private.key
// Checking for verification on Wireguard
- which wg
- sudo mkdir -p /etc/wireguard
- sudo chmod 700 /etc/wireguard
- wg genkey > server_private.key
- cat server_private.key
- cat server_private.key | wg pubkey > server_public.key
- cat server_public.key
// Need to create a public key
- sudo sh -c 'cat /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key'
- sudo cat /etc/wireguard/server_public.key
// Nothing appeared, possibly experiencing Linux redirection + sudo issue
- sudo rm -f /etc/wireguard/server_private.key
- sudo rm -f /etc/wireguard/server_public.key
- wg genkey
- sudo nano /etc/wireguard/server_private.key
- sudo chmod 600 /etc/wireguard/server_private.key
- sudo sh -c "wg pubkey < /etc/wireguard/server_private.key > /etc/wireguard/server_public.key"
- sudo cat /etc/wireguard/server_public.key
// No public key, taking a pause and checking other things
- sudo ls -ld /etc/wireguard
// Mispelled wireguard as wiregurad
- sudo nano /etc/wireguard/wg0.conf

Interface
PrivateKey = ...
Address = 10.0.0.1/24
ListenPort = 51820
SaveConfig = true

PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

- sudo cat /etc/wireguard/server_private.key
- cat /proc/sys/net/ipv4/ip_forward
- sudo sysctl -w net.ipv4.ip_forward=1
- sudo ufw allow 51820/udp
- sudo systemctl start wg-quick@wg0
// Issue starting wg-quick@wg0
- sudo systemctl status wg-quick@wg0
- sudo journalctl -xeu wg-quick@wg0
// Found the issue and VPN is running.

## Notes
- Saying that there was no file or directory when doing the tee function for public key, or cat function for the private key.
- wg was not the correct length or format to generate server keys.
- Successfully have a private key, but I need a public key next.
- Nothing appeared when checking for the public key. However, when checking the private key it was also empty. It put out an example code when checking "wg genkey | sudo tee /etc/wireguard/server_private.key"
- Getting frustuarated so taking an approach to check other things to see what is going wrong.
- Found the issue and I mispelled wireguard, going back to fix the issue.
- The wg0.sevice exied with error codes so I need to go back and check the system control.
- I had found the issue, I had stored the private key into a folder and instead of putting the private VPN key, I put the location of the folder.
- Still testings multi-device connections
