# MIGRATION-TO-THE-LOUD-WITH-CONTAINERIZATION---DOCKER-DOSECKER-COMPO
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
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW -d mysql/mysql-server:latest
```
Flags used  
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
hector@hector-Laptop:~$ export MYSQL_PW="Passw0rd!" #environment variable to store the root password
hector@hector-Laptop:~$ docker exec -i mysql-server mysql -uroot -p $MYSQL_PW < ./create_user.sql
mysql: [Warning] Using a password on the command line interface can be insecure.
hector@hector-Laptop:~$
```

Connecting to **mysql-server** using another container in interactive mode  
``` bash
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u hector -p
```

# PRACTICE TASK