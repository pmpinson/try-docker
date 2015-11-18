# what is docker ?
  platform for developers and sysadmins to develop, ship, and run applications

  main interest containerisation in closed environnement

  comparaison to virtual machine but no need to create a complete server and os it use the current unix kernel to run and shared resource

# install
  https://docs.docker.com/engine/installation/ubuntulinux/
  https://docs.docker.com/engine/installation/mac/


# docker machine for mac and windows
  what is it

# first docker
  `docker run hello-world`

  `docker run -ti ubuntu echo "hello"``

  launch a container and do some shell command in it : `docker run -ti ubuntu /bin/bash`
    * install nginx : `apt-get install nginx`
    * start nginx : `service nginx start`
    * open new terminal and list running container `docker ps`
    * inspect container information `docker inspect b1e9a26244ca` or `docker inspect b1e9a26244ca |grep IP` or `docker inspect  --format '{{ .NetworkSettings.IPAddress }}' b1e9a26244ca`
    * go to a we navigator on ip 172.17.0.2
    * modify the index of nginx `vi /usr/share/nginx/html/index.html`
    * commit the container `docker ps` then `docker commit b1e9a26244ca ppinson/simple-nginx`
    * list images
      -> image are auto pull first time then used local first
      -> docker pull to pull an image (https://confluence.tritondigital.com/display/DEV/Logging+into+the+Artifactory+docker+repository)
    * restart a new container with an image `docker run -ti ppinson/simple-nginx /bin/bash`
      launch nginx `service nginx start`
      inspect container to get ip and go to navigator to see the page
    * restart on backgroud a container with our new images `docker run -d --name othercontainer ppinson/simple-nginx`
      -> fail because no running command
    * restart with a running command `docker run -d --name othercontainer ppinson/simple-nginx nginx -g 'daemon off;'`
    * stop and remove the container
      `docker ps`
      `docker stop 836b0bdba575`
      `docker start 836b0bdba575`
      `docker stop 836b0bdba575`
      `docker rm 836b0bdba575`

# registry
  how can i ship my container, basicly i need to share the different layer i create (each command)
  i can use a registry and push my actual container with a tag (= version)


# docker image
  we have done thing manually but we can do it easly and
  `docker build --tag ppinson/sample-1-nginx sample-1`
  `docker run -d --name othercontainer ppinson/sample-1-nginx`
  go to a web browser with ip


# container or run configuration
## port
  `docker run -d -p 8941:80 ppinson/sample-1-nginx`
  go to browser on container ip on port 80
  go to browser on localhost on port 80
  `docker build --tag ppinson/sample-2-nginx sample-2`
  `docker run -d -P ppinson/sample-2-nginx`
  `docker ps` or `docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' db3ccc0e4c5ae067540526d7f5005128a95cec9fbca8097fa9e49a2eb3c6c01d`

## volume
  add file to container with Dockerfile
  `docker build --tag ppinson/sample-3-nginx sample-3`
  `docker run -d -P ppinson/sample-3-nginx`
  `docker stop be894e3350f3`
  `docker run -d -P -v $(pwd)/sample-3/pages:/usr/share/nginx/html/pages ppinson/sample-3-nginx`
  `docker exec -ti 2c1cfb3eb62a350be1f799425cf3f23a5393fd7cf49d49127354d6a26fb43d66 /bin/bash`

## links
  * start a mysql server `docker run -d --name mysql-server -e MYSQL_ROOT_PASSWORD=pass mysql`
  * list volume `docker volume ls`
  * inspect volume `docker volume inspect `
  * launch a client `docker run -ti --rm --link mysql-server:mysql mysql /bin/bash`
    - `mysql -h $MYSQL_PORT_3306_TCP_ADDR -P$MYSQL_PORT_3306_TCP_PORT -p$MYSQL_ENV_MYSQL_ROOT_PASSWORD`
    - create a database `create database docker_test;`
    - change db to it `use docker_test;`
    - create a table `create table customer (id serial, name varchar(100));`
    - insert data `insert into customer (name) values ('pmp');`
    - `select * from customer;`
  * connect another client `docker run -ti --rm --link mysql-server:mysql mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" docker_test -e "SELECT * FROM customer"'`


### volume from and data container
* create a data volume container `docker create -v /var/lib/mysql --name mysqldata busybox /bin/true`
* start a mysql server `docker run -d --name mysql-server --volumes-from mysqldata -e MYSQL_ROOT_PASSWORD=pass mysql`
* launch a client `docker run -ti --rm --link mysql-server:mysql mysql /bin/bash`
  - `mysql -h $MYSQL_PORT_3306_TCP_ADDR -P$MYSQL_PORT_3306_TCP_PORT -p$MYSQL_ENV_MYSQL_ROOT_PASSWORD`
  - create a database `create database docker_test;`
  - change db to it `use docker_test;`
  - create a table `create table customer (id serial, name varchar(100));`
  - insert data `insert into customer (name) values ('pmp');`
  - `select * from customer;`
* connect another client `docker run -ti --rm --link mysql-server:mysql mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" docker_test -e "SELECT * FROM customer"'`
* kill docker mysql server
* rerun a mysql server with another name `docker run -d --name mysql-server2 --volumes-from mysqldata -e MYSQL_ROOT_PASSWORD=pass mysql`
* connect another client `docker run -ti --rm --link mysql-server:mysql mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" docker_test -e "SELECT * FROM customer"'`

### other options
--net=host
--dns
--dns-search
--rm

### remarks
 - next plugins in docker : volume / Network
 - comming user, only run as root by default

### usefull commands (in dev only)
`docker kill $(docker ps -q)`
`docker rm -v $(docker ps -q -a)`
`docker rmi $(docker images -a)`


# docker compose


#other docker swarn
manage a cluser of docker server
