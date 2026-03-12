## Docker Setup

## Goals
- Learn Docker
- Run a database
- Host apps and web servers

## Tools Used

## Commands
- sudo apt install ca-certificates curl gnupg lsb-release -y (Installing Dependencies for Docker).
- sudo mkdir -p /etc/apt/keyrings
- curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo --dearmor -o /etc/apt/keyrings/docker.gpg (Add official GPG key).
- echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null (Add Docker repository).
- sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y (Install Docker).
- sudo docker run hello-world (Test Docker).

## Notes
- 
