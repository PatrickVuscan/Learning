# Docker & Kubernetes

Patrick Vuscan

Last Updated: *January 9th, 2022*

[Created from watching Docker+Kubernetes tutorial by Amigoscode!](https://youtu.be/bhBSlnQcq2k)

# Contents
- [Introduction](#introduction)
- [Containers vs VMs](#containers-vs-vms)
- [Docker Definitions](#docker-definitions)
- [Dockerfiles](#dockerfiles)
- [Docker Registries](#docker-registries)
- [Docker Debugging](#docker-debugging)
- [Kubernetes](#kubernetes)
- [Kubernetes Basic Architecture](#kubernetes-basic-architecture)


<br/>
<br/>
<br/>
<br/>
<br/>

# Introduction

## What is Kubernetes?
Many Docker containers can interact with eachother. Managing hundreds/thousands of these would be difficult to do manually, so Kubernetes helps. Whenever a container crashes, or problems within two's communications arise, Container Orchestration Tools can help recover an application's state.

Kubernetes is the most popular Container Orchestration Tool.

<br/>

## What is Docker?

- A tool for running applications in an isolated environment, similar to a VM
- App runs in the exact same environment - brings that idea of "it worked on my local" to all environments
- Becoming the standard



<br/>
<br/>
<br/>
<br/>
<br/>

# Containers vs VMs

**Containers** are an **abstraction** at the app layer that packages **code** and **dependencies** together. Multiple containers can run on the same machine and share the OS Kernel with other containers, each  running as isolated processes in  user space. A container doesn't need an OS, it uses the underlying OS.

**VMs** are an abstraction of physical hardware turning **one server into many servers**. The hypervisor allows multiple VMs to run on a single machine. Each VM includes a **full copy** of an OS, the application, necessary binaries and libraries - taking **tens** of GBs. VMs can be slow to boot.

<br/>
<br/>

![Docker vs VMs](Images%20-%20Docker/DockerVsVM.png)

<br/>

### *Benefits of using Docker*

- Containers can be run in seconds instead of minutes
- Uses less disk space and memory (no full OS)
- Deployment and Testing

<br/>
<br/>
<br/>
<br/>
<br/>

# Docker Definitions

## What is a Docker Image?

- A template for creating an environment of your choice
- "Snapshot" / versions of the app at different points in time
- Has everything needed to run Apps
- OS, Software, App Code

<br/>

## What is a Docker Container?

A running instance of an Image.

<br/>

## What is a Docker Volume?

- Allows sharing of data, files, folders between containers, or between the host and containers
- Can be writable or read only

<br/>
<br/>
<br/>
<br/>
<br/>

# Docker Commands

`docker pull nginx` - downloads the nginx image

`docker images` OR `docker image ls` - shows all downloaded / local images

___

`docker ps` OR `docker container ls` - shows all running containers

`docker ps -a` - will show all including stopped (non-removed) containers

`docker ps -aq` - return IDs of all existing containers

`docker ps --format="ID\t{{.ID}}\nNAME\t{{.Names}}\nImage\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"` - great for formatting nicely

- You can use `export FORMAT="..."` to create a variable for it, then use `docker ps --format=$FORMAT`

___

## Running Containers

`docker run nginx:latest` - creates and runs the image, aka is a container - the running image, but does not actually produce output

- `--name website` - names container as website

- `-d` - runs in *detached* mode and returns the ID of the container created

- `-p 8080:80` - runs with host's port 8080(localhost:8080) mapped to container's port 80

  To map multiple, just use `-p` multiple times. **Note you can map multiple host ports to the same container port**

- `-v /some/content:/usr/share/nginx/html:ro` - maps host folder to container's folder as a shared volume

  - `ro rw z` - read only, read write, z is something special...

  - If you're in the folder you want to map, you can instead use `-v $(pwd):/usr/share/nginx/html` (note that the lack of `:ro` makes it writable too)

`docker run --name website-copy --volumes-from website nginx` - sets up container:container volume with `website` (same data?)

___

`docker stop containerIdOrName` - stops a container, but does not remove it

`docker start containerIdOrName` - starts an existing container

`docker rm containerIdOrName` - removes the container. Will not remove running container, unless `-f` is used

- `docker rm $(docker ps -aq)` - removes all existing containers

`docker rmi imageIdOrName` - removes an image

`docker exec -it containerIdOrName bash` - execute the bash command in interactive `-it` mode

`docker tag website:latest website:1.0.0` - creates another tag for the image. Note that they will both point to the same image

`docker inspect containerIdOrName` - returns a JSON formatted container's full config

`docker logs containerIdOrName` will show the logs of the applications running in the container.

- Using an `-f` flag will *follow* the logs realtime.

- Using `--since time` or `--until time` will show logs only since or until the specified time, which can be specified as a timestamp (e.g. *2021-08-02T13:23:38*) or relative (e.g. *42m* for 42 minutes).

<br/>
<br/>
<br/>
<br/>
<br/>

# Dockerfiles

A file that lets us create our own images, through the file which contains the steps to create it.

[This is the reference for it.](https://docs.docker.com/engine/reference/builder/)

Whenever we're in development, it makes sense to mount volumes so that we can easily code, then run it, etc. This doesn't make sense however when building for deployment - for this, we create an image, and instead, copy the contents of the volume into it.

<br/>

## Contents of the Dockerfile

- Must be inside the root of the folder
- Must contain the `FROM` keyword, the base image to use (nginx, alpine, etc)

- `ADD` will copy files, folders, or remote URLs from `source` to the `dest` path in the **image's** filesystem.

  ```docker
  ADD hello.txt /absolute/path
  ADD hello.txt relative/to/workdir
  ```
- `WORKDIR` will set the working directory **INSIDE THE CONTAINER** for any subsequent `ADD`, `COPY`, `CMD`, `ENTRYPOINT`, or `RUN` instructions that follow it in the Dockerfile.

  ```docker
  WORKDIR /path/to/workdir
  WORKDIR relative/path
  ```

  For example, afterwards we can do this `ADD . .`, to copy current host dir contents into the container's `workdir` previously specified

- `RUN` will execute any commands on top of the current image as a new layer and commit the results.

  ```docker
  RUN apt-get update && apt-get install -y curl
  ```

- `CMD` will provide defaults for an executing container. If an executable is not specified, then ENTRYPOINT must be specified as well. **There can only be one CMD instruction in a Dockerfile.**

  ```docker
  CMD [ "/bin/ls", "-l" ]
  ```

<br/>

## .dockerignore Files

Note that in the dockerfile, we may have an `ADD` command, which  will add everything in the folder specified. However, we may not want to add *everything*. Specifically, things like `.env` files, or folders like `/node_modules`.

We can use a `.dockerignore` file, similar to a `.gitignore` file, to not have those added.

<br/>

## Creating the Image

`docker build -t/--tag website:latest .` - builds the image, and provides a tag which contains the image name followed by the version. The `.` is the folder that contains the `Dockerfile`.

This image can then be run as regularly.

<br/>

## Caching During Image Creation

Whenever possible, Docker will cache "layers", aka steps or commands inside of the Dockerfile, so that you don't repeat unneccesary work. It will use cache whenever possible. Once something changes, it will have to recompute that step, **and every step afterwards.**

```docker
FROM node:latest
WORKDIR /app
ADD . .
# If source code changes, must re-run npm install
RUN npm install
CMD node index.js

# Versus

FROM node:latest
WORKDIR /app
# Note that when including specific files in ADD, source must be a folder ending with `/`
ADD package*.json ./
# Will first import package.json and npm install, so that this step can be cached independently of source code
RUN npm install
ADD . .
CMD node index.js
```

Here, the second Dockerfile will be significantly better, because it will no longer re-install, unless your package.json changed!

### Note about size

`node:latest` will likely be very large (~900MB). To benefit from reduced size, use alpine for example! (`node:alpine`)

However, one thing we'll also notice is that using `latest`, or generic `alpine` will mean that what you `docker pull` one day might be different from the next. We can avoid this by using more specific versionings: `node:10.16.1-alpine` for example, so that if `node:alpine` changes, we stick to the same version as before without breaking our app.

<br/>
<br/>
<br/>
<br/>
<br/>

# Docker Registries

- Highly scalable server side application that stores and lets you distribute docker images
- Used in CI/CD pipeline
- Run your applications

![Docker push to registry](Images%20-%20Docker/DockerPushRegistry.png)

There are a few, including Dockerhub, Amazon ECR, and Quay.

<br/>

## Pushing to Registry

We will use Dockerhub. Before pushing, we have to create the repo on the dockerhub website, and then we have to make new tags for our local images, so that they match the name:tags of our docker repo.

`docker tag website:1 PatrickVuscan/website:1` \
`docker tag website:2 PatrickVuscan/website:2` \
`docker tag website:latest PatrickVuscan/website:latest`

Then we can push.

`docker push PatrickVuscan/website:1` \
`docker push PatrickVuscan/website:2` \
`docker push PatrickVuscan/website:latest`

<br/>
<br/>
<br/>
<br/>
<br/>

# Docker Debugging

## Docker Inspect

`docker ps` will only show us so much information about our containers. If we want to *inspect* the entire config of them, we should instead use `docker inspect`

`docker inspect containerIdOrName` will return a JSON formatted container's full config

<br/>

## Docker Logging

`docker logs containerIdOrName` will show the logs of the applications running in the container.

Using an `-f` flag will *follow* the logs realtime.

Using `--since time` or `--until time` will show logs only since or until the specified time, which can be specified as a timestamp (e.g. *2021-08-02T13:23:38*) or relative (e.g. *42m* for 42 minutes).

<br/>

## Docker "Getting into the box"

For things like nginx, which is really a linux box, you can get into the linux machine to inspect what's going on in there.

This is by using the command `docker exec -it containerIdOrName /bin/sh`.

Note that we might not be able to use `bash` as we did elsewhere. To find what we *can* use, one option is to do the `docker inspect`, search in the results for `Cmd`, and see what shell was used for it, and call that.

<br/>
<br/>
<br/>
<br/>
<br/>

# Kubernetes

Kubernetes is the most popular **Container Orchestration Tool**, developed by Google, and open-sourced.

It helps manage containerized applications in different deployment environments; physical, virtual, cloud, etc. These can be multiple containers that create one overarching application, or otherwise.

<br/>

## What problems does Kubernetes solve?

There was a trend to move from Monolith to Microservice architecture, resulting in containerization. Now some apps are comprised of hundreds, even thousands of containers.

Since these were difficult to manage through one's own scripts and tools, there was a need for a tool which properly managed these containers.

Enter, Kubernetes.

<br/>

## What are the tasks of an orchestration tool?

- High availability, or no downtime
- Scalability or high performance
  - Load fast, quick response time
- Disaster recovery - backup and restore

<br/>
<br/>
<br/>
<br/>
<br/>

# Kubernetes Basic Architecture

![Kubernetes architecture](Images%20-%20Docker/KubernetesArchitecture.png)

<br/>

The kubernetes cluster is made up of at least one master/main node, and connected to it, many worker nodes.

<br/>

## Worker Nodes

Each **worker** node has docker containers deployed on it, and also has a `kubelet` process running on it.

The kubernetes `kubelet` process is what connects and communicates with other nodes, and allows the cluster to execute tasks across the nodes.

<br/>

## Main/Master Nodes

On the **main/master** node, it runs several important kubernetes (K8s) processes that are necessary for running and managing the cluster.

One of these includes an **API Server**, which is a container, and is the entrypoint to the K8s cluster. The UI, API and CLI talk to this API server.

Another is the **Controller Manager**, which tracks what's going on in the cluster. Did something die, does something need to be fixed or restarted.

There's also the **Scheduler**, responsible for scheduling different containers on different nodes, depending on the workload, and available server resources on each node. Another terminology for this is "ensuring Pod placement".

Another is the **etcd**, the kubernetes backing key value storage, which holds the current state of the kubernetes cluster. Holds the configuration data and status data, of each node and each container in each node.

Backups are actually done through snapshots of the **etcd**, since this contains all the information of the cluster.

<br/>

## Virtual Network

![Virtual Network](Images%20-%20Docker/VirtualNetwork.png)

One last important part of this architecture, is the **Virtual Network** which allows the main/master node(s?) and worker nodes to communicate amongst themselves. It is what creates one powerful unified machine from the sum of all the nodes.

Worker nodes have the most load, because they're running the actual applications, and thus are usually given more resources.

The main/master node is much more important than individual worker nodes, but is also much more lightweight. Because of it's importance, you usually have at least 2 mains/masters, so that if one goes down, you don't lose access to your cluster.

<br/>
<br/>
<br/>
<br/>
<br/>

# Kubernetes Components

## Pod

A pod is the smallest unit of K8s, essentially an abstraction layer over containers, so that it can work with other technologies, other than Docker.

A pod typically only runs one container inside of it at a time, but you *can* run more than one.

Each pod gets its own IP address in the virtual network, with which pods can communicate between themselves.

Pods are **ephemeral**, as in they die very easily. If the container inside it crashes, or runs out of resources, the pod will die, and a new one will be created in its place. When this happens, a new IP address will be assigned to it on creation.

Since these IPs can change as pods get recreated, we have a service component.

<br/>

## Service

The **Service** component is a permanent IP address for each pod which persists through crashes, since the lifecycles of pods and services are **not connected.**

There are both External and Internal services, differentiated by if we want their IPs exposed.

Services are usually connected to by the IP of the node, and the port. This is usually something ugly like https://128.12.4.0:8080. We don't want to access our apps by these types of things, do we?

<br/>

## Ingress

This is where an **Ingress** component comes in. It essentially exposes an address like https://my-app.com for your pod, which when accessed, forwards requests to the pod's **Service** component

<br/>

## ConfigMap

The **ConfigMap** component is an external configuration of your application attached to a Pod, which can be used so that for small changes like a database URL, you don't have to re-build your repo, push to the repo, pull into the pod, etc.

You should not put credentials into a ConfigMap, since these are stored in plaintext.

<br/>

## Secret

The **Secret** component is a response to needing external config secrets which are *not* stored in plaintext, but rather in base64, for a pod.

One note, the built-in security mechanism is not enabled by default!