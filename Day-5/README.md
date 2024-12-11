# Docker Compose
Docker Compose is a tool used for defining and running multi-container Docker applications. With Docker Compose, you can use a YAML file to configure your application's services, networks, and volumes, enabling you to manage multiple containers as a single application.

### Scenario: 
You’re building a web application consisting of: 

A frontend built with React. 

A backend API developed in Node.js. 

A database using MySQL. 

Instead of running each service separately using docker run commands, you decide to use Docker Compose for simplicity.

## Implementation:

Here we are creating multiple dockerfiles and using docker compose to execute them simultaneously.

1. Install docker on an Linux EC2 instance
```
   curl https://get.docker.com/ | bash
```
2. Install docker compose on the docker ec2 instance
   
   2.1 First move to /usr/local/bin and enter the commands given below
   ```
        cd /usr/local/bin
        wget https://github.com/docker/compose/releases/download/v2.30.3/docker-compose-linux-x86_64
   ```
   2.2 rename the file & chnage permissions
   ```
        mv docker-compose-linux-x86_64 compose
        chmod 700 compose
   ```
   2.3 check for version and go to previous root directory
   ```
         compose --version
         cd
   ```
### Dockerfile1:
```
FROM ubuntu:latest
LABEL name="ravikiran"
RUN apt update && apt install -y jq net-tools curl wget unzip\
    && apt install -y nginx iputils-ping
CMD ["nginx","-g","daemon off;"]
```
2. Create 2 more dockerfiles with names as "Dockerfile2" & "Dockerfile3"

3. create a docker compose file as "docker-compose.yml"
```   
version: "3.9"
services:
  dev:
    build:
      contex: .
      dockerfile: dockerfile1
    container_name: dev_container
  stage:
    build:
      context: .
      dockerfile: dockerfile2
    container_name: staging_container
  prod:
    build:
      context: .
      dockerfile: dockerfile3
    container_name: prod_container
```
4. Run the compose file with "docker compose up -d", this will start building the docker images and run the containers in detached mode.
5. You can see that multiple containers were started simutaneously.

## Explanation:
### version: "3.9" 
specifies the version of the Docker Compose file format.
### services
Services are the individual containers or applications being managed. In this file, there are three services: dev, stage, and prod.
### build: context: .
Specifies the build context, which is the current directory (.). All files needed to build the image should be in this directory or its subdirectories.
### dockerfile: Dockerfile1
Specifies the Dockerfile to use for building the image (Dockerfile1).
### container_name
container_name: dev_container: The container will be explicitly named dev_container.

### You can also try the below project. Just check the code and execute the compose files with "docker compose up -d"
 
 https://github.com/dockersamples/example-voting-app.git
 
 
