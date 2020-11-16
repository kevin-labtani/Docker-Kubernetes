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

There a different kinds of data:

- Application data (souce code and environment) is stored in images and _read-only_

  - Written & provided by us
  - Added to image and container in build phase
  - “Fixed”: Can’t be changed once image is built

- Temporary app data is stored in containers and read-write but temporary

  - Fetched/Produced in running container
  - Stored in memory or temporary files
  - Dynamic and changing, but cleared regularly

- Permanent app data is stored in containers and volumes and read-write an dpermanent
  - Fetched/Produced in running container
  - Stored in files or a database
  - Must not be lost if container stops/restarts

#### Example

We'll be working with `5-data-volumes` for this part, first we'll dockerize it by writing a `Dockerfile` and building an image `docker build -t feedback-node-app .` and start the container `docker run -p 3000:80 -d --name feedback-app --rm feedback-node-app`

The app will work, if we write a feedback message title "awesome", we'll then be able to see it at http://localhost:3000/feedback/awesome.txt; the new file `awesome.txt` isn't saved locally though

If we keep the same container - by creating it without the `--rm` flag - create a file and then stop and restart the container, the new files will still be there though, that extra data is saved in the container layer of the container (the new files are in the file system of the container)

So we have two problems:

1. data written in a container doesn't persist (when the container is deleted)
2. the container can't interact with the host filesystem

The solution to these problems are, resp.

1. Volumes
2. Bind Mounts

#### Volumes

Volumes are folders on your host machine hard drive which are mounted into containers

Volumes persist if a container shuts down. If a container (re-)starts and mounts a volume, any data inside of that volume is available in the container

A container can write data into a volume and read data from it

To add a volume to our container, add an instruction to the `Dockerfile`

```
VOLUME [ "/app/feedback" ]
```

We use the path inside of our container which should be map to a folder outside of the container and where data should persists, docker will control that folder outside, rebuild the image afterwards `docker build -t feedback-node-app:volumes .` and run it, when we try to leave feedback, the container crashes! It's actually the `fs.rename` method in our app that causes an issue, we'll replace it with `fs.copyFile` and delete the tempFile after instead.

Now if we stop, remove and then create a new container... the data is still not persisting!

There are actually two types of volumes:

- Anonymous Volumes
- Named Volumes

In both cases, A defined path in the container is mapped to the created volume/mount; and Docker sets up a folder/path on out host machine, the exact location is unknown to us. The volumes are managed via `docker volume` commands.

`docker volume ls` will list the volumes currenctly managed by docker, currently docker uses an anonymous volume to deal with our app data, but if we stop the container, that anonymous volume is deleted; anonymous volumes only exist as long as the container exist

Named volumes persist after container shutdown, that makes htem great for data which should be persistent but which we don’t need to edit directly

We can't create named volumes inside of the `Dockerfile`, so we'll remove the `VOLUME [ "/app/feedback" ]` instruction and rebuild the image. Instead we have to create a named volume when we run a container with the flag `-v NAME:/PATH/INSIDE/CONTAINER`, `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback --rm feedback-node-app:volumes`. Now if we stop, delete and rerun a new container with the same volume, the data will finally persists!

nb: anonymous volumes are removed automatically when we start/run a container with the `--rm` flag. If we start a container without that option, the anonymous volume will not be removed, even if we remove the container later. We can clear anonymous volumes via `docker volume rm VOL_NAME` or `docker volume prune`

nb: we can also create anonymous volumes on the comman line, `docker run –v /app/data …`

#### Bind Mounts

With bind mounts, we define a folder/path on our host machine. These are great for persistent, editable (by us) data (e.g. source code), so we use them to provide "live data" to the container, without needing to rebuild

We can create a bind mount with the flag `–v /path/to/code:/app/code`, path/to/code need to be the absolute path, and we'll bind the entire app folder, `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app" --rm feedback-node-app:volumes` (put the abs. path between quotes " if the path has special chars or spaces), we can also use `-v $(pwd):/app` instead

As of now it crashes and shut down immediately! , let's rerun it without the `-rm` flag so we can check the logs of the stopped container, and we see `Error: Cannot find module 'express'`

The problem is that we bind the entire app folder and it then overwrite the app folder inside of the container with our local folder, and that local folder don't have the node_modules folder so the `RUN` command from the `Dockerfile` is rendered worthless, docker will not overwrite our localhost folder, it's the opposite that happens. To tell docker that there are certain things that should not be overwritten in the container file system, we add an anonymous volume to the run command: `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app" -v /app/node_modules --rm feedback-node-app:volumes`. Docker evaluates all volumes we set to a container, and if there are clashes, the longer internal path wins

