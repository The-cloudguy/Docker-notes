## Docker command syntax
```
docker  [docker-object] [sub-command] [options] [arguments/commands]
```
- For example:

    - Image
        
        ```
        docker image ls
        ```
    - Container
        
        ```
        docker container ls
        ```
    - Network
        
        ```
        docker network ls
        ```
    - Volume
        
        ```
        docker volume ls
        ```

- Create a new container

    ```
    docker container create httpd
    ```
    - It will pull image if its not available locally.
    - Once created, it will generate 64 character long container id.
    - All object stored into **/var/lib/docker**

- Container list command

    ```
    docker container ls --> list running containers

    docker container ls -a --> list all containers

    docker container ls -l --> last running container

    docker container ls -q --> list container id of running containers

    docker container ls -aq --> list container id of all containers
    ```

- Start the container

    ```
    docker container start <container id> --> tp start container
    ```

- Create and Run a container together

    ```
    docker container run <image name>
    ```
    - Containers are not meant to host OS, it will use to execute a task and exits.
    - while creating ubuntu container it is running **/bin/bash** as executable command, which executes and exists immediately.
    - container only lives if process inside is alive.

- Container Run - with options

    ```
    docker container run -it <image name>
    ```
    - this will run image in interactive mode and attach to hosts terminal.

- Container Run in detached mode

    ```
    docker container run -d <mage name>

    docker container attach <container id>
    ```
    - Run container in detached mode means it will not give any output except the container id.

- Container Run - setting up name for container

    ```
    docker container run -itd --name=webapp <image name>
    ```
    - Each name should be unique
    ```
    docker container rename <existing name> <new name> --> rename the name of container
    ```

- Set container hostname
    
    ```
    docker container run -itd --name=webapp --hostname=webapp <image name>
    ```

- Container exec - executing commands

    ```
    docker container exec <container id> <command which we want to execute in container>
    
    docker container exec -it <container id> /bin/bash --> will access the shell then run command manually
    ```

- Container inspect - Inspect the container for more details

    ```
    docker container inspect <container name or ID>
    ```

- Container stats - will show resource utilization of containers

    ```
    docker container stats
    ```

- Container top - to check which process utilization for container

    ```
    docker container top <container name or ID>
    ```

- Container logs

    ```
    docker container logs <container name or ID> --> will print logs

    docker container logs -f <container name or ID> --> will stream the logs
    ```

- Container events

    ```
    docker system events --since 60m --> will print container events which happend within 60 minutes
    ```

## Linux Signals

- **kill -SIGSTOP $(pgrap [service name])** --> Will pause the process
    - **docker container pause [container name or ID]**

- **kill -SIGCONT $(pgrep [service name])** --> Will resume the process
    - **docker container unpause [container name or ID]**

- **kill -SIGTERM $(pgrep [service name])** --> Will terminate the process. Its the polite way to ask process to shutdown but there is a chance that process might take time to wind up the process or there is a chance that system will ignore completely

- **kill -SIGKILL $(pgrep [service name])** --> it will forcefully terminate the process
    - **docker container stop [container name or ID]**

- **docker container kill --signal=9 [container name or ID]** --> we can send signal as well using this command

## Removing Container

- First need to stop the running container then we can remove it.
    
    ```
    docker container stop <container name or ID>

    docker container rm <container name or ID>
    ```

- Stop and remove all the container

    ```
    docker container stop $(docker container ls -q)

    docker container rm $(docker container ls -aq)
    ```

- Remove all the stopped container and reports the total reclaimed space

    ```
    docker container prune
    ```

- Container to remove itself as soon as it finishes execution

    ```
    docker container run --rm ubuntu expr 4 + 5
    ```

## Restart Policy

- For this first requirement is that container should be started for atleast 10 secs. If container is not started on the first place then we won't go in restart loop.

- Three ways that container will stop and ready for restart.
    
    - Executed the command/process successfully and stopped. (Exit Code 0)
    
        ```
        docker container run ubuntu expr 3 + 5
        ```
    - Didn't execute successfully and failed to run the command or process. (Exit Code 1)

        ```
        docker container run ubuntu expr three + 5
        ```
    - Manually stopped the container. (Exit Code 0)

        ```
        docker container stop httpd
        ```
    
- We can set restart policy for containers for above three cases:

    - **docker container run --restart=[ no | on-failure | always | unless-stopped ]**

        - **no** : Container won't restart in any scenerio.

        - **on-failure** : Container will restart on execution failure means exit code 1.

        - **always** : Container will always restart in all scenerio but if we stop the container manually then container will restart only when we restart the Docker Daemon.

        - **unless-stopped** : Container won't restart if we stop it manually and even if we restart the Daemon. Other than this condition container will restart.

- What if Docker Daemon will stop? By default it will make all container stop. To prevent this we can configure **live-restore** parameter in **/etc/docker/daemon.json**.

    ```
    {
        "debug": true,
        "hosts": ["tcp://<IP>:2376"],
        "live-restore": true
    }
    ```
    - once we configure this then start the daemon. Now if we create the container and daemon will stop then container won't go down.

## Copying files to/from containers

- To copy files from host to container and vice versa.

    ```
    docker container cp <source_path> <container name>:<destination_path> --> host to container

    docker container cp <container name>:<source_path> <destination_path> --> container to host

    docker container cp /tmp/app/ webapp:/opt/app --> copy entire directory from host to container
    ```
## Port Mapping

- When the application container is created, it will expose a port to access that application. We can access from **[container IP]:[container port]**.

- What if we want to access it from host IP. In that case we need to publish container port on host.

    ```
    docker container run -p <host port>:<container port> <image name>

    e.g.
    docker container run -p 80:8080 nginx
    ```
- We can publish multiple containers of same application on different host ports. 

    ```
    docker container run -p 80:8080 --name=nginx1 nginx

    docker container run -p 8080:8080 --name=nginx2 nginx

    docker container run -p 8081:8080 --name=nginx3 nginx
    ```

- For an example, host is having 3 interfaces with different CIDR configured for them. We want container should be accessed through only one interface then we need to publish port with that interface ip as well.

    ```
    docker container run -p <Host IP for that specific interface>:<host port>:<container port> <image name>

    e.g.
    docker container run -p 192.168.1.7:80:8080 nginx
    ```

- If we don't want to set host port and let it set automatically then

    ```
    docker container run -p <container port> <image name>

    e.g.
    docker container run -p 8080 nginx
    ```
    - it will set host port from 32768 to 60999 (Ephemeral Port Range). This we can set into **/proc/sys/net/ipv4/ip_local_port_range**

- What if we wish to publish the port on the container without specifying which ports explicitly? What if the container has services running on multiple ports and we just want it to make all ports where service running on, accessible or mapped to a port on the host?

    ```
    docker container run -P <image name>
    ```
    - Capital **P** option will take automatically exposed ports and publish it on host within ephemeral ports. This exposed ports are defined in Image Dockerfile as **EXPOSE** instruction.

    ```
    docker container run -P --expose=8080 <image name>
    ```
    - We can also add additional ports which are not specified in Dockerfile using **--expose** option.

- To know which ports are exposed by the container use image inspect command.

    ```
    docker image inspect <image name>
    ```
    - there will be section **ExposedPorts**, where we can see what ports are exposed.

- To map all this ports in Docker, it uses **IP Tables** under the hood. It will create port forwarding rules when we expose ports through Docker CLI.

    ```
    iptables -t nat -S DOCKER
    ```
