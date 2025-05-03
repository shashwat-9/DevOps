# Docker for Beginners

### Docker Overview

Containers are completely isolated environments, with their own processes, Network interfaces, Mounts, but they share the
same OS Kernal.

Some of the different types of containers are LXC, LXD, LXCFS. Docker utilises LXC containers.

Setting up these container environments is hard, as they are very low-level, and that is where Docker offers a high-level
tool with several powerful functionalities, making it really easy for us.

Fundamentally, all Linux OS have the same Kernal, and above this Kernal different applications are placed, for user interaction,
developers' tools, etc. making the OS different, like Fedora, Ubuntu, etc. So some custom software on the same kernal
differentiates these Linux Distributions.

Let's suppose a docker container is placed over an ubuntu OS, sharing the kernal, it means the docker can run any flavor
of Linux OS, be it fedora, Debian, SuSE, or CentOS

### Containers vs Virtual Machine

Virtual Machines have OS along with Libraries, Dependencies, and applications. VMs lie over hypervisor, which is placed
over the Hardware infrastructure.

Containers, on the other hand, have only the Libraries, Dependencies, and application, which is placed over the OS, sharing
it. The containers run on docker placed over the OS.

![image](./)

The containers have less utilization, consume less size, and so boot up faster. It has less isolation.
The VMs on the other hand, have the opposite nature to the mentioned params.

Containers could be deployed in VMs, thereby optimizing the cost.

Lots of containerized applications are readily available, on a public repository called dockerhub or dockerstore, including
softwares like OS, DB, many other services.

Running them on a m/c with `docker run` will run the instance of the application.

Image vs Container

Image is a package or template, just like a VM template. Containers are running instances of these images, have their own
environments and set of processes.

### Two editions of Docker

1. Community - Free
2. Enterprise - for managing enterprise grade docker container.

## Docker Commands

### Basic Docker Commands

#### docker run <name>
- starts a container of the given image.
- Run a container from an image, already on docker host, if it already exists, or it will pull it from dockerhub for the
  first time.
  e.g. `docker run nginx`

#### docker ps - list containers
- This cmd lists the name, command, created at, status, ports etc of the docker image running.

#### docker ps -a
- It lists all the information about the currently running, previously stopped or exited containers.

#### docker stop <given_name>
- It stops the docker container with the passed name.
- It will output the name on success.
- checking with 'ps -a' will ensure the container is exited or not.

### docker rm <given_name>
- If it prints the name back, it's successful.
- Remove a container, whatever is lying around and consuming space.
- It doesn't clear the image present on the host.

### docker images
- lists all the images present on the docker host

### docker rmi
- removes image from the docker host
- you must stop the container before deleting the image

### docker pull <name>
- downloads the image from dockerhub, or dockerhost

* Running docker run ubuntu, will fail instantly, as docker isn't supposed to run an OS. It is supposed to run specific
  tasks, webservers, or application or DB or just a basic task.
* once the task completes, it exits the container.

### Append a command
- We can append a cmd, like, `docker run nginx sleep 5`
- sleeps for 5 seconds

### exec - executes a command
- executes a cmd on a running container
  example : `docker exec distracted_mcclintock cat /etc/hosts`

### Run - attach and detach

- `docker run kodekloud/simple-webapp`, by default the container is attached with the standard output, thereby showing logs
- It will show the logs of the application

- `docker run -d kodecloud/simple-webapp`, to detach the standard output of the container.
- It will output an id of the container.

- `docker attach container_id`, to reattach the standard output with the container.

## Demo - Docker command
To use another distribution of Linux using docker image, we can use the following:

`docker run centos bash`
- Running just centos will terminate instantly, as no process is running.
- running the above cmd will open bash, and thus the container will keep on running.

Once a container is exited, it lives on the disk, consuming space.
`docker ps -a`

To remove the container, we can use:
`docker rm <container_name/Id>` , after stopping the container

To stop a running container, we can either pass the name or the container_id, in the `docker stop <image_id>` cmd.
Just providing a few first digit will remove the docker container.
Exit code will be different for containers stopped, and those exited normally.

`docker rmi <image_name>` - remove image from the docker host

- To run a cmd on an already running container, we can use `docker exec <container_id> cmd`

## Docker run

`docker run <image>`
- This will pull the latest image to run.

To run a particular version of an image, we can run the following:
`docker run redis:4.0`

- To use a docker container in an interactive mode, we can use the '-it' with it.
- Using only '-i' will make the container interactive
- The '-t' makes the prompt visible, that is the prompts are enabled. `docker -i run`
- To use the container interactively along with the message prompted, `docker run -it`

## Run - PORT mapping
The underlying host, where docker is installed is called docker-host or docker-engine.

Every docker container is assigned an internal ip, on the docker host. To access the containerized application within the
docker host, we can use the `http:<internal_ip>:port` in the internal browser. The port will be that of the application
server.

