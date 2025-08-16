# Network-Multitool-Base (Fork by EricWasTakenOnDocker)

A compact, multi-arch Docker image packed with common network and container troubleshooting tools. It includes an 
nginx web server to keep the container running and provide a simple landing page with the container's IP and the 
active HTTP/HTTPS ports. Ideal for testing connectivity, DNS, routing, HTTP(S) endpoints, and more.

## Docker Hub

Image: https://hub.docker.com/r/ericwastakenondocker/network-multitool-base

Pull:

```sh
docker pull ericwastakenondocker/network-multitool-base
```

## Tools Included
- apk package manager
- Nginx Web Server (HTTP and HTTPS)
- awk, cut, diff, find, grep, sed, vi, wc
- curl, wget
- dig, nslookup (bind-tools)
- ip, ifconfig, route (iproute2, net-tools)
- traceroute, tracepath, mtr, tcptraceroute
- ping, arp, arping (iputils, busybox-extras)
- ps, netstat (procps, net-tools)
- gzip, cpio, tar
- telnet client (perl-net-telnet)
- tcpdump
- jq
- bash

Size: ~16 MB compressed, ~38 MB uncompressed

Note: SSL certificates are self-signed for "localhost" and placed in `/certs/`. For testing HTTPS with curl, use `-k` 
to ignore certificate warnings.

## How to Run

Basic run (Docker):
```sh
docker run -d --name network-multitool-base \
  -p 80:80 -p 443:443 \
  ericwastakenondocker/network-multitool-base
```

Exec into the container:
```sh
docker exec -it network-multitool-base /bin/bash
```

### Custom HTTP/HTTPS Ports
You can override the default ports 80 (HTTP) and 443 (HTTPS) using environment variables `HTTP_PORT` and `HTTPS_PORT`. 
This is useful when joining another container's namespace, using host networking, or avoiding port conflicts.

Example with custom ports:
```sh
docker run -d --name network-multitool-base \
  -e HTTP_PORT=1180 -e HTTPS_PORT=11443 \
  -p 1180:1180 -p 11443:11443 \
  ericwastakenondocker/network-multitool-base
```

### Host Network Example
```sh
docker run --network host -d --name network-multitool-base \
  ericwastakenondocker/network-multitool-base
```
If ports 80/443 are busy on the host, set custom ports:
```sh
docker run --network host -d --name network-multitool-base \
  -e HTTP_PORT=1180 -e HTTPS_PORT=11443 \
  ericwastakenondocker/network-multitool-base
```

When the container starts, the entrypoint adjusts nginx to listen on the provided ports and writes a concise banner 
to `/usr/share/nginx/html/index.html` showing the hostname, IP, and ports.

## Usage Notes
- Designed for quick network diagnostics and container testing environments.
- Exposed ports: 80, 443 (and alternates 1180, 11443 are available for convenience).
- The webpage auto-updates on start with current IP and port values; mount `/usr/share/nginx/html` if you want to 
- provide your own content (the entrypoint will not overwrite a mounted index.html).

## GitHub
This is a maintenance fork of the original Network-MultiTool. For details and full documentation, see the repository README.

(Repo: https://github.com/ericwastaken/Network-MultiTool-Base)
