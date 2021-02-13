---
layout: post
title: "Docker - Container usage"
description: "Docker - Container usage"
crawlertitle: "Docker - Container usage"
date: 2021-02-12 13:40:47 +0800
categories: Docker
tags: Docker Linux
comments: true

---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Manage containers

### Listing containers

```bash
docker container ls
```

- To list all containers:

  ```bash
  docker container ls -a
  ```

- To list all containers and size (`-s` option):

  ```bash
  docker container ls -sa
  ```

### Run container

- To start new container interactively from an image with its Tag Name or Image ID:

  ```bash
  docker container run <TagName|ImageID>
  ```

   This command always starts a **new** container, use `docker container start <TagName|ImageID>` to start an exixting stopped one.

- Example of start and attach to an [Alpin linux](https://alpinelinux.org) operating system:

  ```bash
  docker run -it --rm alpine /bin/ash
  ```

  - `-i, --interactive`: Remain the `STDIN` accessible.
  - `-t, --tty`: Allocate a pseudo-tty, bind it on this container.
  - `--rm`: Automatically delete the container when it exits.
  - `/bin/ash`: To run the application `/bin/ash` inside the container, without this option, Docker will run `/bin/sh` by default.

- Execute a command: To run an additional command in existing container.

  ```bash
  docker container exec <TagName|ImageID> <command>
  ```

- Publish a container's port(s) to the host (`-p`, `--publish`)

  ```bash
  docker run -p 127.0.0.1:80:8080/tcp <TagName|ImageID>
  ```

  This binds the TCP port `80` on `127.0.0.1` from the host to the port `
  8080` from the container.

- Expose a port or a range of ports

  ```bash
  docker run --expose 80 my_ubuntu /bin/bash
  ```

  This exposes port `80` of the conatiner without publishing the port to the host system's interfaces.

- Set evnironment variable (`-e`, `--env`, `--env-file`)

  ```bash
  docker run -e FOO=bar --env-file ./env.list <TagName|ImageID> /bin/bash
  ```

  - This connect port `3000` between container and host and set host's environment variable `http_proxy` to container.

    ```bash
    docker container run -e http_proxy -e https_proxy -p 3000:3000 <TagName|ImageID>
    ```

### Start container

- Wake up an exited container:

  ```bash
  docker container start <TagName|ImageID>
  ```

  Instead of using a full container ID, we can use just the first few characters, as long as they are uniquely identifying a container.

- Attach to a stoped container:

  ```bash
  docker container start -ai CONTAINER
  ```

  - `-a`, `--attach`: Attach STDOUT/STDERR and forward signals.
  - `-i`, `--interactive`: Attach to container's STDIN.

### Stop containers:

```bash
dokcer stop CONTAINER

# Stop All containers
docker stop $(docker ps -aq)

# Stop specific container
docker stop $(docker ps -aq --filter name="my_container")
```

- `docker ps -aq` will listing all the containers with container ID.

### Keep a container busy living

#### Option 1.

Use `docker run` with `-t` option:

```bash
docker run -td IMAGE
```

- `-d`: Run container in background and print container ID.
- `-t`, `--tty`: Allocate a pseudo-TTY.

#### Option 2.

A Docker container is supposed to be built with only one process running.

As the container will exit when the process itself exits, there's a way to keep a ubuntu container busy by continuously printing nothing to the system blackhole `/dev/null`:

```bash
docker run -d ubuntu tail -f /dev/null
```

### Pause & Unpause

- To suspends all processes in the specified containers.

  ```bash
  docker pause CONTAINER [CONTAINER...]
  ```

### Attach to a running container

Examples:

- `$ docker attach CONTAINER`
- `$ docker exec -it CONTAINER /bin/bash`

### Remove containers

```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

Options:

- `-f`, `--force`: Force remove a container even it's running.
- `-v`, `--volumes`: Remove anonymous volumes associated with the container.

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Monitoring a container:

### Display the running processes of a container

To list processes that running inside of the specific container.

```bash
docker top CONTAINER
```

### Display detailed information from containers

```bash
docker container inspect [OPTIONS] CONTAINER [CONTAINER...]
```

- To shows the IP address of the container:

  ```bash
  docker container inspect --format '{{ .NetworkSettings.IPAddress }}' CONTAINER
  ```

  - `-f`, `--format`: Format the output using the given Go template.

### Display a live stream of container(s) resource usage statistics

```bash
docker stats
```

### Quick port check

```
docker container port CONTAINER
```

### Fetch the logs of a container

```bash
docker logs [OPTIONS] CONTAINER
```

> Note: This command is only functional for containers that are started with the `json-file` or `journald` loggin driver.

Options:

- `--follow`: To continue streaming the new output from the container's `STDOUT` and `STDERR`.
- `--timestamps`: To add an [RFC3339Nano timestamp](https://golang.org/pkg/time/#pkg-constants), for example `2021-02-12T08:32:48.001607450Z` to each log entry.

---

## References

- [Docker official reference documentation](https://docs.docker.com/reference/)
