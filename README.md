# Greenbone Community Containers Compose File for Unix Domain Socket

This repository provides a modified Docker Compose file for Greenbone Community Containers, adapted for use with Unix Domain Sockets, and describes how to manage the container suite launched with that Compose file using gvm-cli from gvm-tools.

## `docker-compose.yml`

The docker-compose.yml file included in this repository is a modified version of [this file](https://greenbone.github.io/docs/latest/_static/docker-compose.yml) that has been adapted for Unix Domain Sockets.
Specifically, the modifications involve bind-mounting a host directory to a container directory to share the Unix Domain Socket between the host and the container.
As a result, Greenbone Community Containers can be managed using gvm-cli in socket mode.

For more information about Greenbone Community Containers, please refer to the following page.

[https://greenbone.github.io/docs/latest/22.4/container/index.html](https://greenbone.github.io/docs/latest/22.4/container/index.html)

## How to Run Containers

Before running `docker compose up`, you must create a directory on the host that will be used for bind mounting into the container.
Please execute the following command to create the directory:
```shell
sudo mkdir -p /var/run/gvmd
```

Next, set the necessary permissions on the created directory as required.
For example, to allow the current user to access the directory, execute the following command:
```shell
sudo chown $(whoami):$(whoami) /var/run/gvmd
```

Finally, execute the following command to launch the Greenbone Community Containers:
```shell
docker compose -f docker-compose.yml up -d
```

## How to Stop Containers

Execute the following command to stop the Greenbone Community Containers:
```shell
docker compose down --volumes
```