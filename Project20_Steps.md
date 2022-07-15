# MIGRATION-TO-THE-CLOUD-WITH-CONTAINERIZATION--DOCKER-COMPOSE
PROJECT 20  

Get docker engine in Ubuntu  
``` bash
hector@hector-Laptop:~$ docker --version
Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1
hector@hector-Laptop:~$
```




I will use a pre-built **MySQL** database container, configure it, and make sure it is ready to receive requests from our PHP application.  

To avoid having to run docker with `sudo`  
``` bash
sudo chmod 666 /var/run/docker.sock #changing permissions
sudo useradd -G docker $USER #add current user to group docker
```

Currently I have no images
``` bash
hector@hector-Laptop:~$ docker images ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
Getting the image I need `mysql/mysql-server:latest`  
``` bash
hector@hector-Laptop:~$ docker pull mysql/mysql-server:latest
latest: Pulling from mysql/mysql-server
1866ca302f77: Pull complete
7208ad90232c: Pull complete
b2d9c817f662: Pull complete
3292176f57b6: Pull complete
d77130a2f17e: Pull complete
47bc31a509ca: Pull complete
fade2a0af17d: Pull complete
Digest: sha256:1a8d2a5584e53a42a43cbd430ae340a36942afee9e14a86624a2cb2d90ce655b
Status: Downloaded newer image for mysql/mysql-server:latest
docker.io/mysql/mysql-server:latest
```

Now I see the image I just downloaded  
``` bash
hector@hector-Laptop:~$ docker images
REPOSITORY           TAG       IMAGE ID       CREATED        SIZE
mysql/mysql-server   latest    5a9594052aec   3 weeks ago    438MB
```


deploying a new MySQL container  
`docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d <docker-image>`  

`docker run --name MySQL -e MYSQL_ROOT_PASSWORD=Passw0rd! -d mysql/mysql-server:latest`  


 MySQL container is running:

``` bash
hector@hector-Laptop:~$ docker run --name MySQL -e MYSQL_ROOT_PASSWORD=Passw0rd! -d mysql/mysql-server:latest
ce2dcf1ec7aab8d2dd5c70919ead800fcdcbdc623a64b292a8876f26fcc0f746
hector@hector-Laptop:~$ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS                             PORTS                       NAMES
ce2dcf1ec7aa   mysql/mysql-server:latest   "/entrypoint.sh mysq…"   31 seconds ago   Up 30 seconds (health: starting)   3306/tcp, 33060-33061/tcp   MySQL
```

Now I will do some cleaning to start fresh  
**Stopping all running containers** `docker stop $(docker ps -aq)`   
**Remove all containers** `docker rm $(docker ps -aq)`  
(alternative I can copy paste individual Container ID)


# CONNECTING TO THE MYSQL DOCKER CONTAINER

First, create a network   
`docker network create --subnet=10.0.0.0/24 tooling_app_network`

I can double check with `docker network ls`  
``` bash
hector@hector-Laptop:~$ docker network ls
NETWORK ID     NAME                  DRIVER    SCOPE
8c221e829dfd   bridge                bridge    local
92a7c27b8b67   host                  host      local
7210e0bcf142   none                  null      local
79c067ecb744   tooling_app_network   bridge    local
hector@hector-Laptop:~$
```

Creating a new container **mysql-server**  
``` bash
export MYSQL_PW="Passw0rd!" #env variable to store root password
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW -d mysql/mysql-server:latest
```
Flags used:  
* **-d** runs the container in detached mode  
* **--network** connects a container to a network  
* **-h** specifies a hostname  
* **-e**  to set  environment variable in the container  

Double checking container creation `docker ps -a`  
``` bash
hector@hector-Laptop:~$ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS                   PORTS                       NAMES
acbcd4888fe6   mysql/mysql-server:latest   "/entrypoint.sh mysq…"   7 minutes ago   Up 7 minutes (healthy)   3306/tcp, 33060-33061/tcp   mysql-server
```

Creating script `create_user.sql` that generates a **user** and **database** called `hector`  
``` bash
hector@hector-Laptop:~$ bat create_user.sql
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: create_user.sql
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 
   1   │ CREATE DATABASE hector;
   2   │ CREATE USER 'hector'@'%' IDENTIFIED BY 'password';
   3   │ GRANT ALL PRIVILEGES ON * . * TO 'hector'@'%';
   4   │
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Now I execute this script inside of the **mysql-server** container

