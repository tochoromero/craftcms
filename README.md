# Craft CMS Docker Image

Lightweight Craft CMS 3 Image

Comes with Craft 3 and pdo_pgsql for use with PostgreSQL.

Bring your own webserver and database.

## Features

* pdo_pgsql
* pg_dump for backups
* redis
* imagemagick

## Example Setup

You only need two files:

* docker-compose.yml
* default.conf

### docker-compose

```yml
# docker-compose.yml
version: '3'

services:
  nginx:
    image: nginx:alpine
    ports:
      - 80:80
    links:
      - craft
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf # nginx configuration (see below)
      - ./assets:/var/www/html/web/assets # For media, js and css files
      - craftdata:/var/www/html

  craft:
    image: urbantrout/craftcms
    volumes:
      - ./templates:/var/www/html/templates # Craft CMS template files
      - craftdata:/var/www/html
    environment:
      DEPENDENCIES: >- # additional composer packages (must be comma separated)
        craftcms/redactor,
        craftcms/element-api

      REDIS_HOST: redis
      SESSION_DRIVER: redis
      CACHE_DRIVER: redis

      DB_SERVER: postgres
      DB_NAME: craft
      DB_USER: craft
      DB_PASSWORD: secret
      DB_DATABASE: craft
      DB_SCHEMA: public
      DB_DRIVER: pgsql
      DB_PORT: 5432
      DB_TABLE_PREFIX: ut

  postgres:
    image: postgres:10.3-alpine
    environment:
      POSTGRES_ROOT_PASSWORD: root
      POSTGRES_USER: craft
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: craft
    volumes:
      # Persistent data
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:4-alpine
    volumes:
      - redisdata:/data

volumes:
  craftdata: # Needed so that nginx and craft services can access the same files
  pgdata:
  redisdata:
```

### nginx configuraion

Create a file called **default.conf** in to your project directory:

```nginx
# default.conf

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name localhost;

    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/html/web;
    charset utf-8;

    # Root directory location handler
    location / {
        try_files $uri/index.html $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri $uri/ /index.php?$query_string;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass craft:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

## Plugins

Just add your plugins to the environment variable DEPENDENCIES. A script then automatically adds or removes those dependencies when you create the container.

In a docker-compose file:

```yaml
    environment:
      DEPENDENCIES: >- # additional composer packages (must be comma separated)
        craftcms/redactor,
```

If you change your dependencies, just run `docker-compose down && docker-compose up` to remove and recreate your container.

## Finish setup

Run `docker-compose up` and visit http://localhost/admin. Voilà!
