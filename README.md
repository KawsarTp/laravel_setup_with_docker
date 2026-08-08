# Laravel Docker Development Environment

A complete Docker-based Laravel development environment using PHP-FPM, Nginx, MySQL, and Composer.

The goal of this setup is simple:

- Developers only need Docker and Git on their host machine.
- PHP runs inside Docker.
- Composer runs inside Docker.
- Nginx runs inside Docker.
- MySQL runs inside Docker.
- Laravel runs inside Docker.
- The source code is shared with the containers through a bind mount.
- MySQL data is persisted using a Docker named volume.

---

# Table of Contents

- [1. Overview](#1-overview)
- [2. Requirements](#2-requirements)
  - [2.1 Docker](#21-docker)
  - [2.2 Docker Compose](#22-docker-compose)
  - [2.3 Git](#23-git)
- [3. Architecture](#3-architecture)
- [4. Services](#4-services)
- [5. Project Structure](#5-project-structure)
- [6. Docker Configuration](#6-docker-configuration)
  - [6.1 PHP Dockerfile](#61-php-dockerfile)
  - [6.2 Nginx Dockerfile](#62-nginx-dockerfile)
  - [6.3 Nginx Configuration](#63-nginx-configuration)
  - [6.4 Docker Compose](#64-docker-compose)
- [7. Environment Configuration](#7-environment-configuration)
  - [7.1 Create `.env`](#71-create-env)
  - [7.2 Database Configuration](#72-database-configuration)
  - [7.3 Docker Database Networking](#73-docker-database-networking)
- [8. First-Time Installation](#8-first-time-installation)
  - [8.1 Clone the Repository](#81-clone-the-repository)
  - [8.2 Create `.env`](#82-create-env)
  - [8.3 Build Images](#83-build-images)
  - [8.4 Start Containers](#84-start-containers)
  - [8.5 Check Containers](#85-check-containers)
  - [8.6 Install Composer Dependencies](#86-install-composer-dependencies)
  - [8.7 Generate Application Key](#87-generate-application-key)
  - [8.8 Clear Laravel Cache](#88-clear-laravel-cache)
  - [8.9 Run Migrations](#89-run-migrations)
- [9. Access the Application](#9-access-the-application)
- [10. Database Access](#10-database-access)
  - [10.1 Laravel to MySQL](#101-laravel-to-mysql)
  - [10.2 Host to MySQL](#102-host-to-mysql)
  - [10.3 MySQL CLI](#103-mysql-cli)
- [11. Laravel Artisan Commands](#11-laravel-artisan-commands)
  - [11.1 Common Commands](#111-common-commands)
  - [11.2 Artisan Shortcut](#112-artisan-shortcut)
- [12. Composer Commands](#12-composer-commands)
  - [12.1 Common Commands](#121-common-commands-1)
  - [12.2 Composer Shortcut](#122-composer-shortcut)
- [13. Docker Commands](#13-docker-commands)
  - [13.1 Start](#131-start)
  - [13.2 Stop](#132-stop)
  - [13.3 Restart](#133-restart)
  - [13.4 Down](#134-down)
  - [13.5 Status](#135-status)
  - [13.6 Build](#136-build)
  - [13.7 Rebuild Without Cache](#137-rebuild-without-cache)
- [14. Container Access](#14-container-access)
  - [14.1 PHP Container](#141-php-container)
  - [14.2 Nginx Container](#142-nginx-container)
  - [14.3 MySQL Container](#143-mysql-container)
- [15. Logs](#15-logs)
- [16. Laravel Permissions](#16-laravel-permissions)
- [17. Cache Management](#17-cache-management)
- [18. Rebuilding After Dockerfile Changes](#18-rebuilding-after-dockerfile-changes)
- [19. Database Reset](#19-database-reset)
- [20. Daily Development Workflow](#20-daily-development-workflow)
- [21. Team / New Developer Setup](#21-team--new-developer-setup)
- [22. Git Rules](#22-git-rules)
- [23. Host vs Docker Ports](#23-host-vs-docker-ports)
- [24. Docker Concepts](#24-docker-concepts)
  - [24.1 Dockerfile](#241-dockerfile)
  - [24.2 Image](#242-image)
  - [24.3 Container](#243-container)
  - [24.4 Docker Compose](#244-docker-compose)
  - [24.5 Network](#245-network)
  - [24.6 Volume](#246-volume)
  - [24.7 Bind Mount](#247-bind-mount)
- [25. Troubleshooting](#25-troubleshooting)
  - [25.1 Port Already in Use](#251-port-already-in-use)
  - [25.2 Database Connection Error](#252-database-connection-error)
  - [25.3 Access Denied for MySQL](#253-access-denied-for-mysql)
  - [25.4 Nginx Default Page](#254-nginx-default-page)
  - [25.5 PHP File Downloads Instead of Running](#255-php-file-downloads-instead-of-running)
  - [25.6 502 Bad Gateway](#256-502-bad-gateway)
  - [25.7 Permission Denied](#257-permission-denied)
  - [25.8 `tempnam()` / Temporary File Error](#258-tempnam--temporary-file-error)
  - [25.9 Composer Network Timeout](#259-composer-network-timeout)
  - [25.10 Unknown `--network` Flag](#2510-unknown---network-flag)
  - [25.11 `.env` Changes Not Taking Effect](#2511-env-changes-not-taking-effect)
  - [25.12 Container Is Not Running](#2512-container-is-not-running)
- [26. Complete Fresh Setup](#26-complete-fresh-setup)
- [27. Quick Reference](#27-quick-reference)
- [28. Final Architecture](#28-final-architecture)

---

# 1. Overview

This project runs Laravel completely inside Docker.

The host machine does not need PHP, Composer, MySQL, or Nginx installed directly.

The application consists of:

```text
Browser
   |
   v
Nginx
   |
   v
PHP-FPM
   |
   v
Laravel
   |
   v
MySQL
```

---

# 2. Requirements

Only the following are required on the host machine:

- Docker
- Docker Compose
- Git

You do **not** need to install:

- PHP
- Composer
- PHP-FPM
- MySQL
- Nginx

---

## 2.1 Docker

Check Docker:

```bash
docker --version
```

Example:

```text
Docker version 29.x.x
```

If Docker is not installed, install Docker according to the official Docker documentation for your operating system.

---

## 2.2 Docker Compose

Check Docker Compose:

```bash
docker compose version
```

This project uses the modern Compose command:

```bash
docker compose
```

not:

```bash
docker-compose
```

---

## 2.3 Git

Check Git:

```bash
git --version
```

---

# 3. Architecture

The Docker environment is structured as follows:

```text
                         Browser
                            |
                            |
                  http://localhost:8080
                            |
                            v
                  +----------------+
                  |     Nginx      |
                  |      :80       |
                  +-------+--------+
                          |
                          | FastCGI
                          | php:9000
                          v
                  +----------------+
                  |   PHP-FPM      |
                  |    PHP 8.x     |
                  |    Laravel     |
                  |    Composer    |
                  +-------+--------+
                          |
                          | MySQL protocol
                          | mysql:3306
                          v
                  +----------------+
                  |     MySQL      |
                  |      8.x       |
                  +-------+--------+
                          |
                          v
                     mysql_data
                        volume
```

---

# 4. Services

| Service | Technology | Container Port | Host Port |
|---|---|---:|---:|
| `nginx` | Nginx | 80 | 8080 |
| `php` | PHP-FPM | 9000 | Not exposed |
| `mysql` | MySQL | 3306 | 3307 |

Important:

- `8080` is the host port for the web application.
- `3307` is the host port for MySQL tools such as DBeaver or MySQL Workbench.
- Laravel does **not** use `3307` to connect to MySQL.
- Laravel uses `mysql:3306`.

---

# 5. Project Structure

Recommended project structure:

```text
project/
│
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
├── tests/
│
├── docker/
│   └── nginx/
│       └── default.conf
│
├── .env
├── .env.example
├── .gitignore
├── artisan
├── composer.json
├── composer.lock
│
├── Dockerfile
├── Dockerfile.nginx
├── docker-compose.yml
└── README.md
```

---

# 6. Docker Configuration

## 6.1 PHP Dockerfile

File:

```text
Dockerfile
```

Content:

```dockerfile
FROM php:8.4-fpm

WORKDIR /var/www/html

# Install system dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
        curl \
        unzip \
        zip \
        libzip-dev \
        libicu-dev \
        libxml2-dev \
        libpq-dev \
        libonig-dev \
        libcurl4-openssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-configure intl \
    && docker-php-ext-install \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        mbstring \
        intl \
        xml \
        curl \
        zip \
        bcmath \
        opcache

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Configure temporary directory
RUN chmod 1777 /tmp

CMD ["php-fpm"]
```

### What this Dockerfile does

It:

1. Uses PHP-FPM.
2. Installs Linux packages required by Laravel and Composer.
3. Installs common Laravel PHP extensions.
4. Copies Composer into the PHP image.
5. Configures `/tmp`.
6. Creates Laravel writable directories.
7. Sets ownership for `storage` and `bootstrap/cache`.
8. Starts PHP-FPM.

---

## 6.2 Nginx Dockerfile

File:

```text
Dockerfile.nginx
```

Content:

```dockerfile
FROM nginx:stable-alpine

COPY docker/nginx/default.conf \
     /etc/nginx/conf.d/default.conf

WORKDIR /var/www/html
```

This image:

- Uses Alpine-based Nginx.
- Replaces the default Nginx configuration.
- Uses `/var/www/html` as the working directory.

---

## 6.3 Nginx Configuration

File:

```text
docker/nginx/default.conf
```

Content:

```nginx
server {

    listen 80;

    server_name localhost;

    root /var/www/html/public;

    index index.php index.html;


    # --------------------------------------------------
    # Laravel routes
    # --------------------------------------------------

    location / {

        try_files $uri $uri/ /index.php?$query_string;

    }


    # --------------------------------------------------
    # PHP-FPM
    # --------------------------------------------------

    location ~ \.php$ {

        try_files $uri =404;

        fastcgi_pass php:9000;

        fastcgi_index index.php;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    }


    # --------------------------------------------------
    # Block hidden files
    # --------------------------------------------------

    location ~ /\. {

        deny all;

    }

}
```

### Important configuration

Laravel's public directory is:

```text
/var/www/html/public
```

Therefore:

```nginx
root /var/www/html/public;
```

is required.

PHP requests are sent to:

```nginx
fastcgi_pass php:9000;
```

Here:

```text
php
```

is the Docker Compose service name.

---

## 6.4 Docker Compose

File:

```text
docker-compose.yml
```

Content:

```yaml
services:

  php:
    build:
      context: .
      dockerfile: Dockerfile

    container_name: laravel_php

    volumes:
      - ./:/var/www/html

    depends_on:
      - mysql

    networks:
      - laravel


  nginx:
    build:
      context: .
      dockerfile: Dockerfile.nginx

    container_name: laravel_nginx

    ports:
      - "8080:80"

    volumes:
      - ./:/var/www/html

    depends_on:
      - php

    networks:
      - laravel


  mysql:
    image: mysql:8.4

    container_name: laravel_mysql

    restart: unless-stopped

    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: laravel
      MYSQL_PASSWORD: laravel

    ports:
      - "3307:3306"

    volumes:
      - mysql_data:/var/lib/mysql

    networks:
      - laravel


networks:

  laravel:
    driver: bridge


volumes:

  mysql_data:
```

### Important

Docker Compose automatically creates a private network for the services.

Inside this network:

```text
php
mysql
nginx
```

can communicate using their service names.

Therefore:

```text
php -> mysql:3306
nginx -> php:9000
```

---

# 7. Environment Configuration

## 7.1 Create `.env`

Copy the example file:

```bash
cp .env.example .env
```

Open:

```bash
nano .env
```

---

## 7.2 Database Configuration

Use:

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel
```

The important values are:

```env
DB_HOST=mysql
DB_PORT=3306
```

---

## 7.3 Docker Database Networking

Inside Docker:

```text
PHP container
      |
      | mysql:3306
      v
MySQL container
```

Therefore Laravel must use:

```env
DB_HOST=mysql
DB_PORT=3306
```

Do **not** use:

```env
DB_HOST=localhost
```

Do **not** use:

```env
DB_HOST=127.0.0.1
```

Do **not** use:

```env
DB_PORT=3307
```

`3307` is only the host-side port.

---

# 8. First-Time Installation

## 8.1 Clone the Repository

```bash
git clone <REPOSITORY_URL>
```

Example:

```bash
git clone https://github.com/company/project.git
```

Enter the project:

```bash
cd project
```

---

## 8.2 Create `.env`

```bash
cp .env.example .env
```

Verify database configuration:

```bash
grep DB_ .env
```

---

## 8.3 Build Images

```bash
docker compose build
```

For a completely clean build:

```bash
docker compose build --no-cache
```

---

## 8.4 Start Containers

```bash
docker compose up -d
```

---

## 8.5 Check Containers

```bash
docker compose ps
```

Expected services:

```text
laravel_php
laravel_nginx
laravel_mysql
```

All should show as running.

---

## 8.6 Install Composer Dependencies

Run:

```bash
docker compose exec php composer install
```

This installs dependencies into:

```text
vendor/
```

---

## 8.7 Generate Application Key

Run:

```bash
docker compose exec php php artisan key:generate
```

Expected:

```text
INFO  Application key set successfully.
```

---

## 8.8 Clear Laravel Cache

Run:

```bash
docker compose exec php php artisan optimize:clear
```

---

## 8.9 Run Migrations

Run:

```bash
docker compose exec php php artisan migrate
```

If seeders are required:

```bash
docker compose exec php php artisan migrate --seed
```

---

# 9. Access the Application

Start containers:

```bash
docker compose up -d
```

Then open:

```text
http://localhost:8080
```

---

# 10. Database Access

## 10.1 Laravel to MySQL

Laravel uses:

```env
DB_HOST=mysql
DB_PORT=3306
```

Docker resolves:

```text
mysql
```

to the MySQL container.

---

## 10.2 Host to MySQL

Database tools running on the host machine should use:

```text
Host: 127.0.0.1
Port: 3307
Database: laravel
Username: laravel
Password: laravel
```

For example, DBeaver or MySQL Workbench can connect using:

```text
127.0.0.1:3307
```

---

## 10.3 MySQL CLI

Connect directly:

```bash
docker compose exec mysql mysql -ularavel -plaravel
```

Root:

```bash
docker compose exec mysql mysql -uroot -proot
```

---

# 11. Laravel Artisan Commands

## 11.1 Common Commands

Migration:

```bash
docker compose exec php php artisan migrate
```

Fresh migration:

```bash
docker compose exec php php artisan migrate:fresh
```

Fresh migration with seed:

```bash
docker compose exec php php artisan migrate:fresh --seed
```

Routes:

```bash
docker compose exec php php artisan route:list
```

Tinker:

```bash
docker compose exec php php artisan tinker
```

Queue worker:

```bash
docker compose exec php php artisan queue:work
```

Tests:

```bash
docker compose exec php php artisan test
```

Create controller:

```bash
docker compose exec php php artisan make:controller ProductController
```

Create model and migration:

```bash
docker compose exec php php artisan make:model Product -m
```

Create migration:

```bash
docker compose exec php php artisan make:migration create_products_table
```

Clear everything:

```bash
docker compose exec php php artisan optimize:clear
```

---

## 11.2 Artisan Shortcut

Typing this repeatedly is inconvenient:

```bash
docker compose exec php php artisan
```

Create a Bash function.

Open:

```bash
nano ~/.bashrc
```

Add:

```bash
art() {
    docker compose exec php php artisan "$@"
}
```

Reload Bash:

```bash
source ~/.bashrc
```

Now you can use:

```bash
art migrate
```

```bash
art migrate --seed
```

```bash
art route:list
```

```bash
art optimize:clear
```

```bash
art tinker
```

```bash
art make:model Product -m
```

---

# 12. Composer Commands

## 12.1 Common Commands

Install dependencies:

```bash
docker compose exec php composer install
```

Update dependencies:

```bash
docker compose exec php composer update
```

Require package:

```bash
docker compose exec php composer require vendor/package
```

Example:

```bash
docker compose exec php composer require laravel/sanctum
```

Remove package:

```bash
docker compose exec php composer remove vendor/package
```

Check Composer:

```bash
docker compose exec php composer --version
```

---

## 12.2 Composer Shortcut

Open:

```bash
nano ~/.bashrc
```

Add:

```bash
dcomposer() {
    docker compose exec php composer "$@"
}
```

Reload:

```bash
source ~/.bashrc
```

Now:

```bash
dcomposer install
```

```bash
dcomposer update
```

```bash
dcomposer require laravel/sanctum
```

```bash
dcomposer remove laravel/sanctum
```

---

# 13. Docker Commands

## 13.1 Start

```bash
docker compose up -d
```

---

## 13.2 Stop

```bash
docker compose stop
```

This stops containers without removing them.

---

## 13.3 Restart

```bash
docker compose restart
```

---

## 13.4 Down

```bash
docker compose down
```

This removes containers and the Compose network.

The named MySQL volume remains.

---

## 13.5 Status

```bash
docker compose ps
```

---

## 13.6 Build

```bash
docker compose build
```

---

## 13.7 Rebuild Without Cache

```bash
docker compose build --no-cache
```

Then:

```bash
docker compose up -d
```

---

# 14. Container Access

## 14.1 PHP Container

Open a shell:

```bash
docker compose exec php bash
```

Check PHP:

```bash
php -v
```

Check Composer:

```bash
composer --version
```

Check Laravel:

```bash
php artisan --version
```

Exit:

```bash
exit
```

---

## 14.2 Nginx Container

Open:

```bash
docker compose exec nginx sh
```

Check Nginx:

```bash
nginx -v
```

Check complete configuration:

```bash
nginx -T
```

Exit:

```bash
exit
```

---

## 14.3 MySQL Container

Open:

```bash
docker compose exec mysql bash
```

Or directly connect:

```bash
docker compose exec mysql mysql -ularavel -plaravel
```

---

# 15. Logs

All services:

```bash
docker compose logs
```

Follow all logs:

```bash
docker compose logs -f
```

PHP:

```bash
docker compose logs php
```

Follow PHP:

```bash
docker compose logs -f php
```

Nginx:

```bash
docker compose logs nginx
```

Follow Nginx:

```bash
docker compose logs -f nginx
```

MySQL:

```bash
docker compose logs mysql
```

Follow MySQL:

```bash
docker compose logs -f mysql
```

---

# 16. Laravel Permissions

Laravel needs write access to:

```text
storage/
bootstrap/cache/
```

Fix ownership:

```bash
docker compose exec php \
chown -R www-data:www-data storage bootstrap/cache
```

Fix permissions:

```bash
docker compose exec php \
chmod -R 775 storage bootstrap/cache
```

Then clear cache:

```bash
docker compose exec php php artisan optimize:clear
```

---

# 17. Cache Management

Clear application cache:

```bash
docker compose exec php php artisan cache:clear
```

Clear config:

```bash
docker compose exec php php artisan config:clear
```

Clear routes:

```bash
docker compose exec php php artisan route:clear
```

Clear views:

```bash
docker compose exec php php artisan view:clear
```

Clear all Laravel caches:

```bash
docker compose exec php php artisan optimize:clear
```

Recommended general command:

```bash
art optimize:clear
```

if the `art` shortcut has been configured.

---

# 18. Rebuilding After Dockerfile Changes

You normally do **not** need to rebuild when changing:

- Controllers
- Models
- Routes
- Blade files
- PHP source code
- CSS
- JavaScript

The project is bind-mounted:

```yaml
volumes:
  - ./:/var/www/html
```

Therefore source-code changes are immediately visible inside the containers.

You **do** need to rebuild after changing:

- `Dockerfile`
- PHP extensions
- Linux packages
- Composer installation method
- `Dockerfile.nginx`
- Nginx base image
- Files copied into an image during build

Run:

```bash
docker compose build
```

Then:

```bash
docker compose up -d
```

For a completely fresh build:

```bash
docker compose down
docker compose build --no-cache
docker compose up -d
```

---

# 19. Database Reset

WARNING:

This deletes the MySQL Docker volume and therefore deletes the database.

Run:

```bash
docker compose down -v
```

Then:

```bash
docker compose up -d
```

Then:

```bash
docker compose exec php php artisan migrate
```

Or:

```bash
docker compose exec php php artisan migrate --seed
```

Never run:

```bash
docker compose down -v
```

against a database containing data you need.

---

# 20. Daily Development Workflow

Start the project:

```bash
docker compose up -d
```

Check services:

```bash
docker compose ps
```

Open:

```text
http://localhost:8080
```

When `.env` changes:

```bash
art config:clear
```

or:

```bash
docker compose exec php php artisan config:clear
```

After a new migration:

```bash
art migrate
```

After changing Composer dependencies:

```bash
dcomposer install
```

For logs:

```bash
docker compose logs -f
```

At the end of the day:

```bash
docker compose stop
```

---

# 21. Team / New Developer Setup

A new developer does not need PHP, Composer, MySQL, or Nginx installed locally.

They only need:

```text
Docker
Docker Compose
Git
```

## Step 1: Clone

```bash
git clone <REPOSITORY_URL>
```

## Step 2: Enter Project

```bash
cd <PROJECT_DIRECTORY>
```

## Step 3: Create Environment

```bash
cp .env.example .env
```

## Step 4: Build

```bash
docker compose build
```

## Step 5: Start

```bash
docker compose up -d
```

## Step 6: Check

```bash
docker compose ps
```

## Step 7: Install Dependencies

```bash
docker compose exec php composer install
```

## Step 8: Generate Key

```bash
docker compose exec php php artisan key:generate
```

## Step 9: Clear Cache

```bash
docker compose exec php php artisan optimize:clear
```

## Step 10: Run Migrations

```bash
docker compose exec php php artisan migrate
```

## Step 11: Open Application

```text
http://localhost:8080
```

The new developer is now ready to work.

---

# 22. Git Rules

## 22.1 Commit These Files

The following Docker-related files should be committed:

```text
Dockerfile
Dockerfile.nginx
docker-compose.yml
docker/nginx/default.conf
.env.example
composer.json
composer.lock
README.md
```

---

## 22.2 Never Commit These

Never commit:

```text
.env
/vendor/
/node_modules/
```

Do not commit:

- Database passwords
- API keys
- Secret keys
- Access tokens
- Production credentials

---

## 22.3 Recommended `.gitignore`

Example:

```gitignore
/vendor/
/node_modules/

.env
.env.backup
.env.production

.phpunit.result.cache

/storage/*.key
/storage/logs/*
```

---

# 23. Host vs Docker Ports

This is an important Docker concept.

## Nginx

Compose:

```yaml
ports:
  - "8080:80"
```

Means:

```text
HOST                  CONTAINER

localhost:8080  --->  nginx:80
```

Browser:

```text
http://localhost:8080
```

Nginx internally listens on:

```text
80
```

---

## MySQL

Compose:

```yaml
ports:
  - "3307:3306"
```

Means:

```text
HOST                  CONTAINER

localhost:3307  --->  mysql:3306
```

A database application on the host uses:

```text
127.0.0.1:3307
```

Laravel inside Docker uses:

```text
mysql:3306
```

This distinction is critical.

---

# 24. Docker Concepts

## 24.1 Dockerfile

A Dockerfile contains instructions for building a Docker image.

Example:

```text
Dockerfile
    |
    v
Docker Image
    |
    v
Container
```

---

## 24.2 Image

An image is a template from which containers are created.

For example:

```text
PHP 8.4 + extensions + Composer
              |
              v
         PHP Image
              |
              v
         PHP Container
```

---

## 24.3 Container

A container is a running instance of an image.

This project has:

```text
laravel_php
laravel_nginx
laravel_mysql
```

---

## 24.4 Docker Compose

Docker Compose manages multiple services together.

The main configuration is:

```text
docker-compose.yml
```

It defines:

- Services
- Networks
- Volumes
- Ports
- Environment variables
- Build instructions
- Dependencies

---

## 24.5 Network

All services are connected to:

```text
laravel
```

network.

Therefore:

```text
nginx -> php:9000
php   -> mysql:3306
```

The service names become DNS hostnames inside the Docker network.

---

## 24.6 Volume

MySQL uses:

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

The named volume:

```text
mysql_data
```

stores MySQL data independently from the container.

Therefore:

```bash
docker compose down
```

does not normally delete the database.

But:

```bash
docker compose down -v
```

removes the volume.

---

## 24.7 Bind Mount

PHP and Nginx use:

```yaml
volumes:
  - ./:/var/www/html
```

This maps the host project directory into the containers.

Therefore:

```text
Host:
./app

        |
        | bind mount
        v

Container:
/var/www/html/app
```

When a developer changes a PHP file, the container immediately sees the change.

A rebuild is not required for normal application-code changes.

---

# 25. Troubleshooting

## 25.1 Port Already in Use

Check port 3306:

```bash
sudo ss -ltnp | grep :3306
```

Check port 3307:

```bash
sudo ss -ltnp | grep :3307
```

Check port 8080:

```bash
sudo ss -ltnp | grep :8080
```

If host port `8080` is already occupied, change:

```yaml
ports:
  - "8080:80"
```

to another host port, for example:

```yaml
ports:
  - "8081:80"
```

Then open:

```text
http://localhost:8081
```

---

## 25.2 Database Connection Error

Check:

```bash
docker compose ps
```

Make sure MySQL is running.

Check logs:

```bash
docker compose logs mysql
```

Check `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel
```

Then:

```bash
docker compose exec php php artisan config:clear
```

Test:

```bash
docker compose exec php php artisan migrate
```

---

## 25.3 Access Denied for MySQL

Make sure credentials match:

```yaml
MYSQL_DATABASE: laravel
MYSQL_USER: laravel
MYSQL_PASSWORD: laravel
```

and:

```env
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel
```

Important:

MySQL environment variables are mainly used when the database is initialized for the first time.

If the MySQL volume already exists with old credentials, changing `docker-compose.yml` does not automatically recreate the existing database users.

For a disposable development database:

```bash
docker compose down -v
docker compose up -d
```

Then:

```bash
docker compose exec php php artisan migrate
```

WARNING: This deletes database data.

---

## 25.4 Nginx Default Page

Check Nginx configuration:

```bash
docker compose exec nginx nginx -T
```

Make sure:

```nginx
root /var/www/html/public;
```

exists.

Make sure:

```text
docker/nginx/default.conf
```

is copied into:

```text
/etc/nginx/conf.d/default.conf
```

Rebuild Nginx:

```bash
docker compose build nginx
```

Start:

```bash
docker compose up -d
```

---

## 25.5 PHP File Downloads Instead of Running

This usually means Nginx is not correctly configured for PHP-FPM.

Make sure:

```nginx
location ~ \.php$ {

    try_files $uri =404;

    fastcgi_pass php:9000;

    include fastcgi_params;

    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

Then:

```bash
docker compose build nginx
docker compose up -d
```

---

## 25.6 502 Bad Gateway

Check PHP:

```bash
docker compose ps php
```

Check PHP logs:

```bash
docker compose logs php
```

Check Nginx logs:

```bash
docker compose logs nginx
```

Make sure Nginx uses:

```nginx
fastcgi_pass php:9000;
```

The PHP service must be named:

```yaml
services:
  php:
```

---

## 25.7 Permission Denied

Run:

```bash
docker compose exec php \
chown -R www-data:www-data storage bootstrap/cache
```

Then:

```bash
docker compose exec php \
chmod -R 775 storage bootstrap/cache
```

Then:

```bash
docker compose exec php php artisan optimize:clear
```

---

## 25.8 `tempnam()` / Temporary File Error

Check:

```bash
docker compose exec php ls -ld /tmp
```

Fix:

```bash
docker compose exec php chmod 1777 /tmp
```

The expected permission includes:

```text
drwxrwxrwt
```

---

## 25.9 Composer Network Timeout

If Composer reports:

```text
curl error 28
Connection timed out
```

test Packagist from the PHP container:

```bash
docker compose exec php curl -I https://repo.packagist.org
```

Test GitHub:

```bash
docker compose exec php curl -I https://github.com
```

If the host has Internet access but the container does not, investigate Docker DNS/network connectivity.

Do not change Laravel database configuration to solve a Docker Internet problem.

---

## 25.10 Unknown `--network` Flag

Do not use unsupported commands such as:

```bash
docker compose build --network host
```

Use:

```bash
docker compose build
```

or:

```bash
docker compose build --no-cache
```

If a build needs special network configuration, solve the Docker/host networking issue instead of adding unsupported flags.

---

## 25.11 `.env` Changes Not Taking Effect

Laravel configuration may be cached.

Run:

```bash
docker compose exec php php artisan config:clear
```

Or:

```bash
docker compose exec php php artisan optimize:clear
```

If using the shortcut:

```bash
art config:clear
```

---

## 25.12 Container Is Not Running

Check:

```bash
docker compose ps
```

Check all logs:

```bash
docker compose logs
```

PHP:

```bash
docker compose logs php
```

Nginx:

```bash
docker compose logs nginx
```

MySQL:

```bash
docker compose logs mysql
```

If necessary, rebuild:

```bash
docker compose down
docker compose build --no-cache
docker compose up -d
```

---

# 26. Complete Fresh Setup

For a completely fresh developer machine:

```bash
git clone <REPOSITORY_URL>
```

```bash
cd <PROJECT_DIRECTORY>
```

```bash
cp .env.example .env
```

Verify:

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel
```

Build:

```bash
docker compose build
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Install dependencies:

```bash
docker compose exec php composer install
```

Generate key:

```bash
docker compose exec php php artisan key:generate
```

Clear cache:

```bash
docker compose exec php php artisan optimize:clear
```

Run migrations:

```bash
docker compose exec php php artisan migrate
```

Open:

```text
http://localhost:8080
```

---

# 27. Quick Reference

## Start

```bash
docker compose up -d
```

## Stop

```bash
docker compose stop
```

## Restart

```bash
docker compose restart
```

## Down

```bash
docker compose down
```

## Status

```bash
docker compose ps
```

## Logs

```bash
docker compose logs -f
```

## Build

```bash
docker compose build
```

## Clean Build

```bash
docker compose build --no-cache
```

## Laravel

```bash
docker compose exec php php artisan
```

## Artisan Shortcut

```bash
art migrate
```

## Composer

```bash
docker compose exec php composer
```

## Composer Shortcut

```bash
dcomposer install
```

## PHP Shell

```bash
docker compose exec php bash
```

## Nginx Shell

```bash
docker compose exec nginx sh
```

## MySQL

```bash
docker compose exec mysql mysql -ularavel -plaravel
```

## Clear Laravel Cache

```bash
art optimize:clear
```

## Migration

```bash
art migrate
```

## Migration + Seeder

```bash
art migrate --seed
```

## Database Reset

WARNING: Deletes the database volume.

```bash
docker compose down -v
```

---

# 28. Final Architecture

```text
                         ┌───────────────────────┐
                         │       Browser         │
                         │ localhost:8080        │
                         └───────────┬───────────┘
                                     │
                                     │ HTTP
                                     ▼
                         ┌───────────────────────┐
                         │        Nginx          │
                         │       Port 80         │
                         │                       │
                         │ /var/www/html/public │
                         └───────────┬───────────┘
                                     │
                                     │ FastCGI
                                     │ php:9000
                                     ▼
                         ┌───────────────────────┐
                         │       PHP-FPM         │
                         │       PHP 8.x         │
                         │                       │
                         │       Laravel         │
                         │       Composer        │
                         └───────────┬───────────┘
                                     │
                                     │ MySQL
                                     │ mysql:3306
                                     ▼
                         ┌───────────────────────┐
                         │        MySQL          │
                         │        Port 3306      │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │      mysql_data       │
                         │    Docker Volume      │
                         └───────────────────────┘
```

## Host Port Mapping

```text
Browser
   |
   | localhost:8080
   v
Nginx container :80


DBeaver / MySQL Workbench
   |
   | localhost:3307
   v
MySQL container :3306
```

## Internal Docker Communication

```text
Nginx
  |
  | php:9000
  v
PHP-FPM
  |
  | mysql:3306
  v
MySQL
```

---

## Application URL

```text
http://localhost:8080
```

The application is now fully Dockerized and can be shared with the development team through Git. Each developer can clone the repository, run the Docker setup, and use the same PHP, Nginx, Composer, and MySQL environment without installing those services directly on the host machine.