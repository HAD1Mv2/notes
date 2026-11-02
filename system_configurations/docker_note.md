# Docker Note
Important steps in docker.
## Enable Nvidia Container Toolkit in Docker
Follows instruction in [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). After installing, do configure step for docker.

## Setting in docker-compose file to enable docker use GPU

In docker-compose.yml, set as follows

``` yaml
services:
  app:
    image: your-image
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

For using certain GPU index

``` yaml
services:
  app:
    image: your-image
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["0"]
              capabilities: [gpu]
```

## Docker Command
- To build and run docker image
    ``` bash
    docker compose up --build
    ```
- Check available docker images
  ```bash
  docker images
  ```
- Check docker containers
  ``` bash
  # Check running containers
  docker ps

  # Check all containers included stopped ones
  docker ps -a
  ```
- Stop a docker container
  ``` bash
  docker stop <container name/id>
  ```
- Start existing container
  ``` bash
  docker start <container name/id>
  ```
- Check docker logs
  
    ``` bash
    docker logs my-api

    #Follow the logs live:
    docker logs -f my-api

    #Show only the last 100 lines:
    docker logs --tail 100 my-api

    #Or logs from the last 10 minutes:
    docker logs --since 10m my-api
    ```

- Enter the container

  ``` bash
    # If your API container is running:

    docker exec -it my-api bash

    # If bash isn't installed:

    docker exec -it my-api sh
  ```

    Now you're inside the container:

    ``` bash
    root@abc123:/app#
    ```

    You can inspect things:
    ``` bash
    ls
    ps aux
    env
    ```

## Mount Volume 

To mount a volume or directory in Docker Compose, you add a volumes block inside your service configuration within the compose.yaml (or docker-compose.yml) file. [1](https://docs.docker.com/engine/storage/volumes/), [2](https://www.geeksforgeeks.org/devops/docker-compose-volumes-for-container-data/)

Depending on your goal, you will choose between a Bind Mount (linking a specific folder on your computer directly to the container) or a Named Volume (letting Docker completely manage the storage location). [1](https://dev.to/lovestaco/docker-volumes-explained-with-docker-compose-2p3f)

### Option 1: Bind Mounts (Best for Local Development)

Use this when you want changes made in your local project folder to immediately reflect inside the running container. [1](https://www.youtube.com/watch?v=y-sMw9937PM)

``` yaml
services:
  web:
    image: node:lts
    ports:
      - "3000:3000"
    volumes:
      # Short Syntax: [HOST_PATH]:[CONTAINER_PATH]
      - ./app:/home/node/app
      
      # Optional: Add ':ro' at the end to make it read-only for the container
      - ./config.json:/home/node/app/config.json:ro

# Use code with caution.
```

• ./app: The relative path to the folder on your host machine.
• /home/node/app: The absolute path where the files will appear inside the container. [1](https://www.warp.dev/terminus/docker-compose-volume)

### Option 2: Named Volumes (Best for Database & Production Data)

Use this when you want data to persist safely (like database storage) but don't need to directly modify the raw files on your host computer. Named volumes must be declared in a top-level volumes: section at the bottom of the file. [1](https://docs.docker.com/reference/compose-file/volumes/), [2](https://www.youtube.com/watch?v=BJ-KwdBvbVE&t=275), [3](https://dev.to/lovestaco/docker-volumes-explained-with-docker-compose-2p3f), [4](https://docs.docker.com/engine/storage/volumes/)

``` yaml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secretpassword
    volumes:
      # Mount the named volume to the container data folder
      - pgdata:/var/lib/postgresql/data

# Define the named volume globally so Compose can create/manage it
volumes:
  pgdata:
# Use code with caution.
```

### Short Form vs. Long Form Syntax

While the short syntax (- host:container) is the most common, Docker Compose also supports a highly explicit long-form syntax that reduces ambiguity: [1](https://www.youtube.com/watch?v=u1q8AyNMxd4&t=735), [2](https://www.warp.dev/terminus/docker-compose-volume)

``` yaml
services:
  web:
    image: nginx:alpine
    volumes:
      - type: bind
        source: ./html
        target: /usr/share/nginx/html
        read_only: true

# Use code with caution.
```

#### Comparison of Mount Types


Type	Syntax Example	Managed By	Typical Use Case
Bind Mount	- ./src:/app	You (points to a specific host folder)	Sharing code files for instant local reloading.
Named Volume	- db_data:/var/lib/mysql	Docker (stored deep in Docker's internal directories)	Databases, upload directories, persistent logs.
Anonymous Volume	- /app/node_modules	Docker (automatically generated with a random ID)	Preventing local host folders from overwriting specific container directories.


### How to apply changes

Once you update your configuration file, spin up your environment with the following command:

``` bash
docker compose up -d
# Use code with caution.
```

*(Note: If you ever change volume paths or types, running docker `compose down` followed by `docker compose up -d` guarantees the new storage layout binds correctly).*

