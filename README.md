# Install-Nextcloud-on-Ubuntu-Server-using-Docker
Install nextcloud on ubuntu server with docker integrate onlyoffice and zoho login


## Required
1. Ubuntu server 22.04.5 
2. Docker and Docker compose


## Step 1: Create docker compose file

### Create folder nextcloud
``` 
mkdir -p nextcloud
cd nextcloud/
```

### Create docker compose file
```
touch docker-compose.yml
nano docker-compose.yml
```

### Copy this code to docker compose file
```
version: '3.8'
services:
  nginx:
    image: nginx:latest
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - nextcloud
      - onlyoffice

  nextcloud:
    image: nextcloud:28.0.10-apache
    restart: always
    volumes:
      - ./nextcloud_data:/var/www/html
    links:
      - db
    environment:
      - MYSQL_PASSWORD=admin
      - MYSQL_USER=admin
      - MYSQL_DATABASE=nextcloud
      - MYSQL_HOST=db
      - TRUSTED_PROXIES=nginx
    depends_on:
      - db

  db:
    image: mysql:8.0.27
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_PASSWORD=admin
      - MYSQL_USER=admin
      - MYSQL_DATABASE=nextcloud
    volumes:
      - ./db_data:/var/lib/mysql

  onlyoffice:
    image: onlyoffice/documentserver:8.1.0
    restart: always
    environment:
      - JWT_SECRET=adminlocal123a@
    ports:
      - "81:80"
```

### Create file nginx.conf

```
mkdir -p nginx
touch nginx/nginx.conf
nano nginx/nginx.conf
```

#### Copy this code to nginx.conf
```
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    upstream nextcloud {
        server nextcloud:80;
    }

    upstream onlyoffice {
        server onlyoffice:80;
    }

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    client_max_body_size 512M;

    server {
        listen 80;
        server_name _;

        # Nextcloud
        location / {
            proxy_pass http://nextcloud;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # Websocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }

        # ONLYOFFICE
        location /onlyoffice/ {
            proxy_pass http://onlyoffice/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

## Step 2: Start docker container

```
docker compose up -d
```

> [!IMPORTANT] 
> You need to wait about some minutes for the containers to start successfully!