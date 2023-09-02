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

## Networking

### Host Networking

- Host networking containers share the host network.
- So, whatever the container port, the same port will be used in host as well.
- There is a limitation with this, as we can't run multiple containers which are using same port or same image of multiple containers.

### Bridge Networking

- There will be a bridge network will be created, so it will have different IP for each container.
- So, each container can use same port of the container as every container is having different IP.
- Container needs to publish the port when running in this case.(e.g. `-p <host-port>:<container-port>`)

## Docker Compose

- Docker Compose is a tool for running multi-container applications on Docker defined using the Compose file format. A Compose file is used to define how one or more containers that make up your application are configured.
- Once you have a Compose file, you can create and start your application with a single command: `docker compose up`. To run in detached mode we can use `docker compose up -d`.
- To tear down all the created containers during `up`, we can call `docker compose down`

## Multi-stage builds

With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image.

```
FROM maven:3.9-amazoncorretto-17

ENV HOME=/usr/app
RUN mkdir -p $HOME
WORKDIR $HOME

ADD ./app $HOME
RUN mvn clean package -DskipTests


FROM amazoncorretto:17.0.8

COPY --from=0 /usr/app/target/app-0.0.1-SNAPSHOT.jar /app/app.jar

EXPOSE 8080

ENTRYPOINT java -jar /app/app.jar
```

Now just run `docker build`

`docker build -t kcsurapaneni/app:latest .`

How does it work? The second `FROM` instruction starts a new build stage with the `amazoncorretto:17.0.8` image as its base. The `COPY --from=0` line copies just the built artifact from the previous stage into this new stage. The `maven:3.9-amazoncorretto-17` and any intermediate artifacts are left behind, and not saved in the final image.

### Name your build stages
By default, the stages aren’t named, and you refer to them by their integer number, starting with 0 for the first `FROM` instruction. However, you can name your stages, by adding an `AS <NAME>` to the `FROM` instruction. Following example improves the previous one by naming the stages and using the name in the `COPY` instruction. This means that even if the instructions in your Dockerfile are re-ordered later, the `COPY` doesn’t break.

```
FROM maven:3.9-amazoncorretto-17 AS builder

ENV HOME=/usr/app
RUN mkdir -p $HOME
WORKDIR $HOME

ADD ./app $HOME
RUN mvn clean package -DskipTests


FROM amazoncorretto:17.0.8

COPY --from=builder /usr/app/target/app-0.0.1-SNAPSHOT.jar /app/app.jar

EXPOSE 8080

ENTRYPOINT java -jar /app/app.jar
```

## depends_on

In Docker, the `depends_on` attribute is used in a Docker compose file to specify the order in which services within a multi-container application should start.

The `depends_on` attribute allows you to define the dependencies between services in your Docker compose setup. When you specify dependencies using `depends_on`, Docker Compose ensures that the specified services are started in the correct order before the dependent services are started.

Here is the basic example of how `depends_on` works in a Docker Compose YAML file.
```
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example

  web:
    image: mywebapp
    depends_on:
      - db
```
In this example we have 2 services: `db` and `web`. The `web` service has a dependency on the `db` service specified using `depends_on`. This means when you run `docker compose up -d`, Docker Compose will start the `db` service before starting the `web` service. It ensures that the necessary dependencies are ready before bringing up the dependent services

However, it's important note a few things.

1. **Dependencies don't guarantee full readiness:** `depends_on` only ensures that the dependent service starts after the service it depends on. It doesn't wait for the dependent service to be fully `healthy` or `ready` before proceeding. If your application needs to be wait for a specific condition (like a database fully initialized), you may need additional mechanisms within your application code.
2. **Not for inter-container communication:** `depends_on` is not designed for inter-container communication. It helps manage the startup order but doesn't manage the availability of network services between containers. If your application requires communication with another container, you should handle retries and connection handling within your application.
3. **Availability:** Starting order doesn't guarantee availability. Even if a service starts, it doesn't guarantee that it's fully operational or responsive. Monitoring and health checks are important for ensuring your services are running as expected.

In summary, `depends_on` is a useful feature in Docker Compose for managing startup order and basic dependencies, but doesn't handle all aspects of application readiness and communication between containers.

## Health Check

It is a feature that allows you to define a test to determine the health status of a container. Health checks help ensure that your containerized applications are running as expected ans can automatically handle situations where a container becomes unhealthy.
> This is particularly useful in orchestrators like Docker Swarm or Kubernetes, where you want to maintain the desired state of your services and automatically replace unhealthy containers.

A health check involves specifying a command or a combination of commands that are periodically executed within the container. The exit status of these commands determine the health status of the container. A healthy container typically has an exit status of 0, while a non-zero exit status indicates unhealthy state.

Here is an example of defining a health check for a container in a Docker Compose file.
```
version: '3'
services:
    db:
    platform: linux/x86_64
    image: mysql:8.0.23
    container_name: local-observability-db
    environment:
      MYSQL_DATABASE: observability
      MYSQL_USER: observability_user
      MYSQL_PASSWORD: observability_password
      MYSQL_ROOT_PASSWORD: root_observability_password
    ports:
      - "${HOST_PORT:-3312}:3306"
    volumes:
      - ./db/init_db.sql:/docker-entrypoint-initdb.d/init_db.sql
    healthcheck:
      test: mysqladmin ping -h 127.0.0.1 -u $$MYSQL_USER --password=$$MYSQL_PASSWORD
      interval: 30s
      timeout: 5s
      retries: 5
```

In this example,
- **`test`**: This specifies the health check command to be executed. In this case, it's using `mysqladmin` command to check if DB is accessible to that user or not.
- **`interval`**: This is the time interval between consecutive health checks. In this case, the health check will be performed every 30 seconds.
- **`timeout`**: This is the maximum amount of time allowed for the health check command to run. If it takes longer than this timeout, the health check is considered failed.
- **`retries`**: This specifies the number of consecutive failures before considering the container unhealthy. In this case, the health check wll be retried up to 5 times before marking the container as unhealthy.

It's important to note that health checks are not a guarantee of the application's functionality, but rather a way to determine if the container is in a healthy state according to the defined criteria. Depending on the orchestrator we are using, unhealthy containers might be replaced automatically or trigger alerts for manual intervention.

Health checks are a powerful tool for ensuring the reliability of our containerized applications, especially in dynamic scalable environments.

## Some more commands

- To copy a file from docker host to running container
  
  - `docker cp <host-file-path> <name-of-the-container>:<path-of-the-container>`
  
  - `docker cp run.sh app:/tmp/run.sh`

## [Using profiles with Compose](https://docs.docker.com/compose/profiles/)

- We can attach profiles information to a service in docker compose.

  ```
  services:
    frontend:
      image: frontend
      profiles: ["frontend"]

    phpmyadmin:
      image: phpmyadmin
      depends_on:
        - db
      profiles:
        - debug

    backend:
      image: backend

    db:
      image: mysql
  ```
- In the above example, `frontend` and `phpmyadmin` services are assigned with `frontend`, `debug` profiles.
- Services without `profiles` are always enabled, when we run `docker compose up`, in this case `backend` and `db` will start.
- If we want to start profile specific services we should mention `--profile`.

- `docker compose --profile debug up` - This command will start `backend`, `db` and `phpmyadmin` services.

- To read more on this, please refer official documentation here - https://docs.docker.com/compose/profiles/
