# Dockerfile | Build ARG | ENV | Dangling Images

## Dockerfile:
A Dockerfile is a text document that contains all the commands to assemble an image for a Docker container. It automates the deployment of applications by packaging an application and its dependencies in a standardized unit called a container.

# Dockerfile - Part 1
This part-1 contains a Dockerfile for setting up an environment with Terraform and Packer on an Ubuntu base image. Below are the steps and commands used in the Dockerfile and for running the container.

### Basic Understanding of Dockerfile
### FROM
Defines the base image for the container. All Docker images start with a base, which can be a minimal OS (like alpine or ubuntu) or an application image (like nginx, node, etc.).

Example: FROM nginx:latest
### LABEL
Adds metadata to the image, such as authorship or application details, and is useful for image documentation.
Example: LABEL maintainer="example@example.com"
### WORKDIR
WORKDIR instruction in a Dockerfile sets the working directory for any RUN, CMD, ENTRYPOINT, COPY, and ADD instructions that follow it. This avoids the need to specify full paths repeatedly. EX: WORKDIR /app
### COPY OR ADD
Copies files or directories from the local machine (host) to the container's filesystem. COPY is generally preferred as it is simpler and more predictable. ADD also supports downloading remote files and extracting tar archives.

EX: COPY index.nginx-debian.html /var/www/html/

EX: ADD  style.css /var/www/html
### ENV
Sets environment variables within the container. These variables can be accessed by applications or scripts inside the container. 
Example: ENV AWS_ACCESS_KEY_ID=SDFSDFSDFSDFSDFSDFSDFSDF
### ARG
Defines build-time variables that users can pass to the Docker build process. Useful for configuring the build without hardcoding values. ARG variables are used to pass build-time variables during the image building process.
Example: ARG VERSION=latest
### ARG vs ENV
ENV: Runtime environment variables for containers.

ARG: Build-time variables for image construction.
### RUN
Executes commands in the container during the image build process. Commonly used for installing software packages, updating the OS, or running custom setup scripts.
Example: RUN apt-get update && apt-get install -y python3
### CMD
Specifies the default command to run when the container starts. If a command is provided in docker run, it will override CMD.
Example: CMD ["nginx", "-g", "daemon off;"]
### ENTRYPOINT
Defines the main process that will run in the container. Unlike CMD, ENTRYPOINT is less easily overridden. It is typically used to set a command that will always run. ENTRYPOINT ["python3", "app.py"]

NOTE: To overwrite the default ENTRYPOINT of a Docker image, you can use the --entrypoint flag with the docker run command. This allows you to specify a different entry point for the container. EX: docker run --rm -d --name demo --entrypoint /bin/bash demo:v1

This command will start a container using /bin/bash as the entry point instead of the default ENTRYPOINT specified in the Dockerfile.
### command to create Dockerfile 
vim Dockerfile
### It is not mandatory to give the name of the dockerfile as "Dockerfile". You can name anything. Suppose you have given the name of the dockerfile as dockfile, then you need to mention "-f dockfile" before "."
docker build -t demo:v1 -f dockfile .

## Dockerfile for setting up an environment with Terraform and Packer on an Ubuntu base image

FROM ubuntu:latest

LABEL name="ravikiran"

ENV AWS_ACCESS_KEY_ID=SDFSDFSDFSDFSDFSDFSDFSDF
AWS_SECRET_KEY_ID=SDSDSDSDSDSDSDSDSDSDSDSD
AWS_DEFAULT_REGION=US-EAST-1A

ARG T_VERSION='1.6.6'
P_VERSION='1.8.0'

RUN apt update && apt install -y jq net-tools curl wget unzip
&& apt install -y nginx iputils-ping

RUN wget https://releases.hashicorp.com/terraform/${T_VERSION}/terraform_${T_VERSION}_linux_amd64.zip
&& wget https://releases.hashicorp.com/packer/${P_VERSION}/packer_${P_VERSION}_linux_amd64.zip
&& unzip terraform_${T_VERSION}linux_amd64.zip && unzip packer${P_VERSION}_linux_amd64.zip
&& chmod 777 terraform && chmod 777 packer
&& ./terraform version && ./packer version

CMD ["nginx", "-g", "daemon off;"]

### CMD ["nginx", "-g", "daemon off;"] 
CMD ["nginx", "-g", "daemon off;"] command, running Nginx in the foreground (by using daemon off;) means that Nginx will not detach and run as a background process. Instead, it will stay active in the main terminal session, ensuring the container continues to run and serve requests.

# Implementaion
### Build the image
docker build .
### Tag the existing image that is without tag
docker tag <image-id> <image_name:version>

EX: docker tag 35825f3277d9 demo:v1
### Tag existing image that is with tag
docker tag <existing_tag_name:version> <desired_tag_name:version>

EX: docker tag demo:v1 demo2:v2

## build args implementation
### Run your container, login to the container & check the Args given in Dockerfile (you can see the packer & terraform versions given in Dockerfile)
docker run --rm -d --name demo1 -p 8000:80 demo2:v2

docker exec -it demo1 bash

ls -al

./packer version

./terraform version
### Give build args while building the image (this will override the packer & terraform versions given in Dockerfile)
docker build -t demo1:latest --progress=plain --build-arg T_VERSION='1.8.0' --build-arg P_VERSION='1.3.3' .

--progress=plain will provide plain text output.

### stop the container(because we are giving same name), run the container again and login into it (will show changed packer & terraform versions)
docker run --rm -d --name demo1 demo1:latest

docker exec -it demo1 bash

ls -al

./packer version

./terraform version

## env implementation
### check the env variables (you will see the env given in Dockerfile)
docker exec -it demo1 env 
### Giving env at runtime of container 
docker run --rm -d --name demo2 -e AWS_ACCESS_KEY_ID=SDFSDFSD -e AWS_SECRET_KEY_ID=SDS -p 8000:80 demo:latest
### check env of container(this will give env which are given at the runtime of the container, overriding the one's in Dockerfile)
docker exec -it demo2 env

## Dangling Images
Dangling images are Docker images that are not tagged and are not referenced by any container. These images can take up unnecessary space on your system.
### Identifying Dangling images
docker images -f "dangling=true"
### Remove Dangling Images
docker images prune
### Remove All Unused Images
docker images prune -a

# Dockerfile - Part 2
