One problem with learning something new is that once you have been through the walkthroughs and examples you end up forgetting about it and months down the line you have forgotten what you did and end up having to go through the docs all over again.

This is exactly what happened to me when going through [the docker docs](https://docs.docker.com/engine/getstarted/). So I made the decision to find a use case for docker and host a website using docker, which is what you are browsing now (well as of 21/02/17).

Below is my Dockerfile to build the image that this website is running off.
```
FROM alpine
RUN apk update
RUN apk add php7-fpm php7-mbstring php7-session php7-json
RUN sed -e 's/127.0.0.1:9000/9000/' -i /etc/php7/php-fpm.d/www.conf
VOLUME ["/srv/kirby"]
CMD ["php-fpm7", "-F"]
```
This uses the alpine linux docker image. This is being used just because of how small the image is.
```
alpine latest 88e169ea8f46 7 weeks ago 3.98 MB
```
Breaking it down it:

* uses a tiny image.
* installs php-fpm and php modules (that the blog depends on)
* configures php-fpm, so that we can connect to it over a tcp socket
* The location of the volume. (more of a reference than anything else)
* The command to run in the foreground to keep the image online.

I could have done all this in apache and when I started and exposed port 80 and 443 to the container, but I already use a lot of nginx and I personally prefer it to apache. or I could have bundled nginx into the image and use something such as [supervisord](http://supervisord.org/).
Well I also listen to a lot of podcast (which I can talk about another time) and one personal favourites is [syscast podcast](http://podcast.sysca.st/). In episode 2 they talk about docker and one thing I took from this was a good rule of thumb is to have one service per container, which seem sensible to me.

So next is to spin up an nginx image. Luckily in this regard there is no need for a Dockerfile as nginx have an image [mainline-apline docker hub](https://hub.docker.com/_/nginx/). So I can spin this up using
```
docker run -d nginx:mainline-alpine
```
We then start up the kirby blog in a similar way. We first create a volume that will contain the data
```
docker volume create --name=kirby-blog
```
we can then start the blog using this to ensure the data persists if the container is removed and re-created
```
docker run -d -v kirby-blog:/srv/kirby kirby-image:latest
```

This means that I can now create an nginx configuration, but there are a few issues
* you need to use docker inspect on the kirby container to get it's internal IP address, so you know the IP nginx has the proxy .php files to. This IP is liable to change if the container is removed and re-created, which means you need to change this in the nginx container all the time.
* The document root for nginx to process requests that are not .php becames `/var/lib/docker/volumes/kirby-blog/_data.` This isn't great as it means you will have to open up world read access, or even group read access and give the nginx user access to those folders.

For point 1 what you can do is create your own docker network and then give the containers static ip addresses.
`docker network create --subnet=172.16.238.0/24 my_first_network`
the use the --ip when running your docker run command to tell the container to always have this IP address. This will allow you to fix point 1.

For point 2 what you need is a way to link the containers together. Well you can link containers using [volumes](https://docs.docker.com/engine/tutorials/dockervolumes/), without going into detail we can mount the volume kirby-blog in the nginx container, so we can then change the $document_root in /srv/kirby, which is the same location in the kirby container.

Ok, so now you can in your docker run command use -v to link the volumes and --ip to ensure that you have the same IP, but now your docker run command is becoming quite unwieldy.
```
docker run --restart=always -d --ip 172.16.238.2 -v kirby-blog:/srv/kirby -v /etc/nginx/conf.d:/etc/nginx/conf.d -v /etc/letsencrypt:/etc/letsencrypt -p80:80 -p443:443 nginx
docker run --restart=always -d --ip 172.16.238.3 -v kirby-blog:/srv/kirby -p127.0.0.1:9000:9000 &lt;kirby-image&gt;
```

This brings me on to an essential part of the docker set up that I came across quite quickly and was also mentioned in the above podcast episode which is [docker compose](https://docs.docker.com/compose/). You can use this to achieve all of the above in a small bit of yaml configuration.
```
version: '2.1'
services:
nginx:
restart: always
image: nginx:mainline-alpine
ports:
- "80:80"
- "443:443"
volumes:
- /etc/nginx/conf.d:/etc/nginx/conf.d:ro
- /etc/letsencrypt:/etc/letsencrypt:ro
- kirby-blog:/srv/kirby:ro
networks:
nginx_web_app:
ipv4_address: 172.16.238.10
depends_on:
- kirby
```
This creates the nginx container, based on the image called nginx:mainline-alpine. It mounts the SSL, nginx configuration and the kirby blog volumes, all read-only as nginx has no reason to write to these volumes. We then tell it to expose the http/https ports and give it a static IP address.
```
kirby:
restart: always
build: kirby/.
ports:
- "127.0.0.1:9004:9000"
networks:
nginx_web_app:
ipv4_address: 172.16.238.14
volumes:
- kirby-blog:/srv/kirby
```
This creates the blog container, but the key differences are the volume is mounted read-write, we only expose port 9000 as localhost:9004, so it can't be seen externally. and we give it a static IP 
```
address.
networks:
nginx_web_app:
driver: bridge
ipam:
driver: default
config:
- subnet: 172.16.238.0/24
gateway: 172.16.238.1
```
This section creates the network that we assign our IP addresses and the gateway from, so the containers can get network access outbound and internally.
```
volumes:
kirby-blog:
external: true
```
This just defines the volumes to use.

What this does is allows us to start up the solution by just running docker-compose up -d we can restart parts of it using docker-compose restart nginx. If we need to upgrade anything run
```
docker-compose build; docker-compose down; docker-compose up -d
```

Although there is a lot to explore in docker, such as [docker swarm](https://docs.docker.com/engine/swarm/) or [kubernetes](https://kubernetes.io/), but what this has done has allowed for that understanding to actually stick and to really see the massive advantages to using docker and the satisfaction of learning something new.
