# Docker
- Docker is **virtualization at application level**.
- Using Docker we can create containers and those containers we can use for one or more micro services.
- Container is package of **os libraries, application server, network feature**.
- Image is a **iso** for container. When we need to launch container, we will ask for image from local or remote registry and once get the image. using that image we can create containers.

## Components of Docker
- Client
- Daemon (Docker Engine)
- ContainerD
- OCI or runc (In windows it will be Compute Service)

## Process of create the container

- First docker client will request docker daemon
- Daemon will check for image in local image registry
- If Image is not present in local registry then it will go to docker hub (remote registry), then image will be download to local registry.(Pull)
- Using that image container will be created

## Process of creating Image

- from existing Container, we export to Docker hub as Image
- From existing VMs, we convert it to Image
- Using dockerfile (Instruction based approach)

## Few useful commands

- **docker info** --> Gives infor of Docker Engine

* **docker image ls** --> To list docker images

- **docker container ls -a** --> To list all containers including exited

- **docker ps** --> shows container which are running

- **docker ps -a** --> shows all containers which are created

- **docker ps -l** --> show container which was lastly created

- **docker ps -s** --> To understand actual size on disk (writable layer size + Virtual size of image layers)

- **docker start {container name or ID}**  --> Start Container

- **docker stop {container name or ID}**  --> Stop Container

- **docker rm {container name or ID}** ---> Remove container (not recovered if not pushed to Docker Hub)

- **docker image rm {Image name}**  --> Remove image locally

- **docker image build -t {image name} {docker file path}** --> create image with docker file

- **docker image build -t {image name} -f {different docker file path}** --> create image to different path

- **docker container run jenkins:2.60.3** --> create container of specific version of jenkins in attached mode (which will give all logs output of container on screen)

- **docker container run -d jenkins** --> create container with latest version of jenkins (Run in detached mode)

- **docker container run -it jenkins** --> Run container in iteractive mode

- **docker container run -p 8081:8080 -d {image name}** --> create container which inbound port is 8081 and application port is 8080

- **docker container rm -f $(docker container ls -a -q)** --> it will remove all container at once

- **docker inspect {image-id} or {image-name}** --> check image components specially layers

- **docker volume create {volume-name}** --> create a volume

- **docker volume inspect {volume-name}** --> inspect volume

- **docker volume ls** --> List all volumes

- **docker volume rm {volume-name}** -- Delete volume

- **docker container run --name cont1 -it --mount "source={volume-name},target/destination={container mountpoint name}" openjdk:8 /bin/bash** ---> to mount created volume on container

- **docker container run --name cont1 -it --mount "type=bind,source={host directory},target/destination={container mountpoint name}" openjdk:8 /bin/bash** --> it will create bind mount in container, basically bind mount is mounting host directory into container

- **docker container run --name cont1 -it --mount "type=tempfs,target/destination={container mountpoint name}" openjdk:8 /bin/bash** --> It will create tempfs mount in container

## Instruction

- **FROM** --> to choose base image
* **RUN** --> where you can execute command while building the image
- **EXPOSE** --> your container is exposed this port for your application
* **CMD** --> Instruction which will run after your container created
- **ENTRYPOINT** --> It will be executable for CMD
- **LABEL** --> This instruction adds metadata
- **ADD** --> This instruction is used to copy the contents from local system and URL. It will take 2 arguments which are source and destination
- **COPY** --> This instruction is used to copy the contents only from local system. It will take 2 arguments which are source and destination
- **VOLUME** --> create a volume
- **ENV** --> Set Enviornment Variables during image build which will be keep while creating containers
- **ARG** --> Set arguments which will only work while buiilding image
- **WORKDIR** --> select the working directory

## Docker Internals

- Containers are isolated area on the top of Operating System.
- To create isolated area, linux kernel has technology which is called namespace.
- namespace could be anything, like namespace for FS is mount namespace, for user its called user namepace, for process tree its called process namespace.
- Container has its own network, Process  tree, FS, required libraries, CPU, RAM etc...
- For Application, container will create a illusion of operating system which has all components available to run application but in size it will be very light weight.
- Resource limits are applied using kernel feature called cgroups (Control Groups).

## Container States

- Running (Started)
- Exited (Stopped)
- Paused

## Container creation Mode

- Attached Mode
	- Our terminal will recieve outputs from container stdout, stderr
- Dettached Mode (Background)
	- Container runs in the background and you will recieve container ID
- Interactive Mode
	- Create a container and login into interactive shell

## Image Layers & Container Thin Layers