``` bash
hector@hector-Laptop:~$ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql #make sure no space after -p
mysql: [Warning] Using a password on the command line interface can be insecure.
hector@hector-Laptop:~$
```

Connecting to **mysql-server** using another container in interactive mode  
``` bash
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u hector -p
```
Flags used:  
* **--name** gives the container a name  
* **-it** runs in interactive mode and Allocate a pseudo-TTY  
* **--rm** automatically removes the container when it exits  
* **--network** connects a container to a network  
* **-h** a MySQL flag specifying the MySQL server Container hostname  
* **-u** user created from the SQL script  	
* **-p**  password specified for the user created from the SQL script  

Connecting to `mysql-server` and displaying database `hector` I created there  
``` bash
hector@hector-Laptop:~$ docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u hector -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 236
Server version: 8.0.29 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| hector             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql>
```

As long as we don't exit, the container will run  
``` bash

hector@hector-Laptop:~$ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS                 PORTS                       NAMES
9135fae03302   mysql                       "docker-entrypoint.s…"   19 seconds ago   Up 18 seconds          3306/tcp, 33060/tcp         mysql-client
acbcd4888fe6   mysql/mysql-server:latest   "/entrypoint.sh mysq…"   3 hours ago      Up 3 hours (healthy)   3306/tcp, 33060-33061/tcp   mysql-server

```
### Prepare database schema
Cloned repo `git clone https://github.com/hectorproko/tooling` to path `/home/hector`  

Creating an environmental variable for `tooling_db_schema.sql`'s path  
``` bash
hector@hector-Laptop:~/tooling/html$ export tooling_db_schema=/home/hector/tooling/html/tooling_db_schema.sql
hector@hector-Laptop:~/tooling/html$ echo $tooling_db_schema
/home/hector/tooling/html/tooling_db_schema.sql
```

Using the variable to execute script on **mysql-server**  
``` bash
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema 
```

Checking the new database created
``` bash  
docker exec -it mysql-server mysql -uroot -p$MYSQL_PW
```
``` bash
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| hector             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| toolingdb          |
+--------------------+
6 rows in set (0.01 sec)

mysql>
```
Updating file `tooling/html/.env`   
``` bash
hector@hector-Laptop:~$ bat tooling/html/.env
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: tooling/html/.env
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # input your environment variables
   2   │
   3   │ MYSQL_IP=mysql-server
   4   │ MYSQL_USER=hector
   5   │ MYSQL_PASS=password
   6   │ MYSQL_DBNAME=toolingdb
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
hector@hector-Laptop:~$

```
### Run the Tooling App  
I will now containerize the application **tooling** by telling Docker how to pack the app into a container using a **Dockerfile**
``` bash
hector@hector-Laptop:~/tooling$ bat Dockerfile
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: Dockerfile
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ FROM php:7-apache
   2   │ MAINTAINER Dare dare@zooto.io
   3   │
   4   │ RUN docker-php-ext-install mysqli
   5   │ RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
   6   │ RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
   7   │ COPY apache-config.conf /etc/apache2/sites-available/000-default.conf
   8   │ COPY start-apache /usr/local/bin
   9   │ RUN a2enmod rewrite
  10   │
  11   │ # Copy application source
  12   │ COPY html /var/www
  13   │ RUN chown -R www-data:www-data /var/www
  14   │
  15   │ CMD ["start-apache"]
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
hector@hector-Laptop:~/tooling$
```

I begin by ensuring my current working directory is **tooling** which contains the **Dockerfile** 

I'll run the following command to `build` 
``` bash
docker build -t tooling:0.0.1 .
```
`-t` parameter is to **tag** the image as **tooling:0.0.1** 
The `.` at the end, says to look for Dockerfile in the current working directory


Now using this new **build** I'll run a container in interactive mode just to look inside `-it` 
``` bash
hector@hector-Laptop:~/tooling$ docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1
```
```
[Thu Jun 30 19:39:01.289793 2022] [mpm_prefork:notice] [pid 9] AH00163: Apache/2.4.53 (Debian) PHP/7.4.29 configured -- resuming normal operations
[Thu Jun 30 19:39:01.289822 2022] [core:notice] [pid 9] AH00094: Command line: 'apache2 -D FOREGROUND'
```
`--network` flag to make sure both **Tooling app** and **database** containers are in the same virtual network  
`-p` flag to map the container port **80** to host port **8085**

