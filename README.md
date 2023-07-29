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
