---
layout: post
title: "How to install Docker on Ubuntu 20.04"
description: "Install Docker on Ubuntu 20.04"
crawlertitle: "Install Docker on Ubuntu 20.04"
date: 2021-01-21 16:44:16 +0800
categories: Docker
tags: Docker Linux
comments: true

---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

This article is my note of Docker installation and troubleshooting process, follow by the [Docker official guide](https://docs.docker.com/engine/install/ubuntu/).

## Set up the repository

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:

   ```shell
   sudo apt update

   sudo apt install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg-agent \
     software-properties-common
   ```

2. Add Docker’s official GPG key:

   ```shell
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

   Verify that you now have the key with the fingerprint `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`, by searching for the last 8 characters of the fingerprint.

   ```shell
   sudo apt-key fingerprint 0EBFCD88
   ```

   ![screen shot of sudo apt-key](https://i.imgur.com/EEguhrZ.png)

3. Set up the **stable** repository.

   ```shell
   sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   ```

## Install Docker engine

1. Update the apt package index, and install the latest version of Docker Engine and containerd.

   ```shell
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

2. Verify that Docker Engine is installed correctly by running the hello-world image.

   ```bash
   sudo docker run hello-world
   ```

   You should see the welcome message below if Docker has been successfully installed:

   ```bash
   Hello from Docker!
   This message shows that your installation appears to be working correctly.

   To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
       (amd64)
    3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
       to your terminal.

   To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

   Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

   For more examples and ideas, visit:
    https://docs.docker.com/get-started/
   ```

   - Troubleshooting: [Error when connect to docker.io](#error-when-connect-to-dockerio)

### Using docker command without sudo

- Add the docker group if it doesn't already exist:

  ```bash
  sudo groupadd docker
  ```

- Add the currently connected user `$USER` to the docker group. Or change the user name to match your preferred user if you do not want to use your current user:

  ```bash
   sudo gpasswd -a $USER docker
  ```

  > [gpasswd](https://man7.org/linux/man-pages/man1/gpasswd.1.html): The gpasswd command is used to administer `/etc/group`, and `/etc/gshadow`.

- To activate the changes to groups:

  ```bash
  # option:
  sg group_name -c "bash"

  # or:
  newgrp docker
  ```

  > [sg](https://man7.org/linux/man-pages/man1/sg.1.html): To execute command as different group ID.
  > [newgrp](https://man7.org/linux/man-pages/man1/newgrp.1.html): To change the current group ID during a login session.

  Or log out/in to activate the changes.

- Check if it's successful:

  ```bash
  docker ps
  ```

## Troubleshooting

### Error when connect to docker.io

This error occurs when using the Docker CLI, failed even simply calling `docker run`, the error message like this:

```terminal
Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection
```

- Fixing in the new `daemon.json` file
  - List current DNS server:

    ```bash
    nmcli dev show | grep 'IP4.DNS'
    ```

  - Create or Edit `/etc/docker/daemon.json` add the DNS server:

    In this case `192.19.8.10` is my proxy server, `8.8.8.8` is Google Public DNS server.

    ```json
    {
      "dns": ["192.19.8.10", "8.8.8.8"]
    }
    ```

  - Restart Docker deamon

    ```bash
    sudo service docker restart
    ```

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## References

- [Docker official guide](https://docs.docker.com/engine/install/ubuntu/)
- [How can I use docker without sudo?](https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo) - ASK UBUNTU
- [docker.io DNS doesn't work, it's trying to use 8.8.8.8](https://askubuntu.com/questions/475764/docker-io-dns-doesnt-work-its-trying-to-use-8-8-8-8) - ASK UBUNTU
