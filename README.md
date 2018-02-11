# nginx and docker configuration

This project is about how to best congifure ngix in docker.

## Installation

The first thing which I am going to do is to pull the latest nginx image so that this does not have to be done again.

> It seems that it was already on my laptop.

```bash
$ sudo docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
e7bb522d92ff: Already exists 
6edc05228666: Pull complete 
cd866a17e81f: Pull complete 
Digest: sha256:285b49d42c703fdf257d1e2422765c4ba9d3e37768d6ea83d7fe2043dad6e63d
Status: Downloaded newer image for nginx:latest
```

Now to check that the image is available to be used
> I have already pull the alpine and php images

```bash
$ sudo docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
php                 latest              14aa3556d5d5        4 days ago          353MB
alpine              latest              3fd9065eaf02        4 weeks ago         4.15MB
nginx               latest              3f8a4339aadd        6 weeks ago         108MB
```

# Create a dockerfile

Make a directory called `service` and then create a Dockerfile in this folder.

```bash
$ mkdir -p service
$ cd service
$ touch Dockerfile
```

Add the following instruction to your Dockerfile. So as to create an image based upon the nginx image

> This example is very redundant, but for the sake of learning lets do it.

```Dockerfile
FROM nginx
```

Build and run our new image.

```bash
$ sudo docker build -t webserver .

Sending build context to Docker daemon  4.608kB
Step 1/1 : FROM nginx
 ---> 3f8a4339aadd
Successfully built 3f8a4339aadd
Successfully tagged webserver:latest
```

Once this has been built we can go ahead and run it to see that our test nginx web server is running and working.

```bash
# Here we are running the container as a deamon with the flag -d
# exposing port 80 on the image and binding port 8080 to port 80 on the container
# --name is giving our container a name of webserver
# and webserver is telling run which image to use
$ sudo docker run -d -p 8080:80 --name webserver webserver
351f6553d438fc0d0f60321b980446613012fad77c18f9509c058c61e974423d
```

If you open [http://localhost:8080](http://localhost:8080) in your browser and your should see a 'Welcome to nginx' webpage.

To check that the container is running we use the ps command

```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
351f6553d438        webserver           "nginx -g 'daemon ..."   6 minutes ago       Up 6 minutes        0.0.0.0:8080->80/tcp   webserver
```

# Looking inside the nginx container

Now that we can start the container, lets have a look inside it to checkout the conf file.

The config file which we are wanting to look at is located here: `/etc/nginx/nginx.conf`

```bash
$ sudo docker exec -it webserver bash
root@351f6553d438:/# cat /etc/nginx/nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

There are many things in this nginx.conf which we could remove, therefore lets reduce it down to the bare minimum to make this server work.

### nginx.conf minimum requirements

```config
events {

}

http {
    server {
        listen 8888;

        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}

```

### events context

> Reference: http://nginx.org/en/docs/ngx_core_module.html#events

```
Syntax:	events { ... }
Default:	—
Context:	main
Provides the configuration file context in which the directives 
that affect connection processing are specified.
```

It seems that we must define the events context within side of this config file. Otherwise the server will not function.

I am not sure about anything else about this, but important to note is that it is a requirement for your web server to work!

### http context 

> http://nginx.org/en/docs/http/ngx_http_core_module.html#http

```
Syntax:	http { ... }
Default:	—
Context:	main
Provides the configuration file context in which the HTTP server directives are specified.
```

The http block { ... } must sit within side of the nginx.conf because it is a core module and all core modules must be defined within side of the main context

The http context has many of its own contexts which can be defined within side of its own block such as the server { ... } context

### server context

> http://nginx.org/en/docs/http/ngx_http_core_module.html#server

```
Syntax:	server { ... }
Default:	—
Context:	http
Sets configuration for a virtual server. There is no clear separation between 
IP-based (based on the IP address) and name-based (based on the “Host” request
header field) virtual servers. Instead, the listen directives describe all 
addresses and ports that should accept connections for the server, and the 
server_name directive lists all server names. Example configurations are provided
in the “How nginx processes a request” document.
```


## Simplify the config file

A simple version of the nginx file could look like this

```conf

events {
    # and event contexts go here
}

http {

    server {
    
        listen 8888; # port which we want to listen on

        server_name localhost; # give the server a name

        location / {
    
            root /usr/share/nginx/html; # tell the server where its root folder is for all requests
            index index.html; # tell the server which file extension or file type to use when not file have been provided in a folder
        }

    }
}
```

## Update Dockerfile

Now that we have learnt more about the nginx configuration we can update our dockerfile to have a custom nginx container

First create a `nginx.conf` in the service folder and add the following code to it

```config
events {
    # and event contexts go here
}

http {

    server {
    
        listen 80; # port which we want to listen on

        server_name localhost; # give the server a name

        location / {
    
            root /usr/share/nginx/html; # tell the server where 
                                        # its root folder is for all requests
            
            index index.html; # tell the server which file extension or 
                              # file type to use when not file have been provided in a folder
        }

    }
}
```

So as to make our own website we need to make a root folder to copy over to the nginx container.

Make a directory called web in the service folder and create `index.html` and save it in the web folder with the following content:

```html
<h1>Hello, world</h1>
```

Update your dockerfile to look like this

```Dockerfile
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf

COPY web /usr/share/nginx/html

```

## House keeping

Now that we have researched everything which we need to know a little house keeping is required.

run: 

```bash
$ sudo docker ps
```

You should see that your webserver container is running

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
ddd6bd45afc9        webserver           "nginx -g 'daemon ..."   49 seconds ago      Up 48 seconds       0.0.0.0:8080->80/tcp   webserver
```

We need to stop it so as it can take on our new changes to our Dockerfile image.

> this will stop the webserver from running

```bash
$ sudo docker stop webserver
webserver
```
> you should now see that it exited

```bash
$ sudo docker ps -a 
CONTAINER ID    ...        STATUS                      PORTS               NAMES
ddd6bd45afc9    ...        Exited (0) 20 seconds ago                       webserve
```

### Build our image again to update it

```bash
$ sudo docker build -t webserver .
Sending build context to Docker daemon  6.144kB
Step 1/3 : FROM nginx
 ---> 3f8a4339aadd
Step 2/3 : COPY nginx.conf /etc/nginx/nginx.conf
 ---> Using cache
 ---> 476feb349398
Step 3/3 : COPY web /usr/share/nginx/html
 ---> Using cache
 ---> 6121bb2d10ff
Successfully built 6121bb2d10ff
Successfully tagged webserver:latest
```

### Running our new container

Prior to running our new container we need to delete the old one so that we can update the containers image id. 

```bash
$ sudo docker rm webserver
```

Now we can run our new image with its updates.

```bash
$ sudo docker run -d -p 8080:80 --name webserver webserver
d436f6e133817876568f26592bfa242728360d732ed12d1e61e538403e838404
```

If you open [http://localhost:8080](http://localhost:8080) in your browser and your should see a 'Welcome to nginx' webpage.


