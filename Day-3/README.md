# DockerFile | Build ARG | ENV | Dangling Images

## DockerFile:
A Dockerfile is a text document that contains all the commands to assemble an image for a Docker container. It automates the deployment of applications by packaging an application and its dependencies in a standardized unit called a container.

### DockerFile - Part 1
This part-1 contains a Dockerfile for setting up an environment with Terraform and Packer on an Ubuntu base image. Below are the steps and commands used in the Dockerfile and for running the container.

### Basic Understanding of DockerFile
### FROM
Defines the base image for the container. All Docker images start with a base, which can be a minimal OS (like alpine or ubuntu) or an application image (like nginx, node, etc.).
Example: FROM nginx:latest
### LABEL
Adds metadata to the image, such as authorship or application details, and is useful for image documentation.
Example: LABEL maintainer="example@example.com"
### ENV
Sets environment variables within the container. These variables can be accessed by applications or scripts inside the container.
Example: ENV AWS_ACCESS_KEY_ID=SDFSDFSDFSDFSDFSDFSDFSDF
### ARG
Defines build-time variables that users can pass to the Docker build process. Useful for configuring the build without hardcoding values.
Example: ARG VERSION=latest
### RUN
Executes commands in the container during the image build process. Commonly used for installing software packages, updating the OS, or running custom setup scripts.
Example: RUN apt-get update && apt-get install -y python3
### CMD
Specifies the default command to run when the container starts. If a command is provided in docker run, it will override CMD.
Example: CMD ["nginx", "-g", "daemon off;"]
FROM ubuntu:latest

LABEL name="saikiran"

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
