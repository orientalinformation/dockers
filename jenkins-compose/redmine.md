How to Run Jenkins on Ubuntu with Docker Compose
Author: Zain Ul Abideen

Last Updated: Tue, Nov 2, 2021 
Containers
DevOps
Programming
Ubuntu
Introduction
Jenkins is an open-source tool written in Java to set up continuous integration and deployment pipelines (CI/CD pipelines) for faster software delivery. It is the main part of Software Development Life Cycle automation. This guide describes the step-by-step procedure of setting up Jenkins on a Vultr Ubuntu instance using Docker Compose.

Prerequisites
A fresh Vultr Ubuntu 20.04 or 18.04 cloud instance
Docker and Docker Compose installed on the server
Writing Docker Compose for Jenkins
First, create a jenkins-compose directory and a new file docker-compose.yml inside the directory.

$ mkdir jenkins-compose

$ cd jenkins-compose

$ touch docker-compose.yml
Now open the newly created file with your favorite editor.

$ nano docker-compose.yml
Copy the following script into the docker-compose.yml file.

version: '3.8'
services:
  jenkins:
    container_name: jenkins
    restart: always
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    volumes:
      - jenkins-home:/var/jenkins_home

volumes:
  jenkins-home:
The syntax of the compose file:

version: ‘3.8’
This version describes the Docker Compose file format for version 3.8. Using some other version for this Docker Compose file may not work.

services:
The services section includes configuration for different docker containers in a single Docker Compose file.

  jenkins:
    container_name: jenkins
    restart: always
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    volumes:
      - jenkins-home:/var/jenkins_home
This section includes the configuration for the Jenkins docker container like container name, restart policy, image to be used for the container, port, and volume mappings.

The restart: always option in the compose file ensures that the Jenkins container remains up even after a reboot.

volumes:
  jenkins-home:
The volumes section creates a new volume used inside the Docker Compose.

After writing the compose file for Jenkins, enable and start the docker service. Enabling the service automatically starts the Docker service on system reboot.

$ sudo systemctl enable docker.service

$ sudo systemctl start docker.service
After writing the Docker Compose file for Jenkins server, use the following command to run it.compose.

$ docker-compose up -d
It pulls the Jenkins image from Docker Hub and starts the container. Use the following command to check if the container is running.

$ docker-compose ps
Configure SSL Certificates for Jenkins
To access the Jenkins server on a secure protocol (HTTPS), install the Nginx server, configure SSL certificates for Nginx, and proxy the traffic to the Jenkins server.

Install Nginx.

# apt install nginx -y
Edit your Nginx configuration file.

# nano /etc/nginx/conf.d/default.conf
Copy the following configurations in the file.

server {
  server_name example.com www.example.com
}
Follow this guide to generate SSL Let's Encrypt certificates for Nginx server. It automatically adds the SSL configuration to the /etc/nginx/conf.d/default.conf file. Open the configuration file.

# nano /etc/nginx/conf.d/default.conf
Add the following configurations to proxy the traffic to the Jenkins server.

location / {
  proxy_pass http://127.0.0.1:8080;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-For $remote_addr;
}
The final configuration file looks like this.

server {
  server_name example.com www.example.com;


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
      proxy_pass http://127.0.0.1:8080;
      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $remote_addr;
    }


}
server {
    if ($host = www.example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  server_name example.com www.example.com;
    listen 80;
    return 404; # managed by Certbot
}
Configure Firewall
To secure your Vultr server, configure the firewall to allow traffic only on ports 80, 443, and 22 (SSH port). You can use the OS firewall or Vultr firewall, or both to secure your server. Use this guide to configure the Vultr Firewall.

To configure the OS firewall, follow these steps:

Install the OS firewall using the following command.

# apt install ufw 
Block all the incoming and allow all the outgoing traffic.

# ufw default deny incoming
# ufw default allow outgoing
Allow traffic from specific ports.

# ufw allow ssh
# ufw allow http
# ufw allow https
NOTE: Before enabling the OS firewall, make sure the SSH port is not blocked; otherwise, you will not be able to access your server.

Now enable the firewall.

# ufw enable
Check the firewall status.

# ufw status
Test Jenkins Server
After running the Jenkins server, access the server on port 443 of your Vultr server like https://www.example.com/. It will ask for the Administrator password.

Use the following command to read the Administrator password from the container.

$ docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Now it will ask whether to install some suggested plugins or not.

Install the suggested plugins.

Now it will ask for the details for the first admin user.

After setting up the admin user, now Jenkins server is ready to set up CI/CD pipelines.

Conclusion
Compose is a tool used to configure and run containerized applications. It makes it easier to deploy a containerized application to the server. This guide explained how to run the Jenkins server with Docker Compose on a Vultr Ubuntu instance.

Want to contribute?