# Docker-Compose Cheat Sheet
This is a docker-compose cheat-sheet
## Table of Content
* #### [Services](#Services)
* #### [Volumes](#Volumes)
    * ##### [Bind Mounting](#Bind-Mounting)
       * [Short Syntax](#Bind-mounting#Short-Syntax)
       * [Long Syntax](#Bind-Mounting#Long-Syntax)
    * ##### [Volume Mounting](#Volume-Mounting)
       * [Short Syntax](#Volume-Mounting#Short-Syntax)
       * [Long Syntax](#Volume-Mounting#Long-Syntax)
    * ##### [Volumes Mix](#Volume-and-Bind-Mix)
       * [Short Syntax](#Volume-and-Bind-Mix#Short-Syntax)
       * [Long Syntax](#Volume-and-Bind-Mix#MountingLong-Syntax)

* #### [Networking](#Networking)

## Services
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
    db-data
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
    db-data
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
    db-data
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
    db-data
```
## Networking

Per default docker containers run over the bridge network. However you can specify your own networks.
Networks are created on the root level just like the services and volumes then every service is assign to a network.

In our case we have created two new networks `front_end` and `back_end`
and we assign the database to the `back_end` and redis server to `front_end`
Networks under a service is a list you can either list your networks with `-` or `[]` 
Both notations are shown below:
server has a  `-` notation
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
      networks:
        [front_end, back_end]

networks:
    front_end:
    back_end:
```

