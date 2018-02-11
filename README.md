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

The thing which is of interest to use is the last line: `include /etc/nginx/conf.d/*.conf` this tells us that we just need to add our configuration to this folder to be included in all http requests.


If we step into the conf.d directory you will notice that there is a default.conf within this folder.

Lets explore it further:

```bash
$ cd conf.d
$ cat default.conf

server {

    listen       80; # the port which nginx is listening on

    
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    ## lots of commented out stuff which is not interest to use right now

}

```

## Simplify the config file

It seems that we do not need all of this stuff in the config file.

We can create our own nginx.conf and copy it over to the container to have more control.

```conf
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
If you run nginx now, you would think that it should work, but it does not.




