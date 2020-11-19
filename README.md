# Docker-Compose Cheat Sheet

This is a docker-compose cheat-sheet

## Table of Content

- [Docker-Compose Cheat Sheet](#docker-compose-cheat-sheet)
  - [Table of Content](#table-of-content)
  - [Services](#services)
  - [Barebone services](#barebone-services)
    - [Image](#image)
    - [Build](#build)
    - [Ports](#ports)
    - [Depends on](#depends-on)
    - [Container Name](#container-name)
    - [Environment Variables](#environment-variables)
    - [Command](#command)
  - [Volumes](#volumes)
    - [Bind Mounting](#bind-mounting)
      - [Short Syntax](#short-syntax)
      - [Long Syntax](#long-syntax)
    - [Volume Mounting](#volume-mounting)
      - [Short syntax](#short-syntax-1)
      - [Long Syntax](#long-syntax-1)
    - [Volume and Bind Mix](#volume-and-bind-mix)
      - [Short Syntax](#short-syntax-2)
      - [Long Syntax](#long-syntax-2)
  - [Networking](#networking)

## Services

## Barebone services

The barebone docker-compose file ways starts with a version and then services

`version` is what docker-compose version should be used.
`services` are basically the list of containers you want to start.

In our case we have 3 three containers we want to orchestrate

1. Database

1. Redis

1. App

```yaml
version: "3.8"

services:
  database:
  redis:
  app:
```

### Image

You can specify what image your services should start from
The image is a dictionary.

```yaml
version: "3.8"

services:
  database:
    image: postgres:9.4
  redis:
    image: redis:alpine
  app:
    image: my-app
```

### Build

If you have a Dockerfile you want to build you can also specify that in the docker compose.

With the dot you are actually saying that the Dockerfile is found in the same directory as the docker-compose file. If the Dockerfile is not found docker-compose will be using the image.

You can find more detail in [Docker Build documentation](https://docs.docker.com/compose/compose-file/#build)

If you already built once and you want to re-build you have to run docker-compose with `--build` options like so:

    docker-compose up --build

```yaml
version: "3.8"

services:
  database:
  redis:
  app:
    build: .
    image: my-app
```

### Ports

You can expose ports inside your container and map them to the port of the host. 
The syntax is `hostPort:ContainerPort` and an example would be `9090:8080` the port `8080` on the container will now be mapped to `9090` in the host.
The ports property is a __list__ and is specified for each service.

```yaml
version: "3.8"

services:
  database:
  redis:
  app:
    ports:
      - 9090:8080
      - 5001:80
      - 5858:5858
    build: .
    image: my-app
```

### Depends on

If you have two or more containers where the startup sequence matter then you can use `dependens_on` to specify that one service needs another service to start.
This is usually if you have an application that needs the database to operate. You want to make sure that the database is fully started before you launch the application.
the depends_on property is a __list__ and specified for one or more services.

In our case our app depends on redis. It is always the service name that goes under `depends_on`

```yaml
version: "3.8"

services:
  database:
  redis:
  app:
    depends_on:
      - redis
    ports:
      - 9090:8080
      - 5001:80
      - 5858:5858
    build: .
    image: my-app
```

### Container Name

You can also specify a custom name for your container

```yaml
version: "3.8"

services:
  database:
  redis:
  app:
    depends_on:
      - redis
    ports:
      - 9090:8080
      - 5001:80
      - 5858:5858
    build: .
    image: my-app
    container_name: my-web-container
```

### Environment Variables

You also specify environment variables either directly or through `.env` file. You specify the variables under `environment` section. `environment` os a __list__ so you can specify several of them.
You can then use the environment variables like so `${VARIABLE}`

In following example we have several variables.
We have `DEVELOPMENT` and `EDITOR` that we have specified and set a value of `1` and `vim` and then we have variables `PATH_TO_DOCKERFILE` and `TAG` and are specified in a .env file. This example shows that you can use the variables alone like `PATH_TO_DOCKERFILE` or you can use it together with other text like `TAG` where you have the image name first.

```yaml
version: "3.8"

services:
  database:
    build: ${PATH_TO_DOCKERFILE}
  redis:
    image: "redis:${TAG}"
  app:
    environment:
      - DEVELOPMENT=1
      - EDITOR=vim
    depends_on:
      - redis
    ports:
      - 9090:8080
      - 5001:80
      - 5858:5858
    build: .
    image: my-app
    container_name: my-web-container
```

### Command

Command is a list where you can specify what command should be overwritten. This acts like the `docker exec` meaning that the command will be executed when the container is up and running. Please do not confuse this with `CMD` inside Dockerfile.
You can either use the list syntax `[]` or not write the command out. The first item in the list will be the main command and the rest will be parameters.

```yaml
version: "3.8"

services:
  database:
    build: ${PATH_TO_DOCKERFILE}
    command: ["bundle", "exec", "thin", "-p", "3000"]

  redis:
    image: "redis:${TAG}"
  app:
    environment:
      - DEVELOPMENT=1
      - EDITOR=vim
    depends_on:
      - redis
    ports:
      - 9090:8080
      - 5001:80
      - 5858:5858
    build: .
    image: my-app
    container_name: my-web-container
    command: bundle exec thin -p 3000
```


## Volumes

In docker compose you can create your own volumes for volume mounting.
You can also specify a volume on the host for bind mounting.
For volume mounting you declare you volumes under volume section on the root level like services then every services is either volumes mounted or bind mounted

- Bind mounting: bind the mount on your host

- Volume mounting: mount the volume created by docker.

### Bind Mounting

For bind mounting you do not need to create a new volume. all you have to do is specify a full-path to the folder you want to bind mount
the synxtax is `source:target` where source is a local folder you want to bind mount

#### Short Syntax

```yaml
version: "3.8"

services:
  database:
    volumes:
      - ~/Document/databases:/var/lib/mysql/data
  redis:
  app:
```

#### Long Syntax

The long syntax is clearly more readable.
You specify the type, source and target

```yaml
version: "3.8"

services:
  database:
    volumes:
      - type: bind
        source: ~/Documents/databases
        target: /var/lib/mysql/data
  redis:
  app:
```

### Volume Mounting

We will create a volume called `db-data` and mount it.
physically the newly created volume is found in `/var/lib/docker/volumes/db-data/`
the syntax is `source:target`

#### Short syntax

```yaml
version: "3.8"

services:
  database:
    volumes:
      - db-data:/var/lib/mysql/data
  redis:
  app:

volumes:
  db-data:
```

#### Long Syntax

`no-copy`: flag to disable copying of data from a container when a volume is created

```yaml
version: "3.8"

services:
  database:
    volumes:
      - type: volume
        source: db-data
        target: /var/lib/mysql/data
  volume:
    no-copy: true
  redis:
  app:

volumes:
  db-data:
```

### Volume and Bind Mix

You can also mix Volume mounting and bind mounting

#### Short Syntax

```yaml
version: "3.8"

services:
  database:
    volumes:
      - db-data:/var/lib/mysql/data
      - ~/Document/databases:/var/lib/mysql/data
  redis:
  app:

volumes:
  db-data:
```

#### Long Syntax

```yaml
version: "3.8"

services:
  database:
    volumes:
      - type: volume
        source: db-data
        target: /var/lib/mysql/data
        volume:
          no-copy: true
      - type: bind
        source: ~/Documents/databases
        target: /var/lib/mysql/data
  redis:
  app:

volumes:
  db-data:
```

## Networking

Per default docker containers run over the bridge network. However you can specify your own networks.
Networks are created on the root level just like the services and volumes then every service is assign to a network.

In our case we have created two new networks `front_end` and `back_end`
and we assign the database to the `back_end` and redis server to `front_end`
Networks under a service is a list you can either list your networks with `-` or `[]`
Both notations are shown below:
server has a `-` notation
app has a `[]` notation

```yaml
version: "3.8"

services:
  database:
    networks:
      - back_end
  redis:
    networks:
      - front_end
  server:
    networks:
      - front_end
      - back_end
  app:
    networks: [front_end, back_end]

networks:
  front_end:
  back_end:
```
