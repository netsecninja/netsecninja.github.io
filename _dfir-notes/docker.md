---
layout: page
title: Docker
permalink: dfir-notes/docker/
---


Notes regarding how to use Docker and some DFIR information that I researched

# Docker images
* List local images: `docker images`
* Remove local image(s): `docker rmi <IMAGENAME or ID>`
* Pull a Docker Hub image: `docker pull <REPONAME>:<TAG>`
* Login to Docker Hub: `docker login`
* Push to your Docker Hub: `docker push <REPONAME>:<TAG>`
* Common Docker file commands:
    * Base image: `FROM <IMAGENAME>`
    * Commands to run: `RUN <COMMAND>`
        * Can chain commands with && or just use another RUN command
* Build from Dockerfile: `docker build <DOCKERFILE>`

# Docker containers
* List running containers: `docker container ls`
    * List all container: `docker container ls -a`
* Create a container and interact: `docker container run <IMAGENAME>`
    * Other common options:
        * Detached (background): `-d`
        * Name: `--name <NAME>`
        * Map exposed ports to ephemeral range: `-P`
        * Map exposed ports to specific port: '`p <HOSTPORT>:<CONTAINERPORT>`
* Start a container: `docker container start <CONTAINERNAME OR ID>`
    * Other common options:
        * Start interactive session: `-it`
* Stop a container: `docker container stop <CONTAINERNAME OR ID>`
* Remove a container: `docker container rm <CONTAINERNAME OR ID>`
* Info on a container: `docker container inspect <CONTAINERNAME OR ID>`
* Interact with a running container:
    * Start a shell: `docker container exec -it <CONTAINERNAME OR ID> bash`
    * Interact with an existing shell: `docker attach <CONTAINERNAME OR ID>`
    * Close interactive session, leaving container running: `<CTRL+p> <CTRL-Q>`
    

# Docker volumes
* Create a new volume: `docker volume create <VOLUMENAME>`
* Info on a volume: `docker volume inspect <VOLUMENAME>`
* Create container with mounted volume:
    `docker contaner run -v <VOLUMENAME>:<MOUNTPATH> <IMAGENAME>`
    OR
    `docker contaner run --mount source=<VOLUMENAME>,target=<MOUNTPATH> <IMAGENAME>`
* View mounted volume contents: `sudo ls /var/lib/docker/volumes/<VOLUMENAME>/_data`

# DFIR
* Save container info: `docker container inspect <CONTAINERNAME OR ID> | tee container.txt`
* Capture container processes: `docker container top <CONTAINERNAME OR ID> -aux | tee processes.txt`
* Capture container port mappings: `docker container ports <CONTAINERNAME OR ID> | tee ports.txt`
* Copy file from container filesystem to local: `docker container cp <CONTAINERNAME OR ID>:<SOURCEFILE>`
* Export a container filesystem: `docker container export -o <OUTPUTFILE> <CONTAINERNAME OR ID>`
* Manually explore differences made since startup: `sudo su && cd /var/lib/docker/`
* Capture differenes made since startup: `docker commit <CONTAINERNAME OR ID> <NEWIMAGENAME>`
* Capture volume contents: `sudo tar -zcf <VOLUMENAME>.tar.gz /var/lib/docker/volumes/<VOLUMENAME>`

# References
* [Docker - Documentation](https://docs.docker.com/)
* [StackRox - Docker Forensics for Containers: How to Conduct Investigations](https://www.stackrox.com/post/2017/08/csi-container-edition-forensics-in-the-age-of-containers/)
