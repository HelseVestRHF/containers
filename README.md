# containers

This course is part of a presentation series "Without Any Gaps" at the Mohn Medical Imaging and Visualization Center that provides introductory presentations on technical solutions supporting open science.

## Motivation

First some words about a related topic "cloud computing" or "virtual machines". Right after this introduction we explain what makes containerization so interesting.

Most cloud computing is done using "virtual machines", which are simulated computers that run on other people's hardware (a definition of a cloud). Such a virtual machine provide secure computing resources and is hosted on large physical computers. One such large computer can host hundereds of virtual machines securely as each VM is separated from its neighbors with its own portion of the CPU, memory and storage. Changing such a setup by adding or removing VM's is easy as only software changes are required on the large computer, which makes this concept cost effective as projects can share compute resources.

One downside of virtual machines is that they have to simulate a full physical computer. Starting a virtual machine therefore requires it to 'boot-up'. The system is more complex as the software needs to simulate all physical aspects of the computer for the boot process to work, this includes the graphic card, sound, storage, and network systems. Such components are already provided by the server that runs the virtual machine - the idea to re-use those components from the server, instead of simulating them was the starting point for container services.

By separating containers from the the host and from each other and by removing the need to boot-up container can be provid security and are better scalable
compared to virtual machines. They provide a much faster and cheaper solution compared to standard virtualization. Whereas a virtual machine might need 10 seconds to boot-up a container is ready in milliseconds. Here are the basic similarities and differences of virtual machines and containers:

- Startup time: for a container its measured in milliseconds, virtual machines needs seconds and sometimes minutes
- Execution time: similar executation times with some overhead compared to native running applications (maybe 5% slower compared to native applications)
- Access to data: virtual machines need to connect to external storage using a shared file system, containers can do the same but can also 'mount' local directories - with some speed penalties if the container host is not Linux
- Use-case: virtual machines are used to replace physical machines, container services are used to replace programs

Especially the last part is of interest in understanding why containers are so useful. As a container starts up very fast it can be used like a program. Most containers are started and are automatically removed after they finish. They only have an effect on the data they process. A container might therefore run only for a second based on the computation it performs. Containers can of course also run forever and react to incoming data, process them and send back the results. Such setups are more involved as containers do not come prepared with system services like 'cron' and 'systemd'.

## Quickstart

Images are blueprints for containers waiting to run. See what images are available on your computer.

```bash
% docker images
```

Download and start a docker container, here a specific version of Ubuntu.

```bash
% docker run --rm -it ubuntu:20.04 /bin/bash
```

See all currently running or stopped docker containers and their container IDs.

```bash
% docker ps -a
```

Stop and remove a container with the ID "abcdedf":

```bash
% docker stop abcdef
% docker rm abcdef
```

Remove an image:

```bash
% docker rmi ubuntu:20.04
```


## How to start

