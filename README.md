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
