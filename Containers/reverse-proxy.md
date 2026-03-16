## Nginx

## Goals
- To increase security and provide access controls for web applications

## Commands
- sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
- sudo nginx -t
- which nginx
- dpkg -l | grep nginx
- 

## Notes
- There was no myapp in the directory, so I uninstalled and reintalled nginx.
- The installation got messed up, so I had to check if Nginx was intalled.
- Found it in usr/sbin/nginx, but needed a folder called /usr/nginx
- 
