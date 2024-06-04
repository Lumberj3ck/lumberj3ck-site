+++
title = 'Nginx Ssl in Docker Container'
date = 2024-06-04T07:21:28+02:00
draft = false
summary = 'There is a common approach to use a nginx inside of a docker container for a web apps. For example using django framework with a gunicorn inside of a one docker container and the second container for a database of your webservice if required and the final docker service is a nginx which binds public port of your pc and the port on nginx container so the nginx inside of container is listening for the incoming request on port 80 or whichever you decide to use'
+++


There is a common approach to use a nginx inside of a docker container for a web apps. For example using django framework with a gunicorn inside of a one docker container and the second container for a database of your webservice if required and the final docker service is a nginx which binds public port of your pc and the port on nginx container so the nginx inside of container is listening for the incoming request on port 80 or whichever you decide to use. And when setting up docker services is done, the least part is left make your nginx work with an https conection. 

By the start of this guide is expected to have already working docker compose configuration. You will need to back it up or make a commit in git to easily revert all the changes in configs. Assume that you have a docker compose and nginx conf something like this. 

{{< emphasize >}} Inside of a docker compose file{{< / emphasize >}}
{{< highlight docker "linenos=table,hl_lines=,linenostart=1" >}}
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

  certbot:
    image: certbot/certbot:latest
    volumes:
      - ./certbot/www/:/var/www/certbot/:rw
      - ./certbot/conf/:/etc/letsencrypt/:rw

    / Example of a configuration for a backend
    backend_service:
    build: .
    volumes:
        / here is our static assets and images saved inside of a volume 
      - static_files:/home/app/static
      - imgs:/home/app/imgs
    env_file:
        / loading some env variables 
        / optional: only if required by your webserver
      - .env.prod
    expose:
        / just exposing port without binding 
        / nginx conf only needs to know a port  
        / where to redirect all the traffic
      - 8000
      / here all the magic is happening gunnicorn server is going to proxy
      / all the traffic which was redirected to this container to the actual python app
    command: gunicorn manga_reader.wsgi:application --bind 0.0.0.0:8000
{{< / highlight >}}

{{< emphasize >}} Inside of a docker configuration file for a backend_service{{< / emphasize >}}
{{< highlight docker "linenos=table,hl_lines=,linenostart=1" >}}
FROM python

WORKDIR /home/app

COPY . . 

RUN python -m venv venv && \
    /bin/bash -c "source venv/bin/activate" && \
    pip install -r requirements.txt
/ since I use django used the command 
/ for collecting all the static files inside of a directory
RUN python manage.py collectstatic
{{< / highlight >}}