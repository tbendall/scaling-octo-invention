---
title: "Docker Introduction"
date: 2025-08-13T14:05:54Z
draft: false
---



In the last few weeks I've been learning and playing with Docker. 

# Introduction

Docker is an open-source platform that enables you to build, ship and run applications in containers. A container is a lightweight and portable self-sufficient unit that includes:

- The application code 
- Runtime
- System tools and libraries
- Settings

The beauty of containers is that it doesn't matter what it runs on; the container could be running on Windows, Linux, a laptop, RasPi, or anything that can install Docker. As it's self sufficient, it's completely agnostic to the underlying OS/compute. It's another way to virtualise compute resources that isn't as heavy-handed as having to install a whole new Virtual Machine.

![Containers](/containers.avif)

One or many containers can run on a single host, virtualising the host's resources and creating separate runtimes for each container.

# Networking

Docker networking is most interesting to me for some reason, and there a few different options you can distribute and network containers.

- bridge (default): An isolated network for containers on a single host
- host: Shares the host’s network stack
- overlay: Enables multi-host networking (used in Docker Swarm)
- macvlan: Assigns MAC addresses to containers for direct LAN access
- none: No networking


Overlay is the type that Docker swarm uses, and is used to link containers hosted on different hosts, for a more distributed deployment.


# Test Deployment

I followed a tutorial to spin up a NGINX reverse proxy which farms out requests to multiple flask servers. See diagram below.

![Distributed Docker Lab](/Distributed_nginx.png)

I used a simple docker compose YAML file to spin up both the NGINX and Flask app.py containers, allowing me to create a distributed infrastructure with one command.

    docker-compose.yml 
    --
    version: '3'
    services:
    web:
        build: ../flask_app
    web2:
        build: ../flask_app
    nginx:
        image: nginx:alpine
        ports:
        - 80:80
        volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf
        depends_on:
        - web
        - web2
<br>
The nginx container takes a prepopulated nginx config file to create the reverse proxy, distributing the HTTP requests across both Flask servers.


    # nginx.conf
    events {}
    http {
        upstream app_servers {
            server web:5000;
            server web2:5000;
        }
        server {
            listen 80;
            location / {
                proxy_pass http://app_servers;
            }
        }
    }
<br>
The Flask server hosts a simple Python that returns "Hello Docker!" and the hostname name of the container responding to the request:


    #!/usr/bin/python3

    # app.py
    from flask import Flask
    import socket
    app = Flask(__name__)
    @app.route('/')
    def home():
        newstring = "Hello, Docker! Hostname is {0}\n".format(socket.gethostname())

        return newstring
    if __name__ == '__main__':
        app.run(host='0.0.0.0')

<br>

Sending multiple requests shows that Nginx is load balancing the requests across each Flask server:

    root@mgmt-host:~/docker# curl localhost
    Hello, Docker! Hostname is d9bcfe1b35d0

    root@mgmt-host:~/docker# curl localhost
    Hello, Docker! Hostname is 36f88db2fc2b
    
    root@mgmt-host:~/docker# curl localhost
    Hello, Docker! Hostname is d9bcfe1b35d0
    
    root@mgmt-host:~/docker# curl localhost
    Hello, Docker! Hostname is 36f88db2fc2b
    
    root@mgmt-host:~/docker# curl localhost
    Hello, Docker! Hostname is d9bcfe1b35d0
    
    root@mgmt-host:~/docker# curl localhost
    Hello, Docker! Hostname is 36f88db2fc2b


Next steps is to create a truly distributed setup; using multiple different hosts to create a converged deployment!