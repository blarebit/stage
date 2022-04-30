---
title: "Running Minio Cluster in Three Different Nodes"
date: 2021-01-21
categories: ["Storage", "Minio"]
draft: false
---
# How to Run Minio in Different Nodes with Docker

In this example I want to have 3 nodes in the Minio cluster.
**1- install docker and docker compose**
To handle the Minio container in case of failure it’s good to have docker compose, so you should install it on each node.

**2- define a docker compose**
There is an image for Minio in docker hub which we can use. According to the Minio documentation we need at least two disks for each node, that’s why I added two directories in /media directory like this:

```
/media/minio1
/media/minio2
```

And repeat it for the other nodes. For introducing nodes to each other we should pass the all nodes with their disks as a start input to Minio. In this example I have three nodes with the following IPs:

```
192.168.1.1
192.168.1.2
192.168.1.3
```

We should pass it for each three nodes Minio start command:

```bash
$ server http://192.168.1.1/minio1 http://192.168.1.1/minio2 http://192.168.1.2/minio1 http://192.168.1.2/minio2 http://192.168.1.3/minio1 http://192.168.1.3/minio2
```

Also all nodes only can connect to each other with the same secret key and access key which we should pass at start command too, in this case I’m gonna set them as the environment variable.

```yaml
version: '3'
services:
 minio:
  image: minio/minio:RELEASE.2020-12-29T23-29-29Z
  volumes:
   \- /media/minio1:/minio1
   \- /media/minio2:/minio2
  expose:
   \- "9000"
  network_mode: host
  restart: always
  environment:
   MINIO_ACCESS_KEY: admin
   MINIO_SECRET_KEY: blahblahblah
   MINIO_PROMETHEUS_AUTH_TYPE: "public"
  command: server http://172.30.5.13/minio1 http://172.30.5.13/minio2 http://172.30.5.51/minio1 http://172.30.5.51/minio2 http://172.30.5.61/minio1 http://172.30.5.61/minio2
  healthcheck:
   test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
   interval: 30s
   timeout: 20s
   retries: 3
```

If you don't want to use docker compose you can use the following command:

```bash
$ docker run -p 9000:9000 -e MINIO_ACCESS_KEY=admin -e MINIO_SECRET_KEY=blahblahblah -v /media/minio1:/minio1 -v /media/minio2:/minio2 --net=host minio/minio:RELEASE.2020-12-29T23-29-29Z server [http://192.168.1.{1...3}/minio{1..2](about:blank)}
```

Now you can run the docker-compose up -d command to start your Minio cluster.

To have a better cluster setting, it’s good to have a Nginx in front of the Minio cluster to have a unique IP to connect to the Minio cluster, for do this we should add Nginx section like the following at the end of one of our docker compose files:

```yaml
 nginx:
  image: nginx:1.19.2-alpine
  volumes:
   \- ./nginx.conf:/etc/nginx/nginx.conf:ro
  ports:
   \- "9797:9292"
  depends_on:
   \- minio
```

Also, you should put the nginx.conf beside the docker compose file with the following config:

```
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log warn;
pid    /var/run/nginx.pid;
events {
  worker_connections 1024;
}
http {
  include    /etc/nginx/mime.types;
  default_type application/octet-stream;
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
           '$status $body_bytes_sent "$http_referer" '
           '"$http_user_agent" "$http_x_forwarded_for"';
  access_log /var/log/nginx/access.log main;
  sendfile    on;
  \#tcp_nopush   on;
  keepalive_timeout 65;
  \#gzip on;
  \# include /etc/nginx/conf.d/*.conf;
  upstream minio {
    server 192.168.1.1:9000;
    server 192.168.1.2:9000;
    server 192.168.1.3:9000;
  }
  server {
    listen    9292;
    listen [::]:9292;
    server_name localhost;
     \# To allow special characters in headers
     ignore_invalid_headers off;
     \# Allow any size file to be uploaded.
     \# Set to a value such as 1000m; to restrict file size to a specific value
     client_max_body_size 0;
     \# To disable buffering
     proxy_buffering off;
    location / {
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_connect_timeout 300;
      \# Default is HTTP/1, keepalive is only enabled in HTTP/1.1
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      chunked_transfer_encoding off;
      proxy_pass http://minio;
    }
  }
}
```

I’m gonna add it end of the docker compose file of the first node, now I start the cluster again.
**3- login**
Now you can access the panel in your browser with 192.268.1.1:9797

![img](https://lh3.googleusercontent.com/10fjrkPGk96ZuMKsr6-3CCP2XUxmAtlOLIbyQmE4d8JoR_EB5C6BG4-YzioNoc2mbSusrHSDfqcUS9bYmEI31rzS5bG5DBZQOf5Dc7-lBt9XauWZUgIVeB9MUm2JpIkMCh7bCPfu)

With the access and secret key which you define before you can login.

