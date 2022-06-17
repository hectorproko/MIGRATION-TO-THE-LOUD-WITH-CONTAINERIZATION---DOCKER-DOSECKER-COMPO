# MIGRATION-TO-THE-LOUD-WITH-CONTAINERIZATION---DOCKER-DOSECKER-COMPO
PROJECT 20  

Get docker engine in Ubuntu  
``` bash
hector@hector-Laptop:~$ docker --version
Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1
hector@hector-Laptop:~$
```




I will use a pre-built **MySQL** database container, configure it, and make sure it is ready to receive requests from our PHP application.  


Currently there I have no images
``` bash
hector@hector-Laptop:~$ sudo docker images ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
Getting the image I need `mysql/mysql-server:latest`  
``` bash
hector@hector-Laptop:~$ sudo docker pull mysql/mysql-server:latest
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

``` bash
hector@hector-Laptop:~$ sudo docker images
REPOSITORY           TAG       IMAGE ID       CREATED        SIZE
mysql/mysql-server   latest    5a9594052aec   3 weeks ago    438MB
```


deploying a new MySQL container
docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d <docker-image>
docker run --name MySQL -e MYSQL_ROOT_PASSWORD=Passw0rd! -d mysql/mysql-server:latest


 MySQL container is running:

hector@hector-Laptop:~$ sudo docker run --name MySQL -e MYSQL_ROOT_PASSWORD=Passw0rd! -d mysql/mysql-server:latest
ce2dcf1ec7aab8d2dd5c70919ead800fcdcbdc623a64b292a8876f26fcc0f746
hector@hector-Laptop:~$ sudo docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS                             PORTS                       NAMES
ce2dcf1ec7aa   mysql/mysql-server:latest   "/entrypoint.sh mysq…"   31 seconds ago   Up 30 seconds (health: starting)   3306/tcp, 33060-33061/tcp   MySQL














# CONNECTING TO THE MYSQL DOCKER CONTAINER

# PRACTICE TASK