## Docker Service
- Check Service
    - systemctl status docker

- Start Service
    - systemctl start docker

- Enable Service so that it will automatically started at reboot of host
    - systemctl enable docker

## Start manually
- we could also run docker daemon directly as a foreground process using the **dockerd** command. This is usually done for troubleshooting and debugging purposes where if the docker daemon is not starting up.

- To debug the docker daemon run **dockerd --debug**. This will add more detailed information in the output along with the debug logs.

- When the docker daemon starts, it listens on internal unix socket at the path **/var/run/Docker.sock**. This socket is used for communication between different processes on the same host.

- This means docker daemon is only accessible within the same host and the Docker CLI is configured to interact with the docker daemon on this socket.

## How to access Docker Daemon from other hosts

- First add the interface to docker host using below command:
    
    ```
    dockerd --debug --host=tcp://<IP>:2375
    ```
    - non secure port 2375

- Now other host can target docker host by first setting an environment variable called **DOCKER_HOST** to point to the IP address of the Docker host.
    
    ```
    export DOCKER_HOST="tcp://<IP>:2375"
    ```
    - then from other host we can run docker commands which will target docker daemon which is configured.

- To enble encryption, we need to create pair of key and pass it while running docker daemon.
    
    ```
    dockerd --debug --host://tcp://<IP>: 2376 --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem
    ```

- Now other host can target docker host by first setting an environment variable called **DOCKER_HOST** to point to the IP address of the Docker host.
    
    ```
    export DOCKER_HOST="tcp://<IP>:2376"
    ```

- above configuration which we are passing while running docker daemon into a configuration file at **/etc/docker/daemon.json**

    ```
    {
        "debug": true,
        "hosts": ["tcp://<IP>:2376"],
        "tls": true,
        "tlscert": "/var/docker/server.pem",
        "tlskey": "/var/docker/serverkey.pem"
    }
    ```
    - once we configure this file we just need to run **dockerd** command, and it will take all required options from this file. If we give options in commannd then it will throw an error.

## Logging Drivers

- Every container is storing logs on the host or stream to other sources like aws, splunk etc

- To see which logs driver is set for Docker Daemon use **docker system info** command and in that check the **Logging Driver** parameter. We can check which logging drivers are supported in **Plugins** parameters.

- We can change loggin driver as well by configuring **/etc/docker/daemon.json** file.

    ```
    {
        "debug": true,
        "hosts": ["tcp://<IP>:2376"],
        "tls": true,
        "tlscert": "/var/docker/server.pem",
        "tlskey": "/var/docker/serverkey.pem"
        "log-driver": "awslogs",
        "logs-opt": {
            "awslogs-region": "us-east-1"
        }
    }    
    ```
    - If we are using awslogs as logging driver then we need to store **AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN** as environment variables in the host.

- We can explicitly configure logging driver for specific container.

    ```
    docker run -d --log-driver <driver name> <image name>

    e.g.
    docker run -d --log-driver json-file nginx
    ```