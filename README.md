# Nginx PHP 7.x/8.x MySQL and PHPMyAdmin (LEMP)

Docker running Nginx, PHP-FPM, Composer, MySQL and PHPMyAdmin.

## Overview

1. [Install prerequisites](#install-prerequisites)

    Before installing project make sure the following prerequisites have been met.

2. [Clone the project](#clone-the-project)

    We’ll download the code from its repository on GitHub.

3. [Configure Nginx With SSL Certificates](#configure-nginx-with-ssl-certificates)

    We'll generate and configure SSL certificate for nginx before running server.

4. [Configure Xdebug](#configure-xdebug)

    We'll configure Xdebug for IDE (PHPStorm or Netbeans).

5. [Run the application](#run-the-application)

    By this point we’ll have all the project pieces in place.

6. [Use Makefile](#use-makefile)

    When developing, you can use `Makefile` for doing recurrent operations.

7. [Use Docker Commands](#use-docker-commands)

    When running, you can use docker commands for doing recurrent operations.

___

## Install prerequisites

For now, this project has been mainly created for Unix `(Linux/MacOS)`. Perhaps it could work on Windows. Make sure to make compatible the relative paths in the `docker-compose.yml` file configuration like `c:\foo\bar`.

All requisites should be available for your distribution. The most important are :

* [Git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

Check if `docker-compose` is already installed by entering the following command : 

```sh
which docker-compose
```

The following is optional but makes life more enjoyable :

```sh
which make
```

On Ubuntu and Debian these are available in the meta-package build-essential. On other distributions, you may need to install the GNU C++ compiler separately.

```sh
sudo apt install build-essential
```

### Images to use

* [Nginx](https://hub.docker.com/_/nginx/)
* [MySQL](https://hub.docker.com/_/mysql/)
* [PHP-FPM](https://hub.docker.com/r/nanoninja/php-fpm/)
* [Composer](https://hub.docker.com/_/composer/)
* [PHPMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
* [Generate Certificate](https://hub.docker.com/r/jacoelho/generate-certificate/)

You should be careful when installing third party web servers such as MySQL or Nginx.

This project use the following ports :

| Server     | Port |
|------------|------|
| MySQL      | 8989 |
| PHPMyAdmin | 8091 |
| Nginx      | 8000 |
| Nginx SSL  | 3001 |
| redis      | 6379 |

---

## Clone the project

To install [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git), download it and install following the instructions : 

```sh
git clone https://github.com/carlosfgti/docker-lemp-php7-phpmyadmin.git
```

Go to the project directory : 

```sh
cd docker-lemp-php7-phpmyadmin
```

### Project tree

```sh
.
├── README.md
├── data
│   └── db
│       ├── dumps
│       └── mysql
│   └── redis
│       └── redis.conf
├── docker-compose.yml
├── etc
│   ├── nginx
│   │   └── default.conf
│   ├── php
│   │   └── php.ini
│   └── ssl
└── web
    └── app
    └── public
        └── index.php
```

---

## Configure Nginx With SSL Certificates (Optional)

1. Generate SSL certificates

    ```sh
    sudo docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=localhost" jacoelho/generate-certificate
    ```

2. Configure Nginx

    Edit nginx file `etc/nginx/default.conf` and uncomment the SSL server section :

    ```sh
    # server {
    #     server_name localhost;
    #
    #     listen 443 ssl;
    #     ...
    # }
    ```

---

## Configure Xdebug (Optional)

If you use another IDE than PHPStorm or Netbeans, go to the [remote debugging](https://xdebug.org/docs/remote) section of Xdebug documentation.

1. Get your own local IP address :

    ```sh
    sudo ifconfig
    ```

2. Edit php file `etc/php/php.ini` and comment or uncomment the configuration as needed.

3. Set the `remote_host` parameter with your IP :

    ```sh
    xdebug.remote_host=192.168.0.100 # your IP
    ```

---

## Run the application

1. Start the application :

    ```sh
    sudo docker-compose up -d
    ```

    **Please wait this might take a several minutes...**

    ```sh
    sudo docker-compose logs -f # Follow log output
    ```

2. Open your favorite browser :

    * [http://localhost:8000](http://localhost:8000/)
    * [https://localhost:3000](https://localhost:3001/) ([HTTPS](#configure-nginx-with-ssl-certificates) not configured by default)
    * [http://localhost:8080](http://localhost:8080/) PHPMyAdmin (username: dev, password: dev)

3. Stop and clear services

    ```sh
    sudo docker-compose down -v
    ```

---

## Use Docker commands

### Checking installed PHP extensions

```sh
sudo docker-compose exec php php -m
```

### Install additional PHP extensions

Currently we have a way to install php extensions:

```sh
sudo docker-compose exec php curl -sSL https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions -o - | sh -s gd xdebug
```
Note: `gd` and `xdebug` is an example which additional extensions will be installed.

After successfully install additional extension, you have to restart php service container. For more information, please refer to this [good source](https://github.com/mlocati/docker-php-extension-installer/tree/master).

### Handling database

#### MySQL shell access

```sh
sudo docker exec -it mysql bash
```

and

```sh
mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
```

#### Backup of database

```sh
mkdir -p data/db/dumps
```

```sh
source .env && sudo docker exec $(shell docker-compose ps -q mysqldb) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
```

or

```sh
source .env && sudo docker exec $(shell docker-compose ps -q mysqldb) mysqldump test -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/test.sql"
```

#### Restore Database

```sh
source .env && sudo docker exec -i $(sudo docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
```

#### Connecting MySQL from [PDO](http://php.net/manual/en/book.pdo.php)

```php
<?php
try {
    $dsn = 'mysql:host=mysql;dbname=test;charset=utf8;port=3306';
    $pdo = new PDO($dsn, 'dev', 'dev');
} catch (PDOException $e) {
    echo $e->getMessage();
}
```

---

## Help us !

Any thought, feedback or (hopefully not!)

By [Carlos Ferreira](https://github.com/carlosfgti)