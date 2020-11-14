## Présentation du Projet

Projet créé le 12 Novembre 2020, dans le but de'apprendre à utiliser Docker.

Le projet est basé sur le cours de Docker de [Academind](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/)

- [Getting Started & Overview](#Getting-Started-&-Overview)
- [Foundation](#Foundation)

## Getting Started & Overview

### What is Docker?

Docker is a container technology, a tool for creating and managing containers

A container is a standardized unit of software, a package of code and dependencies to run that code. The same container always yields the exact same application and execution behavior! No matter where or by whom it might be executed.

### Why Containers?

- Different Development & Production Environments

We want to have the exact same environment for development and production. This ensures that it works exactly as tested

- Different Development Environments Within a Team / Company

It should be easy to share a common development environment/ setup with employees and colleagues

- Clashing Tools / Versions Between Different Projects

We don’t want to uninstall and re-install local dependencies and runtimes all the time

### Docker vs Virtual Machines

Pros:

- Separated environments
- Environment-specific configurations are possible
- Environment configurations can be shared and reproduced reliably

Cons:

- Redundant duplication, waste of space
- Performance can be slow, boot times can be long
- Reproducing on another computer/ server is possible but may still be tricky

|                     Docker Containers                      |                        Virtual Machines                         |
| :--------------------------------------------------------: | :-------------------------------------------------------------: |
|   Low impact on OS, very fast, minimal disk space usage    |      Bigger impact on OS, slower, higher disk space usage       |
|       Sharing, re-building and distribution is easy        |    Sharing, re-building and distribution can be challenging     |
| Encapsulate apps/ environments instead of “whole machines” | Encapsulate “whole machines” instead of just apps/ environments |

### Installation

I'll be using [Docker Desktop on Windows 10 Home with WSL2](https://docs.docker.com/docker-for-windows/install-windows-home/)

Run `docker` in the terminal to check that it works

Also install the Docker extension for VSCode

When using WSL integration, docker create two distros:

- docker-desktop accessible on `\\wsl\$\docker-desktop`
- docker-desktop-data accessible on `\\wsl\$\docker-desktop-data`

### Docker Tools & Building Blocks

- Docker Engine
- Docker Desktop (incl. Daemon & CLI)
- Docker Hub
- Docker Compose
- Kubernetes

### First Project

The code is in the 1-first-demo folder

The app is a simple Node(Express) app

We'll add our first `Dockerfile` to this example app

In the terminal, run `docker build .` in the app directory to build the image based on this docker file, it'll finish by giving is the id of the image, `=> => writing image sha256:508425c72dbf49410d105bcafdd3062bceac172b64494b1f4d`

`docker run -p 3000:3000 508425c72` will run the image in a new container
`-p 3000:3000` is used to publish the port 3000 on port 3000 so we can use `localhost:3000` to access the app, as by default there's no connection between container and host (nb: the termial will be locked by the running web server)

`docker ps` will list all running containers  
`docker stop NAME_OF_CONTAINER` will stop a container (container are assigned a random name)

## Foundation

### Images & Containers

#### Images

Images are the templates/blueprints for containers. It contains the code and required tools/runtime

We can use an image to generate multiple containers (the running instance of that image)

We run containers which are based on images, that is the core fundamental concept of Docker

Images are Read-Only! If we change the code of our app, we'll need to rebuild the image

#### Finding Images

We can use an existing, pre-built image (on Docker Hub)

If we run `docker run node`, docker will automatically download the latest node image from Docker Hub

`docker ps -a` will show all the processes (containers) docker created on our machine

`docker run -it node` will expose an interactive session from inside our container to our hosting machine, and will therefore give us a node terminal

#### Creating Images

We can build our own images by defining a `Dockerfile`

See the starter project in the `2-node-dummy` folder, we'll add a `Dockerfile` to create our own image

#### Dockerfile

Dockerfiles contain instructions which are executed when an image is built,
every instruction then creates a layer in the image. Layers are used to efficiently rebuild and
share images (this keeps Docker images and containers small)  
When we re-build an image, only the layers that changed will be re-built

`FROM` allows us to build our image on top of an existing base image

`WORKDIR` will set the default working directory of the docker container

`COPY . .` will copy files & folders from the `source` to the `dest` path in the image's filesystem. The first `.` tells docker to copy everything (excluding the `Dockerfile`) from the source to the image, the second `.` tells docker that the destination directory in the image is the same folder that contains the `Dockerfile`, excluding it. If we set a working directory, the second `.` will copy to the working directory, but we can still set an absolute directory if we want to be explicit

`RUN` will tell docker to run a specific command, by default the commands are executed in the working directory (root by default) of the docker container

`EXPOSE` define the network port that this container will listen to at runtime, it's optional but recommended for documentation purpose

`CMD` is a special instruction, it's not executed when the image is built but when a container
is created and started based on that image. `CMD` takes an array of strings

Our final `Dockerfile`:

```
FROM node

WORKDIR /app

COPY . /app

RUN npm install

EXPOSE 80

CMD ["node", "server.js"]
```

#### Running a Container Based on Our Own Image

`docker build .` will build an image based on the `Dockerfile`, we'll get an `id` in the last build line

`docker run -p 3000:80 ID_OF_IMAGE` with the id of the image we just built will start a container and we'll be able to reach the app on port 3000, 80 being the exposed port from the container

if we modify our app, we'll need to rebuild our image, we can optimise our `Dockerfile` so that `npm install` doesn't need to be re-run on each build everytime we change our source code (if a layer of the image is modified, every subsequent layer will be re-run at build time instead of using the cached results):

```
FROM node

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

CMD ["node", "server.js"]
```

#### Containers

Containers are running instances of images. When we create a container, a thin read-write layer is added on top of the Image  
Multiple containers can be started based on one image  
All containers run in isolation, ie. they don't share any application state or written data  
We need to create and start a Container to start the application which is inside of a container, So it's containers which are executed - both in development and production

#### Managing Images & Containers

`docker run IMAGE_NAME` creates and run a new container

`docker start NAME/ID` restarts a container

`docker rm CONTAINER_NAME` will remove containers, you can't remove a running container, you can pass multiple names spaced with white space

`docker images` lists images

`docker rmi IMAGE_ID` will remove an image, you can only remove an image no longer used by any containers (started or stopped)

`docker image prune` will remove all unused images

`docker run --rm IMAGE_NAME/ID` will automatically remove a container when it exits eg. `docker run -p 3000:80 --rm 63dfe`

`docker images inspect IMAGE_ID` can be used to inspect an image

`docker cp SOURCE CONTAINER_NAME:DEST` (SOURCE is a file or folder) can be used to copy files into a container, eg. `docker cp dummy/. boring_vaughan:/test`. Reverse the source and destination to copy files from a container, eg. `docker cp boring_vaughan:/test dummy/.`

#### Attached and Detached Containers

When we restart a container, the process finishes immediately - the terminal isn't locked - and the container is running in the background, in detached mode. When we use then run command instead, the terminal is locked and the container run in the foreground, in attached mode.

Attached means we're listening to the output of the container

In our sample app from `2-node-dummy` we're logging info to the console, so running in attached mode is useful

Using `docker run -p 3000:80 -d IMAGE_ID` with `-d` flag will run a container in detached mode

To restart a stopped container in attached mode, use `docker start -a NAME/ID`

We can attach ourself to a detached container again by running `docker container attach CONTAINER_NAME`

Another way to getting access to stderr/stdout from the container is `docker logs CONTAINER_NAME`; `docker logs -f CONTAINER_NAME` enables follow mode to keep on listening to the logs form the container

#### Interactive Mode

`docker run -it IMAGE_ID` will run a container in interactive mode (`-i` keeps stdin open, `-t` allocate a pseudo-TTY, ie. creates a pseudo terminal)

We'll use `3-python-app` as the demo project for this part

first we add a `Dockerfile`

```
FROM python

WORKDIR /app

COPY . /app

CMD ["python", "rng.py"]
```

Then we can run the app with `docker run -it IMAGE_ID`

Use `docker start -a -i CONTAINER_NAME` to restart a container in interactive mode

#### Naming and Tagging Images & Containers

Images can be tagged when created with the flag `-t NAME:TAG`, eg. `docker build -t goals:latest .`  
`name` defines a group of images eg. "node", `tag` defines a specialized image within a group of images eg. "14"

To rename an image use `docker tag OLD_NAME NEW_NAME`, it'll create a clone of the old image under the new name

Containers can be named when created with the flag `--name PROVIDED_NAME`, eg. `docker run -p 3000:80 -d --rm --name goalsapp goals:latest`

#### Sharing Images

We could share a `Dockerfile` and have people run `docker build .` but we'd need to share the entire app folder so others can build it!!

Another possibility is to share a built image

Sharing can be done through Docker Hub or a Private Registery

To share an image, use `docker push IMAGE_NAME`, the image name/tag must include the repository name/url  
we'll also need to create a repository on [docker hub](https://hub.docker.com/repository/create)  
and give our local image the name of the repository:

```
docker tag node_assignment kevinlabtani/node-hello-world
docker push kevinlabtani/node-hello-world
```

nb: we'll need to be logged in with `docker login` to be able to push

To use an image, use `docker pull IMAGE_NAME`, this is done automatically if you just `docker run IMAGE_NAME` and the image wasn't pulled before

```
docker pull kevinlabtani/node-hello-world
```

IMAGE_NAME Needs to be HOST:NAME to talk to private registry  

docker will not check if the images we have locally are the latest version available, we need to manually run `docker pull` to update an image  

### Data & Volumes (in Containers)

### Containers & Networking

## Real Life Scenarios

Multi-Container Projects  
Using Docker-Compose  
“Utility Containers”  
Deploying Docker Containers

## Kubernetes

Kubernetes Introduction & Basics  
Kubernetes: Data & Volumes  
Kubernetes: Networking  
Deploying a Kubernetes Cluster
