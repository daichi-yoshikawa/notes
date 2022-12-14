## Frequently Used Commands of Docker
** Show containers and image **
```
$ docker ps -a <<< -a: Show all containers
$ docker containers ps -a
$ docker images ps -a
$ docker ps -aq <<< Show list of container id
```

** Show volume and network **
```
$ docker volume ls [-f dangling=true] <<< Show volumes in use
$ docker network ls [--no-trunc] <<< --no-trunc: Not truncate network id
```

** Remove containers **
```
$ docker rm [-f] <container id1> <container id2> ... <<< -f: Force to remove
$ docker container prune [-f] <<< -f: Skip confirmation

```

** Remove images **
May be better to remove containers first before removing images.
```
$ docker rmi [-f] <image id>
$ docker image prune <<< Remove all intermadiate images(images with no tag and referred by no containers)
$ docker image prune -a <<< -a: Remove all unused images
```

** Remove volume **
```
$ docker volume prune
```

** Remove network **
```
$ docker network prune
```

** Remove all **
```
$ docker system prune [--volumes] [-f] <<< --volumes: Remove volume as well, -f: Skip confirmation
```

** Pull docker image from docker hub **
```
$ docker login
$ docker pull <image name>:<tag>
$ docker pull -q ubuntu:xenial <<< -q: Surpress verbose
```

** Build docker image from Dockerfile **
```
$ docker build [--no-cache] [--target <target name specified AS in Dockerfile>] -t <image name>:<tag> . <<< --no-cache: Do not use cache
$ docker image build -t <account name>/<image name>:<tag> .
$ docker image build --no-cache -t daichi_in_dockerhub/my_postgres:latest .
```

** Run container **
* -it: -i -t
* -i(=--interactive): Keep stdin open
* -t(=--tty): Allocate a virtual terminal session within the container
* --rm: Automatically clean up the container and the remove the file system when the container exists
* -v \<dir abs path in host>:\<dir abs path in container>: Mount volume
* -p \<port in host>:<port in container>: Relay port
* -d: Start the container detached (in the background)
* --network host: Use host network
* --name: Explicitly add name to container
```
$ docker run [-d] [--rm] [-it] [-v XXX:YYY] [-p 12345:56789 or --network host] [--name foo_container] <image name>[:tag]
$ docker run -d --rm -it -v .:/root --network host --name my_app_container selenium:latest
```
** Note **: What is a detached mode(running in the background)? By default, you have to wait until the command finishes when you run some command in a terminal. It's called "running the command in foreground" or "foreground process". You can interact with the foreground process but you can't run the other command while you're waiting for the end. Running in the background is the opposite to it obviously. You can't interact with the background process but you can run the other command right after you launched the foreground process.

** Stop container **
```
$ docker stop <container name or container id>
```

** Get in terminal on the running container **
```
$ docker exec -it <container name> /bin/bash
$ docker exec -it my_app_container /bin/bash
```

## Dockerfile

Create account of dockerhub and login in terminal.
```
$ docker login
```

Dockerfile template/example
```
FROM selenium/standalone-chrome:104.0 <- Search in dockerhub

USER root <- Make sure that you execute commands below as root user

RUN apt-get update && \ <-- For docker use apt-get over apt
    apt-get install -y --no-install-recommends \ <- Recommended options
      software-properties-common \ <- In most case you'll need this.
      ...

# For python setup
RUN add-apt-repository ppa:deadsnakes/ppa && \
    apt-get update &&
    apt-get install -y --no-install-recommends \
      python3.8 \
      python3-pip && \
    ln -s /usr/bin/python3 /usr/bin/python && \ <- By default python not found
    pip install --upgrade pip

COPY file_in_host.txt file_in_container.txt

WORKSPACE /root <- You can change directory

CMD tail -f /dev/null <- To persistent container
```

## docker-compose

### docker-compose sample
```
version: <version>
  services:
    my_webapp:
      build:
        context: .
        dockerfile: Dockerfile
      container_name: my_webapp_container
      image: ubuntu:20.04
      ports:
        - "<host port>:<container port>"
      volumes:
        - $PWD/<host dir abs path>:<container dir abs path>
      [network_mode: "host"]
      command: tail -f /dev/null
    postgres:
      container_name: my_postgres
      image: postgres:12.4
      restart: always
      environment:
        - POSTGRES_DB=$POSTGRES_DB_IN_DOT_ENV
        - POSTGRES_USER=$POSTGRES_USER_IN_DOT_ENV
        - POSTGRES_PASSWORD=$POSTGRES_PASSWORD_IN_DOT_ENV
      ports:
        - <host port>:5432
      volumes:
        - <host dir abs path>:<container dir abs path>
```

### docker-compose commands
```
$ docker-compose up -d --rm --name=my_webapp
$ docker-compose run -d --service-ports --rm my_webapp <- --service-ports enable ports(See below)
$ docker-compose start -d --rm my_webapp
```
**Note:** [By default no ports will be created in case they collide with alread opened ports](https://github.com/docker/compose/issues/1256). In other words, docker-compose is not a complete equivalent of docker run and you need to use --service-ports option to open ports.

** Difference between up, run and start **
* docker-compose up: Start or restart all the services defined in docker-compose.yml.
* docker-compose run: Requires service name and it only starts it. Similar to "docker run -it".
* docker-compose start: Restart containers which were previously created, but were stopped. Never creates new containers.

```
$ docker-compose down <- Stop and remove containers, networks, volumes and images created by docker-compose up.
$ docker-compose kill <- Stop containers by sending a SIGKILL signal.
$ docker-compose stop <- Do not remove containers, so they can be started again with docker-compose start.
```

* pause/unpause -> hibernate
* kill -> soft shut-down
* stop/start/restart -> hard shut-down
* down/up/create -> factory reset
