## Présentation du Projet

Projet créé le 12 Novembre 2020, dans le but de'apprendre à utiliser Docker.

Le projet est basé sur le cours de Docker de [Academind](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/)

- [Getting Started & Overview](#Getting-Started--Overview)
- [Foundation](#Foundation)
  - [Images & Containers](#images--containers)
  - [Data & Volumes (in Containers)](#data--volumes-in-containers)
  - [Containers & Networking](#Containers--Networking)
- [Real Life Scenarios](#Real-Life-Scenarios)

  - [Multi-Container Projects](#Multi-Container-Projects)
  - [Using Docker-Compose](#Using-Docker-Compose)
  - [“Utility Containers”](#Utility-Containers)
  - [More Complex Setups](#More-Complex-Setups)
  - [Deploying Docker Containers](#Deploying-Docker-Containers)

- [Kubernetes Introduction & Basics](#Kubernetes-Introduction--Basics)
  - [Kubernetes Core Concepts](#Kubernetes-Core-Concepts)
  - [Kubernetes: Data & Volumes](#Kubernetes-Data--Volumes)
  - [Kubernetes: Networking](#Kubernetes-Networking)
  - [Deploying a Kubernetes Cluster](#Deploying-a-Kubernetes-Cluster)

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

```dockerfile
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

```dockerfile
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

`docker container prune` will remove all unused containers

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

We'll use `3-python-app` as the demo project for this module

first we add a `Dockerfile`

```dockerfile
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

- Permanent app data is stored in containers and volumes and read-write and permanent
  - Fetched/Produced in running container
  - Stored in files or a database
  - Must not be lost if container stops/restarts

#### Example

We'll be working with `5-data-volumes` for this module, first we'll dockerize it by writing a `Dockerfile` and building an image `docker build -t feedback-node-app .` and start the container `docker run -p 3000:80 -d --name feedback-app --rm feedback-node-app`

The app will work, if we write a feedback message title "awesome", we'll then be able to see it at http://localhost:3000/feedback/awesome.txt; the new file `awesome.txt` isn't saved locally though and is lost if we get rid of the container

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

```dockerfile
VOLUME [ "/app/feedback" ]
```

We use the path inside of our container which should be mapped to a folder outside of the container and where data should persists, docker will control that folder outside, rebuild the image afterwards `docker build -t feedback-node-app:volumes .` and run it, when we try to leave feedback, the container crashes! It's actually the `fs.rename` method in our app that causes an issue, we'll replace it with `fs.copyFile` and delete the tempFile after instead.

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

Now if we change something in our internal files, eg. edit `feedback.html` and then reload the page, the change takes effect without having to create and run a new image

If we change something in the `server.js`, eg. add a `console.log`, we need to add nodemon to see the logs with `docker logs feedback-app` as the code in `server.js` is executed by the node runtime (the alternative is to stop de container and restart/recreate one)

#### Volumes - Comparison

|                       Anonymous Volumes                        |                         Named Volumes                          |                           Bind Mounts                            |
| :------------------------------------------------------------: | :------------------------------------------------------------: | :--------------------------------------------------------------: |
|          Created specifically for a single container           |    Created in general – not tied to any specific container     | Location on host file system, not tied to any specific container |
|   Survives container shutdown / restart unless --rm is used    | Survives container shutdown / restart – removal via Docker CLI |    Survives container shutdown / restart – removal on host fs    |
|              Can not be shared across containers               |                Can be shared across containers                 |                 Can be shared across containers                  |
| Since it’s anonymous, it can’t be re-used (even on same image) |      Can be re-used for same container (across restarts)       |       Can be re-used for same container (across restarts)        |

#### Read-Only Volumes

The container isn't able to write to the app folder, we can enforce this by turning the bind mount into a read only volume by adding an extra `:ro` to the bind mount command, but since we write to some of those folders from inside the container, we'll need to specify another sub-volume to allow that, we have a named volume for /app/feedback already, we need to add one for the temp folder, and we'll use an anonymous volume `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:volumes`

nb: if on the contrary we wanted to write the files to our local feedback folder, we can run the container without the named volume but with the bind mount for the entire app, `docker run -p 3000:80 -d --name feedback-app -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app" -v /app/node_modules --rm feedback-node-app:volumes`

#### Managing Volumes

`docker volume ls` to list volumes
`docker volume create` to create a volume
`docker volume inspect VOL_NAME` to display detailed information on a volume
`docker volume remove VOL_NAME` to remove a volume
`docker volume prune` to remove unused volumes

#### COPY vs Bind Mount

We still `COPY . .` (ie. copy everything) in our `Dockerfile` even though we bind the entire app, it's not actually necessary right now, but the `docker run` command we currently run with the bind mount is used for development purpose so we can change our code without having to rebuild the image every time, once we're done with developing and want to host our app in production, we won't run it with a bind mount, so we do need to keep the `COPY` command

#### .dockerignore

we can add a `.dockerignore` file to specify which folders and file shouldn't be copied with a `COPY` command

#### ARGuments & ENVironment Variables

Docker supports build-time ARGuments and runtime ENVironment variables

- Runtime ENVironment
  - Available inside of Dockerfile & in application code
  - Set via ENV in Dockerfile or via `--env KEY=VALUE` on `docker run`

Let's change our port in the example app to use an env variable, `app.listen(process.env.PORT);` and update the `Dockerfile` to set a default environment variable

```dockerfile
ENV PORT 80

EXPOSE $PORT
```

Now we rebuild, `docker build -t feedback-node-app:env .` and runthe container again, `docker run -p 3000:80 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:env`

We don't have to stick to the default ENV variable, we can use the `--env KEY=VALUE` flag to run the app on another port, `docker run -p 3000:8000 --env PORT=8000 -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:env`

`-e` also works as a shortcut for `--env`, if we have more than one env variable, we simply add multiple flags

We can also specify a file that contains the env variables, eg if we have a `.env` file, run the container with `--env-file ./.env`, `docker run -p 3000:8000 --env-file ./.env -d --name feedback-app -v feedback:/app/feedback -v "/home/kevin/code/docker-bootcamp/5-data-volumes:/app:ro" -v /app/node_modules -v/app/temp --rm feedback-node-app:env`

Using a `.env` file is better for security than including secure data directly into the `Dockerfile`. Otherwise, the values are "baked into the image" and everyone can read these values via `docker history IMAGE`  
We still want to add env variables keys with default values to the `Dockerfile`, though

- Build-time ARGuments
  - Available inside of Dockerfile, NOT accessible in CMD or any application code
  - Set on image build (docker build) via `--build-arg KEY=VALUE`

Let's lock in the default port value with a build time argument, in our `Dockerfile`, we add `ARG DEFAULT_PORT=80`, this argument can't be used on the `CMD` instruction, but it can be used on the other instructions, eg, `ENV PORT $DEFAULT_PORT`, now we can set a dynamic arg that is used as a default value for the dynamic env variable for the port! Let's rebuild, `docker build -t feedback-node-app:web .`, and if we wanted to change the default value of the arg, we build again with the `--build-arg KEY=VALUE` flag, `docker build -t feedback-node-app:dev --build-arg DEFAULT_PORT=8000 .`

In the `Dockerfile`, remember that all subsequent layer are rerun when we modify one layer, so it's best to put the `ARG` and `ENV` after the `RUN npm install` so we don't reinstall all packages everytime we pass in a new default ARG

### Containers & Networking

In many applications, we'll need more than one container as it's hard to configure a container that does more than one "main thing" (eg. run a web server AND a database) and it's considered a good practice to focus each container on one main task.

Containers need to communicate

- with each other
- with the host machine
- with the world wide web

We'll be working with `6-networks` for this module, an api based on swapi with a db implemented (MongoDB needs to be installed locally to be able to follow along)

#### Communicating with the World Wide Web

Our example app uses axios to send a web request to the star wars api, ie. there's http communication between our app and a website

Our `Dockerfile` is a standard dockerfile for conainerizing a node app, so we can just build n image with `docker build -t favorite-node .` and run a container with `docker run --name favorites -d --rm -p 3000:3000 favorite-node`. We do not need a volume as we don't need anything to survive except for the data that is stored in the db, which is not part of this container

As of now, our app crashes (we need to run it in non detached mode to see the error, or not with the --rm falg and then look at the logs), we get a mongoDB connect error as the container can't "talk" to the mongoDB running on our localhost

If we temporarily remove the connection to the db `mongoose.connect(...)` and rebuild the image and rerun the container, the container now doesn't crashes we can use Postman to check that the `/movies` and `/people` endpoints work fine by sending GET request to `http://localhost:3000/movies` and `http://localhost:3000/people`. The other endpoint `/favorites` won't work as we need our DB

Communicating with the world wide web works out of the box with docker!

#### Communicating with the Local Host Machine

Our example app uses MongoDB to fetch and store data in our own db, ie. there's communication between our app and a service running on our host machine  
nb: Communicating to the Host Machine typically is a requirement during development, eg. because we're running a development database on our machine

We add the `mongoose.connect(...)` code back in our code and will now make our app communicate with the locally installed MongoDB database

For communication between the dockerized app and our localhost, we need to change the code in the app, specifically the address mongoose tries to connect to, from `"mongodb://localhost:27017/swfavorites"` to `"mongodb://host.docker.internal:27017/swfavorites"`, and docker will do the rest "under the hood" (we need to rebuild and rerun of course)

`host.docker.internal` is a special address/identifier which is translated to the IP address of
the machine hosting the container by bocker

#### Communicating with Other Containers

What if we had another container with a service (database,...) we want to comunicate with from our main app container?  
Our example app is a great use case as it would be best to have the MongoDB in a separate container from our main nodejs app

First we need a second container for our MongoDB, there's an [official mongoDB image](https://hub.docker.com/_/mongo) on Docker Hub, we'll run `docker run -d --name mongodb mongo` to pull and spin up a container with a MongoDB database

Then, we'll alter our app code so that it connects to that containerized db, `docker container inspect mongodb` will give us the IP address of the running MongoDB container under the `"IPAddress"` key; so we can now change the address mongoose tries to connect to this IP address, eg. `"mongodb://172.17.0.2:27017/swfavorites"`

After an image rebuild and container rerun, our app now works and the app container is connected to the mongodb container, we can use Postman to test the GET and POST routes to the `/favorites` endpoint

This isn't convenient though, as we have to look up the IP address and it will change everytime we rerun the db container, forcing us to change our code and rebuild an image

#### Docker Networks

With Docker, we can create networks via `docker network create NETWORK_NAME` and you can then attach multiple Containers to one and the same network

First, we create our network, `docker network create favorites-net`; `docker network ls` can be used to list all existing networks

Then, we'll start our mongodb container on that network with `docker run -d --name mongodb --network favorites-net mongo`

Finally, in our app, we can now change the address mongoose tries to connect to, to `"mongodb://mongodb:27017/swfavorites"` (we're using the name of the mongoDB container as the address, `mongodb`) and docker will translate the name of the MongoDB container to the IP address "under the hood" as long as both containers are on the same network. After an image rebuild `docker build -t favorite-node .`, we can run a container with the `--network NAME` flag, `docker run --name favorites --network favorites-net --rm -d -p 3000:3000 favorite-node`; we can use Postman or check the logs with ` docker logs favorite` to see that our app works

Use `docker network prune` or `docker rm NETWORK_NAME` to remove docker networks

nb: We don't have to publish ports from our mongoDB container as it's just another container that needs to connect to it

nb: Docker Networks actually support different kinds of "Drivers" which influence the behavior of the network, the default driver is the "bridge" driver

nb: right now we lose all the data from the db when the MongoDB container is stopped, this is something we can (and will) solve with volumes

## Real Life Scenarios

### Multi-Container Projects

We'll be working with `7-multi-service-app` for this module, a demo project with 3 "building blocks":

- Database: MongoDB
  - Data must persist
  - Access should be limited
- Backend: NodeJS REST API
  - Data must persist
  - Live Source Code Update
- Frontend: React SPA
  - Live Source Code Update

For this module, we'll focus on development only, ie. not deployment & production

#### Dockerizing the MongoDB Service

We're going to use the docker hub mongo image, `docker run -d --rm --name mongodb -p 27017:27017 mongo`, we need to expose the default mongo port from the container to our localhost machine as the rest of our backend is not dockerized yet and needs to communicate with it

#### Dockerizing the Node App

We'll write our own `Dockerfile`

```dockerfile
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

CMD ["node", "app.js"]
```

Then build our own image, `docker build -t goals-node .` (we need to be in the backend folder to do that)

Finally, we spin up a container, `docker run --name goals-backend --rm -d -p 80:80 goals-node` (we need to publish the port 80 so that the react app can still connect to it)

Right now the app crashes, we need to change the address in `app.js` for `mongoose.connect` from `mongodb://localhost:27017/course-goals` to `mongodb://host.docker.internal:27017/course-goals`, then rebuild the image and rerun a container. We see in the logs that the app is successfully connecting to the mongodb container

#### Moving the React SPA into a Container

We'll write our own `Dockerfile`  
(by default the port is 3000 for a react app)

```dockerfile
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Then build our own image, `docker build -t goals-react .` (we need to be in the frontend folder to do that)

Finally, we spin up a container, `docker run --name goals-frontend --rm -p 3000:3000 -it goals-react`, we need to run the container in interactive mode otherwise the dev server spun up by the react app will immediately stop (we won't actually interact with it)

Our MERN stack app is now entirely dockerized and works, it needs to be polished more though

#### Adding Docker Networks

Right now, the containers are communicating through our localhost and their published port, we want to use a docker network instead

First, we'll create a docker network `docker network create goals-net`

Second, start a mongoDB container running on the network `docker run --name mongodb -d --rm --network goals-net mongo`, there is no longer a need to publish a port

Third, we need to change again the address in `app.js` for `mongoose.connect` from `mongodb://host.docker.internal:27017/course-goals` to `mongodb://mongodb:27017/course-goals`, then rebuild the image and start a container, this time on the network, `docker run --name goals-backend --rm -d --network goals-net -p 80:80 goals-node`

nb: for the React app, as it is run in the browser and has no idea about our containers and how to translate the `goals-backend` domain, we don't do the same as for our backend, we keep all our domains as `localhost`, so it means we still need to publish port 80 form our backend app, and we don't need to put the React app on the docker network, we'll still start a container with `docker run --name goals-frontend --rm -p 3000:3000 -it goals-react` as before

#### Adding Data Persistence & Security to MongoDB

For our MongoDB container, we want the data to persist and the access to be limited

We'll add a volume to the container to save the data, we ge the path to the directory from the [doc](https://hub.docker.com/_/mongo) `docker run --name mongodb -v data:/data/db -d --rm --network goals-net mongo`

nb: we just want ot persist data, so we use a volume and not a bind mount

For security, we can use the env variables `-e MONGO_INITDB_ROOT_USERNAME=VALUE -e MONGO_INITDB_ROOT_PASSWORD=VALUE` so that a username and password are necessary to connect ot the container `docker run --name mongodb -v data:/data/db -d --rm --network goals-net -e MONGO_INITDB_ROOT_USERNAME=kevin -e MONGO_INITDB_ROOT_PASSWORD=test mongo`

Now we need to modify our backend to reflect the changes to access, we need to change the address in `app.js` for `mongoose.connect` to `mongodb://kevin:test@mongodb:27017/course-goals?authSource=admin`, then rebuild and start a container; our access to the mongoDB container is now secure (nb: we need to remove the volume created without identifiers and make a new one)

#### Volumes, Bind Mounts & Polishing for the NodeJS Container

For our NodeJS container, we want the data to persist (the log files, in this case), and we want live source code update

We need to add nodemon, modify the `package.json` and the `Dockerfile`

```json
{
  [...]
  "scripts": {
    "start": "nodemon app.js"
  },
  [...]
  "devDependencies": {
    "nodemon": "^2.0.4"
  }
}
```

```dockerfile
CMD ["npm", "start"]
```

We'll add a volume to the container to save the logs, a bind mount for the entire app, and an anonymous volume for node_modules, `docker run --name goals-backend -v /home/kevin/code/docker-bootcamp/7-multi-service-app/backend:/app -v logs:/app/logs -v /app/node_modules --rm -d --network goals-net -p 80:80 goals-node`

Right now the MongoDB username and pwd are hard coded, we'll use env variables instead through a `.env` file, we need to change the address in `app.js` for `mongoose.connect` to `mongodb://${process.env.NONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`, then rebuild and start a container, `docker run --name goals-backend -v /home/kevin/code/docker-bootcamp/7-multi-service-app/backend:/app -v logs:/app/logs -v /app/node_modules --rm -d --env-file ./.env --network goals-net -p 80:80 goals-node`

Finally, we'll add a `.dockerignore`

#### Live Source Code Updates for the React Container (with Bind Mounts)

For our React app container, we want live source code update

We'll add a bind mount for the `src`, run a container with `docker run -v /home/kevin/code/docker-bootcamp/7-multi-service-app/frontend/src:/app/src --name goals-frontend --rm -p 3000:3000 -it goals-react`

nb: no need to add nodemon to a create-react-app app

Finally, we'll add a `.dockerignore`

### Using Docker-Compose

We'll be working with `8-compose` for this module, the same MERN stack app as we used before

See `docker-commands.txt` for a list of all the commands we have to execute to get our project up and running, there's quite a lot of them!

Docker compose is a tool which helps with orchestration of multiple containers (It can also be used for single Containers to simplify building and launching) with just one configuration file and a set of orchestration command (build, start, stop,...)

nb: Docker compose does NOT replace Dockerfiles for custom Images  
 Docker compose does NOT replace Images or Containers  
 Docker compose is NOT suited for managing multiple containers on different hosts (machines)

We can put our container configuration into a `docker-compose.yaml` file and then use just one command to bring up the entire environment, `docker-compose up`

As we'll see later, we can use `docker-compose.yaml` to overwrite/add instructions from a `Dockerfile`, or even instead of a `Dockerfile` for simple cases, as `COPY` and `RUN` aren't available in a docker compose file, but `working_dir` and `entrypoint` are

#### docker-compose.yaml

nb: YAML uses indentation (2 spaces)  
use VSCode docker extension for autocomplete help

Use `docker-compose version` to check the installed version

The `services` key lists the containers as its children

Use the `image` key when using an existing image, the `build` key to build a custom one

If our `Dockerfile` is named Dockerfile we can use the shorter form for the `build`

```yaml
build:
  context: ./backend
  dockerfile: Dockerfile
# is the same as
build: ./backend
```

There are two options for listing env variables:

```yaml
environment:
  - MONGO_INITDB_ROOT_USERNAME=kevin
  - MONGO_INITDB_ROOT_PASSWORD=test
# or
environment:
  MONGO_INITDB_ROOT_USERNAME: kevin
  MONGO_INITDB_ROOT_PASSWORD: test
```

We can also specify an `.env` file - the path we specify is relative to the `docker-compose.yaml`

```yaml
env_file:
  - ./env/mongo.env
```

When using docker compose, docker will automatically create a new network for all the services specified in the `.yaml` file and add them to that network, so while we can use a `network` key to specify a network, it isn't necessary

For named volumes, we also need to add a top level `volumes` key listing all named volumes used by our services  
nb: the same volume can be shared between different containers

```yaml
volumes:
  data:
  logs:
  # no values after the key to the name
```

For bind mounts, we don't need the absolute path, we can use a relative path

The `depends_on` key/option is specific to docker compose, it tells docker compose that a container is depending on another container being already running, we list the services names as values

To run a service in interactive mode, we can set the keys/vals

```yaml
stdin_open: true
tty: true
```

the full `docker-compose.yaml` for our NEMRN stack app:

```yaml
version: "3.8"
services:
  mongodb:
    image: "mongo"
    volumes:
      - data:/data/db
    env_file:
      - ./env/mongo.env
  backend:
    build: ./backend
    ports:
      - "80:80"
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    env_file:
      - ./env/backend.env
    depends_on:
      - mongodb
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src
    stdin_open: true
    tty: true
    depends_on:
      - backend
volumes:
  data:
  logs:
```

#### Docker Compose Up & Down

`docker-compose up` will build images & start a container for each service, it takes two flags:

- `-d` : Start in detached mode
- `--build` : Force docker compose to re-evaluate / rebuild all images (otherwise, it only does that if an image is missing)

`docker-compose down` will stop and remove all containers/services as well as the networks it created, if run with the `-v` flag it will also remove all volumes used for the containers

`docker-compose build` will just build the images and not start containers

#### Docker Compose Naming

Docker compose will rename our containers and volumes based on the folder name and the service name we specify in the `docker-compose.yaml`, and an incrementing number for containers

We can assign our own container names with the `container_name` option

### “Utility Containers”

An "Utility Container" is a container that just has an environment in them, with no application, it allows eg. to create a npm project on our local machine without having node & npm installed on it

nb: [some comments on running utility containers on linux](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/learn/lecture/23074458#questions/12977214/)

`docker exec` allows use to run certain commands inside the running container beisdes the default command the container runs, eg `docker exec -it node npm init` to init a npm project inside a container using the official node image

We can also overwrite the default command of the image with `docker run -it node npm init`

Still not that useful since these container will stop as soon as the npm project is initialized and we're left with nothing!

#### Building a utility container

We'll be working with `9-utility-container` for this module, we will create a utility container, nb: we don't want a starting command

```dockerfile
FROM node:14-alpine

WORKDIR /app
```

we'll build our image, `docker build -t node-util .` and run it to create a npm app in the container _and locally_ thanks to a bind mount `docker run -it -v /home/kevin/code/docker-bootcamp/9-utility-container:/app node-util npm init`

Running the command will create a `package.json` on our localhost machine, meaning we can create a npm project on our local machine without having node & npm installed on our local machine!

nb: the container shuts down once the command is done

#### ENTRYPOINT

What if we wanted a more restricted utility container, eg. we only want container that runs npm commands? We can add an `ENTRYPOINT` key to the `Dockerfile`

```dockerfile
FROM node:14-alpine

WORKDIR /app

ENTRYPOINT [ "npm" ]
```

Then build an image `docker build -t mynpm .` and run it with our volume `docker run -it -v /home/kevin/code/docker-bootcamp/9-utility-container:/app mynpm init`; once again, running the command will create a `package.json` on our localhost machine. We could then add express, with `docker run -it -v /home/kevin/code/docker-bootcamp/9-utility-container:/app mynpm install express`

#### Using Docker Compose

Right now, we have to write long command prompts, let's use docker compose to solve that problem

```yaml
version: "3.8"
services:
  npm:
    build: ./
    stdin_open: true
    tty: true
    volumes:
      - ./:/app
```

Use `docker-compose run mynpm init` to run the mynpm service with docker compose and create a npm project. Then run `docker-compose run mynpm install express` to install express

There is no up/down with `docker-compose run` so the containers won't be removed, add the `--rm` flag to auto-remove them, eg. `docker-compose run --rm mynpm init`

### More Complex Setups

We'll be setting up a Laravel & PHP dockerized Project for this section

Our target setup will allow us to build laravel apps without having to install anything on our host machine, in total we'll have 6 containers,

- Our host machine with a folder for the source code for the laravel php app
- An app container with the PHP interpreter that has access to our source code
- An app container with a Nginx web server that takes incoming request and send them to the PHP interpreter container, get the response back and sends it to the client
- An app container with a MySQL database that communicates with the PHP interpreter container
- An utility container for Composer, the package manager that we'll use to create the laravel app (in our source code folder) and install dependencies
- An utility container for laravel Artisan, a command tool for eg. running migrations
- An utility container for npm, used for the views of the app

We'll be working with `10-laravel` for this module

#### Adding a Nginx (Web Server) Container

We'll use `docker-compose.yaml` to setup the entire project

For Nginx we'll use the official image from docker hub

The default port for Nginx is 80, we'll publish it to our local machine port 8000

We need a bind mount so we can provide our own configuration to Nginx, we'll bind just the specific `nginx.conf` file, in read-only mode as the container shouldn't have to a need to overwrite this conf file

```yaml
server:
  image: "nginx:stable-alpine"
  ports:
    - "8000:80"
  volumes:
    - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
```

#### Adding a PHP Container

For PHP we'll build our own image as there's no finished image with everything we need so we need to write a `Dockerfile`

We need to use a php-fpm image form the Nginx config we're using as the base for our custom image

We'll install pdo pdo_mysql within our image

```dockerfile
FROM php:7.4-fpm-alpine

WORKDIR /var/www/html

RUN docker-php-ext-install pdo pdo_mysql
```

We don't have a command in our `Dockerfile`, so the command from the base image will be used, in this case it's a command that invokes the PHP interpreter

In our `docker-compose.yaml` file, we need a bind mount to make the source code available to the PHP interpreter in the right folder, `/var/www/html`, and we want to be able to work on that source code in the container and for our changes to be reflected on the local machine. We use the `delegated` option to improve performance when it comes to writing to the local machine

The port we expect is defined in `nginx.conf`, 3000, and the default port is in the source code for the base image, 9000; but while we can map the ports `- "3000:9000"` in case we want to interact with the PHP container from our host machine, it's not actually needed just to send requests as we have direct container to container communication, we can just modify the `nginx.conf` from `fastcgi_pass php:3000;` to `fastcgi_pass php:9000;`

```yaml
php:
  build:
    context: ./dockerfiles
    dockerfile: php.dockerfile
  volumes:
    - ./src:/var/www/html:delegated
```

#### Adding a MySQL Container

For MySQL, we'll use the official mysql image

The container will communicate with the PHP container through the docker network

We do need env variables to set up the MySQL image, we'll use a `.env` file

```yaml
mysql:
  image: mysql:5.7
  env_file:
    - ./env/mysql.env
```

#### Adding a Composer Utility Container

We need composer to setup a laravel application

We'll build our own image base on the existing composer image, with a custom `ENTRYPOINT`

```dockerfile
FROM composer:latest

WORKDIR /var/www/html

ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
```

In our `docker-compose.yaml` file, we need a bind mount to expose our source code directory to this image so that if we use composer to create a laravel app inside of the container, this will be mirrored to our local machine, we also create an empty /src folder on our local machine

```yaml
composer:
  build:
    context: ./dockerfiles
    dockerfile: composer.dockerfile
  volumes:
    - ./src:/var/www/html
```

#### Creating a Laravel App via the Composer Utility Container

These four container are sufficient for us to create the laravel app

In our terminal, we want to run just the compose service to create our laravel app, `docker-compose run --rm composer create-project --prefer-dist laravel/laravel .`, we create the app in the root folder of the container as we specified a `WORKDIR`

This is working as intended and we now have our laravel app in the local machine /src folder

#### Permission Errors on Linux

When using Docker on Linux, we might face permission errors when adding a bind mount as shown in the next part. If we do, try these steps:

Edit the `php.dockerfile` to this:

```dockerfile
FROM php:7.4-fpm-alpine

WORKDIR /var/www/html

RUN docker-php-ext-install pdo pdo_mysql

RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

USER laravel
```

Also edit the `composer.dockerfile` to this:

```dockerfile
FROM composer:latest

RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

USER laravel

WORKDIR /var/www/html

ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
```

nb: we need to delete the existing laravel app (i.e. the src/ folder) and re-create it after updating all these files. Also make sure to prune all images before re-running the commands again with `docker image prune -a`

Find extra comments and solutions [here](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/learn/lecture/23172898#questions/12986850/)

#### Launching Our App

In the `.env` file in the /src folder of our newly created app, we need to adjust the DB\_ keys values so that laravel is able to connect to the database, from

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

to

```
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

Right now, our server service - the main entry point that will serve the application - doesn't have any way to communicate with the local source code, we need to add a bind mount to it

We can also add dependencies to the server service so that just running the server service will also run the dependencies, and we don't have to type `docker-compose up -d server php mysql`

```yaml
server:
  image: "nginx:stable-alpine"
  ports:
    - "8000:80"
  volumes:
    - ./src:/var/www/html
    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
  depends_on:
    - php
    - mysql
```

We can now run our app, `docker-compose up -d --build server`, we want to rebuild the images in case something changed

Our laravel app is now running on [localhost:8000](http://localhost:8000/)

#### Adding More Utility Containers

We still need to add an artisan utility container

We'll use the php `Dockerfile` for it, but overwrite/add some of the instructions through the `docker-compose.yaml` file, specifically, the entrypoint. (alt. we could write another `Dockerfile`)

```yaml
artisan:
  build:
    context: ./dockerfiles
    dockerfile: php.dockerfile
  volumes:
    - ./src:/var/www/html
  entrypoint: ["php", "/var/www/html/artisan"]
```

Finally, we'll add a npm utility container, using the official npm image, and with a bind mount

```yaml
npm:
  image: node:14-alpine
  working_dir: /var/www/html
  entrypoint: ["npm"]
  volumes:
    - ./src:/var/www/html
```

With our app running (`docker-compose up -d --build server`), we'll start artisan and run migrations, `docker-compose run --rm artisan migrate`

We now have a working setup for a laravel app

Our final `docker-compose.yaml`,

```yaml
version: "3.8"

services:
  server:
    image: "nginx:stable-alpine"
    ports:
      - "8000:80"
    volumes:
      - ./src:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - php
      - mysql
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build:
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  artisan:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html
    entrypoint: ["php", "/var/www/html/artisan"]
  npm:
    image: node:14-alpine
    working_dir: /var/www/html
    entrypoint: ["npm"]
    volumes:
      - ./src:/var/www/html
```

#### Bind Mounts vs COPY

nb: this is a setup for development purpose, not deployment. The idea of containers for deployment is that everything a container needs is inside the container, so there's no local machine with the source code in a deployment setup

So, we might want to consider creating a `Dockerfile` for the server service, and copying over our source code and nginx config into that image so that besides the bind mount used during development, we also copy a snapshot of the source code and our nginx config in the image when it's being build, so that when we deploy that image, it already contains the source code and nginx config it needs, let's do that, in `nginx.dockerfile`

```dockerfile
FROM nginx:stable-alpine

WORKDIR /etc/nginx/conf.d

COPY nginx/nginx.conf .

RUN mv nginx.conf default.conf

WORKDIR /var/www/html

COPY src .
```

We'll also update the server service in our `docker-compose.yaml`. We need to modify our usual paths for the `context` and `dockerfile` keys because the `context` key also sets the folder where the dockerfile will be built, but the src and nginx folders we refer to in the `nginx.dockerfile` are outside the dockerfiles folder so it would fail with our usual paths

```yaml
server:
  # image: "nginx:stable-alpine"
  build:
    context: .
    dockerfile: dockerfiles/nginx.dockerfile
  ports:
    - "8000:80"
  volumes:
    - ./src:/var/www/html
    - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  depends_on:
    - php
    - mysql
```

We need to also modify the `php.dockerfile` for the php container, so that we copy the source code into the container so it works without the bind mount once deployed

```dockerfile
[...]
COPY src .
[...]
```

And as with the server service, we need to adjust the `context` & `dockerfile` keys in the `docker-compose.yaml` file

```yaml
php:
  build:
    context: .
    dockerfile: dockerfiles/php.dockerfile
  volumes:
    - ./src:/var/www/html:delegated
mysql:
  image: mysql:5.7
  env_file:
    - ./env/mysql.env
```

Now if we remove the bind mounts for the server and php services,

```yaml
server:
  [...]
  # volumes:
  #   - ./src:/var/www/html
  #   - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  [...]
php:
  [...]
  # volumes:
  #   - ./src:/var/www/html:delegated
  [...]
```

and launch our app with `docker-compose up -d --build server`, it works but we have a laravel permission error linked to our php service; we can fix this by giving write access to our php container to the default user created by the base image by adding a command, our deployment ready `php.dockerfile`

```dockerfile
FROM php:7.4-fpm-alpine

WORKDIR /var/www/html

COPY src .

RUN docker-php-ext-install pdo pdo_mysql

RUN chown -R www-data:www-data /var/www/html
```

nb: on linux we might have permissions issues

Our app then runs without bind mounts and is ready for deployment. We do want to add the bind mounts back in, though

We'll finally also need to update the `docker-compose.yaml` for the artisan service

```yaml
artisan:
  build:
    context: .
    dockerfile: dockerfiles/php.dockerfile
  volumes:
    - ./src:/var/www/html
  entrypoint: ["php", "/var/www/html/artisan"]
```

### Deploying Docker Containers

#### Development to Production

Bind Mounts shouldn’t be used in Production

Containerized apps might need a build step (eg. React apps)

Multi-Container projects might need to be split (or should be split) across multiple hosts / remote machines

Trade-offs between control and responsibility might be worth it, if we have to do everything ourselves we are responsible for security, updates, etc

#### Bind Mounts, Volumes & COPY

In Development:

- Containers should encapsulate the runtime environment but not necessarily the code
- Use “Bind Mounts” to provide your local host project files to the running container
- Allows for instant updates without restarting the container

In Production:  
Image / Container is the “single source of truth”

- A container should work as a standalone, we should not have source code on your remote machine
- Use `COPY` to copy a code snapshot into the image
- Ensures that every image runs without any extra, surrounding configuration or code

#### Basic Example with AWS & EC2

We'll be working with `11-deployment` for this module, a simple node app without a database or anything else, and with a single container

We'll build an image of the app, `docker build -t node-dep-ex .` and run a container `docker run -d --rm --name node-dep -p 80:80 node-dep-ex` to make sure it works (it does!)

We'll install docker on a remote host (eg. via SSH), push our local docker image to Docker Hub and then pull the image to the remote host, and run a container based on that image on the remote host and expose the necessary ports from that remote host to the www so end-users can use our app

We'll deploy to AWS EC2. AWS EC2 is a service that allows us to spin up and manage our own remote machines, the steps are:

1. Create and launch EC2 instance, VPC and security group
2. Connect to instance (SSH), install Docker and run the container
3. Configure security group to expose all required ports to WWW

On the [EC2 Dashboard](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:) click _launch instance_

Choose the Amazon Linux 2 AMI x86 (Amazon Machine Image), then the t2.micro instance (free tier), click _next_ and make sure that a VPC is created, leaving all other settings as default, and finally click _review & launch_

Clicking _launch_ will bring up a screen allowing us to create and download a new key pair, that will be needed later to connect to the instance via ssh. nb: if we loose the key, we'll have to delete the instance and create a new one, we can't get the key back!

On the EC2 dashboard instances list screen, select our instance and click _connect_ and pick _ssh client_, then follow the instructions:

1. Open an SSH client.
2. Locate your private key file. The key used to launch this instance is `docker-basic.pem`
3. Run this command, if necessary, to ensure your key is not publicly viewable.  
   `chmod 400 docker-basic.pem`
4. Connect to your instance using its Public DNS:  
   `ec2-18-205-243-1.compute-1.amazonaws.com`
   Example:  
   `ssh -i "docker-basic.pem" ec2-user@ec2-18-205-243-1.compute-1.amazonaws.com`

Now that we're connected, we need to install docker on this aws instance, we used an amazon AMI as they make it easy to install docker, we could always use the [traditional way](https://docs.docker.com/engine/install/) of installing docker on a linux machine

1. `sudo yum update -y`
2. `sudo amazon-linux-extras install docker`

To start the docker service, run `sudo service docker start`

Now we need to bring our local image to the aws instance,

- Option 1: Deploy Source

  - Build image on remote machine
  - Push source code to remote machine, run `docker build` and then `docker run`

- Option 2: Deploy Built Image

  - Build image before deployment (on local machine)
  - Just execute `docker run`

Option 2 avoids unnecessary remote server work, so we'll do that

Login on Docker Hub, Create a new repository, `node-deploy-1`, we can now push to this repo with `docker push kevinlabtani/node-deploy-1:tagname`

We need to first build our image (we'll also add a `dockerignore`), `docker build -t node-dep-1 .` and then rename it, `docker tag node-dep-1 kevinlabtani/node-deploy-1`

We can now push the image to docker hub, `docker push kevinlabtani/node-deploy-1` (we need to be logged in to be able to push)

Back on the aws instance, we can now pull/run the image we just pushed to docker hub, `sudo docker run -d --rm -p 80:80 kevinlabtani/node-deploy-1` (need `sudo` for permissions, there are better ways though...)

To test this app, go back to the aws EC2 dashboard where we can find an ipv4 public ip for our instance, `18.205.243.1`

In the _Netword & Security_ tab of the dashboard, click on _security groups_, then click on the right security group (can be found on the instances tab, by clicking on our active instance) and add an inbound rule (only the ssh port, 22, is open by default) to open the port 80, allowing **HTTP** form **anywhere**

We can now connect to our app!

We could of course also use docker-compose, if we had a multi container app for example

Say we update our code, the workflow to push the updated code is:

- Locally
  1. Rebuild `docker build -t node-dep-1 .`
  1. Retag `docker tag node-dep-1 kevinlabtani/node-deploy-1`
  1. Push to docker hub `docker push kevinlabtani/node-deploy-1`
- On the aws instance
  1. Stop the running container `sudo stop CONTAINER_NAME`
  1. We need to pull manually, otherwise docker will just rerun a container based on the existing local image `sudo docker pull kevinlabtani/node-deploy-1`
  1. Run a container based on the updated image `sudo docker run -d --rm -p 80:80 kevinlabtani/node-deploy-1`

To shut the aws instance forever, from the EC2 dashboard, instance tab, select our instance and under _instance state_, pick terminate

The disadvantages of this DIY approach is that we fully “own” the remote machine and we’re responsible for it and it’s security, we need to keep essentials software updated, manage network and security groups / firewall, manage scaling. On top of that, SSHing into the machine to manage it can be
annoying

#### Managed Remote Machines with AWS ECS

With a managed remote machine, creation, management and updating is handled automatically, monitoring and scaling is simplified

We'll use the same app as before, `11-deployment`

Go to the [ECS Console](https://console.aws.amazon.com/ecs/home?region=us-east-1#/getStarted) and click on _get started_ and then _custom_ to create our own container

The side drawer that appears allows us to specify the _Container definition_, or how ECS should execute `docker run`, ie. setting _container name_ is the same as using the `--name` flag. Pick a name (eg. node-demo), put our repo on docker hub as image `kevinlabtani/node-deploy-1`, map our ports `80` nb. the container internal ports will always be mapped to the same port outside of the container. Under _Advanced container configuration_, we could eg. overwrite the default `entrypoint` or `cmd` executed, or setup env variables. Click _update_ when we're done configuring

In the _Task definition_, we can tell aws how to run our containers, think of the task as one remote machine that runs one or more containers. We're using **FARGATE** by default, so aws isn't really running an EC2 istance for our container, it runs it in serverless mode and starts the container whenever there's a request to it, so we just pay for the time the container is executed, not the time it's just sitting around idle. Just click _Next_

In the _Service_ tab, we can control how a task should be executed, it's here we could eg. add a load balancer, jut click _Next_

In the _Cluster_ tab, we configure the overall network in which our services run. If we had mutiple containers, we could group multiple containers in one cluster so they all can communicate to each other, jut click _Next_

On the review page, click _Create_, then _view service_ once it's been created. To see our running app, click on _tasks_ and then on the one task id that's running to find our public ip where we can see our running application!

Say we update our code, after rebuilding locally and pushing the new image to docker hub, to see our updated app online, in the _clusers_, _tasks_ tab, click on the _task definition_ for the running task and then click on _create new revision_, keep all the settings and click _create_ (aws will auto pull the new image), then click on _Actions_ and pick _Update Service_, keep all the settings and click on _Skip to review_ and then _Update Service_. A new task is now visible under the tasks tab, grab the new Public IP and we'll be able to see the new app

nb: while a new IP is assigned to the updated app, it's still possible to link a domain name to the running Fargate ECS Task independent of the IP by using a load balancer as described [in the doc](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html)

To delete a service and cluster, in the cluster tab, click on the running service and click _delete_, then delete the cluster

#### Multi-Container App on AWS ECS

We'll use the app in `11-deployment-multi-container`, it's the same goals app used in the docker-compose section but missing its front-end, sow e have a node container and a mongodb container

We won't use docker-compose for deployment as we need to configure our containers separately when it comes to cpu capacity, etc. We'll just use the `docker-compose.yaml` as a source for what we need to do

First the backend container, we need to create an image and push it to docker hub

When deploying, we can't use the docker network approach of using services names (in our `mongoose.connect` address string) and have docker auto assign the right ip because once deployed the different containers might be on different machines. There is an exception, if with ECS, the containers are added to the same task then they'll be run on the same machine, but still, ECS won't create a docker network for them, but we can use `localhost` as an address, so we can change the connect string to use an env variable for the host, `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}:27017/course-goals?authSource=admin`, and add that variable to the `backend.env`, `MONGODB_URL=mongodb`, and to the `Dockerfile`, `ENV MONGODB_URL=secret` we'll then use another value when we build for deployment. We're using env variables so we can use the exact same image in dev and prod

Let's build our backend image, `docker build -t goals-node .`

We now need to push it to docker hub. Create a new repository, `goals-node`, we can now retag, `docker tag goals-node kevinlabtani/goals-node`, then push to this repo with `docker push kevinlabtani/goals-node` nb. we're just pushing the image, so things like env variables aren't being used at the moment, these are being set in when we run a container based on the image

On ECS, click on _Create Cluster_ (alternatively, we coumld use the wizard again), seleckevinlabtani/goals-nodet _Networking only_, then _Next_, name the cluster, check _create vpc_ while keeping all the defaults and click _create_

Click on _view cluster_ and under _Task definitions_, Create a new task definition, pick FARGATE, then _Next step_. Name this Task Definition, pick the ecsTaskExecution as _Task Role_, pick the smallest size for the cpu and memory, click _add container_

Now in the _Container definitions_ tab, name the container goals-backend, add the image from our docker hub, map port 80, under ENVIRONMENT, we don't want to use `nodemon` and bind mount in deployment, so we'll change the Command key to `node,app.js`.  
We also need to specify our 3 environment variables; here the `MONGODB_NAME` need to be `localhost` as we've seen. We don't need to setup anything else, so click _Add_. nb: the volumes we have in dev are a bind mount for auto-reload and an anonymous volume so that the bind mount doesn't overwrite the node_modules folder, these aren't needed in production

Second, the MongoDB container, click _add container_ again in the _Task definitions_ we're working on, name it mongodb, use mongo as the image, map port 27017, finally add the environment variables for root username and password.  
We'll need to add storage, but we'll do that after, so click _Add_ to add the container

We can leave all other settings as they are and click _Create_ to create our _Task definition_

We can now launch a service based on this task. Go back to _clusters_, pick our goals-app cluster and click _Create_ under the Services tab. Pick FARGATE and under _Task Definition_ pick the _goals_ task we just created, name the service goals-service, nr of tasks is 1, we can keep the deployment type and click _Next step_.  
On the next page, _Configure network_, pick the VPC that was created when we created the cluster and under subnets, add both subnets we're able to choose from, make sure auto-assign public ip is enabled and under _Load balancing_ choose _Application Load Balancer_, create the load balancer (next paragraph) and select it by its name, choose the `goals-backend:80:80` mapping and click _Add to load balancer_, add it to the target group we created in advance, finally click _Next step_

We'll need to create an application load balancer and name it, it should be internet-facing, expose port 80 and be connected to our VPC, check the availability zone checkboxes and click _next_, pick the right security group besides the default one, click _configure routing_, name the group and choose ip as a target type, also set the health check path to `/goals` (requests sent to `/` will get a 404 which is treated as unhealthy by our load balancer) then click _next:register target_, click _next:review_ and finally _create_. Ignore auto scaling and click _Next step_ again, finally click _Create Service_ and then _View Service_

On our Cluster > goals-serviceapp page, we now have a service, if we click on that service we see the task associated with that service and if we click on that task, we can find the public ip (and the two containers belonging to that task) and use Postman to test the api endpoints

The public IP address change every time we deploy an updated container, that's why we created a load balancer. If we search load balancers in the AWS service list, we'll find the load balancer we created under the EC2 dashboard. That load balancer has a DNS name that looks like a domain name, `ecs-lb-783189166.us-east-2.elb.amazonaws.com` we can go (use in Postman) to that url instead of the ip address

#### EFS Volumes with AWS ECS

Right now, all the data is lost if we push an update or recreate new containers

AWS EFS --elastic file system - is a service that allows us to attach a file system to our serverless executed containers, these will survive even if the containers are recreated

To add a volume, go to _Task Definitions_, click on our goals Task definition and pick the latest and click _Create new revision_. Scroll down o volumes and click _add volume_, name it data and pick EFS as volume type, then pick the _File System ID_ for the security group we're creating in the next paragraph, then click _add_; we just defined the volume

Click on _Amazon EFS console_ to create an EFS volume, click _create file system_, name it db-storage and add it to the right VPC, then click _Customize_. Move on to Step 2, network access, we should have the mounts, under security groups, choose the security group we're creating in the next paragraph for all mount targets (2?), click _next_ through to _create_

On the EC2 server page, go to _security groups_ and create a new security group, name it efc-sc, add in to the VPC and add an inbound rule for type:NFS, source:custom, being the goals security group we're using to manage our containers, the port is 2049 (automatically assigned). Click _crreate security group_

After having defined the volume, we want to connect it to our containers. Click on the mongo>DB container name to access its config, go down to _STORAGE AND LOGGING_, pick the data volume we just added as _Mount point_ and bind it to the `/data/db` path inside of the container, then click _update_, then _create_ to create this new task revision. Th ego to actions, pick _update service_ to force redeployment. nb. as of now, pick 1.4 for _Platform version_ as "latest" will fail; click through to _update service_

Now even if we restart everything with brand new containers, as long as we keep that volume, our data will persist.

There's still a problem though, as we're using rolling deployment, so a new task starts before the previous one finishes, and the previous one stops only if the new one is running successfully (which won't happen since we have an error), meaning two tasks with two mongoDB containers are trying to connect to the same file system at the same time. That will only happen when we deploy a new version though, so we won't look for a workaround as we plan to replace the mongoDB container anyway, so for now we'll just manually stop and remove the currently running task so that the to be deployed task has a change of becoming active

#### Moving to MongoDB Atlas

We can absolutely manage your own Database containers, but there are a few pitfalls

- Scaling & managing availability can be challenging
- Performance (also during traffic spikes) could be bad
- Taking care of backups and security can be challenging

So we should consider using a managed Database service (eg. AWS RDS, MongoDB Atlas), we're going to switch to MongoDB Atlas

We'll be working with `11-deployment-multi-container-atlas` for this section, it's the same app as before

Create a cluster, get the connection string `mongodb+srv://<username>:<password>@cluster0.jjobt.mongodb.net/<dbname>?retryWrites=true&w=majority` and replace the `mongoose.connect` connection string. If we wanted to use the cloud db for prod and a local mongodb container for dev, we'd need to make sure that the version of MongoDB used by the docker hub mongo image is the same as the version used by MongoDB Atlas, 4.2.10 right now. We'll also use MongoDB atlas in dev though, but with a different db, so we can change the connection string to `mongodb+srv://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}/${process.env.MONGODB_NAME}?retryWrites=true&w=majority`, adding an env variable for the db name

We can remove the mongodb service in the `docker-compose.yaml` and its associated volume, as well as the `mongodb.env`

We need to add an user and use credentials in MongoDB Atlas, we also need to manage Network Access, whitelist `0.0.0.0./0` to allow access from anywhere

Give it a try, `docker-compose up`  
Now we need to rebuild `docker build -t kevinlabtani/goals-node .` and push the new image to docker hub

In AWS ECS, we now want to get rid of the MongoDB container. Go to _Task Definitions_, pick the latest goals task definition and create a new revision, now remove the mongodb container and the volume, and related EFS resources and security group. On the Node app container, we need to change the env variables to use the new ones from MongoDB Atlas. Then, create this new task revision and actions update service to force redeploy (we can go back to latest _Platform version_ as we don't use EFS anymore)

#### Multi-Stage Builds

We'll work with `11-deployment-multi-container-atlas-frontend`, the same app as before with the frontend added back in

Some apps require a build step, eg. an optimization script that needs to be executed after development but before deployment, so the dev setup is different from the prod setup. Our react frontend is like that

The react app needs to be executed differently in dev and in prod so we'll have to setup two different environments. In production we don't need node like we do in dev, the js code itself once build with react is not executed with nodejs. We'll add a second dockerfile, `Dockerfile.prod` with the instructions needed to create a web server to serve the built react app

First, we'll focus on building the react app with the `Dockerfile.prod`,

```dockerfile
FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

CMD ["npm", "run", "build"]
```

Then, we'll add a second stage to serve the app with nginx. Every `FROM` instruction create a new stage in the `Dockerfile`  
We have a `COPY` because we don't want our previous changes (ie. the build step) discarded. Note how we're aliasing the first stage and then copying from it. See the notes on docker hub [for the nginx image](https://hub.docker.com/_/nginx) for more info on the command we run

```dockerfile
FROM node:14-alpine as build

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

RUN npm run build

FROM nginx:stable-alpine

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

If we wanted, we could execute just the first stage `docker --target build -f Dockerfile.prod build .`, the `--target STAGE_NAME` flag allows us to target just a stage (not helpful here, but we could have eg. one stage for testing)

#### Deploy React SPA

There's one adjustment we have to make on our code as we're fetching from `localhost` right now, in prod the react code runs in the browser as we've seen before. If we planned to deploy the react app in the same task as our backend, the request would be to the same url, so we could just do the request to `/goals/` instead of `http://localhost/goals/`. We'll create a new task for the react app though, so the request wil be to a different ulr, we'll use an env variable to set the url. nb: the react app manage its own env variable separate from docker (once again, the code isn't run in the docker container but on the client's machine), when we run `npm start`, `process.env.NODE_ENV` is always equal to 'development'. We can get the production server url from the backend load balancer

```js
const backendUrl =
  process.env.NODE_ENV === "development"
    ? "http://localhost"
    : "http://ecs-lb-783189166.us-east-2.elb.amazonaws.com";
```

Let's build the image for prod, `docker build -f Dockerfile.prod -t kevin-labtani/goals-react .` and push it to dockerhub on a new repo, `goals-react`

To deploy our react frontend to ECS, the problem we have right now is that both our frontend and backend are mapped to port 80. We could serve our frontend from the backend since it's already running a node server and merge both in a single container. We could expose the backend on a different port. We won't do either, though. (This is not an ECS restriction btw, we can't have two webservers on the same host!) So what we need to is create a new _Task Definition_ in addition to the goals task definition we already have.

Create a new Task Definition, pick FARGATE, name it goals-frontend, pick the same _task role_ as we did for the backend, pick the min amount of memory and cpu resources, then click _Add container_

In the _Container definitions_, name the container goals-frontend, put our image from dockerhub, map port 80 (different task from the backend so this will work) Click _add_ to add the container, the create the Task definition

We need to add a new application load balancer for the frontend app, from the AWS EC2 panel. Name it goals-react-lb It should be internet-facing, expose port 80 and be connected to our VPC, check the availability zone checkboxes and click _next_, pick the right security group besides the default one, click _configure routing_, name the group react-tg and choose ip as a target type, also set the health check path to `/` then click _next:register target_, click _next:review_ and finally _create_. Go to the load balancer config to find his url we'll later use to reach our app.

We now need to add a service based on the task definition we just created, on the Task Definition tab, click _actions_ and pick _Create Service_.

Pick FARGATE and under _Task Definition_ pick the _goals_ task we just created, name the service goals-service, nr of tasks is 1, we can keep the deployment type to rolling update and click _Next step_.  
On the next page, _Configure network_, pick the VPC that was created when we created the cluster and under subnets, add both subnets we're able to choose from, edit the _security groups_ to pick the existing one that is already exposing port 80 (goals--xxx), make sure auto-assign public ip is enabled and under _Load balancing_ choose _Application Load Balancer_, and select it by its name, goals-react-lb. Pick the `goals-react:80:80` container and click _Add to load balancer_, add it to the target group we created, react-tg, finally click _Next step_. Ignore auto scaling and click _Next step_ again, finally click _Create Service_ and then _View Service_. This starts our fronent ap in a new service to run alongside our backend service, use the load balancer (goals-react-lb) url to check the app

In then end we still have the same environment in dev and prod, but with the react app we need a different `Dockerfile` because of the build process

## Kubernetes Introduction & Basics

Kubernetes (K8s) is an open-source system for automating deployment, scaling & Load Balancing, and management of containerized applications

Manual deployment of containers is hard to maintain, error-prone and annoying:

- Containers might crash / go down and need to be replaced
- We might need more container instances upon traffic spikes
- Incoming traffic should be distributed equally

Kubernetes simplifies the deployment and configuration of complex containerized applications and it helps with topics like scaling and load balancing

Services like AWS ECS also help with that (with container health checks, automatic re-deployment, autoscaling and Load balancer) but we have to follow the AWS-specific rules, so we are "locked in", if we want to switch to a different provider/host, we have to "translate" the deployment configs, etc.

With Kubernetes, we can set up a configuration which will work on any host that supports Kubernetes, no matter if it's a cloud provider or our own, Kubernetes-configured data center

Typically, we will write down Kubernetes configuration files which describe our target state with a couple of Kubernetes commands, and then we bring that state to life on our cluster

Kubernetes is not just a software you run on some machine, It’s a collection of concepts and tools  
Basically, Kubernetes is like Docker-Compose for multiple machines

A Kubernetes **Cluster** is required to run our Containers on, that cluster is simply a network of machines that are called "Nodes" in the Kubernetes world - and there are two kinds of **Nodes**:

- The Master Node, it hosts the "Control Plane", so it's the control center which manages our deployed resources. It hosts:
  - An API Server, responsible for communicating with the Worker Nodes kubelets
  - A Scheduler, responsible for managing the Containers, e.g. determine on which Node to launch a new Container
  - Kube-Controller-Manager, that watches and controls Worker Nodes, manages the correct number of Pods & more
  - Cloud-Controller-Manager, that works like the Kube-Controller-Manager but for a specific Cloud Provider (eg. AWS)
- Worker Nodes, machines where the actual containers are running on inside of Pods, with the following running "tools"/processes:

  - Kubelet service, the counterpart for the Master Node API Server, communicates with the" control plane"
  - Container runtime (e.g. Docker), used for actually running and controlling the containers
  - Kube-proxy service, responsible for container network (and cluster) communication and access

  With Kubernetes, we don't manage containers but rather **Pods** which then manage the containers, a pod contains one or more container (typically one though) and any configuration as well as volumes required by the container/s

Finally, We also have _Services_, a logical set of Pods with a unique, Pod and Container independent IP address

nb. If we create our own Kubernetes Cluster from scratch, we need to create all these machines and then install the Kubernetes software on those machines, manage permissions etc. Once the Cluster is up and running, Kubernetes will create, run, stop and manage Containers for us

### Kubernetes Core Concepts

#### Installation

What Kubernetes Will Do:

- Create your objects (e.g. Pods) and manage them
- Monitor Pods and re-create them, scale Pods etc.
- Kubernetes utilizes the provided (cloud) resources to apply our configuration/goals

What we need to do for Kubernetes to work:

- Create the cluster and the node instances (worker + master nodes)
- Setup API Server, kubelet and other Kubernetes services / software on Nodes
- Create other (cloud) provider resources that might be needed (e.g. Load Balancer, Filesystems)

(We can use tools like [kubermatic](https://www.kubermatic.com/) or a managed service like AWS EKS though)

On our local machine, we need the `kubectl` command line tool used to send instructions to the cluster

For testing purpose, we'll setup everything on our local machine, ie. the cluster too. We'll use [Minikube](https://minikube.sigs.k8s.io/docs/start/) for that, it'll setup a VM on our machine to simulate a single-node Cluster - the Master and Worker nodes combined in one single VM

Since we're using Docker Desktop with WSL2, we can also simply enable kubernetes there; see more info [here](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) and [here](https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/)  
To add the K8s dashoard, run `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc6/aio/deploy/recommended.yaml`.  
Check the resources it created with `kubectl get all -n kubernetes-dashboard`. ClusterIP are internal network address, we cannot reach it if we type the URL in our Windows browser, we need to create a temporary proxy `kubectl proxy`.  
Now, go to http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/  
To get a token to connect, run `kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')` and copy then paste the token.

#### Kubernetes Objects (Resources)

Kubernetes works with Objects, and these objects can be created in two ways, Imperatively or Declaratively. We'll start with the imperative approach

- The “Pod” Object is the smallest “unit” Kubernetes interacts with

  - Contains and runs one or multiple containers (most common is one container per Pod)
  - Pods contain shared resources (eg. volumes) for all Pod containers
  - Has a cluster-internal IP by default (Containers inside a Pod can communicate via localhost)

  Pods are designed to be ephemeral: Kubernetes will start, stop and replace them as needed  
  For Pods to be managed for us, we need a “Controller” (eg. a “Deployment”)
  nb: pods are somewhat similar to a task in AWS ECS

- The “Deployment” Object controls (multiple) Pods

  - We set a desired state, Kubernetes then changes the actual state (It defines which Pods and containers to run and the number of instances)
  - Deployments can be paused, deleted and rolled back
  - Deployments can be scaled dynamically and automatically (We can change the number of desired Pods as needed)

  Deployments manage a Pod for us, we can also create multiple Deployments  
  We therefore don’t directly control Pods, instead we use Deployments to set up the desired end state

- The “Service” Object exposes Pods to the Cluster or Externally

  - Pods have an internal IP by default – it changes when a Pod is replaced
  - Services group Pods with a shared IP
  - Services can allow external access to Pods

  Without Services, Pods are very hard to reach and communication is difficult  
  Reaching a Pod from outside the Cluster is not possible at all without Services

#### First Deployment - Using the Imperative Approach

We'll work with `12-kub-first-app` for this section

First, we build an image, `docker build -t kub-first-app .`

Second, we push this image to dockerhub, create a new repo named kub-first-app, then retag `docker tag kub-first-app kevinlabtani/kub-first-app` and push our local image `docker push kevinlabtani/kub-first-app`

Now we want to "send" the image to K8s as part of a deployment that will then create the pod and the container running in the pod for us, and manage it for us

The cluster should be already running though Docker Desktop

Let's send the instruction to our cluster to create a new deployment object, `kubectl create deployment first-app --image=kevinlabtani/kub-first-app`

We can't reach our app right now though, but we can check the K8s dashboard (see Installation section for how) to see that our app is indeed running without issues

`kubectl get deployments` to get all the deployments  
`kubectl get pods` to get all the pods  
`kubectl delete deployment DEPL_NAME` to delete a deployment

We now want to create a service to expose our pods, `kubectl expose deployment first-app --type=LoadBalancer --port=8080`.

- `--type=ClusterIP` is the default type that would make the deployment reachable only form inside the cluster but with an addresss that doesn't change
- `--type=NodePort` will expose the deployment with the ip address of the worker node
- `--type=LoadBalancer` utilises a load balancer to generate an unique address for this service and evenly distribute incoming traffic accross all pods that are part of this service

`kubectl get services` will list our services (the kubernetes service was created automatically)

| NAME       | TYPE         | CLUSTER-IP    | EXTERNAL-IP | PORT(S)        | AGE  |
| ---------- | ------------ | ------------- | ----------- | -------------- | ---- |
| first-app  | LoadBalancer | 10.107.53.227 | localhost   | 8080:32331/TCP | 12s  |
| kubernetes | ClusterIP    | 10.96.0.1     | <none>      | 443/TCP        | 143m |

We can now go to http://localhost:8080/ to see our app being served

#### Restarting Containers, Scaling, Updating Deployments, Rollbacks & Deleting

We can crash our app by visiting `/error`, but if we go back to the root of our app, we see it still works! If we run `kubectl get pods` we see the status is _running_, but thats because it was restarted once. If we crash it again, the restart time takes longer - K8s takes longer an longer to rerun crashing pods to prevent infinite loops - but eventually it'sll get running again. We get that behavior because we created a deployment so the pod is being monitored and restarted if it fails

If we don't have autoscaling in place, we can manually scale with `kubectl scale deployment/first-app --replicas=3`, a replica is an instance of a pod. Now we see 3 pods if we run `kubectl get pods` (we'll scale it back to 1, though)

If we update our source code, we need to rebuild `docker build -t kevinlabtani/kub-first-app:2 .` - K8s will only download new images if they have a different tag, so we're using one here - and repush `docker push kevinlabtani/kub-first-app:2`  
nb: If we specify the `latest`tag, `kub-first-app:latest`, then K8s will always pull the Image when building  
Now to update a deployment, we run `kubectl set image deployment/first-app kub-first-app=kevinlabtani/kub-first-app:2`  
`kubectl rollout status deployment/first-app` will give us the current updating status

If we mess up and run `kubectl set image deployment/first-app kub-first-app=kevinlabtani/kub-first-app:3` trying to update to an image that doesn't exist, we can rollback the deployment update with `kubectl rollout undo deployment/first-app`

`kubectl rollout history deployment/first-app` will get a list of all the deployments we made, add the `--revision=REV_ID` to get more info about a specific deployment. If we wanted to go back to a specific deployment, we can run `kubectl rollout undo deployment/first-app --to-revision=REV_ID`

We can delete a service with `kubectl delete service first-app`
We can delete a deployment with `kubectl delete deployment first-app`

#### The Declarative Approach

We'll work with `13-kub-first-app-declarative` for this section

Imperative

- `kubectl create deployment ...`
- Individual commands are executed to trigger certain Kubernetes actions
- Comparable to using `docker run` only

Declarative

- `kubectl apply –f config.yaml`
- A config file is defined and applied to change the desired state
- Comparable to using `docker-compose` with compose files

We'll create a `deployment.yaml` (the name is up to us), see the [docs](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#deployment-v1-apps). The `kind` key specifies what kind of K8s object we want to create. The `metadata` key specifies things like app name.

The `spec` key defines the specifications for this deployment. The `template` key defines the pods to be created by this deployment

The `selector` key defines which pods match this deployment as deployments are dynamic objects, it tells K8s which pods this deployment should control. we can `matchLabels` or `matchExpressions`. nb: the labels keys/values are up to us, we have two as an example btw

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: second-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: second-app
      tier: backend
  template:
    metadata:
      labels:
        app: second-app
        tier: backend
    spec:
      containers:
        - name: second-node
          image: kevinlabtani/kub-first-app:2
        # - name: ...
        #   image: ...
```

`kubectl apply -f=deployment.yaml` will apply the config to the connected cluster, we can now see our deployments and pods running, `kubectl get deployments`; The app is not reachable yet though, we need to add a service object

We now need to create a `service.yaml`  
nb:the syntax for the selector is different because the Service ressource is an older spec, and it can olny match labels. We also don't need to select by all the labels, here we only need one. Type can be `ClusterIP`, `NodePort`, `LoadBalancer`, as we've seen before. `port` is the port we want to expose the app on, `targetPort` is the port inside the container

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: second-app
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 8080
    # - protocol: 'TCP'
    #   port: 443
    #   targetPort: 443
  type: LoadBalancer
```

`kubectl apply -f=service.yaml` will apply the config to the connected cluster, the app is now served to the ourside world, `kubectl get services`

To make changes (update) to our resources, we can just change the `.yaml` file and apply it again, eg. change the image source in the `deployment.yaml` and then apply the `.yaml` again. (We still need to build then push the image to docker hub, of course)

To delete resources, run `kubectl delete -f=deployment.yaml -f=service.yaml`, we could still use the imperative way though, `kubectl delete deployment second-app`

If we want, we can merge `.yaml` files with all the related resources in it, separating the reources with 3 dashes `---`. Here we'll create `master-deploy.yaml`. nb: it's good practice to put services first

#### More on Selectors

We'll work with `13-kub-first-app-declarative-2` for this section

`matchExpressions` is a more powerful way of selecting things, as we can use operators `In`, `NotIn`, `Exists` and `DoesNotExist`

eg, in our `deployment.yaml`, select all pods where `app` label key has a value of `second-app` or `first-app`

```yaml
matchExpressions:
  - { key: app, operator: In, values: [second-app, first-app] }
```

We can also use selectors with the imperative approach, the `-l` flag allow us to select objects by their label, eg. (we've added labels in the two yaml files first) `kubectl delete deployments, services -l group=example` will tell K8s to delete the deployments and services with the label `group: example`

#### Liveness Probes & Extra Config Options

A liveness probe is used by K8s to check if a pod & container in pods are healthy or not

In our `deployment.yaml`, we can add a `livenessProbe` key to override the default K8s behavior,

We'll also add an extra option, `imagePullPolicy` because we want K8s to always pull images when we update the image, even if no image tag is specified

```yaml
[...]
      containers:
        - name: second-node
          image: kevinlabtani/kub-first-app:2
          imagePullPolicy: Always
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            periodSeconds: 10
            initialDelaySeconds: 5
```

### Kubernetes Data & Volumes

We'll work with `14-kub-data` for this section, an API with a POST and a GET request handler, the POST route allow us to create a `.txt` file saved to the hdd

Run the app locally with `docker-compose up -d --build`; we can then use Postman requests to `localhost/story` to interact with our API. We're using a named volume, so if we take our container down and recreate a new one, the data is still saved

Let's now see how we work with K8s and volumes; we need to define "state" first

State is data created and used by your application which must not be lost; We have 2 kind of states:

- User-generated data, user accounts, …: Often stored in a database, but could also be files (e.g.uploads)
- Intermediate results derived by the app; Often stored in memory, temporary database tables or files

##### Getting our Demo App Running on K8s Cluster

We'll build `docker build -t kevinlabtani/kub-data:latest .` and push `docker push kevinlabtani/kub-data:latest` our local app

We'll create `.yaml` files for the deployment and services object

`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: kevinlabtani/kub-data:latest
```

`service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: story-service
spec:
  selector:
    app: story
  type: LoadBalancer
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
```

We now apply our service and deployment, `kubectl apply -f=service.yaml -f=deployment.yaml`, and test that GET and POST Postman requests to `localhost/story` are working correctly

#### Kubernetes Volumes

Right now we're not using volumes, so the data will be lost if the container restarts

Kubernetes can mount Volumes into Containers  
A broad variety of Volume types / drivers are supported such as ”Local” Volumes (i.e. on Nodes) and Cloud-provider specific Volumes  
Volume lifetime depends on the Pod lifetime. Volumes survive Container restarts (and removal), but Volumes are removed when Pods are destroyed ie. Volumes are not necessarily persistent

##### "emptyDir" Volume Type

Right now if we go to `/error` we'll crash the app and K8s will relaunch a container and all our data is gone

We have to define volumes in the place where we define and configure our pods (as volumes are attached to pods), in the `deployment.yaml` in our case, and all the containers in a pod wil lbe able to use the volume

```yaml
[...]
      containers:
        - name: story
          image: kevinlabtani/kub-data:latest
          imagePullPolicy: Always
          volumeMounts:
            - mountPath: /app/story
              name: story-volume
      volumes:
        - name: story-volume
          emptyDir: {}
```

An [emptyDir volume](https://kubernetes.io/docs/concepts/storage/volumes/) is first created when a Pod is assigned to a node, and exists as long as that Pod is running on that node. Containers can then write to that directory

Now if we create some data with a POST request to `/story`, then crash the app with a GET request to `/error`, after the container has restarted, we can still get the data with a GET request to `/story` (initially we can't get story at the app tries to read from a file that isn't there)

##### "hostPath" Volume Type

If we have multiple (say, 2) replicas, when we crash one pod, traffic is redirected to the other one, and the data isn't available anymore until the first pod is restarted

One way of working around that is to use a hostPath volume. A hostPath volume mounts a file or directory on the host's Node filesystem into our pod. This way multiple pods can share the same hostPath volume. nb: it won't solve the problem of having multiple nodes

`path` is the path on the host machine  
`type: DirectoryOrCreate` will create the directory specified under `path` if it doesn't exist

```yaml
[...]
spec:
  replicas: 2
[...]
      containers:
        - name: story
          image: kevinlabtani/kub-data:latest
          imagePullPolicy: Always
          volumeMounts:
            - mountPath: /app/story
              name: story-volume
      volumes:
        - name: story-volume
          hostPath:
            path: /data
            type: DirectoryOrCreate
```

After applying our deployment, we see that the existing pods are getting terminated and 2 new ones are recreated. Now if we crash one pod, we still get the saved data from the other pod

#### "CSI" Volume Type

The Container Storage Interface volume type defines a standard interface for container orchestration systems (like Kubernetes) and anyone can build driver solutions utilizing this interface, eg. there's an AWS EFS CSI driver

#### Persistent Volumes

All the volumes we've seen so far Volumes are destroyed when a is removed

hostPath partially works around that in “OneNode” environments, but it won't work in a real cluster on eg. AWS

Pod- and Node-independent Volumes are sometimes required

Persistent Volumes are inside the cluster and detached from the pod & node, instead we have Persistent Volume claims that are part of pods that reach out to the Persistent Volume.

”Normal” Volumes

- Volume is attached to Pod and Pod lifecycle
- Defined and created together with Pod
- Repetitive and hard to administer on a global level

Persistent Volumes

- Volume is a standalone Cluster resource (NOT attached to a Pod)
- Created standalone, claimed via a PVC
- Can be defined once and used multiple times

#### Defining a Persistent Volume

We'll work with `15-kub-data-persistent` for this section

We'll write a `host-pv.yaml` to create a persistent volume

We'll use the hostPath type, this only work on a single node testing environment like we have there, but what we'll learn will also hold true later

The `capacity` is the overall available capacity for the persisent volume, `1Gi` is 1 gigabyte

`VolumeMode` is either `Filesystem` or `Block`, see [here](https://www.computerweekly.com/feature/Storage-pros-and-cons-Block-vs-file-vs-object-storage) for ref

`accessModes` is `ReadWriteMany`:can be mounted as a r/w volume by many node, `ReadOnlyMany`:read only by multiple nodes and `ReadWriteOnce`:can be mounted as a r/w volume by a single node (multiple pods ok as long as there's on the same node). We'll pick `ReadWriteOnce` as that's the only one available with the hostPath type

The storage class defines to K8s defines how the storage should be provisionned, we'l use the default one `storageClassName: standard`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

#### Creating a Persistent Volume Claim

In order to use the Persistent Volume, the pods need to define a Persistent Volume Claim

We'll write a `host-pvc.yaml` to create a persistent volume claim

nb: we could write everything in one single `.yaml` file

In `spec`, we could claim volume by ressources rather than by name

The `resources` key specifies which resource we want to get with this claim, we can pick eg. how much storage we want to request

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-pvc
spec:
  volumeName: host-pv
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
```

Now we need to connect the claim to a pod, in `deployment.yaml`, through the `volumes` key

```yaml
volumes:
  - name: story-volume
    persistentVolumeClaim:
      claimName: host-pvc
```

#### Using a Claim in a Pod

Apply the new & updated yaml files with `kubectl apply -f=host-pv.yaml -f=host-pvc.yaml -f=deployment.yaml`

`kubectl get pv` to get a list of all persistent volumes
`kubectl get pvc` to get a list of all persistent volumes claims

The biggest difference is that the volume is now independent from the pod AND the node, and not just with hostPath but with any supported type for persistent volumes

If we go back to our definition of state, we should note that for intermediate results, pod specific volumes might be enough, we might not need a persistent volume

#### Environment Variables & ConfigMaps

We'll work with `16-kub-data-env` for this section. The same as before, but we're replacing the `story` folder with an env variable

We'll need to rebuild the image and push it

In the `deployment.yaml` we'll add an `env` key to the containers definition

```yaml
containers:
  - name: story
    image: kevinlabtani/kub-data:latest
    imagePullPolicy: Always
    env:
      - name: STORY_FOLDER
        value: "story"
    volumeMounts:
      - mountPath: /app/story
        name: story-volume
```

If we don't want to setup our env variables in the container specs, we can use a separate `ConfigMap` ressource. We'll create an `environment.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: data-store-env
data:
  folder: "story"
  # key: value..
```

Then, we apply it, `kubectl apply -f=environment.yaml`

`kubectl get configmap` will list all oru configmaps

Now we want to utilise this configmap, in the `deployment.yaml`

```yaml
containers:
  - name: story
    image: kevinlabtani/kub-data:latest
    imagePullPolicy: Always
    env:
      - name: STORY_FOLDER
        valueFrom:
          configMapKeyRef:
            name: data-store-env
            key: folder
    volumeMounts:
      - mountPath: /app/story
        name: story-volume
```

Then, we need to reapply the `deployment.yaml`

### Kubernetes Networking

We'll work with `17-kub-network` for this section. This application consists in 3 different backend API that will work together, an auth API, an users API to create/login users (in dummy mode, ie. we're nto storing data) and a tasks API (tasks will be stored in a file) that gets a token which identifies a logged in user and reach ou tto the auth API to verify this user

The Auth API and Users API are in the same pod and are using Pod--internal comunication  
The Tasks API run in an other pod  
Both pods are part of the cluser and both the users & tasks API are reachable by the client (eg. Postman). The auth API isn't directly reachable by the client

Eventually, we'll want to run the Auth API in its own separate pod

#### Creating a First Deployment

We'll start with just the users API, `docker build -t kevinlabtani/kub-demo-users .` then `docker push kevinlabtani/kub-demo-users`

We'll create kubernetes folder for all the `.yaml files`

`kubectl apply -f=users-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
        - name: users
          image: kevinlabtani/kub-demo-users
```

`kubectl apply -f=users-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

We can test the API by sending a POST request to localhost:8080/login and localhost:8080/singup, we'll get back a token

We'll now add the auth API to the same pod as the users API. First, we'll replace the dummy code in the `users-app.js` with real calls to the auth API (line 26 & 58), using env varaible for the domain name, we'll also update the `docker-compose.yaml` file to make sure that it still works. We'll then rebuild and repush the users API image

Then we'll build and push the auth API image `docker build -t kevinlabtani/kub-demo-auth .`  
We'll use the same deployment file as we want it in the same pod. We're not changing the service yaml file as we don't wish to expose the auth API to the outside wold.

For pod internal communication, we can just send the request through the localhost address

`kubectl apply -f=users-deployment.yaml`

```yaml
[...]
containers:
  - name: users
    image: kevinlabtani/kub-demo-users:latest
    env:
      - name: AUTH_ADDRESS
        value: localhost
  - name: auth
    image: kevinlabtani/kub-demo-auth:latest
```

`kubectl get pods` shows us we do indeed have 2 container running, and testing with Postman confirms that our routes are working

#### Creating Multiple Deployments

We also want to deploy the Tasks API in a separate pod and make sure we can send request there too. We also need to make sure the tasks API is able to communicate with the auth API even though it's in a different pod

First, we need a new deployment for the auth API so that the auth and user containers no longer run in the same pod.

`auth-deployment.yaml`  
we've also removed the auth container from `users-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: kevinlabtani/kub-demo-auth:latest
```

This new pod will reachable only inside the cluster. The problem is that the IP address for that pod could change if it's scaled or it has crashed and restarted, so we do need to create a `auth-service.yaml` for it

```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

`kubectl apply -f=auth-service.yaml -f=auth-deployment.yaml`

##### Pod-to-Pod Communication with IP Addresses & Environment Variables

If we run `kubectl get services`, we can get the ip for the auth-service, (again, that IP is only reachable inside the cluster) and use it as AUTH_ADDRESS inside the `users-deployment.yaml`, then reapply it

```yaml
[...]
containers:
  - name: users
    image: kevinlabtani/kub-demo-users:latest
    env:
      - name: AUTH_ADDRESS
        value: "10.98.68.222"
```

` kubectl get pods` show we now have two pods running, and we can test our APIs (localhost:8080/login, localhost:8080/signup) with Postman and see it's still working

While the ip address is stable and won't change all the time, getting the ip address is a bit anoying and there's a more convenient way; K8s provides us with auto-generated env variables with info about the services that are running in the cluser

`AUTH_SERVICE_SERVICE_HOST` is the env variable we need, with the ip address auto assigned for the auth service. We'll need to modify the code in `users-app.js` again to use that env variable name (ie. `process.env.AUTH_SERVICE_SERVICE_HOST`), though, and then rebuild, repush and redeploy. We'll also remove the env variable from the `users-deployment.yaml` since it's no longer needed

```yaml
[...]
containers:
  - name: users
    image: kevinlabtani/kub-demo-users:latest
```

Again, we can test our APIs (localhost:8080/login, localhost:8080/signup) with Postman and see it's still working

##### Using DNS for Pod-to-Pod Communication

There's a more convenient way that using env variables for pod to pod communication, because K8s clusters by default come with a build-in service called CoreDNS that can be used to create domains that are known inside of the cluster

The pattern for these domains is quite simple, if we want to send a request to the auth service named "auth-service.NAMESPACE_NAME", we can just use that name as a domain to send request to; it's quite similar to docker-compose

` kubectl get namespace` will list the namespaces, the default namespace is the one our services and deployments are assigned if we do'nt specify one

We'll go back and remodify our code in `users-app.js` to use `AUTH_ADDRESS` as an env variable name, rebuild, repush and add it back to `users-deployment.yaml` with the right value and then reapply it

```yaml
[...]
containers:
  - name: users
    image: kevinlabtani/kub-demo-users:latest
    env:
      - name: AUTH_ADDRESS
        value: "auth-service.default"
```

Again, we can test our APIs (localhost:8080/login, localhost:8080/signup) with Postman and see it's still working

#### Adding our Tasks API

We now will add our Tasks api, it also send a request to the auth api, we'll need to modify the address we send a request to though, using an env variable. We'll be using our own env variable `AUTH_ADDRESS` again, rahter than the one auto-generated by K8s. We'll also update the `docker-compose.yaml` file to make sure that it still works

Build and push the tasks API image, `docker build -t kevinlabtani/kub-demo-tasks .` & `docker push kevinlabtani/kub-demo-tasks`

`kubectl apply -f=tasks-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tasks-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tasks
  template:
    metadata:
      labels:
        app: tasks
    spec:
      containers:
        - name: tasks
          image: kevinlabtani/kub-demo-tasks:latest
          env:
            - name: AUTH_ADDRESS
              value: "auth-service.default"
            - name: TASKS_FOLDER
              value: tasks
          volumeMounts:
            - mountPath: /app/tasks
              name: tasks-volume
      volumes:
        - name: tasks-volume
          emptyDir: {}
```

`kubectl apply -f=tasks-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tasks-service
spec:
  selector:
    app: tasks
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
```

We can now test the task API with Postman, GET to localhost:8000/tasks, add a header "Authorization" "Bearer abc"; we'll get `Loading the tasks failed.` initially because there are no tasks, add one with a POST request to the same address and then the GET route will work

#### Adding a Containerized Frontend

We'll work with `18-kub-network-fe` for this section, we've now added a react frontend to the app.

In the `App.js` file we're fetching from the backend in two places. The domain we're fetching from is http://localhost:8000/tasks, we get it by doing a `kubectl get services`. We also need ot add the beared token with the request

We have a multistage setup for the frontend, build the app then use nginx to serve it, `docker build -t kevinlabtani/kub-demo-frontend .`

Let's test it locally without K8s to test it, `docker run -p 80:80 --rm -d kevinlabtani/kub-demo-frontend`. It works but we have a CORS error when we try to fetch from the backend. We need to update the tasks api `task-app.js` file and set the right headers

```js
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Methods", "POST,GET,OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type,Authorization");
  next();
});
```

The we rebuild, repush the tasks image and then delete and reapply the task deployment; it now works alright.(we do need at least one task in the backend though since it's a very basic frontend app)

We now want to serve this frontend by our cluster, in a new pod (and a new deployment)

`frontend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: kevinlabtani/kub-demo-frontend:latest
```

`frontend-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

`kubectl apply -f=frontend-deployment.yaml -f=frontend-service.yaml`

We can now reach our frontend at http://localhost:80

We now want to add a reverse proxy for the frontend so that we don't have to hardcode the address for the fetch in our frontend `App.js`

We want to send the request to ourselves, that is, the nginx server that serves our react frontend, we can modify `nginx.conf` to both serve our frontend but also redirect requests to a specific domain if they target a specific path

old `nginx.conf`

```conf
server {
  listen 80;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html =404;
  }

  include /etc/nginx/extra-conf.d/*.conf;
}
```

new `nginx.conf`

```conf
server {
  listen 80;

  location /api/ {
    proxy_pass http://tasks-service.default:8000/;
  }

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html =404;
  }

  include /etc/nginx/extra-conf.d/*.conf;
}
```

In `nginx.conf` we can use `http://tasks-service.default:8000/` for the `proxy_pass`. The `nginx.conf` is executed on our cluster, so we can't use `http://localhost:8000` anymore, so we're using cluster internal ip addresses, or auto generated domain name  
In `App.js` we can now send the fetch requests to `/api/tasks`. Then rebuild, repush and reapply. We can see our app now works without errors.

### Deploying a Kubernetes Cluster

We need to setup the infrastructure for K8s to run on, we need to replace minikube or Docker Desktop we've used so far

Our deployment options are

- Custom Data Center & Install + configure everything on our own
- Cloud Provider & Install + configure most things on our own (eg. EC2 + [kOps](https://github.com/kubernetes/kops))
- Cloud Provider & Install + Use a managed service (then we just have to define our cluster architecture)

We'll be using AWS EKS (Elastic Kubernetes Service), the advantages over AWS ECS that we used before are:

- Managed service for Kubernetes deployments
- No AWS-specific syntax or philosophy required
- Use standard Kubernetes configurations and resources

#### Preparing the Starting Project

We'll work with `19-kub-deploy` for this section. We have an users and an auth API, The users API communicates with the auth API and the auth API isn't reachable from the outside

We already have `auth.yaml` and `users.yaml`, both combine a deployment and a service in a single `.yaml` file

Note that for the users API Deployment we have an env variable for the AUTH_API_ADDRESS much like we had in the last section. We also have an env variable for mongoDB as we're using mongoDB to store our data

We of course have to build and push both our images, `docker build -t kevinlabtani/kub-dep-users .`, `docker build -t kevinlabtani/kub-dep-auth .`, etc.

This time we won't apply the `.yaml` locally to our cluster, we'll directly dive into AWS EKS

#### Creating & Configuring the Kubernetes Cluster with EKS

Go to the [AWS Management Console](https://console.aws.amazon.com/console/) and search EKS

Pick a cluster name - kub-dep-demo - and click _Next step_, we're then taken to a page where we can configure our cluster

step1 Configure cluster:  
Pick the _Cluster Service Role_ we're creating next paragraph; EKS behind the scenes will create some EC2 instances, EKS needs permissions to do that, and permissions are managed through the IAM console, click on it.

Click _create role_ in the IAM console, pick AWS service, scroll down to EKS, click on it and choose _EKS - Cluster_ (a predifine role with all the neede dpermissions already assigned). Click _Next:Permissions_ then _Next:Tags_, then _Next:Review_, then name the role eksClusterRole and click _create role_

step2 Networking:  
Click on services in the navbar, search _CloudFormation_ and open it in a new tab

Click _create stack_, leave the default and paste in the url from [this page](https://docs.aws.amazon.com/eks/latest/userguide/create-public-private-vpc.html#create-vpc) as _S3 URL_. Click _Next_

Pick a name - eksVpc - and leave all defaults, click _Next_

At the end, click _Create stack_; this will create a VCP network for us; close this page

Go back to the _Specify networking_ page and pich the VPC we just created

For _Cluster endpoint access_, pick _Public and private_, then click _Next_

step3:  
click _Next_

step4:  
click _create_

We now have our cluster created

We need to modify `kubectl` to be able to communicate with our EKS cluster. The config file created by docker-desktop is located at `C:\Users\kevin\.kube\config`, it's best to save a copy of the existing config file and rename it. We'll use the aws cli to do that so we need to [install it](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)

On AWS click on our account nameang go to _My Security Credentials_. Pick _Access Key_ and _Create New Access Key_, download the key file and find an AWSAccessKeyID and a AWSSecretKey. Run `aws configure` and enter our credentials

Now, once the EKS cluster is active, run `aws eks --region us-east-1 update-kubeconfig --name kub-dep-demo`, this will update our .kube config file with everything it needs for `kubectl` to communicate with our AWS EKS cluster

Now, eg. `kubectl get pods` will communicate with our AWS EKS cluster

#### Adding Worker Nodes

Go to the _Compute_ tab and click _Add Node Group_

step1:  
Name it demo-dep-nodes, and add the IAM role eksNodeGroup we're creating in the next paragraph through the IAM console, leave eveything else default and click next

The nodes (EC2 instances) wil need their own specific permissions. Click _create role_ in the IAM console, pick AWS service, scroll down to EC2, click on it and then click _Next:Permissions_, search for EKS and add AmazonEKSWorkerNodePolicy, search for cni and add AMAZONEKS*CNI_Policy, search for ec2container and add AMAZONEC2ContainerRegistryReadOnly. Then click on \_Next:Tags*, then _Next:Review_, make sure the 3 policies are there then name the role eksNodeGroup and click _create role_

step2:  
Pick the kind of EC2 instances that will be launched and managed by EKS. Keep the default AMI type and pick a cheap instance type, t3.small (don't use the micro instance, it might cause errors)

Pick our scaling policy, we'll keep with 2, the default. Click _next_

step3:  
we can dissallow remote access to nodes (means we can't connect through ssh to the nodes), click _next_

step4:  
click _create_, this will spin up a couple EC2 instances and add them to our cluster, it will also install all the K8s software required on these nodes and add them to the cluster network. That's the part we would have to do manually otherwise

On the EC2 service pages, we can see our running EC2 instances, we also see that at the moment there's no load balancer

#### Applying Our Kubernetes Config

As we've seen before, `kubectl` is now linked to our AWS EKS custer, so we can just run `kubectl apply -f=auth.yaml -f=users.yaml`

`kubectl get services` lists our two services, we can also see an EXTERNAL-IP associated with the users-service than we can use to send requests to with Postman

On the EC2 service pages, we can see we now have a load balancer that was creaued automatically by AWS because we depload the services

If we make a change, we can just reapply the `.yaml` files

reminder: pods are the thing that contains containers and run on nodes, they will be distributed by K8s accross the available nodes, 2 in this cases. Nodes are not started with replicas, nodes and configured with node groups in the EKS console

nb: cluster internal communication (ie. using `auth-service.default`) works just fine even on EKS! `kubectl get namespaces` show we have a default namespace created by EKS

#### Getting Started with Volumes

We'll now modify the app so that the users API writes to a folder, now we need a volume to persist that data

We can add the volume directly, or we could setup a persistent volume ressource and create a persistent volume claim, we'll do the later

As we've seen, hostPath volumes aren't doable here, as we got more than one node, and a hostPath would only be created on one node and not the other, and we don't have control on which node the hostPath xwould be created, adn even if we did, different nodes would have different data!

We'll use the CSI volume type, specifically, AWS EFS CSI

#### Adding EFS as a Volume (with the CSI Volume Type)

Check the [github](https://github.com/kubernetes-sigs/aws-efs-csi-driver) for the driver. We need to run `kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.0"` to deploy the stable driver in our cluster

First, go to EC2 console and to _security groups_, click on _create security group_, name it eks-efs, pick the eksVpc we created. Add an _inbound rule_, pick _NFS_ with _Custom source_ and add the ip we get by going to the VPC service page and go to our eksVPC to get the _IPv4 CIDR_. Click _Create security group_

We now need to create an EFS, search for it in the AWS console and click _Create file system_, name it eks-efs, pick the eksVps as VPC then click on _Customize_. On step2 - Network access - remove the security groups and pick our newly created eks-efs, then click through to _Create_. Copy and save the _File system ID_

#### Creating a Persistent Volume for EFS

We need to add a persistent volume to the `users.yaml`. We also need to add an empty users folder to our users api so that we can write to it. `volumeHandle` is the _File system ID_. `storageClassName: efs-sc` also need to be added as it's a custom one that doesn't exist yet (`kubectl get sc` to list all storage classes)

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-59d14521
```

We then need to claim this volume, in the deployment ressource:

```yaml
volumes:
  - name: efs-vol
    persistentVolumeClaim:
      claimName: efs-pvc
```

And add the persistent volume claim itself

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
---

```

#### Using the EFS Volume

We need to rebuild, and push the users API image since we modified it.  
We then need to delete the users deployment `kubectl delete deployment users-deployment` and reapply `kubectl apply -f=users.yaml`

We can now test the app with Postman

In the EFS console, click on the file system we created and go to the _Monitoring_ tab, we'll see some write access since we added some data with Postman, we also see 3 client connections (since we have 3 replicas of the pod)

If we shutdown all pods by setting the nr of replicas to 0, and then rechange it to 3 to create new pods, we can see that our log data persists since our volume works.
