## Pi Hole

## Goal
- The objective is to block ads and enhance the performance speed of my computer while surfing the web.

## Commands
- nano docker-compose.yml
- sudo docker compose up -d
- sudo docker ps
 Ran into an issue saying port 53 was occupied.
- sudo docker compose up -d
- sudo docker ps
- sudo ss -tulpn | grep :53
- sudo systemctl stop systemd-resolved
- sudo systemctl disbale systemd-resolved
- sudo rm /etc/resolv.conf
- echo "namesserver 1.1.1.1" | sudo tee /etc/resolv.conf
- sudo docker compose down
- sudo docker compose up -d
- sudo docker ps
- sudo ss -tulpn | grep :53
- sudo docker exec -it pihole pihole setpassword

## Notes
- I went into the docker-compose.yml and configured it to run pihole.
- I wrote out the image from the latest updates of pihole, environment, volumes and ports (53)
- Ran into an issue saying that port 53 was in use so I switched to port 54 and it worked.
- Turns out that it did not work and found out that it did need to be on port 53 and my password was not working. Decided to tackle the port problem first.
- Was able to get pihole on port 53, but password was being weird. I had to force it to work on terminal.
- After that, I was able to log into my pihole account.
- Pi-hole is not working correctly.
- Gave up for a while, but went into my pihole settigns and allowed it to permit all origins and now it is working properly. 
