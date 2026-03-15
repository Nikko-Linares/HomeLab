## Docker Setup

## Goals
- Learn Docker
- Run a database
- Host apps and web servers

## Commands
- sudo apt install ca-certificates curl gnupg lsb-release -y (Installing Dependencies for Docker).
- sudo mkdir -p /etc/apt/keyrings
- curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo --dearmor -o /etc/apt/keyrings/docker.gpg (Add official GPG key).
- echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null (Add Docker repository).
- sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y (Install Docker).
- sudo docker run hello-world (Test Docker).
- sudo rm /etc/apt/sources.list.d/docker.list (removing file)
- sudo nano /etc/apt/sources.list.ddocker.list

## Notes
- Linux did not have the intsallation so I had to resort to checking Ubuntu version, check if the repo existed and found that the website did not have a release file.
- The offical Docker repo did not fully support noble yet. I had to remove the file and switch to a different one.
- Found a new repo called Jammy and tried setting it up. I ran into an issue saying "Invalid value set for option Signed-By regarding source https://download.docker.com/linux/ubuntu jammy (not a fingerprint)"
- I found why I got that error. I used docker.gng when I should have used docker.gpg.
