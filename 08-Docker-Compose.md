## Docker Compose

- If we need to set up a complex application running multiple services, a better way to do it is to use **Docker Compose**.

- With Docker Compose, we could create a configuration file in YAML format called **dockercompose.yaml** put together the different services and the options specific to running them in this file. Then we could simply run a **docker compose up** command to bring up the entire application stack. This is easy to run and maintain because all the configurations are stored into dockercompose.yaml file. This is all only applicable to running containers on a **single Docker host**.

## Voting Application Sample

- If we use docker run commands individually to run all the components then:

    ```
    docker container run -d --name=redis redis --> to create redis container

    docker container run -d --name=db postgres --> to create postgresdb container

    docker container run -d --name=vote -p 5000:80 --link redis:redis voting-app --> to create voting-app and link it with redis container. Format of link is <container name>:<Host which appliction looking for>

    docker container run -d --name=result -p 5001:80 --link db:db result-app --> to create result-app and link it with db container.

    docker container run -d --name=worker --link redis:redis --link db:db worker --> to create worker and link it with redis and db container.

    ```

- Same application using docker compose:

    **docker-compose.yaml**
    **version: 1**
    
    ```
    redis:
        image: redis
    db:
        image: postgres
    vote:
        image: voting-app
        ports:
        - 5000:80
        links:
        - redis
    result:
        image: result-app
        ports:
        - 5001:80
        links:
        - db
    worker:
        image: worker
        links:
        - redis
        - db
    ```

    - **Note:** if container name and host name which application wants to connect is same then we can specify one name.
        **e.g. db:db --> db**
    
    - Once docker-compose.yaml file is ready then run **docker compose up** command to run all containers.

    - We image is not build in local system then instead of **image: redis** mention **build: ./redis**. (location of build context)

    - **version: 2** 
        - Docker compose will create **dedicated bridge network** for the application and attach all containers to that new network. All containers are now able to reach to each other by their name. Hence we **don't need links** in version 2.

        - Version 2 introduce **depends on** feature, if we wish to specify a startup order.

        ```
        version: 2
        services:
            redis:
                image: redis
            db:
                image: postgres
            vote:
                image: voting-app
                ports:
                - 5000:80
                depends_on:
                - redis
            result:
                image: result-app
                ports:
                - 5001:80
                depends_on:
                - db
            worker:
                image: worker
                depends_on:
                - db
                - redis
        ```

    - **version: 3**
        - Similar like version 2 but some of the options are removed and added. For that need to see documentation.

- **Networks in Docker Compose**

        ```
        version: 2
        services:
            redis:
                image: redis
            db:
                image: postgres
            vote:
                image: voting-app
                ports:
                - 5000:80
                depends_on:
                - redis
                networks:
                - front-end
                - back-end
            result:
                image: result-app
                ports:
                - 5001:80
                depends_on:
                - db
                networks:
                - front-end
                - back-end
            worker:
                image: worker
                depends_on:
                - db
                - redis

        networks:
            front-end:
            back-end:
        ```

## Additional Commands

- **docker-compose up --detach** or **docker-compose up -d** --> Run application stack in detached mode

- **docker-compose logs** --> to see application stack logs

- **docker-compose ps** --> to see all containers under application stack

- **docker-compose down** --> Stop and remove resources

- **docker-compose top** --> Display the running processes

## Sub-command list

**Commands:**

  - ***build:***          Build or rebuild services
  - ***config:***         Validate and view the Compose file
  - ***create:***             Create services
  - ***down:***               Stop and remove resources
  - ***events:***             Receive real time events from containers
  - ***exec:***               Execute a command in a running container
  - ***help:***              Get help on a command
  - ***images:***             List images
  - ***kill:***               Kill containers
  - ***logs:***               View output from containers
  - ***pause:***              Pause services
  - ***port:***               Print the public port for a port binding
  - ***ps:***                 List containers
  - ***pull:***               Pull service images
  - ***push:***               Push service images
  - ***restart:***            Restart services
  - ***rm:***                 Remove stopped containers
  - ***run:***                Run a one-off command
  - ***scale:***              Set number of containers for a service
  - ***start:***              Start services
  - ***stop:***               Stop services
  - ***top:***                Display the running processes
  - ***unpause:***            Unpause services
  - ***up:***                 Create and start containers
  - ***version:***            Show version information and quit