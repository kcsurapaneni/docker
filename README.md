# docker

## Architecture

There are 3 main components in docker architecture.

1. Docker Client
2. Docker Server/Daemon
3. Docker Hub/Registry

## Commands

### help

- `docker --help` - It'll show usage and some commands info

    Example
    ```
    Usage:  docker [OPTIONS] COMMAND

    A self-sufficient runtime for containers
    ```

- `docker <command> --help` - List specific usage of that command
Example
    ```
    docker inspect --help

    Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

    Return low-level information on Docker objects

    Options:
    -f, --format string   Format output using a custom template:
                            'json':             Print in JSON format
                            'TEMPLATE':         Print output using the given Go template.
                            Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
    -s, --size            Display total file sizes if the type is container
        --type string     Return JSON for specified type
    ```

### common commands

- `docker pull <image>` - Download an image from a registry to server
- `docker ps` - List running containers in the docker daemon
- `docker ps -a` - List all the containers(including exited) in the docker daemon
- `docker images` - List images which are downloaded from registry and available in your local server
- `docker inspect <image-id>` - List metadata information like id, created, config, file system, OS, architecture of that image
- `docker run <image-id>` - It will run the image as container
- `docker run -d <image-id>` - It will run the image as container in detached mode. So, we can safely close the terminal or shell
- `docker run -p <host-port>:<container-port> <image-id>` - It will run the image as container and expose the port of the specified container to local defined port
- `docker port <container-id>` - It will show the ports info, that are exposed from container to localhost
- `docker exec -it <container-id> <cmd>` - It will execute the `cmd` on the running container and returns the output
- `docker restart <container-id>`
- `docker start <container-id>`
- `docker stop <container-id>`
- `docker logs <container-id>`
- `docker logs <container-id>` - Shows logs along with timestamp
- `docker rm <container-id>` - Remove container
- `docker rmi <image-id>` - Remove image

## Dockerfile

Docker files contains a set of steps, instructions or directives which are used via `docker build` command to create a docker image

### Structure

- Each line will have instruction and arguments
- It's a convention to have all the instructions on CAPS to distinguish between arguments. (`FROM nginx`)

### Instructions

- `FROM` - sets the base image for a build and it is a must instruction (e.g. alpine)
- `LABEL` - adds metadata of an image (e.g. description, maintainer)

> <p style="color:red"> Following 3 instructions will created a new layer per instruction </p>

- `RUN` - runs commands in a new layer (e.g. installs or configuration)
- `COPY` - will copy the files from source (client machine) to destination (new image layer)
- `ADD` - It also does the same as `COPY`, but in addition to that we can use remote URL and do the extraction

- `CMD` - It defines default executables of a container. If you want to run a web server on creating a container from image by default, then you can specify this and it is automatically starts. This can be overridden by docker run command arguments.

- `ENTRYPOINT` - does the same thing as `CMD`, but can't be overridden

> We can also use both `CMD` and `ENTRYPOINT` together in a single file, where `CMD` specifies default options to the executable specified in the `ENTRYPOINT` instruction

- `EXPOSE` - informs docker what port the container app is running on (metadata only, no network configuration)

## Environment Variables

- We can pass environment variables like `-e <env-name>=<env-value>` 
- e.g. `docker run --name db -e MYSQL_ROOT_PASSWORD=somewordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -d mysql:8.0.23`
- We can see the environment values of container by inspect (e.g. `docker inspect <container-id>`)
```
"Env": [
    "MYSQL_ROOT_PASSWORD=somewordpress",
    "MYSQL_PASSWORD=wordpress",
    "MYSQL_DATABASE=wordpress",
    "MYSQL_USER=wordpress",
    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "GOSU_VERSION=1.12",
    "MYSQL_MAJOR=8.0",
    "MYSQL_VERSION=8.0.23-1debian10"
]
```

## Container Storage

### Writable Layer

- Container image is made of a stack of immutable or read-only layers. When the Docker engine creates a container from such an image, it adds a writable container layer on top of this stack of immutable layers.

- The container layer is marked as read/write. Another advantage of the immutability of image layers is that they can be shared among many containers created from this image. All that is needed is a thin, writable container layer for each container.

- Docker uses storage drivers to store image layers, and to store data in the writable layer of a container.

- The container’s writable layer does not persist after the container is deleted, but is suitable for storing ephemeral data that is generated at runtime.

- Use Docker volumes for write-intensive data, data that must persist beyond the container’s lifespan, and data that must be shared between containers.

### TMPFS

- It uses host memory not disk, so it will be fast
- Not persisted. Hence, once the container is deleted we lost the data
- Can't be shared across containers
- Generally used for storing sensitive or temp files

### Bind Mounts

- It maps host folders to container folders
- As it rely on host folder structure, not portable. Because, our system might not have the same folder structure if we want to run that image as container
- Multiple containers can access same host folder
- Using `-v` flag
    > ```docker run --name db -e MYSQL_ROOT_PASSWORD=somewordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -v /Users/kc/docker_mysql8_data:/var/lib/mysql -d mysql:8.0.23```
- Using `--mount` flag
    > ```docker run --name db -e MYSQL_ROOT_PASSWORD=somewordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress --mount type=bind,source=/Users/kc/docker_mysql8_data,target=/var/lib/mysql -d mysql:8.0.23```

> The difference between `-v` and `--mount` is that, `-v` can create a directory if it didn't exist, BUT `--mount` gives an error if the directory doesn't exist 

### Volumes

- These are same as `bind mounts`, but managed by docker
- Storage is available outside of container lifecycle
- Can be attached to multiple containers as well, but careful as there is no locking, we might get inconsistent results
- Can be moved between containers
- Using `-v` flag
    > `docker run --name db -e MYSQL_ROOT_PASSWORD=somewordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -v docker_mysql8_data:/var/lib/mysql -d mysql:8.0.23`
- Using `--mount` flag
    > `docker run --name db -e MYSQL_ROOT_PASSWORD=somewordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress --mount source=docker_mysql8_data,target=/var/lib/mysql -d mysql:8.0.23`
- We can list all the volumes by `docker volume ls`
- We can also manually create a volume by `docker volume create <volume-name>` and delete it by `docker volume rm <volume-name>`
