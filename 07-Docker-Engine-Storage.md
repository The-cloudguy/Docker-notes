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

	- **Volume** --> First separate volume needs to be created then it should mounted into container. it will long last if container also removes, because it will created on host FS. When volume itself deleted then data will be lost.

        ```
        docker volume create <volume name> --> create volume first. Files related to volume will be stored in /var/lib/docker/volumes/<volume name>

        docker container run -v <volume name>:<PATH to container where to mount> mysql  --> older way to mount volumes

        docker container run --mount 'type=volume,source|src=<name of volume>,destination|dest|target=<PATH to container where to mount>' mysql --> Newer way to mount volumes

        ```
	
    - **Bind Mount** --> Mount Host directory in container as NFS shared directory. No volumes will be created.

        ```
        docker container run --mount 'type=bind,source|src=<Path on Host>,destination|dst|target=<PATH to container where to mount>' mysql
        ```
	
    - **tempfs** ---> when mounted in Container, data stored in Host RAM. But it will not persist longer. Once container host rebooted data will be gone

        ```
        docker container run --mount 'type=tempfs,destination|dest|target=<PATH to container where to mount>' nginx:latest
        ```

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

## Additional Commands

- **docker volume ls** --> List all volumes

- **docker volume inspect [volume ID]** --> Inspect the volume

- **docker volume remove [volume ID]** --> Remove the volume

- **docker volume prune** --> Remove unused volumes

- **docker container run --mount 'type=volume,src=data_vol1,dst=/var/www/html/index.html,readonly' httpd** --> Mount readonly volume into container