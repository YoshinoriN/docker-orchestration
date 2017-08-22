![](https://img.shields.io/badge/Redmine-v3.4.2-blue.svg)
[![CircleCI](https://circleci.com/gh/YoshinoriN/docker-redmine-orchestration.svg?style=svg)](https://circleci.com/gh/YoshinoriN/docker-redmine-orchestration)

# docker-redmine-orchestration 

A easy and fast docker-compose for Redmine (Nginx + Unicorn + MariaDB)

[日本語はこちら](/doc/README_JA.md)

# Architecture

* MariaDB 10.3.0
* Nginx 1.13.3
* Ruby 2.4.1
* Unicorn 5.3.0

# what is difference between official docker-redmine ?

|-|Official|This application|
|---|---|---|
|DB|MySQL|MariaDB|
|Web server|-|Nginx|
|Rails server|webrick or passenger|unicorn|

# Requirements

* Higher than Docker compose 1.6

# Overview

![](./doc/img/overview.png)

# Default installed VCS

The git were installed in the Redmine container and the Redmine's `configuration.yml` has been already setted up to link git.

# Install and execute

## Install

* At first. Download Redmine's source code using by `redmine_download.sh`.

* Second. Please change Redmine's setting.
    * Redmine's setting files are contain in the `./src/config` directory.

## Docker compose up

Please execute `docker-compose up` in root directory.

```sh
docker-compose up
```

Please access `http://yourdomain:3000`.

After first build, please change `RAILS_MIGRATE` value to `0` in the `docker-compose.yml`.

# Settings

## MariaDB

Please change `redmine/mariadb/config/my.cnf`.
Also you can change it after build image.

### Character set

Default character set is `UTF-8`.

### User and password

**User and paswword are decided when build docker image.** You can specify user and password using by `docker-compose.yml`.

```yml
  mariadb:
    build: ./mariadb
    container_name: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: mypass
      MYSQL_USER: redmine
      MYSQL_DATABASE: redmine
```

You can remove `MYSQL_ROOT_PASSWORD` key in `docker-compose.yml`, after first docker image build.

If you change `MYSQL_USER` and `MYSQL_DATABASE`. You have to change `redmine/redmine/config/database.yml` and `DB_PING_USER` and `DB_PING_USER_PASSWORD`.

```yml
  redmine:
    build: ./redmine
    container_name: redmine
    environment:
      RAILS_MIGRATE: 1
      PLUGINS_MIGRATE: 1
      DB_PING_USER: redmine
      DB_PING_USER_PASSWORD: redmine 
```
Redmine container have to start after MariaDB container. So,
`DB_PING_USER` and `DB_PING_USER_PASSWORD` are wait to start MariaDB container.

### Directory connecting

The default setting you can't connect MariaDB directly.
If you want to connect directly, please add below key in `docker-compose.yml`.

```yml
mariadb:
    ports:
      - "3306:3306"
```

You can connect MariaDB directory using by `3306` port. Also you can change port number.


## Nginx

Please change `./nginx/config/nginx.conf`.
Also you can change it after build image.

### HTTPS

You have to change `./nginx/config/nginx.conf` and `docker-compose.yml`.

At first please configure `./nginx/config/nginx.conf`.
And change server settings. Below is example.

```yml
server {
        #NOTE : for TLS connection.
        ssl on;
        ssl_prefer_server_ciphers on;
        ssl_protocols TLSv1.2;
        ssl_certificate <key's path in nginx container>
        ssl_certificate_key <key's path in nginx container>
```

Next please change `docker-compose.yml`. You have to put key on **host** machine. And mount these keys on nginx container. So you have to specify host key's path and mount volume path in `docker-compose.yml`.

```yml
    #NOTE : TLS key's path for HTTPS
     - <host ssl_certificate key's path> : <nginx.conf ssl_certificate key's path>
     - <host ssl_certificate_key key's path> : <nginx.conf  ssl_certificate_key key's path>
```

Please change Redmine connection settings to HTTPS using by Redmine's management console.

## Redmine

Please change `./redmine/src/config`.
Also you can change it after build image.

### Install plugins

Please put plugins in the `redmine/redmine/redmine/plugins` directory.

And please set `PLUGINS_MIGRATE` key's value to `1`.

```yml
  redmine:
    build: ./redmine
    container_name: redmine
    environment:
      RAILS_MIGRATE: 1
      PLUGINS_MIGRATE: 1
```

After that these plugins were installed 

Are plugins installed every docker compose up. This behavior for flexible install. (User can select what plugin do you use and uninstall every time easily.)

# Container's time zone

If you want to change container's time zone. Please add `TZ` key to each container using by `docker-compose.yml`.

Below is example.

```yml
  mariadb-storage:
    build: ./storage/mariadb-storage
    container_name: mariadb-storage
    environment:
      TZ: Asia/Tokyo
    volumes:
     - ./storage/mariadb-storage/data:/var/lib/mysql
```

# Others

You can customize other settings using by `docker-compose.yml`

# git

If you integrate git repositories with Redmine. Please create repositories below directory.

You can create some repositories in the directory.

```sh
./storage/git-storage/repositories/<your-repository>
```

And above directory are mount below directory in Redmine container.

```sh
/usr/src/git/<your-repository>
```

# Back up

Please execute `backup.sh`. Back up files are create in `buckups` directory by `tar` format.

# Direcroty Hierarchy

```sh
.
├── backups
├── backup.sh
├── circle.yml
├── create_images.sh
├── doc
│   ├── img
│   │   └── overview.png
│   ├── overview.pptx
│   └── README_JA.md
├── docker-compose.yml
├── export_images.sh
├── images
├── import_images.sh
├── LICENSE
├── load_images.sh
├── logs
│   ├── nginx
│   └── redmine
├── mariadb
│   ├── config
│   │   └── my.cnf
│   └── Dockerfile
├── nginx
│   ├── config
│   │   └── nginx.conf
│   └── Dockerfile
├── README.md
├── redmine
│   ├── docker-entrypoint.sh
│   ├── Dockerfile
│   ├── Gemfile
│   ├── README.md
│   ├── redmine_download.sh
│   └── src
│       └── config
│           ├── configuration.yml
│           ├── database.yml
│           └── unicorn.rb
└── storage
    ├── git-storage
    │   ├── Dockerfile
    │   └── repositories
    │       └── README.md
    ├── mariadb-storage
    │   ├── data
    │   └── Dockerfile
    └── redmine-files
        ├── Dockerfile
        └── files
```
