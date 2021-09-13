# containers

This course is part of a presentation series "Without Any Gaps" at the Mohn Medical Imaging and Visualization Center that provides introductory presentations on technical solutions supporting open science.

## Motivation

First some words about a related topic "cloud computing" or "virtual machines". Right after this introduction we explain what makes containerization so interesting.

Most cloud computing is done using "virtual machines", which are simulated computers that run on other people's hardware (a definition of a cloud). Such a virtual machine provide secure computing resources and is hosted on large physical computers. One such large computer can host hundereds of virtual machines securely as each VM is separated from its neighbors with its own portion of the CPU, memory and storage. Changing such a setup by adding or removing VM's is easy as only software changes are required on the large computer, which makes this concept cost effective as projects can share compute resources.

One downside of virtual machines is that they simulate a full physical computer. Starting a virtual machine therefore requires it to 'boot-up'. The system is complex as the software needs to simulate all physical aspects of the computer for the boot process to work, this includes the graphic card, sound, and a storage system. Such components are already provided by the server that runs the virtual machine - the idea to re-use those components from the server, instead of simulating them was the starting point for container services.

By providing similar security by separating containers from the the host and from each other and by removing the need to boot-up such container's can be provided security and are scalable similar to virtual machines but provide a much faster and cheaper solution compared to standard virtualization solutions. Whereas a virtual machine might need 10 seconds to boot-up a container is ready in milliseconds.
