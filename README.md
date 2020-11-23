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

Using a `.env` file is better for security than including secure data directly into the `Dockerfile`. nb: otherwise, the values are "baked into the image" and everyone can read these values via `docker history IMAGE`

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

Trade-offs between control and responsibility might be worth it, if we have to do everything ourselves, we are responsible for security, etc

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
2. Configure security group to expose all required ports to WWW
3. Connect to instance (SSH), install Docker and run the container

## Kubernetes

Kubernetes Introduction & Basics  
Kubernetes: Data & Volumes  
Kubernetes: Networking  
Deploying a Kubernetes Cluster
