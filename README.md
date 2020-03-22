# Run VDJbase within docker

This configuration uses  Docker Compose to orchestrate two containers: one running vdjbase, and the other running nginx. The vdjbase container is built on two layers. 
The first, `vdjbase_base`, has all the Python and R libraries, including `vdjbaseviz`. It is slow to build, because many of the R libraries are built from source. The
second container, `vdjbase`, pulls the current source from bitbucket. It is quick to build, making it easy to pull down updates by rebuilding the container. It is also
possible, during development for example, to log in to the vdjbase container and make modifications there, but these will not be persisted when the container is
stopped.

The nginx container is from [linux.io](https://hub.docker.com/r/linuxserver/nginx/). It has extensive logging and configuration options: check their documentation 
for details.

## Installation

## Make the Persistent File Systems

`docker-compose.yml`:
```yaml
version: '2'

services:
  nginx:
    image: linuxserver/nginx
    volumes:
        - /share/vdjbase/config/nginx:/config
        - /share/vdjbase/shared:/vdjbase
    ports:
       - 6000:80
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
  vdjbase:
    image: vdjbase
    container_name: vdjbase
    volumes:
        - /share/vdjbase/shared:/scratch
    restart: unless-stopped
```

VDJbase will be listening on port 6000. If you would like to use some other port, modify the setting in `docker-compose.yml`.

You will need to create a directory to hold some shared files created by vdjbase and served by nginx, and also a directory to hold configuration info. In the compose
file, these are `/share/vdjbase/shared` and `/share/vdjbase/config/nginx`. Amend the compose file as necessary and create the directories:

```shell script
mkdir /share/vdjbase/shared
mkdir /share/vdjbase/config
mkdir /share/vdjbase/config/nginx
```

## Build the Images

Build the `vdjbase_base` and `vdjbase` docker images:

```shell script
cd vdjbase_base
docker build --no-cache -t vdjbase_base .
cd ../vdjbase
docker build --no-cache -t vdjbase .
```

## First Run

In the directory containing `docker-compose.yml`:

```shell script
docker-compuse up -d
````

This initial run will establish the configuration files for nginx in the persistent folder. Once the containers have been running for 2 minutes or so, take them 
down, and copy the site configuration file into the appropriate location (modify the path appropriately), and restart:

```shell script
docker-compose down
cp vdjbase_nginx_config /share/vdjbase/config/nginx/nginx/site-confs/default
docker-compuse up -d
````

VDJbase should now be running on the port defined in `docker-compose.yml` (port 6000, unless you modified the file).

## Logs and debugging

Nginx logs are in /share/config/nginx/log/nginx. You can log in to either container to see what it's doing.
