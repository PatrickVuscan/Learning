# Docker & Kubernetes

## What is Kubernetes?
Many Docker containers can interact with eachother. Managing hundreds/thousands of these would be difficult to do manually, so Kubernetes helps. Whenever a container crashes, or problems within two's communications arise, Container Orchestration Tools can help recover an application's state.

Kubernetes is the most popular Container Orchestration Tool.

___

## What is Docker?

- A tool for running applications in an isolated environment, similar to a VM
- App runs in the exact same environment - brings that idea of "it worked on my local" to all environments
- Becoming the standard

### Containers vs VMs

**Containers** are an **abstraction** at the app layer that packages **code** and **dependencies** together. Multiple containers can run on the same machine and share the OS Kernel with other containers, each  running as isolated processes in  user space. A container doesn't need an OS, it uses the underlying OS.

**VMs** are an abstraction of physical hardware turning **one server into many servers**. The hypervisor allows multiple VMs to run on a single machine. Each VM includes a **full copy** of an OS, the application, necessary binaries and libraries - taking **tens** of GBs. VMs can be slow to boot.

![Docker vs VMs](Images/Docker%20vs%20VM.png)

### *Benefits of using Docker*

- Containers can be run in seconds instead of minutes
- Uses less disk space and memory (no full OS)
- Deployment and Testing

___

## What is a Docker Image?

- A template for creating an environment of your choice
- "Snapshot" / versions of the app at different points in time
- Has everything needed to run Apps
- OS, Software, App Code

## What is a Docker Container?

A running instance of an Image.

___

## Docker Commands

`docker pull nginx` - downloads the nginx image

`docker images` - shows all downloaded / local images

`docker ps` OR `docker container ls`- shows all running containers

### Run

`docker run nginx:latest` - runs the image, aka is a container - the running image, but does not actually produce output. Note that the tag latest is apparently **required**. Ctrl-C'ing will terminate the container

`docker run -d nginx:latest` - runs the container in *detached* mode. Returns the ID of the container created

`docker run -p 8080:80 nginx:latest` - runs the container with host's port 8080(localhost:8080) mapped to container's port 80

`docker stop containerID` - stops container