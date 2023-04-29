# Nginx-Proxy-Manager
### Introduction
Nginx Proxy Manager is a reverse proxy management tool that makes it possible to create configuration files, set up security exceptions, streams, and SSL certificates through a web interface. In this write-up, you can install Nginx Proxy Manager on a Ubuntu 20.04 server running docker.
Prerequisites
•	Deploy a One-Click Docker instance on Vultr.
•	Login with SSH.
•	Install SQLite3.
Setup the Database and Data Directories
Create the Nginx Proxy Manager directory in a widely accessible location like /opt.
```
# mkdir /opt/nginxproxymanager
```
Under the directory, create a new databases subdirectory.
```
# mkdir /opt/nginxproxymanager/databases
```
### Create a new SQLite database file using the following command.
```
# touch /opt/nginxproxymanager/databases/nginxproxy.db
```
Exit the SQLite database console.
Control + d
### Install Nginx Proxy Manager
Create a custom Docker network.
```
# docker network create nginxproxyman
```
The Nginx Proxy Manager network allows management and monitoring of attached containers.
Using a text editor, create and edit a docker-compose.yml file in the main /opt/NginxProxy directory.
```
# nano /opt/nginxproxymanager/docker-compose.yml
```
Enter the following configurations to the file:
```
version: "3"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: 'nginxproxymanager'
    restart: unless-stopped
    ports:
      - '80:80' 
      - '443:443' 
      - '81:81' 
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```
networks:
  default:
    external:
      name: nginxproxyman
      
Nginx Proxy Manager listens on ports 80 and 443 for HTTP and HTTPS traffic, respectively. Port 81 provides access to the web management dashboard. To tighten server security, change the management port to a random combination.
Save and close the file.
Switch to the Nginx Proxy Manager directory.
```
# cd /opt/nginxproxymanager
```
Install Nginx Proxy Manager by starting docker-compose in detached mode.
```
# docker-compose up -d 
```
Verify that Nginx Proxy Manager is up and running.
```
# docker ps
```
The command output should look like the one below:
CONTAINER ID   IMAGE                             COMMAND                  CREATED             STATUS             PORTS                                            NAMES
f3a37d391293   jc21/nginx-proxy-manager:latest   "/init"                  33 minutes ago      Up 33 minutes      0.0.0.0:80-81->80-81/tcp, 0.0.0.0:443->443/tcp   nginxproxymanager
Configure Firewall
If you use UFW (enabled by default), allow the following access ports through the server.
Allow HTTP
```
# ufw allow 80
```
Allow HTTPS
```
# ufw allow 443
```
Allow the Nginx Proxy Manager web management dashboard.
```
# ufw allow 81
```
#### Setup Nginx Proxy Manager
Visit your server's IP address and load the Nginx Proxy Manager web management dashboard on port 81.
### http://Server_IP:81
Log in to the management dashboard with the following credentials:
### •	USERNAME: admin@example.com
### •	PASSWORD: changeme
Change your default username, email address, and password to secure the server.
To proxy and forward requests to a backend application, attach a docker container to the Nginx Proxy Manager network. For example, the following command creates a new ownCloud container attached to the nginxproxy network.
```
# docker run --network nginxproxyman --name owncloud -d owncloud:latest
```
Access the Nginx Proxy Manager web dashboard, and navigate to Hosts on the main navigation menu.
Click Add Proxy Host to enter a domain name, choose between HTTP or HTTPS scheme access, set up the target container name in the Forward Name/IP field, and toggle Save to continue.