- Image consists of layers which are R/O in nature mostly.
- When we create Dockerfile, whaterver instruction consist changes in FS, will create a extra layer e.g.
- By default openjdk:8 image has 7 layers but when we create a Dockerfile as below
    ```
    FROM openjdk:8
    RUN mkdir /app
    COPY . /app
    CMD ["sleep" , "1d"]
    ```
- Now in this file FS is updating thorugh COPY and RUN instruction hence, a new layers will be add when we will create image

- When we will create container using image, it will add another thin rw layer on the top of image layers.

- All the actions will be happen in this perticular thin layer, it won't affect to actual image layers.

- Now when you will create 100 containers using the same image, underlying image layers will be same in all containers but each containers will have its separate RW thin layer as container layer on the top of the image layers. 

- Problem with this RW layer is, as much as container alives, layer will be available. Once container dies, data on that layer will be vanished.

## Docker Volumes

- Volume adds persistence to the docker container
- Volumes add disk sharing across containers and also between container and host
- Volume life cylce is different from containers life cycle
- Volume types
	- Volume --> First separate volume needs to be created then it should mounted into container. it will long last if container also removes, because it will created on host FS. When volume itself deleted then data will be lost. 
	- Bind Mount --> Mount Host directory in container as NFS shared directory. No volumes will be created.
	- tempfs ---> when mounted in Container, data stored in Host RAM. But it will not persist longer. Once container host rebooted data will be gone

## Example Scenario:

- Create a docker container with mysql running:
	- For the containers which generate/store data it is important to preserve it, so we have to use volumes

## Application Categories:

- Stateless Applications:
	- Application which do not store any state and rely on other applications to store/preserve the data

- Stateful Application
	- Applications which store the data/state locally and for these applications the data has to preserved by using volumes.


## Storage Driver

- overlay2 - by default latest storage driver installed on host system
- aufs - in ubuntu flavour sometimes you will see this storage driver
- devicemapper - older type of storage driver
- zfs - special linux filesystem storage driver
- brtfs - special linux filesystem storage driver

- for Amazon S3, Azure cloud storage - 3rd party storage driver plugins are available

## Docker Network Drivers

- Docker networking way back was part of docker daemon
- Docker networking is implemented as a different component "libnetwork"
- Networking in Docker is implemented as Network Drivers. Popular drivers are
	1) Bridge
	2) Host
	3) MACVLAN
	4) None
	5) Overlay (multi host)

## Linux Networking Fundamentals

- Docker networking uses the kernel’s networking stack to create high level networking feature of Docker
- Docker Networking is Linux Networking

## Building Blocks

- Linux Bridge

	- This is layer 2 device with virtual implementation of physical switch inside Linux Kernel
	- Forwards the traffic based on MAC address by inspecting traffic.
- Network Namespace

	- Isolated network stack in kernel with its own interfaces, routes & firewall rules
- Virtual Ethernet Devices or veth

	- Linux networking interface that acts as connecting wire between two two network namespaces
- Iptables

	- Generic table structure that defines rules & commands as part of the netfilter framework.

- CNM Constructs – Sandbox

	- Contains the configuration of the containers network stack.
	- Includes routing tables, container’s interfaces, DNS setting etc.
	- Sample implementation could be Linux Network Namespace or any other similar concept.
- CNM Constructs – Endpoint

	- Joins sandbox to the Network.
	- This abstracts actual connection to the network away from application.
	- Maintains portability for applications, so that they can use different network drivers
- CNM Constructs – Network

	- Collection of endpoints having connectivity b/w themselves
- CNM Driver Interfaces

	- CNM provides two pluggable & open interfaces to leverage additional functionality and control in the network
		- Network Drivers
		- IPAM Drivers

## Docker Native Network Drivers

- Host

	- Container uses the networking stack of the host.
- Bridge

	- Creates a bridge on the host that is managed by Docker.
	- All containers on Bridge Driver can communicate among themselves
	- Default Driver
- Overlay

	- Used for multi-host networks.
	- Uses local linux bridges & VXLAN to overlay container-to-container networking
- MACVLAN

	- Uses Linux MACVLAN bridge to establish connection b/w container interfaces & parent host interfaces
	- MAC address can be attached to each container
- None

-  Host Network Driver

	- In host network driver all the containers are in same network namespace(sandbox)

- Bridge Network Driver

	- Default Bridge Network Driver
	- User-Defined Bridge Networks