It is easiest to start using containers on MacOS, Windows or Linux with the 'docker' runtime (https://docs.docker.com/get-docker/). Your computer becomes the docker-host that will run your containers. It is possible to share and run a container on all three platforms so the choice of what system to use for development is not too important. Here a first test of an ubuntu container providing us with the current date.

```bash
> docker run --rm ubuntu date

Mon Sep 13 13:46:50 UTC 2021
```

On a MacOS host the above call of running a docker container and the 'date' program and removing it again took 1.4seconds (on a Linux host the same call took about 0.5seconds).

## Some background information

Running a container on a linux system is probably the easiest to explain. Linux comes with security measures like control groups (cgroups) that allocate resources from the host computer like CPU, memory and networking and provide them to programs. Programs that are running in an environment provided by cgroups (and namespaces) are separated from the host system. To these programs other resources do not exist. Containers use this to to provide a minimal Linux inside the container with no access to host users or host storage.

On MacOS and Windows the process is a little bit more complex. Containers are linux based so in order to provide the 'docker run' command from above the docker environment simulates a Linux computer using a virtual machine. This is done fully transparent to the user but all commands go through this additional virtualization layer. One of the side-effects of this tunneling is that container can only access the storage that the virtualization solution provides - usually much less compared to the storage on the host system. You might therefore run into problems of 'not-enough-memory'. In such cases 'docker' needs to extend its space reserved for containers, which is done in the docker dashbord. After a restart of docker the containers will have more storage space available.

## Where do containers come from?

There is a way to create a container starting from zero - but not many people do this. Easiest is to start with an already existing container. Container's consist of layers (yes, just like an onion). The innermost layer is usually a basic Linux operating system like debian or ubuntu. The layer is part of a 'union filesystem'. Such a filesystem was initially developed for test operating systems starting from a CD or DVD. A DVD is only written once but a running linux operating system require a writable file system to function. In order to be able to write to a read-only filesystem the union filesystem was developed to sit as a layer 'ontop' of the read-only first layer. If a program tried to write something instead of writing to DVD the written data ends up in a separate layer (in memory). For all programs such a onion filesystem looks like one single system where layers are merged with files in the places that the OS expects. The same mechanism is used in containers to allow them to be build in pieces. Container may share such underlying layers to save storage space. 

So the answer to where containers come from is, they come from other containers. Containers all the way down...

A downloaded container is called an *image*. The same image can be run multiple times. Each of those instances is a *container*. 

We can download and start our first image from the docker registry. Here we ask for 'debian', which is a small image only 125MB in size.

```bash
% docker run --rm -it debian /bin/bash 
docker run --rm -it debian /bin/bash 
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
955615a668ce: Already exists 
Digest: sha256:08db48d59c0a91afb802ebafc921be3154e200c452e4d0b19634b426b03e0e25
Status: Downloaded newer image for debian:latest
root@61dc11963da5:/# 
```

The docker command we use is 'run'. The created container has the random ID "61dc11963da5". If the image 'debian' does not exist it is downloaded for us from the docker registry on the internet. The option '--rm' will clean up any new layer we might create inadvertenty, it is a good option to always include to save space. The option '-it' is a shortcut for 'interactive terminal', so we can interact with the running container. The program we start inside the container is '/bin/bash' a shell whos prompt we see on the last line. A simple 'quit' typed in there will close the container again and remove any trace of it. In the above call we end up on a new shell inside the container (id: 61dc11963da5). We are listed as the root user - a default user inside a docker container.

Instead of /bin/bash we can also start any other program that is available inside the debian container:

```bash
docker run --rm debian date

Mon Sep 13 14:39:35 UTC 2021
```

Here we don't need the option '-it' as *date* does not need an interactive terminal. We just want to see its output. Here another example using the disk free (df) program:

```bash
docker run --rm -it debian df -h
```

In truth the number of programs already present in a debian container is just

```bash
docker run --rm -it debian /bin/bash -c "ls /bin | wc -l"

69
```

For any real work we need to add a layer to have our program inside our container.

## Add one layer

To add a program to a container we can use the package manager of the base layers operating system. For our debian OS we can use the 'apt-get' command to install for example R.

```bash
docker run --rm -it debian /bin/bash
root@54da72ddb3cc:/# apt-get update
...
root@54da72ddb3cc:/# apt-get install r-base
...
```

The above three lines will install R inside the container but closing the container and opening it again any change inside the container - like the installation of R will be lost. In order to keep our changes we will have to keep the new layer in our own docker image.

Keep the first container running and open another terminal. In order to 'commit' the above container with all its changes we need to know its docker identifier. A running container has a random ID (some letters and numbers) that are displayed on the prompt above but can also be seen using:

```bash
docker ps
```

Now that we have the id of our container (54da72ddb3cc) we can 'commit' it as a new container with:

```bash
docker commit 54da72ddb3cc mydebianr
```

If this works you can see your new container in the list of docker images:

```bash
docker images
```

Using that container is now as easy as:

```bash
docker run --rm -it mydebianr R
```

## Doing the same with more structure - Dockerfile

If there are many layers the above manual way of adding software lacks structure and cannot be easily repeated without lots of documentation. Docker provides a better way of keeping track of how containers are build using Dockerfiles. These files are human readable text files that specify in detail how a container should be build. Sharing such a Dockerfile is sufficient to allow other users to build the same container. At a minimum a dockerfile just contains the name of the image it is based on and a command that adjusts the image, installs some software for example:

```bash
FROM: ubuntu:20.04

RUN apt-get update && apt-get install -y bc
```

## How to keep your data around - outside the container

Containers are very much suited as a means to separate programs from data. That way updates of the software can happen while keeping all the data in place.

### Share a directory on your laptop

A basic use for containers is to run some analysis on data available on your system. For those purposes the container just need access to a folder for reading and to one folder for writing - if there are outputs. Such simple folder shares are specified during the run command with '-v'. One important issue is that the option needs a full path.

```bash
% docker run --rm -it -v /path/to/data:/input -v /path/to/output:/output ubuntu /bin/bash
```

Started like above the two '-v' options will make a '/path/to/data' outside the container available inside as '/input'. The '/path/to/output' from outside the container will be mounted inside the container in '/output'.

This very easy option has some downsides, especially if used on MacOS and Windows. The mounts on both of those platforms are not very performant as data is copied first into the virtual machine and afterwards becomes available to the container.

### Share a volume

One step up from the mounts is to use the containers own union file system for storage. In this case a container has a reference to a 'volume'. Such volumes are data containers that don't have any software, just storage. If a Dockerfile references such a volume and writes data to it those files will still be available after a container reboots. A volume called 'mydata' can be created and added to an ubuntu docker container with

```bash
% docker volume create mydata
% docker run --rm -it --mount source=mydata,destination=/mydata ubuntu /bin/bash
```

The data will be available in the containers /mydata directory.

With the above tools we can create more complex scenarios. Lets say we have an application (like a website) that needs to talk to a database. The application would go into one container, the database software would go into another container and the data from that database goes into a volume. The main reason to do those complex setups is to allow the solution to a) update the different pieces independently from each other without worries about data migration and b) to make the solution scale with more users as we can duplicate some of the containers. That of course requires another container that contains a software like 'traffic' to distribute users to the different web-containers. In such a scenario all web-server containers are containers of the same container image and based on the current number of users one could add or remove more of them.

