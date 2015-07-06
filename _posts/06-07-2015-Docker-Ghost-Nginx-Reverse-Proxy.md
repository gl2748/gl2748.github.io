---
layout: post
category : docker
tagline: "Setting up a secure ghost blog with a persistent data container and an Nginx reverse proxy"
tags : [ docker, ghost, nginx, reverse-proxy]
---

Ghost suggests that it not be run on Port 80.
    * For Ghost to run on port 80 directly it must be run as the root user.
    * With a single Ghost instance hogging the single port 80 on the server we cannot have a mulitsite or multi subdomain set-up.

Therefore the official docs suggest setting up a reverse proxy to forward requests upstream to Ghost. There are a few options for your reverse-proxy server but here will be using NGINX.

All this means is that the NGINX webserver will listen on port 80 and forward requests to the correct process running 'upstream'. For example, you can have a ghost instance at blog.mywebsite.com as well as something at www.mywebsite.com and maybe something else at myotherwebsite.com running on the same server, as well as your process protected behind non-root access.

Here we will be working towards a simple ghost blog running at blog.mywebsite.com. Furthermore we will put all of our processes in seperate docker containers.

####1.
Create a non-running data-only volume to hold our ghost data. This is a persistent data container.

    docker run -v /data --name ghostdata dockerfile/ubuntu echo Data-only container for Ghost

####2.
Run the ghost container linked to our data-volume. NOTE: Later we will have to edit our ghost config.js file, specifically the database connection/filename, so that it matches the path of the data located in our data volume i.e. '/data/ghostblog.db'. 
    docker run -d --volumes-from ghostdata --name ghostblog  ghost:latest

####3.
Create our nginx sites enabled config locally on the host machine. i.e. at `~/docker/nginx/sites-enabled. Importantly here'server_name' is where you will navigate to in your browser to see your running ghost blog. The proxy_pass setting should reflect the name of your ghost container. Note: Later we will have to edit the nginx config to include the 'sites-enabled' directory.

    server {
        listen      80;
        server_name blog.mywebsite.com;
        access_log  /var/log/nginx/nginx.blog.access.log;
        error_log   /var/log/nginx/nginx.blog.error.log;
        location    /   {
            proxy_pass          http://ghostblog:2368;
            proxy_set_header    X-Forwarded-Host    $server_name;
            proxy_set_header    Host                $server_name;
            proxy_set_header    X-Real-IP           $remote_addr;
        }
    }

####4.
Run your nginx contianer and inject the sites-enabled config.

    docker run -d -p 80:80 --name nginx --link ghostblog:ghostblog -v ~/docker/nginx/sites-enabled:/etc/nginx/sites-enabled nginx

####5.
Enter the running nginx container and edit the /etc/nginx/nginx.conf file to include the sites-enabled directory. 

    docker exec -it nginx bash

You will need to install your text-editor of choice.

    apt-get update
    apt-get install vim

edit the nginx config file to include the sites-enabled directory

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

exit the running container and restart it. if it fails to stay running check your config files for typos.

    ctrl+d
    docker restart nginx

####6.


####Road map

Set up a grand ambassador (although nginx's proxy_pass is kinda nice.)
In
