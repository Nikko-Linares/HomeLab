## NextCloud

## Goals
- The goal is to create my own cloud server so I can be responsible for holding my own data from multiple devices.
- To add a Git server, VPN gateway and Internal DNS.

## Commands
- mkdir ~/cloud
- cd /~cloud
- nano docker-compose.yml
- docker compose up -d
- sudo nano /etc/nginx/sites-available/nextcloud
- sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
- sudo rm /etc/nginx/sites-enabled/default (Removing the old file to enable the proxy)
- sudo nginx -t

## Notes
- Had to configure the docker-compose.yml file to write the nextcloud services with image, prots, and volumes.
- Got an error saying "validating /home/clouduser/cloud/docker-compose.yml: services.nextcloud additional properties 'images' not allowed.
- It was suppose to say image not images.
- Ran the ip address in a browser and set up an account. Next is to move the Nextcloud behind a Reverse Proxy.
- Set up the location for the proxy pass.
- Ran into an error that the service could not listen to /etc/nginx/sites-enabled/nextcloud:2.
- I ended up writing listen: 80; instead of listen 80;
- I ended up trying to set up my ip address, but kept running into issues. Went to the VM settings and changed it from NAT to Bridged Adapter.