A word of caution: the scalability and improvements in keeping the system updated come at the cost of a more complex environment in which suddenly it becomes important to understand and debug networking and routing. Also, nothing is faster than running directly on the provided hardware. If speed is the ultimate goal a containerized environment is a good testing environment, ultimately such a system should run on dedicated hardware.


## Containers talk to containers

Connections between containers is done on the network level. Such connections are also of interest to interact with software inside a container from the outside - lets say a website running inside the container should become accessible on the hosts web-browser. An example for such a setup is R-server, an environment for data analysis. Similar to the 'docker run' commands to make directories accessible from the outside with '-v' there is also a command to map ports from inside the container to the outside. System services are using predefined ports to provide such services. For a normal website for example its port number 80 or 443. A container with a running webserver inside will have this port inside mapped to another port outside. Only the outside port is the one we can access using the web-interface:

```bash
% docker run --rm -it -p 80:99 ngnix /bin/bash
```

In the above example the port 80 from inside the container is mapped to port 99 on our machine. We can reach the website afterwards if we navigate to localhost:99.

 The simplest of such solutions is to assign one container to one network port. This way two container will each know the port of the partner and can send messages (REST-API).

As soon as we reach such systems we need a way to orchestrate containers making sure that containers that belong together are started together, that they can find each other on the network and that no other container can eavesdrop on the connection. Of course the best known software to do this orchestration is kubernetes. Before we go there here a first step in that direction - docker-compose. Similar to the Dockerfile format to specify individual containers 