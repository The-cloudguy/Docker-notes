## Docker Networks

- When we install docker, it creates 3 networks automatically. **Bridge, None and Host**. Bridge is the default network a container gets attached to.

- If we like to associate the container with any other network then:

    ```
    docker container run ubuntu --network=<network name>

    e.g.

    docker container run ubuntu --network=host
    ```

- **Bridge network** is private internal network created by docker on the host. All containers attached to this network by default and they get internal IP address usually in the range 172.17 series. The containers can access each other with internal IP if required and if we want to access containers from outside we can map the ports of these containers to ports on the docker host.

- **Host network** takes out any network isolation between the docker host and the docker container. Meaning, if we want to access webapp on port 5000 then we don't need to map anything, webapp will be accessible on 5000 port of host. Drawback of this network type is we can't run multiple containers on the same port because on the host we can't map 2 containers on same ports.

- With the **None network**, the containers are not attached to any network and doesn't have any access to the external network or other containers.

- We can create custom network,

    ```
    docker network create --driver <driver name> --subnet <CIDR range> <Network name>

    e.g.

    docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network

    docker network ls --> list docker networks

    docker network rm <network name> --> Delete docker network 
    ```

## Embedded DNS

- Every container can connect to other container by its name or IP. IP is not recommended because after reboot of host, IP might change.

    ```
    If webapp wants to connect mysql database:

    mysql.connect( mysql ) --> here mysql is container name
    ```

- Docker has built-in DNS server which helps the containers to resolve each other using the container name. Note that built-in DNS server always running at address 127.0.0.11.

- Embedded DNS will not work with default bridge network. To use DNS, create user defined network.

## How does docker implement networking?

- Docker uses network namespaces that creates a separate namespace for each container. It then uses virtual ethernet pairs to connect containers together.

## Additional Commands

- To inspect network

    ```
    docker network inspect <network id>
    ```

- To connect/disconnect a container to a custom network

    ```
    docker network connect <network name> <container name>

    docker network disconnect <network name> <container name>
    ```

- To remove all unused custom networks (Except default networks)

    ```
    docker network prune
    ```