# Ericwastaken Network-Multitool (A maintenance FORK of https://github.com/wbitt/Network-MultiTool)

A (**multi-arch**) multitool for container/network testing and troubleshooting. The main docker image is based on Alpine 
Linux.

The container image contains lots of tools, as well as a `nginx` web server, which listens on port `80` and `443` by 
default. The web server helps to run this container-image in a straight-forward way, so you can simply `exec` into the 
container and use various tools.

## Attribution

This project is based on WBITT’s & PRAGMA's original work (MIT-licensed) and modified by Eric Soto in 2025, also 
under MIT, primarily to be able to rev-up the Alpine base under Docker. Because of this, references to custom builds and 
automated builds are removed. Automated GitHub Action builds/pushes are disabled and we will maintain Dockerhub manually 
as needed.

## Available on Docker Hub

The docker repository to pull this image is: [https://hub.docker.com/r/ericwastakenondocker/network-multitool-base](https://hub.docker.com/r/ericwastakenondocker/network-multitool-base)

Or:

```
docker pull ericwastakenondocker/network-multitool-base
```

### Supported platforms: 
* linux/386
* linux/amd64
* linux/arm/v7
* linux/arm64

## Variants / image tags:
* **latest**

## Build and publish (multi-platform + Docker Hub)
The repository includes helper scripts to build multi-platform images and publish them to Docker Hub.

Prerequisites:
- Docker with Buildx enabled (Docker Desktop on macOS/Windows includes this; on Linux install buildx and QEMU emulators)
- Logged into Docker Hub: `docker login`
- Configure docker-build-manifest.env with your values:
  - BUILDER_NAME: a local buildx builder name (e.g., network-multitool-base)
  - NAME: full image name, e.g., ericwastakenondocker/network-multitool-base
  - CURR_TAG: the version tag you want to build/push (e.g., 20250816)

Supported multi-platform targets used by the scripts: linux/amd64, linux/arm64

1) Build the image
- Interactive build (choose multi-platform when prompted):
  ```sh
  ./x-build.sh
  ```
  - Option 1 builds a multi-arch image using buildx
  - Option 2 builds only for the current machine’s architecture

2) Publish to Docker Hub
- Interactive publish (choose multi-platform when prompted):
  ```sh
  ./x-deploy-dockerhub.sh
  ```
  - Multi-platform (Option 1) will build and push both $CURR_TAG and latest in one step using buildx --push
  - Current platform (Option 2) will tag and push the locally built image for $CURR_TAG and latest

Notes:
- The env file docker-build-manifest.env is sourced by both scripts; adjust it before running.
- For repeatable releases, bump CURR_TAG and re-run the two scripts.
- If the buildx builder named in BUILDER_NAME does not exist, the scripts will create and bootstrap it.

## Tools included in "latest":
* apk package manager
* Nginx Web Server (port `80`, port `443`) - with customizable ports!
* awk, cut, diff, find, grep, sed, vi editor, wc
* curl, wget
* dig, nslookup
* ip, ifconfig, route
* traceroute, tracepath, mtr, tcptraceroute (for layer 4 packet tracing)
* ping, arp, arping
* ps, netstat
* gzip, cpio, tar
* telnet client
* tcpdump
* jq
* bash
* whatever else is part of Alpine (see the Dockerfile for the latest version)

**Size:** 16 MB compressed, 38 MB uncompressed

**Note:** The SSL certificates are generated for "localhost", are self signed, and placed in `/certs/` directory. During your testing, ignore the certificate warning/error. While using curl, you can use `-k` to ignore SSL certificate warnings/errors.

------

# How to use this image? 
## How to use this image in normal **container/pod network** ?

### Docker:
```
$ docker run -d --name network-multitool-base ericwastakenondocker/network-multitool-base
```

Then:

```
$ docker exec -it network-multitool-base /bin/bash
```

## How to use this image on **host network** ?

Sometimes you want to do testing using the **host network**.  This can be achieved by running the multitool using host networking. 

### Docker:
```
$ docker run --network host -d --name network-multitool-base ericwastakenondocker/network-multitool-base
```

**Note:** If port 80 and/or 443 are already busy on the host, then use pass the extra arguments to multitool, so it can listen on a different port, as shown below:

```
$ docker run --network host -e HTTP_PORT=1180 -e HTTPS_PORT=11443 -d --name network-multitool-base ericwastakenondocker/network-multitool-base
```

# Configurable HTTP and HTTPS ports:
There are times when one may want to join this (multitool) container to another container's IP namespace for 
troubleshooting, or on the host network. This is true for both Docker and Kubernetes platforms. During that time if 
the container in question is a web server (nginx, apache, etc), or a reverse-proxy (traefik, nginx, haproxy, etc), 
then network-multitool cannot join it in the same IP namespace on Docker, and similarly it cannot join the same pod on 
Kubernetes. This happens because network multitool also runs a web server on port 80 (and 443), and this results in 
port conflict on the same IP address. To help in this sort of troubleshooting, there are two environment variables 
**HTTP_PORT** and **HTTPS_PORT** , which you can use to provide the values of your choice instead of 80 and 443. When 
the container starts, it uses the values provided by you/user to listen for incoming connections. Below is an example:

```
$ docker run -e HTTP_PORT=1180 -e HTTPS_PORT=11443 \
    -p 1180:1180 -p 11443:11443 -d local/network-multitool
4636efd4660c2436b3089ab1a979e5ce3ae23055f9ca5dc9ffbab508f28dfa2a


$ docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                                             NAMES
4636efd4660c        local/network-multitool   "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds        80/tcp, 0.0.0.0:1180->1180/tcp, 443/tcp, 0.0.0.0:11443->11443/tcp   recursing_nobel
6e8b6ed8bfa6        nginx                     "nginx -g 'daemon of…"   56 minutes ago      Up 56 minutes       80/tcp                                                            nginx


$ curl http://localhost:1180
Praqma Network MultiTool (with NGINX) - 4636efd4660c - 172.17.0.3/16 - HTTP: 1180 , HTTPS: 11443


$ curl -k https://localhost:11443
Praqma Network MultiTool (with NGINX) - 4636efd4660c - 172.17.0.3/16 - HTTP: 1180 , HTTPS: 11443
```  

If these environment variables are absent/not-provided, the container will listen on normal/default ports 80 and 443.

------

# FAQs

## Why this multitool runs a web-server?
Well, normally, if a container does not run a daemon/service, then running it (the container) involves using 
*creative ways / hacks* to keep it alive. If you don't want to suddenly start browsing the internet for "those creative 
ways", then it is best to run a small web server in the container - as the default process. 

This helps you when you are using Docker. You simply execute:
```
$ docker run -d --name network-multitool-base ericwastakenondocker/network-multitool-base
```

The multitool container starts as web server - so it remains `UP`. Then, you simply connect to it using:
```
$ docker exec -it network-multitool-base /bin/sh 
```

This is why it is good to have a web-server in this tool. 

## I can't find a tool I need for my use-case?
We have tried to put in all the most commonly used tools, while keeping it small and practical. We can't have all the 
tools under the sun, otherwise it will end up as [something like this](https://www.amazon.ca/Wenger-16999-Swiss-Knife-Giant/dp/B001DZTJRQ).  

However, if you have a special need, for a special tool, for your special use-case, then I would recommend to simply 
build your own docker image using this one as base image, and expanding it with the tools you need.

## Why not use LetsEncrypt for SSL certificates instead of generating your own?
There is absolutely no need to use LetsEncrypt. This is a testing tool, and validity of SSL certificates does not 
matter.