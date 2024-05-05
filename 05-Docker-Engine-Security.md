## TLS enable

- To enable TLS, we need to configure **/etc/docker/daemon.json** file.

    ```
    {
        "hosts": ["tcp://<IP>:2376"],
        "tls": true,
        "tlscert": "/var/docker/server.pem",
        "tlskey": "/var/docker/serverkey.pem"
    }
    ```
    - On client side we need to just define TLS flag while running command or we can set environment as well.

        ```
        export DOCKER_TLS=true
        export DOCKER_HOST="tcp://<IP>:2376"

        docker container ls
        ```

- But this will not make secure connection between host and Daemon server. For that we need to setup client side authentication as well.

## Client Side Authentication

- To enable client side authentication, first we need to create client certificate pair. Then we need to share client cert, client key and ca cert with client machine. Then configure **daemon.json** file on server.

    ```
    {
        "hosts": ["tcp://<IP>:2376"],
        "tls": true,
        "tlscert": "/var/docker/server.pem",
        "tlskey": "/var/docker/serverkey.pem",
        "tlsverify": true,
        "tlscacert": "/var/docker/caserver.pem"
    }
    ```
    - On client side we need to define TLS Verify flag and in command we need to give client side cert locations. This parameters we can define into **~/.docker** directory.

        ```
        export DOCKER_TLS=true
        export DOCKER_TLS_VERIFY=true
        export DOCKER_HOST="tcp://<IP>:2376"

        docker --tlscert=<> --tlskey=<> --tlscacert=<> container ls        
        ```

## Namespaces and capabilities

- Docker uses namespaces to isolate workspace, process IDs, Network, Interprocess Communication, Mounts and Unix Timesharing systems are created in their own namespace, thereby providing isolation between containers.

- Containers and Host share the same kernel but containers are isolated using namespaces in linux. The host has a namespace and containers have their own namespace. All the processes run on the containers are running on the host itself but thier own namespaces. Container can see its own processes only.

- By default docker runs processes into containers as the root user. We can change that by running below command.

    ```
    docker container run --user=1000 ubuntu sleep 3600
    ```
    - Now container will run with 1000 user.

    - We can set this while creating image as well.

    ```
    Dockerfile:

    FROM ubuntu

    USER 1000
    ```

- We can set capabilities of the USER as well. Which we can see in this location **/usr/include/linux/capability.h**.

    ```
    docker container run --cap-add MAC_ADMIN ubuntu --> add capability

    docker container run --cap-drop KILL ubuntu --> drop capability

    docker container run --privileged ubuntu --> run container in privileged mode
    ```

## CGroups

- CGroups or Control Groups are a linux feature that allows to allocate resources such as CPU, Memory, Network Bandwidth or block I/O among different processes within our host.

- Docker uses CGroups to share or limit resources among different containers.

## Resource Limits CPU

- By default, there are no restrictions to how much CPU or Memory resources a container can consume on a host.

- When the kernel detects that the system is out of memory, it starts killing processes on the host to free up memory. This need not just be the containers, it may even kill the native processes running on the host to free up memory.

- If there is one CPU and two process running on host then how CPU will be shared between two processes? Each process gets an equal amount of time of the CPU. The first process runs for a few cycles and then waits, while the second process gets its time and then back to the first process again and this is continued until both processes ends. The time allocated to each process is in microseconds and switch happens so quick that it goes unnoticed by the end user.

- If one of the process is high priority then we can assign more CPU shares for it. The way the CPU is shared between two processes is by allocating shares of the CPU to define the time each process gets to use the CPU. For example, if you assign a CPU share of 1024 for one process and 512 for another, the first process gets doubled the time than the first process.

- So the shares only matter if there is competition from other processes. This single process can consume as much of the CPU as it needs. The scheduler will managing CPU and allocate the time of the CPU between the different processes. The default scheduler on linux system is the **CFS scheduler** stands for **Completely Fair Scheduler**. Docker version 1.13 and higher supports another scheduler known as **Real-Time Scheduler**.

- CGroups or Control Groups is what enables restricting resources to containers. Each container gets a CPU share of 1024 assigned by default. That means all containers get equal amount of CPU time.

- To modify the CPU share of a particular container,

    ```
    docker container run --cpu-shares=512 webapp
    ```

- Restrict or limit the CPU usage,

    ```
    1) If there are multiple CPUs for example 4 CPU then it will numbered as CPU-0, CPU-1, CPU-2 and CPU-3, to limit a container to use first 2 CPUs then,

    docker container run --cpuset-cpus=0-2 webapp1 --> webapp1 will use CPU-0 to CPU-2 (Range)

    docker container run --cpuset-cpus=2,3 webapp2 --> webapp2 will only use CPU-2 and CPU-3 (Seperated by comma)

    docker container run --cpuset-cpus=3 webapp3 --> webapp3 will only use CPU-3

    2) We can set the CPU count a container can use maximum,

    docker container run --cpus=2.5 webapp1 --> webapp1 will use 2.5 count of CPU out of 4 CPU

    docker container update --cpus=0.5 webapp1 --> we can extend count from 2.5 to 3 by adding 0.5 more cpu
    ```

- If no restriction for CPU usage then container ends up using all the CPUs of host which might affect process running on containers as well as host. Which makes server unresponsive due to HIGH CPU usage.

## Resource Limits MEMORY

- Unless limited, a process and consume as much memory as available on the physical RAM. When all physical memory is used, process start using swap space configured on disk for additional memory.

- When there is no memory left to use then container will throw and error **OOM(Out of Memory)** and start killing processes to free up the memory.

- To restrict memory consumption for container,

    ```
    docker container run --memory=512m webapp
    ```
    - If container start using more CPU than it will throttle but if container start using more memory than it assigned, then it will be killed with OOM exception.

    - In above command, if webapp consumes more than 512MB then it can consume another 512MB as swap space, if it is enabled on the host.

    - To limit container's swap space usage then,

        ```
        docker container run --memory=512MB --memory-swap=512MB webapp --> Swap size = 512MB - 512MB = 0 MB

        - memory-swap option must be used in conjuction with the memory option. The memory-swap option, is in fact, the sum of memory and the swap space. it's not just the swap space.

        - If we want to set 256MB of swap size then,

        docker container run --memory=512MB --memory-swap-768MB webapp --> Swap size = 768MB - 512MB = 256MB
        ``` 