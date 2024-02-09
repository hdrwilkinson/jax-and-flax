# Instructions

## Creating an Image

Firstly, create a Dockerfile (no extension) as we have done. We have followed the following steps to create a Flask application:

1. Initiates the OS or image on which to base the image (e.g. ubuntu)
2. Updating the apt repo
3. Installing dependencies using apt
4. Install python dependencies using pip
5. Copy source code to a folder
6. Run the webserver using the flask command

Using the command `docker build -f Dockerfile -t [username]/[desired_image_name] .` you can create the image. We have used `-f` to specify the name of the Dockerfile (not necessary in this case when the Dockerfile is called Dockerfile) and `.` to specify that the location is the current folder.

To push the image that has just been built to the registry simply use the command `docker push [username]/[desired_image_name]`. Note that you must be logged in to do this. To gain temporary access to docker through the terminal use `docker login`.

To view the history of the build use `docker history [username]/[desired_image_name]`. This can also be used to view what 'breaks' in the process of building an image. Each 'layer' of the build is built on the previous layer and so it is simple to view the process.

## Networking

Docker creates 3 networks automatically: 'bridge', 'none' and 'host'. By nefault, when creating a container, it attaches to the 'bridge'. However, you can attach it to other networks e.g. `docker run [image_name] --network=none` (or `host`).The network information can be found by inspecting a container.

Containers attached to the 'host' are automatically accessible to the same port externalls as internally i.e. they are fixed. This means, for example, that you can't run multiple web contaienrs on the same 'host' and port as the ports are now common to all containers in the 'host' network.

In the 'none' network, the containers don't have any external access i.e. can't access other containers. Ther ay isolated.

All containers attached to 'bridge' get an internal IP address (usually in the 172.17 series e.g. 172.12.0.1:5000). Containers attached to the 'bridge' can be accessed by other containers (through ports) in the 'bridge' or 'host'. Though, they are not directly attached to the 'host' (think of them as floating containers, waiting to be ported to the 'host'). In the 'bridge' network, you can create multiple IP addresses to attach containers to. This is done through the following command `docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network`, where we have created a subnet (182.18.0.0/16) with a custom name (custom-isolated-network) on the bridge network. You can see all networks available through the `docker network ls` command.

### Embedded DNS Server

Within a docker host the embedded DNS server stores the 'names' associated with the IP addresses. This makes communication between instances easier because you can use the simplified name rather than the IP address to connect to it.

Each network has it's own DNS server for tracking the container names (think of it like looking up someone via a phone book).

## File System

Docker has a lyered architecture with the base file system within a docker application being:

```
/var/lib/docker
  /aufs
  /containers
  /images
  /volumes
  /etc
```

Docker images are created with each layer existing on-top of the previous layer. This simplifies the creation and storage of containers and images. Each container uses the cache from the previous creation and then can contain unique information that would overwrite the information held in the image on which it is based (if it exists, else it is simply created).

The image layers are 'read only' whereas the container layers are 'read write'. However, if a container is removed then the unique elements associated with that are lost. To stop this happening, you can create 'volumes' which exist within the container itself, and then attach the container elements to it e.g. a mysql database.

### Volume Mounting

Yout can mount a container (e.g. mysql) to a volume. For instance, to mount a mysql database use the following command `docker run --mount type=volume, source=[volume_name], target=/var/lib/mysql mysql` (or, in previous versions `docker run -v [volume_name]:/var/lib/mysql mysql`).

