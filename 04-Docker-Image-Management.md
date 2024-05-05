## Image Registry

- It is the online place where contaier images are stored. It can be private or public registry.

    - Docker Trusted Registry
    - Google Container Registry
    - Amazon Container Registry
    - Azure Container Registry

- Images are having tags, which we can use while pulling the image.

    ```
    docker run ubuntu --> if we don't define tags, docker will pull latest image

    docker run ubuntu:18.04 --> pulling specific version of image

    docker run ubuntu:trusty
    ```

- To View images

    ```
    docker image ls --> list images
    ```

- To search images on registry

    ```
    docker search httpd --> Search for httpd images (at the time 25 images it will show)

    docker search httpd --limit=2 --> limit the search result (max 100 images we can limit)

    docker search --filter stars=10 httpd --> Show only those images which are having 10 stars

    docker search --filter stars=10 --filter is-official=true httpd --> Show only those images which are having 10 stars and those are official images
    ```

- To pull Images

    ```
    docker image pull httpd --> Download the image on local machine
    ```

## Image Adressing Convention

- Image Addressing Convention is as below 

    ```
    image : docker.io / httpd / httpd  :   latest
               |        |         |           |
            Default    User/      Image/     Tag
            Registry   Account    Repository
    ```

    e.g.

    ```
    docker image pull gcr.io/httpd/httpd:latest
    ```

## Authenticating to Registries

- Pull image from default/public registry, we don't need to authenticate.

- Pull image from private registry directly will throw an error. So for that we need to first authenticate to that registry.

    ```
    docker login docker.io --> login to docker.io registry

    username:
    password:

    docker login gcr.io --> login to gcr.io registry

    username:
    password:
    ```

    - After login to registry we can push images to remote registry.

- We can retag the image as per our requirement. Tag is just a soft link to the actual image having the same image id.

    ```
    docker image tag <image>:<older tag> <image>:<newer tag>

    e.g

    docker image tag httpd:latest httpd:customv1

    docker image tag httpd:latest gcr.io/<company>/httpd:customv1 --> if we have repository in Google container registry the we need to tag like this in our company and then push
    ```

- To see the actual size of docker objects such as Images, Containers, Volumes and Build Cache

    ```
    docker system df
    ```

## Removing Images

- We can't remove images those are used by containers. First we need to stop container, then remove container and then we can remove image.

- If its not in use we can remove image by using below command.

    ```
    docker image rm httpd:latest
    ```

- If we remove retagged image then it won't delete the layers of image because it's just a soft link to the original image. We need to remove original image to remove all the layers of the image and reclaim some space.

- To delete all unused images

    ```
    docker image prune -a
    ```

- To see all the layers of image along with all the commands which are used to create that layers

    ```
    docker image history <image name>
    ```

- To inspect the image which will show Parent/Base Image, Exposed ports, author details, size, All configs in Dockerfile etc

    ```
    docker image inspect <image name>
    ```

## Image save and load

- In restricted network or where the internet is not working, its hard to pull images. In that case we can save image into **tar** file and then store it in pendrive or somewhere where all machine can access and then download or copy that tar file in other machine and restore it as image.

    ```
    docker image save alpine:latest -o alpine.tar

    docker image load -i alpine.tar
    ```

## Import and Export Operations

- We can convert containers into images in a **tar** format (Read only template). Say for we have container and we want to export it as tar format then

    ```
    docker export <container-name> > testcontainer.tar

    docker image import testcontainer.tar newimage:latest --> import tar file as image
    ```

## Create your own custom image

- Dockerfile : Instruction-Argument format

    ```
    FROM Ubuntu
    RUN apt-get update && apt-get -y install python
    RUN pip install flask flask-mysql
    COPY .   /opt/source-code
    ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
    ```
- once create this Docker file build the image using this file

    ```
    docker build . -f Dockerfile -t <Registry>/<account name>/<image name>:<tag>

    if we want to push the image to remote repository

    docker push <Registry>/<account name>/<image name>:<tag>
    ```
- Every line of instruction will be creating new layers in docker image with just the changes from the previous layer. Each layer is having its on size independently. We can see layer instructions and size in **docker image history** command

- Layered architecture helps you restart Docker build from that particular step in case it fails or if you want to add new steps in the build process, you wouldn't have to start all over again because all the layers built are cached by Docker.

