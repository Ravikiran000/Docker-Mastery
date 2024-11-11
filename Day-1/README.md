## What is a Virtual Machine ?
Virtual Machines (VMs) are like having a complete, separate computer within your computer. Each VM runs its own full operating system, like having another Windows or Linux inside your main system. Because they need to load an entire OS, VMs are big and take a while to start. They are very isolated from each other, making them secure, as each VM operates independently without knowing about the others.


## What is a Container?
A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

Ok, let me make it easy !!!

A container is a bundle of Application, Application libraries required to run your application and the minimum system dependencies.
Containers share the main computer’s operating system but have their own space for running applications. Containers are much smaller and start very quickly since they don’t need a full OS. They offer enough isolation to keep applications separate, but are less isolated than VMs because they share the same OS.


![image](https://github.com/user-attachments/assets/cae4fb34-2fb2-4358-8f18-eabddaf65c1e)


## Containers vs Virtual Machine

1. Resource Utilization: Containers share the host operating system kernel, making them lighter and faster than VMs. VMs have a full-fledged OS and hypervisor, making them more resource-intensive.

2. Portability: Containers are designed to be portable and can run on any system with a compatible host operating system. VMs are less portable as they need a compatible hypervisor to run.

3. Security: VMs provide a higher level of security as each VM has its own operating system and can be isolated from the host and other VMs. Containers provide less isolation, as they share the host operating system.


## Container Runtime
Container Runtimes are used for running the containers. There are several runtimes for running containers, including: Container-D Docker CRI-O
### Note: In production environments, Container-D is typically used. Developers often use Docker for local testing. For example, KIND (Kubernetes IN Docker) is used to create Kubernetes clusters in Docker for testing.
# Docker
Docker is a containerization platform that provides easy way to containerize your applications, which means, using Docker you can build container images, run the images to create containers and also push these containers to container regestries such as DockerHub, Quay.io and so on.


## Docker Architecture
![image](https://github.com/user-attachments/assets/229ce935-c7ee-43fc-80c2-65878065219c)


### Docker Life cycle
docker build -> builds docker images from Dockerfile
docker run -> runs container from docker images
docker push -> push the container image to public/private regestries to share the docker images.
![image](https://github.com/user-attachments/assets/f819855c-3dd3-44de-b7a5-a84189657d9a)


### Docker client
The Docker client (docker) is the primary way that many Docker users interact with Docker. When you use commands such as docker run, the client sends these commands to dockerd, which carries them out. The docker command uses the Docker API. The Docker client can communicate with more than one daemon.

### Docker daemon
The Docker daemon (dockerd) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes. A daemon can also communicate with other daemons to manage Docker services.

### Docker registries
A Docker registry stores Docker images. Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default. You can even run your own private registry.

When you use the docker pull or docker run commands, the required images are pulled from your configured registry. When you use the docker push command, your image is pushed to your configured registry. Docker objects

When you use Docker, you are creating and using images, containers, networks, volumes, plugins, and other objects. This section is a brief overview of some of those objects.

### Dockerfile
Dockerfile is a file where you provide the steps to build your Docker Image.

### Docker Image
An image is a read-only template with instructions for creating a Docker container.


## Installing Docker 
curl https://get.docker.com/ | bash
## Listing namespaces & Running basic docker commands
### Check Docker version: 
docker version

### List namespaces: 
lsns

### List PID namespaces: 
lsns -t pid

### Run a container: 
docker run --name app1 nginx:latest

### Run a container in the background: 
docker -d run --name app1 nginx:latest

### Create multiple containers: 
for i in {1..10}; do 
docker run -d nginx:latest; 
done

### List running containers: 
docker ps

### List all containers including runinng & stopped: 
docker ps -a

### Stop all containers: 
docker stop $(docker ps -aq)

### Deploy a container and remove it when container is stopped: 
docker run --rm -d --name app1 nginx:latest

### Inspect a container: 
docker inspect app1

### Port forwarding: 
docker run --rm -d --name app1 -p 8000:80 nginx:latest

### View logs: 
docker logs app1 -f

-f stands for “follow.” It allows you to see the logs in real-time, as they’re generated, similar to tail -f on a log file.
With -f, new log entries will continuously appear in your terminal as the application writes more logs, which is useful for monitoring an active process.



## Default directory change in Docker
The default directory for Docker is /var/lib/docker. As you continue downloading images and generating logs, this directory will consume more space and eventually get busy. To prevent this, we can store all our Docker data in a separate directory.

### steps to change Default directory
1. create one EBS volume and attach it to your instance.
2. check the volume

    lsblk

3. create partition to the volume and make file system to it and attach it to the fstab

   3.1 fdisk /dev/xvdf
   
       n > p > w >

       lsblk

   3.2 mkfs.ext4 /dev/xvdf1

       copy UUID Somewhere

   3.3 mkdir /dockerdata

       nano /etc/fstab

       Paste Like below
![image](https://github.com/user-attachments/assets/d92b4dd5-7630-4bcc-9b77-f61193819e05)

4. Change docker service directory

   4.1 sudo systemctl stop docker.service
  
   4.2 sudo systemctl stop docker.socket
  
   4.3 sudo nano /lib/systemd/system/docker.service Add the following line with the custom directo
  
   4.4 #ExecStart=/usr/bin/dockerd -H fd:// -- containerd=/run/containerd/containerd.sock
  
       ExecStart=/usr/bin/dockerd --data-root /dockerdata -H fd:// --containerd=/run/containerd/containerd.sock

   4.5 sudo rsync -aqxP /var/lib/docker/ /dockerdata
  
   4.6 sudo systemctl daemon-reload && sudo systemctl start docker