- Creating a new bridge network

	- Create a new bridge network with range as 10.11.0.0/16 and name as mybridge
	- Command: 
        ```
        docker network create --driver bridge --subnet 10.11.0.0/16 mybridge

        docker network ls
        
        docker network inspect mybridge
        ```
	- Create two container p1 and p2 on the mybridge
	- Command:

        ```
        docker container run --name p1 --network mybridge -d alpine sleep 1d
  
        docker container run --name p2 --network mybridge -d alpine sleep 1d
        ```
	- Now test for the connectivity between p1 and p2 using ip and hostnames
        ```
        docker network inspect mybridge
  
        # make a note of ip addresses
        docker exec p1 ping -c 3 10.11.0.3
  
        docker exec p1 ping -c 3 p2
        ```
	- In the above both containers can connect using names and ip addresss

	- Try to add container c1 which is running in bridge network to mybridge network
        ```
        docker network connect mybridge c1
  
        docker network inspect mybridge
  
        docker exec p1 ping -c 3 c1
        ```
	- If disconnected then container will attach to default bridge network
        ```
        docker network disconnect mybridge c1
        ```
- When you create overlay network, two network drivers are create
	- **Overlay**: This points to the overlay network
	* **docker_gwbridge**: The egress bridge is for traffic leaving the cluster. only one docker_gwbride exists per host. This bridge is used for ingress/egress communications not for container to container communications with in overlay.

- Docker Swarm
	- The cluster maanagement & Orchestration features are embedded inside Docker Engine.
	- Docker swarm consists of multiple docker hosts which run in swarm mode.
	- Two Roles managers and workers exist in Docker swarm
	- Manager is responsible for membership & delegation
	- Worker is responsible for running swarm services
	- Each Docker Host can be a manager, a worker or both.
	- In Docker Swarm Desired State is maintained. For instance if you are running one container in swarm on a particular node (Worker) 	   and that node goes down, then Swarm schedules this nodes task on other node to maintain the state.
	- Task is a running container which is part of swarm service managed by Swarm Manager
- Nodes
	- It is instance of the docker engine participating in Swarm.
	- There are two kinds of nodes
		- **Manager nodes**:
			- You communicate to manager node to deploy applications in the form of Service Definitions.
			- Manager nodes dispatch unit of work called as tasks to the Worker nodes
		- **Worker nodes**:
			- They receive & execute the tasks dispatched from manager nodes.
			- An agent runs on the worker node & reports on the tasks assigned to it
- Services and tasks
	- Service is the definition of the task to be executed.
	- Typically it would be the application to be deployed.
	- Two kinds of Service models are available
		- **Replicated Services model**: In this case swarm manager distributes a specific number of replica task among the nodes based upon the scale you set in the desired state
		- **Global Services Model**: In this case swarm runs one task for the service on every available node in the cluster.
- Task
	- carries a Docker container and the commands to run inside the container.
	- It is the atomic secheduling unit of swarm.
	- Once a task is assigned to node, it cannot move to another node.
	- It can only run on the assigned node or fail.

- Gameoflife application through code to container flow

    1) clone the code
        ```
        git clone https://github.com/wakaleo/game-of-life.git
        ```
    2) build package
        ```
        cd game-of-life
        
        mvn package
        ```
    3) build image using below Dockerfile
        ```
        cd /root/game-of-life/gameoflife-web
        
        cat Dockerfile
        
        FROM tomcat:8-jre8
        RUN rm -rf /usr/local/tomcat/webapps/*
        COPY target/gameoflife.war /usr/local/tomcat/webapps/ROOT.war
        EXPOSE 8080
        CMD ["catalina.sh", "run"]

        docker image build -t mygol:0.1 .
        ```
    4) Run container using the image
        ```
        docker container run -d -p 8080:8080 mygol:0.1
        ```
    5) Check <publicip>:8080 URL

    6) Now need to push image to registry (DockerHub , ECR, ACR)
        ```
        docker login
        username:
        password:
        
        docker image tag mygol:0.1 <username>/<imagename>:<tag>
        
        docker push <username>/<imagename>:<tag>
        ```
- For multi stage build use below dockerfile for gameoflife application
    ```
        FROM maven:3-openjdk-8 as builder
        RUN git clone https://github.com/wakaleo/game-of-life.git
        WORKDIR ./game-of-life
        RUN mvn package
        CMD ["sleep" , "1d"]

        FROM tomcat:8-jre8
        RUN rm -rf /usr/local/tomcat/webapps/*
        COPY --from=builder /game-of-life/gameoflife-web/target/gameoflife.war /usr/local/tomcat/webapps/ROOT.war
        EXPOSE 8080
        CMD ["catalina.sh", "run"]
    ```


































