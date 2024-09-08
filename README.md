# WEB
Динамический веб
## Развертывание веб приложения
1. Варианты стенда:
* nginx + php-fpm (laravel/wordpress) + python (flask/django) + js (react/angular);
* nginx + java (tomcat/jetty/netty) + go + ruby;
* своя комбинация.
2. Реализации на выбор:
* на хостовой системе через конфиги в /etc;
* деплой через docker-compose.
# Описание, выполнение домашнего задания:
1. Для выполнения задания используется Docker. 
* database из образа mysql:8.0 и вольюмом ./dbdata:/var/lib/mysql;
* wordpress из образа wordpress:6.0.1-php8.0-fpm-alpine и вольюмом ./wordpress:/var/www/html;
* nginx из образа nginx:1.22.0-alpine и вольюмами ./wordpress:/var/www/html ./nginx-conf:/etc/nginx/conf.d;
* node из образа node:16.13.2-alpine3.15 и вольюмом /node:/opt/server;
* app из собственного docker-образа.
2. Деплой осуществляется с помощью docker-compose:
```
version: '3'

services:  
  database:
    image: mysql:8.0
    container_name: database
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    image: wordpress:6.0.1-php8.0-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./wordpress:/var/www/html
    networks:
      - app-network
    depends_on:
      - database

  nginx:
    image: nginx:1.22.0-alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - 8083:8083
      - 8081:8081
      - 8082:8082
    volumes:
      - ./wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
    networks:
      - app-network
    depends_on:
      - wordpress

  node:
    image: node:16.13.2-alpine3.15
    container_name: node
    working_dir: /opt/server
    volumes:
      - ./node:/opt/server
    command: node test.js
    networks:
      - app-network

  app:
    build: ./python
    container_name: app
    restart: always
    env_file:
      - .env
    command: "gunicorn --workers=2 --bind=0.0.0.0:8000 mysite.wsgi:application"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```
3. Конфигурационные файлы NGINX:
```
server {
     listen 8083;
     listen [::]:8083;

     server_name example.com www.example.com;

     index index.php index.html index.htm;

     root /var/www/html;

     location ~ /.well-known/acme-challenge {
             allow all;
             root /var/www/html;
     }

     location / {
             try_files $uri $uri/ /index.php$is_args$args;
     }

     location ~ \.php$ {
             try_files $uri =404;
             fastcgi_split_path_info ^(.+\.php)(/.+)$;
             fastcgi_pass wordpress:9000;
             fastcgi_index index.php;
             include fastcgi_params;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             fastcgi_param PATH_INFO $fastcgi_path_info;
     }

     location ~ /\.ht {
             deny all;
     }

     location = /favicon.ico {
             log_not_found off; access_log off;
     }
     location = /robots.txt {
             log_not_found off; access_log off; allow all;
     }
     location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
             expires max;
             log_not_found off;
     }
}
```
* django.conf
```
upstream django {
 server app:8000;
}

server {
    listen 8081;
    listen [::]:8081;
    server_name localhost;
    location / {
 	   try_files $uri @proxy_to_app;	
    }

    location @proxy_to_app {
 	   proxy_pass http://django;
 	   proxy_http_version 1.1;
 	   proxy_set_header Upgrade $http_upgrade;
 	   proxy_set_header Connection "upgrade";
 	   proxy_redirect off;
 	   proxy_set_header Host $host;
 	   proxy_set_header X-Real-IP $remote_addr;
 	   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 	   proxy_set_header X-Forwarded-Host $server_name;
    }
}
```
* node.conf
```
server {
 listen 8082;
 listen [::]:8082;
 server_name localhost;
    location / {
 	   proxy_pass http://node:3000;
 	   proxy_http_version 1.1;
 	   proxy_set_header Upgrade $http_upgrade;
 	   proxy_set_header Connection "upgrade";
 	   proxy_redirect off;
 	   proxy_set_header Host $host;
 	   proxy_set_header X-Real-IP $remote_addr;
 	   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 	   proxy_set_header X-Forwarded-Host $server_name;  
    }
}
```
4. Запуск контейнеров:
```
docker-compose up -d
...
docker ps
 CONTAINER ID   IMAGE                               COMMAND                  CREATED       STATUS                          PORTS                                                                   NAMES
8d555c5152be   project_app                         "gunicorn --workers=…"   4 hours ago   Up 4 hours                                                                                              app
62ce3d8c2fa6   mysql:8.0                           "docker-entrypoint.s…"   4 hours ago   Restarting (1) 54 seconds ago                                                                           database
860032539f1e   nginx:1.22.0-alpine                 "/docker-entrypoint.…"   4 hours ago   Up 4 hours                      80/tcp, 0.0.0.0:8081-8083->8081-8083/tcp, :::8081-8083->8081-8083/tcp   nginx
f0a811abf220   wordpress:6.0.1-php8.0-fpm-alpine   "docker-entrypoint.s…"   4 hours ago   Up 4 hours                      9000/tcp                                                                wordpress
26544677b744   node:16.13.2-alpine3.15             "docker-entrypoint.s…"   4 hours ago   Up 4 hours                                                                                              node  
```
5. Проверяем работоспособность.
![alt text](./Pictures/1.png)
![alt text](./Pictures/2.png)
