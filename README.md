## Présentation du Projet

Projet créé le 12 Novembre 2020, dans le but de'apprendre à utiliser Docker.

Le projet est basé sur le cours de Docker de [Academind](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/)

- [Getting Started & Overview](#Getting-Started-&-Overview)
- [Foundation](#Foundation)

## Getting Started & Overview

### What is Docker?

Docker is a container technology, a tool for creating and managing containers

A container is a standardized unit of software, a package of code and dependencies to run that code. The same container always yields the exact same application and execution behavior! No matter where or by whom it might be executed.

### Why containers?

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

`docker run -p 3000:3000 508425c72` will run the image  
`-p 3000:3000` is used to publish the port 3000 on port 3000 so we can use `localhost:3000` to access the app, as by default there's no connection between container and host (nb: the termial will be locked by the running web server)

`docker ps` will list all running containers  
`docker stop NAME_OF_CONTAINER` will stop a container (container are assigned a random name)

## Foundation

### Images & Containers

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