Alternatively, if you already had the information stored externally on the host you can 'bind' the container to it (i.e. you don't want to store it in the default /var/lib/volumes/ folder). This can be done through the command `docker run --mount type=volume, source=[storage_path], target=/var/lib/mysql mysql` (or in pervious versions `docker run -v [storage_path]:/var/lib/mysql mysql`).

## Composing

To set up an application you can per form a series of `docker run ...` commands. However, it can be convoluted to properly link them and can seem redundant. It is far simpler to create a docker-compose.yaml file and then run that using `docker-compose up`. An example of a docker-compose.yaml file is:

```
version: '3.0'
services:
  redis:
    image: redis-alpine
  clickcounter:
    image: kodekloud/click-counter
    ports:
    - 8085:5000
```

This sets up two containers called redis and clickcounter on their respective images and allows the user to intersct with the application through port 5000 i.e. localhost:5000.

## Commands

'Run Container' Command: `docker run [image_name]`

 - Runs a container of an image on the local machine if it already exists. If it doesn't it will search the docker hub, and, if it exists, it will 'pull it down' and run the applicaiton.
 - To run a container in the background, use `docker run -d [image_name]` i.e. in a 'detached' mode.
 - To specify a tag (e.g. a version) simply add it at the end after a ':' `docker run [image_name]:[version_number]`. Docker will upload the latest version within the 'major', 'minor', and 'patch' system e.e. if you specifc the major and minor, it will install the latest patch release associated with that major and minor release.
 - To map between ports from (host e.g. 80)/to (container e.g. 5000) use `docker run -p 80:5000 [image_name]`
 - To map data from container to host source (avoids deleting data when removing container) use `docker run -v [host_dir]:[container_dir] [image_name]`
 - To run a container with a specifric name use the `--name [desired_name]`  flag.

'List Containers' Command: `docker ps`

 - Lists all currently running containers. 
 - To see all containers, running or not, add `-a` at the end of the command.

'Stop Container' Command: `docker stop [container_name]`

 - Each container will have a unique name. To stop a container, insert that unique name into the command above. 
 - Note, that this does not delete the container, it simply stops it.

'Remove Container' Command: `docker rm [container_name]`

 - To permanently remove a contaienr, use the above command. 
 - If it prints the name back, then the container has been removed.

'List Images' Command: `docker images`

 - Displays a list of images available on host machine.

'List Networks' Command: `docker network ls`

 - Displays a list of availale networks.

'Remove Image' Command: `docker rmi [image_name]`

 - Removes an image from the host. 
 - You must ensure that no containers are running off this image before attempting to remove it (they must be removed, not stopped).

'Download Image' Command: `docker pull [image_name]`

 - Downloads an image but doesn't run it as a container.

'Appending or Extending' a Command: `docker run [image_name] [command] [param]`

 - Docker is built to host a container whena process is running. As ubuntu is an OS and has no processes running, the command `docker run ubuntu` would exit very shortly after it starts. Adding the `sleep 5` tells the application to sleep for 5 seconds after setting up the container. Then it would exit, as before.
 - Dockerfiles, the file on which docker images are built contain a 'CMD' line at the end e.g. `CMD ['command1', 'param1', 'command2', 'param2']`. This tells the application what to run when the container is set up. By adding a command into the terminal here, we are setting the 'CMD' list.
 - The difference between 'CMD' and 'ENTRYPOINT' is that for 'CMD' they are replaced, whereas with 'ENTRYPOINT' they are appended.
 - As an example, if you're wanting to run an ubuntu container that sleeps for 10s, but you don't want to have to specify `sleep 10` everytime, you can add an entrypoint into the Dockerfile e.g. `ENTRYPOINT ['sleep']`, that way you can simply run `docker run ubuntu-sleeper 10` and the `10` is appended onto the end of the entry point list e.g.  `ENTRYPOINT ['sleep', '10']`.
 - If both 'ENTRYPOINT' and 'CMD' are both present then 'CMD' is appended to 'ENTRYPOINT'. But, this can be overwritten. As an example if you had `ENTRYPOINT ['sleep']` and `CMD ['5']` if you used `docker run ubuntu-sleeper 10` then 'CMD' is removed and 'ENTRYPOINT' becomed `ENTRYPOINT ['sleep', '10']`.
 - If you want to rewrite the 'entrypoint' from the command line then you can use the `--entrypoint` flag e.g. `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`.

'Exec' Command: `docker exec [container_name] cat /etc/hosts`

 - Executes a command on a running container. 
 - In the example above, the command `cat /etc/hosts` would allow you to see the contents of the file within the container.

'Volume' Command: `docker volume create [volume_name]`

 - This creates a 'data_volume' folder within the file system.

'Attach' Command: `docker attach [container_name or the first few characters of the container_id]`

 - The simple run command `docker run [image_name]` runs a container in the foreground or in the 'attached' mode i.e. it is attached to the console. To exit, you must also exit the container. 
 - However, if you run the container in a detached mode, then it runs in the background, freeing up the terminal. The 'attach' command allows the user to reattach to the container.

 'Prune Images' Command: `docker system prune -a`

 - Where `-a` works for all images, not just dangling ones.

This can also be done in stages by performing actions on all images using the following commands where `-aq` means 'all' and 'quiet' i.e. don't print everything:
 - `docker stop $(docker ps -aq)`
 - `docker rm $(docker ps -aq)`
 - `docker rmi $(docker list -aq)`

'Inspect' Command: `docker inspect [container_name]`

 - This displays more information and details about the container.

'Logs' Command: `docker logs [container_name]`

 - This displays the logs from a container.

'History' Command: `docker history [image_name]

- Prints out the history of the image including it's building.

Interactive Mode:

 - To run a container in interactive mode add `-i` in the command e.g. `docker run -i [image_name]`
 - To run a container in interactive mode within the terminal add `-it` in the command e.g. `docker run -it [image_name]`

'Tagging' Containers: `docker run [image_name]:[tag]

- Use this to add tags to images where applicable.

## Additional Points

### Port Mapping

Imagine you're operating a mailroom in a large building, and each department within the building is like a separate application running in its own Docker container. The building itself represents the Docker host. Now, this building has its main address (the host's IP address) and each department has its own internal room number (similar to the port inside a container).

Port mapping in Docker is like instructing the building's receptionist on how to direct external mail (incoming network traffic) to the correct department based on the room number listed on the mail (the container's internal port), even though the mail only has the main building address (the host IP).

For example, if you have a web server running in a container that listens on port 80 internally, but you already have another service using port 80 on the Docker host, you can tell Docker to map an unused external port (say, 8080) to the internal port 80 of the container. This way, when someone sends a request to the Docker host on port 8080, Docker knows to forward that request to port 80 in the specified container, just like the receptionist knows to deliver mail for "Room 8080" to "Room 80" in the correct department.

### Volume Mapping

Data stored (e.g. a table within a mysql container) is removed when a container is removed. To maintain data regardless of the containers existence you can use 'Volume mapping'. Volume mapping stores the container data in a host data location.

### Environment Variables

To set an environment variable for a container add `-e [env_variable_name]=[env_variable_value]`. To find the environment variables associated with a running container use the 'inspect' command and you will find them in 'Config > Env'.