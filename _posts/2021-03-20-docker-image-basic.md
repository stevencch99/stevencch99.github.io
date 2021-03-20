---
layout: post
title: "Docker - Basic Understanding of Image"
description: "Docker - Basic Understanding of Image"
crawlertitle: "Docker - Basic Understanding of Image"
date: 2021-03-20 17:07:04 +0800
categories: Docker
tags: Docker Linux
comments: false

---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## What is image?

It is...

- App binaries and dependencies
- Metadata about the image data and how to runt he image
- A read-only template used to build [containers]({{site.baseurl}}{% link _posts/2021-02-12-docker-container-usage.md%}).
  - Including everything required to run an application: code, runtime, system tools, system libraries and settings.
- Not a complete operating system - **No** kernel or kernel modules (e.g. drivers)

**Official definition**: "And Image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime."

## Build image

To build an image, first we need a `Dockerfile` which described a instruction of how to run our program:

![Dockerfile explanation](https://training.play-with-docker.com/images/ops-images-dockerfile.svg)

  > Ref: <https://training.play-with-docker.com>

> ## To increase the build's performance, exclude files and directories by adding a `.dockerignore` file to the context directory (the same directory where running docker build).

- Prepare `Dockerfile` file, e.g.

    ```bash
    touch Dockerfile
    ```

    ```bash
    FROM alpine
    RUN apk update && apk add nodejs
    COPY . /app
    WORKDIR /app
    CMD ["node","index.js"]
    ```

- Then build the image from current directory through the Dockerfile use Docker CLI.

    ```bash
    # build from the current directory as context:
    docker build .

    # build with tag
    docker image build -t <TagName>
    ```

- Layers & Cache:

    Docker recognized that we had already built some of these layers in our eariler image builds and since nothing had changed in those layers it could simply use a cached version of the layer.

    ![](https://training.play-with-docker.com/images/ops-images-cache.svg)

      > Ref: <https://training.play-with-docker.com>

### Image layer

An Docker Image is composed by series of layers, Docker managed to combine these layers into a single image.

**A layer is basically a change** on an image, or an intermediate image.

Every command specified in Dockerfile causes the previous image to change, created a new layer upon previous image as well.

## Create image from a container

- Change container content (e.g. docker container run -it TagName bash, then change something.)
- See the difference against original version:
    ```bash
    docker container diff ContainerID
    ```
- Commit the change:
    ```bash
    docker container commit ContainerID
    ```
    Once it has been commited, we can see the newly created image in the list of available images.
    ```bash
    docker image ls
    ```

## What is Registry & Repository

- Registry: A service that is storing your docker images, it can be public or private.
  - e.g. Docker Hub, Quary, Google Container Registry, AWS Container Registry
- Repository: A collection of different images with same name, that has different tags

  > - Registry => Docker Hub
  > - Repository => Repository for certain application images

## Tag an existing Image

<!- TODO: supplement description ->
Tag is useful to identify different version of the image.

```bash
docker tag ImageID TagName
```

- Add a new tag

    ```bash
    $ docker tag <Image_ID> <tag_name> <newName>/<repoName>:<tagName>
    # e.g.
    $ docker tag <ID> stevencch99/rails:6.0.2
    ```

## `.dockerignore` file

Here is an example of syntax:
- `*/temp*`: Exclude files and directories whose names start with temp in any immediate subdirectory of the root.
- `*/*/temp*`: Exclude files and directories starting with temp from any subdirectory that is two levels below the root.
- `temp?`: Exclude files and directories in the root directory whose names are a one-character extension of temp. For example, `/tempa` and `/tempb` are excluded.

## Image Inspection

Using inspect function to show information from an image:

```bash
docker image inspect <TagName>
```

- The layers the image is composed of
- The driver used to store the layers
- The architecture / OS it has been created for
- Metadata of the image

### Inspect with format:

```bash
docker image inspect --format "{{ json .RoofFS.Layers }}" <TagName|ID>
```

## Removing Docker Images

- Use `docker system df` to see space usage.

- List: `$ docker images -a` or `$ docker ps -a`
- Remove: `$ docker rmi Image` || `$ docker image rm image`
- Prune: `$ docker [image|container|volume|network] prune`
- Prune everything: `$ docker system prune`
    Docker >= 17.06.1: `$ docker system prune --volumes`
    > prunes images, containers, and networks.
    > In Docker 17.06.1 and higher, must specify the `--volumes` flag.

    `$ docker system prune --volumes`

## Dockerfile reference

> [builder ref:](https://docs.docker.com/engine/reference/builder/)

- `CMD`
- `COPY`
- `ENTRYPOINT`
- `ENV`
- `EXPOSE`
- `MAINTAINER`
- `RUN`
    The `RUN` insturction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the `Dockerfile`.
    `RUN` has 2 forms:
    - `RUN <command>` (*shell* form, the command is run in a shell, which by default is `/bin/sh -c` on Linux or `cmd /S /C` on Windows)
    - `RUN ["executable", "param1", "param2"]` (`exec` form)
- `workdir`


## Environment replacement

Environment variables (declared with the `ENV` statement) can also be used in certain instructions as variables to be interpreted by the `Dockerfile`.

```
FROM busybox
ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
```

- The `${variable_name}` syntax also supports a few of the standard `bash` modifiers as specified below:
  - `${foo:-bar}` indicates that if `foo` is set then the result will be that value, or the `bar` will be the result.
  - `${foo:+bar}` indicates that if `foo` is set then `bar` will be the result, otherwise the result is the empty string.
      ```
      b=2
      echo ${foo:-b} # return 2
      echo ${foo:+b} # empty string
      ```

---

## References
