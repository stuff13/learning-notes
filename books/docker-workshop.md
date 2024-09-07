# [Docker Workshop](https://www.packtpub.com/en-gb/product/the-docker-workshop-9781838983444)
I am currently reading this book. These notes are therefore incomplete

- [Chapter One: Running My First Docker Container](#chapter-one)
- [Chapter Two: Getting Started with Dockerfiles](#chapter-two)
- [Chapter Three: Managing Your Docker Images](#chapter-three)
- Chapter Four
- Chapter Five
- Chapter Six
- Chapter Seven
- Chapter Eight
- Chapter Nine
- Chapter Ten
- Chapter Eleven
- Chapter Twelve
- Chapter Thirteen
- Chapter Fourteen
- Chapter Fifteen


## Chapter One

To see running containers
`docker ps`
- to display all the containers, even stopped ones
`docker ps -a`
To view the images stored locally
`docker images`

### Managing Docker Containers
#### docker pull
- downloads a container image to the local cache
`docker pull <name>`
#### docker stop
#### docker start
#### docker restart
#### docker attach
- allows users to gain access to the primary process of a running Docker container instance
- using **exit** will end the bash shell AND close the container
- If you want to quit the shell, use ctrl-p then ctrl-q to exit without closing the container.
#### docker exec
- executes a command inside a running container
`docker exec <command>`
`docker exec -it <name> /bin/bash`
- uses your current command prompt as a bash shell for your docker container
- `exit` returns to the normal shell
#### docker rm
- deletes a stopped container
#### docker rmi
- deletes a container image
`docker rmi -f $(docker images -a -q)`
- to delete all containers
#### docker inspect
- shows verbose details about the state of a container
#### docker run
- `-i` makes the session interactive
- `-d` tells the docker to run the container in the background (daemon mode)
	- without `-d`, the current terminal will be taken over by the container
- `-t` allows bash to run in interactive mode 
- `--name` gives the container a human-readable name
#### docker system prune
`docker system prune -fa`
- removes any container images not tied to an existing running container

### PostGres CLI commands
- Logging In
`psql --username <username> --password`
- Listing the database
`\l`
- Quitting the PSQL shell
`\q`

## Chapter Two
## Getting Started With Dockerfiles

A *Dockerfile* is a text file that contains instructions (*directives*) on how to create a Docker image.
```
	# This is a comment
	DIRECTIVE argument
```
*Directives* are case in-sensitive, but by convention, we distinguish them from arguments by writing them in all-caps

### Common Directives

#### FROM
`FROM <image>:<tag>`
- specifies the parent image for the build
#### LABEL
- `LABEL <key>=<value>`
`LABEL version=1.0
- can include multiple labels on a single line by separating them with spaces
#### RUN
- `RUN <command>`
- executes commands during image build time
	i. creates a new layer on tome of the existing layer
	ii. executes the specified command
	iii. commits the results to the newly created layer 
`RUN apt-get update`
- best if we chain commands in a single line:
`RUN apt-get update && apt-get install nginx -y`
#### CMD
- used to provide the default initialization commandexecuted when a container is created
- a dockerfile can execute only one CMD directive
- with multiple, only the LAST one is executed
- `CMD ["executable", "params1", "params2", "params3",...]`
- `CMD ["echo", "Hello World"]`
- command arguments passed with the docker run command will take precedence over this directive
- command is run after the image is launched
#### ENTRYPOINT
- also used to provide initialisation
- can only be overriden by using the --entrypoint flag with the docker container run command
- `ENTRYPOINT ["executable", "params1", "params2", "params3",...]`
- when used with CMD, CMD holds arguments for ENTRYPOINT
- with no ENTRYPOINT set, the default ENTRYPOINT is run: `/bin/sh -c`, allowing the first CMD parameters to be run as a command

### Building Docker Images

- `docker image build <context>`
`docker image build .`
- `.` means that the Dockerfile should be found in the current directory
- `-t`: add a tag to the image
`docker image build -t my-tagged-image:v2.0`
- my-tagged-image == repository name
- v2.0 == tag name
`docker image list`
- list the available Docker images

### Other Dockerfile Directives

#### ENV
- sets environment variables
- `ENV <key>:<value>`
`ENV PATH $PATH:/usr/local/bin/`
- multiple environment variables can be set with one line by separating them with spaces
- once set, available in further layers and containers launched from this image
#### ARG
- `ARG <varname>`
`ARG VERSION`
- used to define variables the user can pass at build time
- only directive that can precede the FROM directive
- can have a default value set
`ARG VERSION=1.0.0`
- can be set from the build command: `docker image build --build-arg <varname>=<value>`
#### WORKDIR
- used to specify the working directory of the docker container
- `WORKDIR <path to container>`
- if the directory does not exist, mkdir will create the directory
#### COPY
- copies files from the local file system to the docker container
- `COPY <source> <destination>`
`COPY *.html /usr/www/html/`
- **--chown** can be used to specify ownership
> `COPY --chown=myuser:mygroup *.html /usr/www.html/`
#### ADD
- similar to COPY, but allows us to use a URL as a source
- `ADD <source> <destingation>`
`ADD http://sample.com/test.txt /tmp/test.txt`
- compressed files added are automatically uncompressed
#### USER
- Docker uses the root user as the default user of a Docker container
- The Dockerfile can use USER to specify which user should be used
- `USER <user> [<group>]`
- these need to be valid users and groups
`USER www-data`
- The default user for Apache web server is www-data
#### VOLUME
- In Docker, the data generated and used by Docker containers will be stored within the container filesystem
- If we delete the container, the data is lost
- We can use the VOLUME directive to specify a volume outside the container to separate the data from the container and even share data between containers
- `VOLUME ["<path>"]`
- a sequence of paths sparated by spaces can be used to create multiple volumes
- To inspect volume information: `docker container inspect <container>`
- to display detailed information about the volume: `docker volume inspect <volume>`
#### EXPOSE
- used to inform Docker that the container is listening on the specified ports at runtime
- `EXPOSE <port>`
- these ports are only available from within other Docker containers
- to expose ports outside the docker container, we publish the ports with the -p flag
`docker container run -p <host_port>: <container_port> <image>`
- when using the -p command, we ALSO have to use the EXPOSE directive
- the path is specified for within the container and Docker sets it somewhere on the host system
# HEALTHCHECK
- `HEALTHCHECK [OPTIONS] CMD <command>`
- only the last HEALTHCHECK option in a Dockerfile will take effect
`HEALTHCHECK CMD curl -f http://localhost/ || exit 1`
- the exit value at the end dictates the health: 0 == healthy, 1 == unhealthy
- **--interval**: specifies the period between each health check (default 30s)
- **--timeout**: if no success response isreceived within this period, check fails (default 30s)
- **--start-period**: how long to wait before running the first health check (0s)
- **--retries**: HEALTHCHECK will retry this many times, and only fail after all of them (3 tries)
#### ONBUILD
- creates a reusable docker image to be used for another docker image
- `ONBUILD <instruction>`
`ONBUILD ENTRYPOINT ["echo", "Running ONBUILD directive"]`
- does not run when built, only runs when this image is used for a child image

#### Sample PHP Dockerfile:
```
	FROM ubuntu:18.04
	LABEL version=1.0.0
	ENV DEBIAN_FRONTEN=non-interactive
	
	RUN apt-get update && apt-get -y install apache php curl
	COPY *.php /var/www/html
	WORKDIR /var/www/html
	
	HEALTHCHECK --interval=5s --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
	
	EXPOSE 80
	ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]
```

## Chapter Three
## Managing your Docker Images