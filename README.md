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

We can download and start our first container from the docker registry. Here we ask for 'debian', which is a small container only 125MB in size.

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

The docker command we use is 'run'. If the container 'debian' does not exist it is downloaded for us from the docker registry on the internet. The option '--rm' will clean up any new layer we might create inadvertenty (to save space). The option '-it' is a shortcut for 'interactive terminal' so we can interact with the running container. The program we start inside the container is '/bin/bash' a shell whos prompt we see on the last line. A simple 'quit' typed in there will close the container again and remove any trace of it.

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

If there are many layers the above manual way of adding software lacks structure and cannot be easily repeated without lots of documentation. Docker provides a better way of keeping track of how containers are build using Dockerfiles. These files are human readable text files that specify in detail how a container should be build. Sharing such a Dockerfile is sufficient to allow other users to build the same container.

