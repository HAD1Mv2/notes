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