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
Execute the following command to create the directory:
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

## How to communicate with Greenbone Community Containers using gvm-cli from gvm-tools

First, install gvm-tools.
In this guide, pipx is used.
Execute the following command:
```shell
pipx install gvm-tools
```

Execute the following command to check the version of the Greenbone Community Containers:
```
gvm-cli socket --xml "<get_version/>"
```
When the above command is executed, you will be prompted to enter a username and password (By default, the password for the admin user is set to "admin").
```shell
$ gvm-cli socket --xml "<get_version/>"
Enter username: admin
Enter password for admin: 
```
When the correct username and password are provided, the following output is produced:
```xml
<get_version_response status="200" status_text="OK"><version>22.6</version></get_version_response>
```

For detailed information about gvm-tools and gvm-cli, please refer to the following page.

[https://greenbone.github.io/gvm-tools/install.html](https://greenbone.github.io/gvm-tools/install.html)

## Communication with Greenbone Community Containers Using the Improved gvm-cli from gvm-tools

The standard gvm-cli requires entering the username and password for every command, which I find extremely inconvenient. To address this issue, I developed an improved version that is available in the following repository.

[https://github.com/zackey-heuristics/gvm-tools](https://github.com/zackey-heuristics/gvm-tools)

In this section, we describe the method for communicating with the Greenbone Community Containers using the gvm-tools from the repository mentioned above.

For installation, pipx is used.
Execute the following command:
```shell
pipx install git+https://github.com/zackey-heuristics/gvm-tools --force
```

Execute the following command to check the version of the Greenbone Community Containers. However, replace <GMP_USERNAME4SOCKET> and <GMP_PASSWORD4SOCKET> with the username and password you have configured. By default, both are set to `admin`.
```shell
gvm-cli  --xml "<get_version/>" --gmp_username4socket <GMP_USERNAME4SOCKET> --gmp_password4socket <GMP_PASSWORD4SOCKET>
```
When the correct username and password are provided, the following output is produced:
```xml
<get_version_response status="200" status_text="OK"><version>22.6</version></get_version_response>
```

## How to Stop Containers

Execute the following command to stop the Greenbone Community Containers:
```shell
docker compose down --volumes
```

## Method for Changing the Location of gvmd.sock and the Corresponding Configuration Procedures

The method described in the "How to Run Containers" section requires recreating the directory and running commands with administrative privileges each time the machine is started.
This section explains how to address these issues.

First, create an appropriate directory under a directory where the current user has, by default, read (r), write (w), and execute (x) permissions. 
For example, if the current user is "heuristics", create a directory with a path such as:
```shell
/home/heuristics/run/gvmd/
```
When creating the directory, execute the following command:
```shell
mkdir -p $HOME/run/gvmd
```

Next, modify the relevant section of the docker-compose.yml file as follows:
```yaml
volumes:
  gpg_data_vol:
  scap_data_vol:
  cert_data_vol:
  data_objects_vol:
  gvmd_data_vol:
  psql_data_vol:
  vt_data_vol:
  notus_data_vol:
  psql_socket_vol:
  gvmd_socket_vol:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /home/heuristics/run/gvmd/
  ospd_openvas_socket_vol:
  redis_socket_vol:
  openvas_data_vol:
  openvas_log_data_vol:
```
Using the above configuration, launch the Greenbone Community Containers.

Finally, execute the following command to check the version of the Greenbone Community Containers:
```shell
gvm-cli socket --socketpath $HOME/run/gvmd/gvmd.sock --xml "<get_version/>"
```

## Note: Disabling the Port Exposure of the Greenbone Security Assistant (GSA) Web Interface

This repository is configured to interact with the Greenbone Community Containers using Unix Domain Sockets. As a result, the port for the Greenbone Security Assistant (GSA) Web Interface is not exposed by default.

To enable the GSA Web Interface, remove the comment markers from the relevant section as indicated below:
```yaml
  gsa:
    image: registry.community.greenbone.net/community/gsa:stable
    restart: on-failure
    # ports:
    #   - 127.0.0.1:9392:80
    volumes:
      - gvmd_socket_vol:/run/gvmd
    depends_on:
      - gvmd
```