Now if we change something in our internal files, eg. edit `fedback.html` and then reload the page, the change takes effect without having to create and run a new image

If we change something in the `server.js`, eg. add a `console.log`, we need to add nodemon to see the logs with `docker logs feedback-app` as the code in `server.js` is executed by the node runtime (the alternative is to stop de container and restart/recreate one)

#### Volumes - Comparison

|                       Anonymous Volumes                        |                         Named Volumes                          |                           Bind Mounts                            |
| :------------------------------------------------------------: | :------------------------------------------------------------: | :--------------------------------------------------------------: |
|          Created specifically for a single container           |    Created in general – not tied to any specific container     | Location on host file system, not tied to any specific container |
|   Survives container shutdown / restart unless --rm is used    | Survives container shutdown / restart – removal via Docker CLI |    Survives container shutdown / restart – removal on host fs    |
|              Can not be shared across containers               |                Can be shared across containers                 |                 Can be shared across containers                  |
| Since it’s anonymous, it can’t be re-used (even on same image) |      Can be re-used for same container (across restarts)       |       Can be re-used for same container (across restarts)        |

#### Read-Only Volumes

The container isn't able to write to the app folder, even with a bind mount, we can enforce this by turning the bind mount into a read only volume by adding an extra `:ro` to the bind mount command, but since we write to some of those folders from inside the container, we'll need to specify another sub-volume to allow that, we have a named volume for /app/feedback already, we need to add one for the temp folder, and we'll use an anonymous volume `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:volumes`

#### Managing Volumes

`docker volume ls` to list volumes
`docker volume create` to create a volume
`docker volume inspect VOL_NAME` to display detailed information on a volume
`docker volume remove VOL_NAME` to remove a volume
`docker volume prune` to remove unused volumes

#### COPY

We still `COPY . .` (ie. copy everything) in our `Dockerfile` even though we bind the entire app, it's not actually necessary right now, but the `docker run` command we currently run with the bind mount is used for development purpose so we can change our code without having to rebuild the image every time, once we're done with developing and want to host our app in production, we won't run it with a bind mount, so we do need to keep the `COPY` command

#### .dockerignore

we can add a `.dockerignore` file to specify which folders and file shouldn't be copied with a `COPY` command

#### ARGuments & ENVironment Variables

Docker supports build-time ARGuments and runtime ENVironment variables

- Runtime ENVironment
  - Available inside of Dockerfile & in application code
  - Set via ENV in Dockerfile or via `--env KEY=VALUE` on `docker run`

Let's change our port in the example app to use an env variable, `app.listen(process.env.PORT);` and update the `Dockerfile` to set a default environment variable

```
ENV PORT 80

EXPOSE $PORT
```

Now we rebuild, `docker build -t feedback-node-app:env .` and runthe container again, `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:env`

We don't have to stick to the default ENV variable, we can use the `--env KEY=VALUE` flag to run the app on another port, `docker run -p 3000:8000 --env PORT=8000 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:env`

`-e` also works as a shortcut for `--env`, if we have more than one env varialbe, we simply add multiple flags

We can also specify a file that contains the env variables, eg if we have a `.env` file, run the container with `--env-file ./.env`, `docker run -p 3000:8000 --env-file ./.env -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:env`

Using a `.env` file is better for security than including secure data directly into the `Dockerfile`. nb: otherwise, the values are "baked into the image" and everyone can read these values via `docker history IMAGE`

- Build-time ARGuments
  - Available inside of Dockerfile, NOT accessible in CMD or any application code
  - Set on image build (docker build) via `--build-arg  KEY=VALUE`

Let's lock in the default port value with a build time argument, in our `Dockerfile`, we add `ARG DEFAULT_PORT=80`, this argument can't be used on the `CMD` instruction, but it can be used on the other instructions, eg, `ENV PORT $DEFAULT_PORT`, now we can set a dynamic arg that is used as a default value for the dynamic env variable for the port! Let's rebuild, `docker build -t feedback-node-app:web .`, and if we wanted to change the default value of the arg, we build again with the `--build-arg KEY=VALUE` flag, `docker build -t feedback-node-app:dev --build-arg DEFAULT_PORT=8000 .`

In the `Dockerfile`, remember that all subsequent layer are rerun when we modify one layer, so it's best to put the `ARG` and `ENV` after the `RUN npm install` so we don't reinstall all packages everytime we pass in a new default ARG

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
