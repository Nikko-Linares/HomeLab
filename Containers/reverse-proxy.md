## Nginx

## Goals
- To increase security and provide access controls for web applications

## Commands
- sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
- sudo nginx -t
- which nginx
- dpkg -l | grep nginx
- dpkg -L nginx
- sudo ss -tulpn | grep 8080
- sudo docker run -d -p 8080:80 nginx
- curl http://IP_ADDRESS:8080

## Notes
- There was no myapp in the directory, so I uninstalled and reintalled nginx.
- The installation got messed up, so I had to check if Nginx was intalled.
- Found it in usr/sbin/nginx, but needed a folder called /usr/nginx.
- Was messing around with nginx and found that it could not connect to my ip address or port.
- Learned that servers fail not because of configs are wrong, but because servers aren't running properly.