## Instruction

- **FROM** --> to choose base image
* **RUN** --> where you can execute command while building the image
- **EXPOSE** --> your container is exposed this port for your application
- **CMD** --> Instruction which will run after your container created
- **ENTRYPOINT** --> It will be executable for CMD
- **LABEL** --> This instruction adds metadata
- **ADD** --> This instruction is used to copy the contents from local system and URL. It will take 2 arguments which are source and destination
- **COPY** --> This instruction is used to copy the contents only from local system. It will take 2 arguments which are source and destination
- **VOLUME** --> create a volume
- **ENV** --> Set Enviornment Variables during image build which will be keep while creating containers
- **ARG** --> Set arguments which will only work while buiilding image
- **WORKDIR** --> select the working directory
- **USER** --> Run container with the user which is specify after this instruction.

## Docker Container Commit

- What if we want to create image from modified container? First we need to use existing image then create a container from that image. Login to that container and modified its files then commit the change using below command.

    ```
    docker container commit -a "author name" <existing image> <new image>

    e.g.

    docker container commit -a "guy" httpd customhttpd

    To add any instruction while commiting:

    docker container commit -a "author name" -c 'CMD ["httpd", "-D", "FOREGROUND"]' <existing image> <new image>
    ```
- Now we can create new version of container from modified image.

- But this is not the recommended way to modified the image, we should always go for updating Dockerfile and create new image and use it to create new container. This way we can record our changes in Dockerfiles.

## Build Context

- When we are building an image at that time we are giving a path in the command where docker will see the Dockerfile and supporting files.

    ```
    docker build .  --> In this . is the build context

    docker build /opt/my-custom-app --> In this /opt/my-custom-app is the build context
    ```

- When the image build command runs, it will transfer files under the build context at a temporary directory under the docker's filesystem at **/var/lib/docker/tmp**. Now remember this transfer will happen always even if the docker daemon is on the same host or different.

- It is recommended to have limited files under build context to reduce transfer time and time which required to build image and disk space as well. If we want to ignore some of the files or directories then add list of files and directories name into **.dockerignore** file, so those won't copy or transfer to **/var/lib/docker/tmp**.

- We can also specify code repo URL as build context, in that case first code will be clonned and then image will build.

    ```
    docker build https://github.com/myaccount/my-app

    docker build https://github.com/myaccount/my-app#<branch>

    docker build https://github.com/myaccount/my-app:<folder>

    docker build -f Dockerfile.dev https://github.com/myaccount/my-app
    ```
    - Docker build command expects the Dockerfile under the build contexts with the instructions to build the image.

## Build Cache

- When **docker image build** command runs, it will run each instruction wise and create a layer for that and cache them. So if image build fails at some layer then next time it will use that cache and will start from failed layer.

- In future, if we are building image then how docker will know whether to use cache or not? 

    - First thing docker will see is **change in instruction sets**. If any changes happend in instruction set then that layer and following all layers will become invalid and docker will use cache of those layers which are not invalid and build new layers which are invalid.

    - Second thing docker will see is **checksum of files in ADD and COPY instruction**. If we modify the file so in that case we need new content should be reflect in our image. So when modification happen in file, its checksum will change and this change make that layer invalid and docker build new layer for that invalid layer in next build.

- Technique of forcing docker to update package list before installing packages by combining two instructions into one is known as **cache busting**.

- When you specify multiple packages , it is a best practice to add them each in a new line for better readability as well as to be sorted in an alphabetical order which would be helpful to locate packages and not have duplicates when you have too many packages to install.

    ```
    e.g.

    RUN apt-get update && apt-get install -y \
        python \
        python-dev \
        python3-pip
    ```

- We can also pin a package to a particular version, its called **Version Pinning**. This technique can also reduce failures due to unanticipated changes in required packages.

    ```
    e.g.

    RUN apt-get update && apt-get install -y \
        python \
        python-dev \
        python3-pip=20.0.2
    ```

- Remember that the instructions related to most frequently modified code such as our application code must always be at the bottom of the Docker file, and those that are least frequently modified code such as package update or installation of packages must always be at the top of the Docker file. It will save time for building image.

## Difference between COPY and ADD