If everything works, I can open the browser and type `http://localhost:8085`  

![logo](https://raw.githubusercontent.com/hectorproko/MIGRATION-TO-THE-LOUD-WITH-CONTAINERIZATION---DOCKER-DOSECKER-COMPO/main/images/page.png)  

# PRACTICE TASK

## Practice Task №1 – Implement a POC to migrate the PHP-Todo app into a containerized application.

### Part 1
**Tasks**:  
1. Write a Dockerfile for the TODO app [Repo Here](https://github.com/hectorproko/php-todo)
2. Run both database and app on your laptop Docker Engine  
3. Access the application from the browser  

First I make sure I have my `mysql-server` **container** created/running with the `.sql` script executed
``` bash
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW-d mysql/mysql-server:latest

docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql
```

The 3 files we need to focus on are:  
[Dockerfile](https://github.com/hectorproko/php-todo/blob/main/Dockerfile)  
[start-apache.sh](https://github.com/hectorproko/php-todo/blob/main/start-apache.sh)  
[.env](https://github.com/hectorproko/php-todo/blob/main/.env)  
 
 ``` bash
 #This portion of .env
DB_HOST=mysql-server
DB_DATABASE=hector
DB_USERNAME=hector
DB_PASSWORD=password
```
Now I will build my `php-todo` app **image** using `docker build` in the root directory of [TODO app Repo](https://github.com/hectorproko/php-todo) where the desire [Dockerfile](https://github.com/hectorproko/php-todo/blob/main/Dockerfile) file is located  
``` bash
docker build -t phptodo . 
```

Double checking **image** was created  
``` bash
$ docker images | grep phptodo
phptodo                              latest       d11eee293df9   27 minutes ago      548MB
```

Using newly created **image** (phptodo) to start a **container**  
``` bash
docker run --network tooling_app_network -p 8085:8000 -it phptodo
```

![logo](https://raw.githubusercontent.com/hectorproko/MIGRATION-TO-THE-LOUD-WITH-CONTAINERIZATION---DOCKER-DOSECKER-COMPO/main/images/todocontainer.gif)  

**Part of the output:**  
Using [Laravel](https://laravel.com/) to server the php application   
``` bash
Generating autoload files
> php artisan clear-compiled
> php artisan optimize
Generating optimized class loader
Nothing to migrate.
Application key [lG6XbWSiRUu19AeD49zXUu1NZtdd7F7j] set successfully.
Application cache cleared!
Configuration cache cleared!
Route cache cleared!
Laravel development server started on http://0.0.0.0:8000/
[Wed Jul  6 21:25:11 2022] PHP 7.4.30 Development Server (http://0.0.0.0:8000) started
```

![logo](https://raw.githubusercontent.com/hectorproko/MIGRATION-TO-THE-LOUD-WITH-CONTAINERIZATION---DOCKER-DOSECKER-COMPO/main/images/todopage.gif) 

### Part 2
**Tasks**:
1. Created an account in [Docker Hub](https://hub.docker.com/)  
2. Created a new Docker Hub repository **project20**  
3. Pushed the docker images from local machine to the repository  

Image I want to push is `phptodo`  
``` bash
hector@hector-Laptop:~$ docker images | grep phptodo
phptodo                              latest       d11eee293df9   20 hours ago    548MB
hector@hector-Laptop:~$
```
Authenticate docker logging `docker login`  
``` bash
$ docker login --username=hectorproko
Password:
WARNING! Your password will be stored unencrypted in /home/hector/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
I'll tag the **image** to specify the **user**, **repo** and **name** of image (at destination) using `docker tag` `image ID` `hectorproko/project20:phptodo`  

``` bash
hector@hector-Laptop:~$ docker tag d11eee293df9 hectorproko/project20:phptodo
hector@hector-Laptop:~$ docker images | grep phptodo
hectorproko/project20                phptodo      d11eee293df9   21 hours ago    548MB
phptodo                              latest       d11eee293df9   21 hours ago    548MB
```

### Part 3


## Practice Task №2 – Complete Continuous Integration With A Test Stage