But to access the application from another machine, we should use the ip of the docker host and not the internal ip of the
container. And there must be a port mapping done with the port of the docker host and port on which the containerized application
is running.

The containerized application port is isolated from that of the docker host. This containerized application port should be
mapped with a port on docker host.

`docker run -p <host_port>:<containerized_application_port> <application_name>`

We can map many different ports of the docker host to the same containerized application, thereby creating multiple entry
points.

## RUN - Volume mapping
Whenever the container is stopped, the files, whatever is stored in the container, is also deleted.
To ensure the persistence of data, we can map the folder of the container where files are stored to an external volume on the disk.

`docker run -v <External_path>:<Internal_path> <image_name>`

## Inspect Container

`docker inspect <container_name>`
Returns all the details of the container in the json format.

## Container logs

`docker logs <container_image>`
To view the logs of a container, that may or mayn't have been run in a detached mode.

Task/Practice:
Testing the port mapping thing could be done with jenkins.

## Docker Image
### What am I containerizing?
- An application that is built using the python-flask framework.

### How to create my own image
- Create a new file, say docker file, write the instructions
```dockerfile
FROM ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

```
docker build <DockerFileName> -t username/tagname
docker push username/tag-name -> push to dockerhub
```

### Docker File
INSTRUCTIONS and ARGUMENT
- FROM <image> -> FROM is an instruction and image is the base docker image. Every docker file is based on some other docker image
  , be it OS or etc
- RUN pip install flask -> RUN cmd runs teh command on the base image
- COPY . /opt/source-code -> copy the file from local system to the base docker image
- ENTRYPOINT -> allows us to specify a cmd that will be run when the image is run

### Layered Architecture
- The dockerfile comes in layered format. The base image is followed by the instructions in the new file.
- When we run the docker build, the logs show the execution of each step, and it also caches each step, thus when running
  again, the process is started from where it was left.
- Thus will use the previous steps in the layer.

If the default organization is not defined in the docker, it will assume it to be library.

So, we can build the docker image with a tag, with our account name following the build name. <account_name>/<build_name>

And before pushing the build, we must login to the docker
``` docker login```
- Enter username
- Enter password

### Environment Variable
- To set an environment variable while running an image, we can use the following
```
    docker run -e abc=def <application_name>
```
- To find the currently assigned environment variable on a running container, we can use the following:
```
    docker inspect <docker_image>
```

## docker CMD vs ENTRYPOINT
#### Command Versus EntryPoint

- Containers are meant to run or host a specific task or process such as to host a web server/ Database/ carry out computation etc.
- Once the task is completed, the container exits. The container is alive as long as the process inside is alive.
- For e.g., when we run a ubuntu docker image, it exits, because there's a `CMD bash`, which requires a terminal to work.
  As the terminal is unattached by default, the program exits.

##### CMD Command
- This command in a docker file tells on what program should run in a container. e.g. `CMD nginx`, `CMD mysqld`
- To specify a different command to run a container image, we can append a command to the `docker run` command. This appended command overrides the default command.
- To make a different command run at start, than the default specified, we can create a new docker file, with the required `CMD <command>`
- `CMD command param 1` or `CMD ["command", "param1"]`

```
FROM ubuntu
CMD sleep 5
```

`docker build -t ubuntu-sleeper .`
`docker run ubuntu-sleeper`

##### ENTRYPOINT Command
- `ENTRYPOINT ["cmd"]` inside a docker file instead of `CMD`, makes the `docker run <tag>` open to arguments, i.e. `docker run <tag> <args>`. Earlier with `CMD`, this cmd `docker run <tag> <cmd> <args>` was supported.
- Example, `docker run <tag> <cmd> <args>` works with `CMD` inside docker file. `docker run <tag> <args>` works with `ENTRYPOINT`.
- If we're having a cmd like sleep with EntryPoint inside a docker file, it requires an args. If not passed in the CLI, it will through error. To get rid of this, we can add a default value inside the docker file using both `ENTRYPOINT` and `CMD`.

```
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]

```
- If specified, that one will be picked up. Else, the default will be picked up.
- If not passed, 5 will be picked up as an arg for the ubuntu cmd.
- To change the command altogether when ENTRYPOINT is used, we can use the following :
  `docker run --entrypoint sleep2.0 <tag> <arg>`


## Docker Compose
- Configurations are in yaml files.
- If our application requires multiple services to be running, we can use multiple `docker run` cmds, each for an individual
  components.
```
    docker run services(1)
    docker run services(2)
    ...
    docker run services(n)
```
- Or we can use a better way called docker-compose file, where we store the configurations in a yaml file.

docker-compose.yaml
```
    services:
        web: 
            image: "<username>/simple-webapp"
        database:
            image: "mongodb"
        messaging:
            image: "redis:alpine"
        orchestration:
            image: "ansible"