- COPY and ADD instruction both copies files from local file system to containers or the image file system.

    ```
    Dockerfile:                                            Dockerfile:

    FROM centos:7                                          FROM centos:7
    COPY /testdir /testdir                                 ADD /testdir /testdir
    ```

- However, ADD is having some more features compare to COPY

    - ADD is also use to copy tar file and untar it in destination container file system.

    ```
    Dockerfile:

    FROM centos:7
    ADD app.tar.xz /testdir
    ```

    - ADD is also use to download content from URL.

    ```
    Dockerfile: (More Instructions)                             Dockerfile: (Best way - less instructions)

    FROM centos:7                                               FROM centos:7
    ADD http://app.tar.xz /testdir                              RUN curl http://app.tar.xz \
    RUN tar -xJf /testdir/app.tar.xz -C /tmp/app                    | tar -xcJ /testdir/app.tar.xz \
    RUN make -C /tmp/app                                            && yarn build \
                                                                    && rm -rf /testdir/app.tar.xz
    ```

## Difference between CMD and ENTRYPOINT

- CMD is the instruction which shows which process should be run or which argument should be passed.

    ```
    docker container run ubuntu sleep 10 --> so here sleep 10 is CMD which we are passing
    ```

- If we want to run an container which created and sleeps after 10 seconds.

    ```
    Dockerfile:

    FROM ubuntu
    CMD sleep 10
    ```
    - We can define CMD in shell command or in json format.

        - CMD sleep 10 or CMD ["sleep", "10"]
        - In json format first part always must be command or executable.

- If we want to run container with only argument as executable should be hardcoded inside image.

    ```
    Dockerfile:

    FROM ubuntu
    ENTRYPOINT ["sleep"]

    Now we can run command as below:

    docker container run ubuntu-sleeper 10 --> we are only passing 10 as part of argument to our executable sleep
    ```

- If we don't give argument in above command then it will throw an error like operend is missing. In that case:

    ```
    Dockerfile:

    FROM ubuntu
    ENTRYPOINT ["sleep"]
    CMD ["10"]

    Now we can run command as below:

    docker container run ubuntu-sleeper --> here we are not defining anything in command, it will take from image

    docker container run ubuntu-sleeper 15 --> this will override value if CMD

    docker container run --entrypoint sleep2.0 ubuntu-sleeper 10 --> this will override entrypoint
    ```

## Base and Parent Image

- If image is build **FROM scratch** then its base image.

- If image is build **FROM ubuntu:xenial** or **FROM httpd** means it is not build **FROM scratch** those are parent image.

- There might be multiple parent image but always one base image.

    ```
    scratch             scratch             scratch
      |                    |                   |
    ubuntu(Base)        debian(Base)        debian(Base)
      |                    |                   |
    MongoDB             php(Parent)         php(Parent)
                            |                  |
                        Wordpress           Wordpress(Parent)
                                               |
                                            Custom-wordpress
    ```

- Scratch image is blank image which is docker's reserved image name. We can't create it.

- We can create base image by using below approach.

    ```
    Dockerfile: debian:buster-slim

    FROM scratch
    ADD rootfs.tar.xz /
    CMD ["bash"]
    ```

## Multi-stage builds

- If we have application which needs to be containerzied then below is the steps: (Hard Way)

    - First we build the artifact which will be use as executable for our image to run as container
    - This artifact needs to extract to local host
    - Artifact should be copy to our application container so it can be execute while starting up

- We can achieve this using multi-stage image build (Easy Way) (Cosider its nodejs applicaiton)

    ```
    Docker file:
    ## Stage 0 : Build Stage

    FROM node AS builder
    COPY . .
    RUN npm install
    RUN npm run build

    ##This will create artifact which needs to be copy into second image
    ## Stage 1 : Containerize for production

    FROM nginx
    COPY --from=builder dist /usr/share/nginx/html
    ##OR##
    #COPY --from=0 dist /usr/share/nginx/html
    CMD [ "nginx", "-g", "daemon off;" ]
    ```
    - now we can build this image using **docker build -t my-app .** command.

    - we can individually target stages using **docker build --target builder -t my-app .** command.

- **Pros**:
    
    - Optimize Dockerfiles and keeps them easy to read and maintain.
    - Helps keep size of images low
    - Helps avoid having to maintain multiple Dockerfiles - Builder and Production
    - No intermediate images

