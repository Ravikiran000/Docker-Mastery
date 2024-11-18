# Docker Compose
Docker Compose is a tool used for defining and running multi-container Docker applications. With Docker Compose, you can use a YAML file to configure your application's services, networks, and volumes, enabling you to manage multiple containers as a single application.

### Scenario: 
You’re building a web application consisting of: A frontend built with React. A backend API developed in Node.js. A database using MySQL. Instead of running each service separately using docker run commands, you decide to use Docker Compose for simplicity.

## Implementation:
### Install docker compose on the docker ec2 instance

1. Here we are creating multiple dockerfiles and using docker compose to execute them simultaneously.

Dockerfile1:

FROM ubuntu:latest

LABEL name="ravikiran"

ENV AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXXXXXXXXXXX\
    AWS_SECRET_KEY_ID=XXXXXXXXXXXXXXXXXXXXXXXX\
    AWS_DEFAULT_REGION=US-EAST-1A

ARG T_VERSION='1.6.6'\
    P_VERSION='1.8.0'

RUN apt update && apt install -y jq net-tools curl wget unzip\
    && apt install -y nginx iputils-ping

CMD ["nginx","-g","daemon off;"]

2. Create 2 more dockerfiles with names as "Dockerfile2" & "Dockerfile3"
3. create a docker compose file as "docker-compose.yml"
   
version:
services:
  dockerfile1:
    build:
      contex: .
      dockerfile: dockerfile1
    container_name: dev
  dockerfile2:
    build:
      context: .
      dockerfile: dockerfile2
    container_name: staging
  dockerfile3:
    build:
      context: .
      dockerfile: dockerfile3
    container_name: prod

4. run the compose file with "docker-compose up", this will start building the docker images and run the containers.
5.       
