## Docker Engine

- Docker first released to the public in 2013.

- Docker Engine consist of 3 components.
    - Docker Daemon
        - Docker Daemon is the actual server or process which is responsible for creating and managing objects such as images, containers, volumes and networks on a host.
    - Rest API
        - The REST API provides an interface to manage objects in Docker.
    - Docker CLI
        - Docker CLI is the command line utility which we are using to run commands to manage objects in Docker.

- In the past, Docker used a technology called Linux Containers or LXC to manage containers on Linux. LXC used capabilities of Linux kernel such as namespaces and cgroups to create isolated ennvironments on Linux known as containers. It was hard to work with LXC directly for a normal user.

- With the release of version 0.9(2014), Docker introduced its own execution environment known as libcontainer. With libcontainer Docker could now directly interact with the linux kernel features such as namespaces and cgroups and thus, libcontainer replaced LXC as the default execution enviornment of Docker.

- There where no standards or guidelines for container software, The Open Container Initiative or OCI was formed.

- OCI Components:
    - Image-Spec
        - Image specification defines the standards for Image of the containers.
    - Runtime-Spec
        - Runtime Specification defines the life cycle of the container technology, such as create command should create the container, start command should start the container, delete command should delete the container.

- Until then, Docker daemon was a single monolithic code base which is managing lots of functionality all together e.g. Running containers, managing volumes, networks and images. Pushing and pulling images was also a part of that functionality.

- In version 1.11(2016), with the OCI standards in place, the architecture of the Docker Engine was restored and broken down into smaller reusable components.

- The part that run containers became an independent component known as runC.

- The part that manages containers became an independent component known as Containerd. It now manages runC which internally uses libcontainer to create containers and run it on the host.

- What happpend if the Docker Daemon is down? who will manage the containers? the component known as containerd-shim was added to make container daemon less. Instead of containerd creating containers, it's the containerd-shim that does it and monitors the state of containers. Even if the Docker daemon shuts down or it is restarted the containers run in the background and is attached back when the daemon comes back up.

## Docker Objects

- Docker has 4 main objects:

    - Image
        - Docker Images are read only templates which are used to create containers.
        - Images could be base images like os images or application like webserver or database images.
        - we can create our own custom application image.

    - Container
        - Container is running instance of an image. We can create, start, stop, delete the container. it's a read write running template.

    - Network
        - Docker network is providing various solution to communicate containers internally or with outer world.

    - Volume
        - Containers are ephemeral in nature, which means the data inside a container is not persistent and is bound with the lifecycle of a container. When container dies, the data that was created by the container also dies along with it. Volume will help to persist data across container restarts.

## Registry

- Stores image and can be shared publicly as well.

- While Enterprise version, we can create private repository as well which is know as DTR or Docker Trusted Registry.

- Image can be pulled or pushed to the registry.

## Docker Engine Components involvement while creating container

- When we execute command in CLI to create container **docker container run -itd ubuntu**

- Docker Client converts the command into a **RESTful API**, which then passed to the Docker Daemon.

- **Docker daemon** receives the instruction, first it will check whether ubuntu image is available locally or not, if it is available then it will be used, if not available then it will connect to default docker register **DockerHUB** and download the image.

- Then it makes call to **containerd** to start the container. Containerd is responsible for converting the image that was downloaded into an OCI compliance bundle.

- It then passes the bundle to **containerd-shim**, which in turn calls the **runC** to start a container.

- **runC** interacts with the **namespaces** and **cgroups** on the kernel to create a container and that's how container gets created.

## Commands

- **docker version** --> show Docker Engine (Client/ Server) Version

- **docker --version** --> CLI version of Docker Client

- **docker system info** --> To get more information about Docker hosts like the number of containers running, paused or stopped, the Docker server version, storage drivers, file systems, the configurations, default configurations etc.