```

- We can use the `docker-compose up` cmd to up the entire application stack.
- Easier to store and maintain. It is however applicable on running all the containers on a single docker host.


#### Sample Application - Voting application
VOTING_APP(python) -> IN_MEMORY_DB(Redis)
|
v
Worker(.NET)
|
- - - - -  - - - - - - -
|
v
DB(Postgres) -> RESULT_APP(Node.js)

- We will see how the application stack is put up on the same docker engine, using the cmds and then the docker-compose.

```
    docker run --links
    This cmd is used to link different containers together. Like a container running worker application is required to
    link with container running Postgres DB.
    e.g. docker run -d --name=vote --link redis:redis voting-app
    
    This cmd will make the redis container visible to the voting app.
    This link argument have <Image_name>:<Host_name>
    What it does is, adds an entry in the /etc/hosts file on the voting-app
    Container, with the host name redis along with the internal ip of the container.
    This way of using links is deprecated in docker, and support maybe removed sooner.
    Advanced way exists in docker networking and swarm to do the same task.
```

1. Sequence of cmds
```
    docker run -d --name=redis redis
    docker run -d --name=db postgres:9.4
    docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
    docker run -d --name=result -p 5001:80 --link db:db result-app
    docker run -d --name=worker --link redis:redis --link db:db worker
```

equivalent docker-compose file
------------------------------
docker-compose.yml
```
redis:
	image: redis
db:
	image: postgres:9.4
vote:
	image: voting-app
	ports:
		- 5000:80
	links:
		- redis
result:
	image: result-app
	ports:
		- 5001:80
	links:
		- db
worker:
	image: worker
	links:
		- redis
		- db
```

- We create a key for each image name.
- Under each item, we specify which image and other parameters
- db:db = db, in context of the cmds and compose file
- To bring up the application stack, we can use the cmd `docker-compose up`
- If we want to build the application, than to pull, we can replace the image tag, with the
  `build : /path/to/application/with/docker/file`

### Docker compose version
#### Version 1
- The basic docker compose file
- Limitations include, no option to deploy the application than the default networking bridge.
- Further, there's no way to specify the ordering of the containers to getting up.

###### Version 2
- Create a property `services` in the root of the file. Under this Services, put all the configs under this service.
- Add the `version: 2` at the top of the compose file.
- Use the `docker-compose up` cmd to bring up the stack.

- Another difference is with networking. In version one, the docker-Compose attaches all the containers to the default-bridge network, and then use links between the containers as we did before.
- With version 2, the docker creates a dedicated bridge network for this application and then attaches all the Containers to that new network. All Containers communicate to each other using the service name.
- So in version 2, we can get rid of the `links` tag in the docker compose. Also, in version 2, `depends_on` tag is available, which specify the building/running order of the containers.
- Then there's version 3, the latest version, which comes up with the support of docker swarm. It will be discussed in much detail, when docker stack is discussed.

###### Networking
- So far, we've been deploying all the containers on the same default network bridge. Suppose we want to separate the user-generated traffic from the internal traffic like the backend, DB, cache, etc.
- We can use this property from version 2. Like the `services` key, we can use the `networks` key, and put the required network names under that.
- Then we can put the relevant network under the application key in the service key.

```
docker-compose file
----------------------
version: 2
services:
	redis:
		image: redis
		networks:
			- back-end
	db:
		image: postgres:9.4
		networks:
			- back-end
	vote:
		image: voter-app
		networks:
			- front-end
			- back-end

networks:
	front-end:
	back-end:

```

 - Docker compose is not installed by default, when docker is installed.
 - Whichever directory has the docker-compose file, will be suffixed before the container name when `docker-compose up`
is used.
 - From version 2 onwards, a dedicated network bridge is created, and all tns resolution is taken care of.
 - And therefore, no need to have `links` section in the docker-compose file.

### Docker Registry
 - It's a central repository of all the docker images.
 - `docker run <image_name>`, <image_name> is the name of the image that will be pulled. The image naming convention is
`<User/account>/<Image/repository>`. If we don't specify the <User/account> name, it will be taken as of the <Image/Repository>
name.
 - The username is the name of the user account on the dockerhub, or if it's an organization, then it's the organisation name.
 - If we don't specify from where the images are to be pulled, it is the default, central Docker hub, the DNS of which is `docker.io`
 - The registry is where you upload and image, or the update of it.
 - The other popular registries include, Google's registry(gcr.io).
 - Many cloud services, like AWS, GCP, Azure, etc. provide a private registry when you open an account with them,
 - We should always login to the private registry before running/pulling a docker image from there `docker login private-registry.io`
 - Then we can pull/run the image like `docker run private-registry.io/apps/internal-app`

What if you're running your application on-prem and don't have a private registry?
 - We can then deploy our own private registry, as registry is itself an application, an available as an image, ofcourse.
 - It exposes the api on port 5000
`docker run -d -p 5000:5000 --name registry registry:2`
 - We can use the docker image tag cmd, to tag an image with the url of the registry, `docker image tag my-image localhost:5000/my-image`
 - `docker push localhost:5000/my-image`