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

## Notes
- I went into the docker-compose.yml and configured it to run pihole.
- I wrote out the image from the latest updates of pihole, environment, volumes and ports (53)
- Ran into an issue saying that port 53 was in use so I switched to port 54 and it worked.
