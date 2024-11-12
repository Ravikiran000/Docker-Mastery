# Docker Volumes & Bind Mounts
## Why Volumes & Bind Mounts are needed:
Imagine you have an application packaged in a container that stores data in a database. If that container unexpectedly goes down, any data saved by the application would typically be lost. However, by using Docker volumes or bind mounts, you can ensure that the data is preserved and remains intact, even if the container stops running.

1. Docker Volumes:  Volumes are stored in a part of the host filesystem managed by Docker. Volumes provide a layer of abstraction over the host filesystem, ensuring that container data is decoupled from the host.
2. Bind Mounts: Bind mounts can map any file or directory on the host into the container. Provides direct access to the host filesystem, which can result in better performance for certain use cases.

## Volumes vs Bind Mounts
### Volumes:

Managed by Docker.
Stored in a part of the host filesystem which is managed by Docker.
Preferred method for data persistence.

### Bind Mounts:

Maps a file or directory on the host to a file or directory in the container.
More complex but provides flexibility to interact with the host system.

# Implementation
You can continue with Day-1 EC2 & EBS setup

## Using Volumes

### List existing volumes:
docker volume ls
### Create a new volume:
docker volume create test_volume
### Run a mongoDB container & login into it
docker run -d --name app1 mongo:latest

docker exec -it app1 mongosh
### check for databases
show dbs;

Add the below data

db.helo.insertMany([
{ "_id" : 1, "name" : "Matt", "status": "active", "level": 12, "score":202},
        	{ "_id" : 2, "name" : "Frank", "status": "inactive", "level": 2, "score":9},
        	{ "_id" : 3, "name" : "Karen", "status": "active", "level": 7, "score":87},
        	{ "_id" : 4, "name" : "Katie", "status": "active", "level": 3, "score":27, "status": "married", "emp": "yes", "kids": 3},
        	{ "_id" : 5, "name" : "Matt1", "status": "active", "level": 12, "score":202},
        	{ "_id" : 6, "name" : "Frank2", "status": "inactive", "level": 2, "score":9},
        	{ "_id" : 7, "name" : "Karen3", "status": "active", "level": 7, "score":87},
        	{ "_id" : 8, "name" : "Katie4", "status": "active", "level": 3, "score":27, "status": "married", "emp": "yes", "kids": 3}
        	])

Show data in database

db.helo.find()

exit from container

exit
### stop & start the container again and login into it and search for data (it won't be present there since the container is down)
docker stop app1

docker start app1

docker exec -it app1 mongosh

show dbs;
### Attach volume to the container, run & login to the container, add data and exit
docker stop app1

docker rm app1

docker run -d --name app1 -v test_volume:/app mongo:latest

docker exec -it app1 mongosh

(refer above steps to add data)
### stop & start the container again and login into it and search for data (data will be present since you atttached volume)
docker stop app1

docker start app1

docker exec -it app1 mongosh

show dbs;

db.helo.find()

## Using Bind Mounts
### Run a container without network access:

docker run --rm -d --name troubleshootingtools --network none troubleshootingtools:v1

### Run a container with Docker socket mounted:

docker run --rm -d --name troubleshootingtools -v /var/run/docker.sock:/var/run/docker.sock --network none troubleshootingtools:v1
