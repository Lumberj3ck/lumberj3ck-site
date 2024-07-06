+++
title = 'Nginx Ssl in Docker Container'
date = 2024-06-04T07:21:28+02:00
draft = false
summary = 'This guide covers setting up Nginx with HTTPS in a Docker environment, using Certbot to obtain and renew Lets Encrypt certificates. It provides a detailed configuration for Docker Compose, Nginx, and Certbot, ensuring secure handling of web traffic. Finally, it includes steps to configure Django settings for HTTPS compatibility.'
+++

Using Nginx inside a Docker container is a common approach for web applications, especially when paired with the Django framework and Gunicorn. Typically, you have one Docker container for **Django** and **Gunicorn**, another for the database, and a final one for Nginx. Nginx acts as a reverse proxy, forwarding requests from the server to Gunicorn. For this setup, Nginx listens for requests by binding the server's public port to the Nginx container's port (e.g., {{< emphasize >}}80{{< / emphasize >}} for HTTP or {{< emphasize >}}443{{< / emphasize >}} for HTTPS).
In this guide, we’ll set up Nginx with HTTPS in a Docker environment.

### Prerequisites
Ensure you have a working **Docker Compose** configuration. Back up your configuration files and commit any changes to Git for easy rollback if needed. Assume the following file structure:

**Posible file tree**
{{< highlight bash >}}
  ├── Dockerfile
  ├── docker-compose.yaml
  ├── nginx
  │   ├── Dockerfile
  │   └── nginx.conf
  └── src
{{< / highlight >}}

**Nginx Docker file**
{{< highlight dockerfile "linenos=table,hl_lines=,linenostart=1" >}}
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
{{< / highlight >}}

**Nginx configuration**
{{< highlight nginx "linenos=table,hl_lines=,linenostart=1" >}}
# the name for the service which we gave inside of a docker compose
upstream backend_service{
    server backend_service:8000;
}

server {
    listen 80;

    server_name domain.site www.domain.site;
    location / {
        proxy_pass http://backend_service;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static/ { 
      root /home/app; 
    }

    location /imgs/ {
      root /home/app/;
    }
}
{{< / highlight >}}

I posted docker compose and main Docker file inside of this [gist](https://gist.github.com/Lumberj3ck/d9bd5c87c08e26a4aa8eee1075a667a9).

### Docker Compose Configuration  
Map the necessary ports in docker-compose.yaml:
80 port for the incomming http traffic and also 443 for the https.

{{< highlight nginx "linenos=table,hl_lines=4-5,linenostart=1" >}}
nginx_service:
  build: ./nginx
  ports:
    - 80:80
    - 443:443
  volumes:
    - static_files:/home/app/static
    - imgs:/home/app/imgs
...
{{< / highlight >}}

### Adding Certbot
Add Certbot to your **docker-compose.yaml** to automatically obtain Let's Encrypt certificates:
Certbot it is an open source software, which automatically provides letsencrypt certificates. Certbot will produce files required for the nginx, thats why we need to have a volumes and inject those volumes to the nginx container.

{{< highlight nginx "linenos=table,hl_lines=1-5,linenostart=1" >}}
certbot:
  image: certbot/certbot:latest
  volumes:
    - ./certbot/www/:/var/www/certbot/:rw
    - ./certbot/conf/:/etc/letsencrypt/:rw
{{< / highlight >}}

Inject these volumes into the Nginx service:

{{< highlight nginx "linenos=table, hl_lines=9-10,linenostart=1" >}}
nginx_service:
  build: ./nginx
  ports:
    - 80:80
    - 443:443
  volumes:
    - static_files:/home/app/static
    - imgs:/home/app/imgs
    - ./certbot/www:/var/www/certbot/:ro
    - ./certbot/conf/:/etc/nginx/ssl/:ro
...
{{< / highlight >}}

Full docker compose configuration is [here](https://gist.github.com/Lumberj3ck/b9b7678b0d54a83bf54801d6a5505e0a#file-docker-compose-yaml).

### Configuring HTTPS
Modify your Nginx configuration to transfer all incoming traffic from 80 port (which is default port for the http) to 443 (which is default port for the https). Create a new server block which will listen for the 443 port. Basically we just shifted all the listeners to the new server block.

{{< highlight nginx "linenos=table, hl_lines=,linenostart=1" >}}
upstream backend_service{
    server backend_service:8000;
}

server {
    listen 443 default_server ssl;
    listen [::]:443 ssl;

    server_name domain.site www.domain.site;

    ssl_certificate "/etc/nginx/ssl/live/domain.site/fullchain.pem";
    ssl_certificate_key "/etc/nginx/ssl/live/domain.site/privkey.pem";

    location / {
        proxy_pass http://backend_service;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static/ {
        root /home/app;
    }

    location /imgs/ {
        root /home/app/;
    }
}
{{< / highlight >}}

Currently configuration is working, but only with the https and to start accepting request from the http we need to add new block.

{{< highlight nginx "linenos=table, hl_lines=11,linenostart=1" >}}
server {
    listen 80;

    server_name domain.site www.domain.site;
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        # redirecting every http request to the https which we already set up
        return 301 https://domain.site$request_uri;
    }
}
{{< / highlight >}}

Full nginx configuration is [here](https://gist.github.com/Lumberj3ck/b9b7678b0d54a83bf54801d6a5505e0a#file-nginx-conf)

### Obtaining SSL Certificates
Run Certbot to obtain the certificates

{{< blockquote >}}
Don't forget to **build the docker images** and make sure that **nginx container is up**, and you specified right domain name inside of the command bellow.
{{< / blockquote >}}

{{< highlight bash >}}
  sudo docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d example.org
{{< / highlight >}}

If there are no errors, remove the --dry-run flag to obtain the actual certificates:

{{< highlight bash >}}
  sudo docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ -d example.org
{{< / highlight >}}



### Renewal
Remember to renew your certificates every three months:
{{< highlight bash >}}
  docker compose run --rm certbot renew
{{< / highlight >}}


{{< blockquote >}}
Also if you're using Django don't forget to add follwing options in your settings.py  
{{< / blockquote >}}

{{< highlight python >}}
  CSRF_TRUSTED_ORIGINS = ['https://domain.site']  
  CSRF_COOKIE_HTTPONLY = False
{{< / highlight >}}

**It's done! Your server is oficially working with the https traffic!**