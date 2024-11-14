# Distroless Images & Docker Multistage Builds 
## Distroless Images
Distroless images are Docker images built by Google that contain only your application and its runtime dependencies, without a package manager, shell, or other operating system utilities. They are designed to be minimal, secure, and efficient, as they reduce the attack surface and potential vulnerabilities.
### Why do we need Distroless Images?
1. Reduces Attack Surface: Without a shell or other utilities, there are fewer vulnerabilities that an attacker could exploit.
2. Minimizes Image Size: By including only what is necessary for running the application, the image becomes smaller, which can improve load and deployment times.

Distroless images are especially useful in production environments where you want to minimize the size of the image and reduce exposure to security risks. By removing unnecessary components, these images also tend to be smaller and load faster.

## Distroless Images Implementation
pull a distoless image and check it's size

https://github.com/GoogleContainerTools/distroless

docker pull gcr.io/distroless/static-debian12 

If you check the size of the above image, it will be very low.

You can also use this distroless images in your dockerfile, but you won't be able to login to your container, since it doesn't have shell.

Consider Day-3 Dockerfile, using distro image in it (you can't build the image, because we're trying create a directory, group, user and distroless image can't have shell)

FROM gcr.io/distroless/static-debian11

LABEL name="ravikiran"

ARG T_VERSION='1.6.6'\

    P_VERSION='1.8.0'
    
RUN mkdir /app \

    && groupadd appuser && useradd -r -g appuser appuser \
    && apt update && apt install -y jq net-tools curl wget unzip\
    && apt install -y nginx iputils-ping \
    && wget https://releases.hashicorp.com/terraform/${T_VERSION}/terraform_${T_VERSION}_linux_amd64.zip \
    && wget https://releases.hashicorp.com/packer/${P_VERSION}/packer_${P_VERSION}_linux_amd64.zip\
    && unzip terraform_${T_VERSION}_linux_amd64.zip  && unzip packer_${P_VERSION}_linux_amd64.zip\
    && chmod 777 terraform && chmod 777 packer\
    && ./terraform version && ./packer version
    
WORKDIR /app

USER  appuser

ENV AWS_ACCESS_KEY_ID=DUMMYKEY\
    AWS_SECRET_KEY_ID=DUMMYKEY\
    AWS_DEFAULT_REGION=US-EAST-1A
    
#COPY index.nginx-debian.html /var/www/html/
#COPY scorekeeper.js /var/www/html
#ADD  style.css /var/www/html

ADD https://releases.hashicorp.com/terraform/1.9.0/terraform_1.9.0_linux_amd64.zip /var/www/html

EXPOSE 80

USER  appuser

CMD ["nginx","-g","daemon off;"]

In the above file, you can observe that we included the things to be downloaded are placed under in one RUN command, that is for decreasing the layers of the image, which is a best practice.

## Multistage builds
Multistage builds allow you to use multiple stages in a Dockerfile to separate build steps and dependencies from the final production image. With multistage builds, you can create a smaller, optimized final image by only copying the necessary artifacts from previous stages and discarding any unnecessary files or dependencies.

Docker multi-stage builds allow you to create more efficient Dockerfiles by using multiple FROM statements. Each stage in the build process can have its own base image and set of instructions, enabling you to compile code, run tests, and package your application without including unnecessary build tools and dependencies in the final image.

### Scenario for Using Multistage Builds
Consider a Java application that requires Maven to compile but doesn’t need Maven to run in production. You could use a multistage build to compile the application in a Maven container and then copy the compiled files to a lightweight Java runtime image for production.

## Implementation of Docker Multistage Builds

clone the below repository (this is a java application, we are just containerizing the application, which is already coded)

git clone https://github.com/spring-projects/spring-petclinic.git

### Move to the cloned Repo/Directory & write a dockerfile
   cd <directory>
   
   vim Dockerfile
   
   #Stage 1: Build stage
   
   FROM maven:3.8.5-openjdk-17-slim AS build
   
   WORKDIR /app
   
   COPY . .
   
   RUN ./mvnw package -DskipTests
   
   #Stage 2: Runtime stage
   
   FROM openjdk:17-jdk-slim AS runtime
   
   WORKDIR /app
   
   COPY --from=build /app/target/*.jar app.jar

   EXPOSE 8080
   
   CMD ["java", "-jar", "app.jar"]

### Expplanation:
### Stage-1 (Build Stage)
### FROM maven:3.8.5-openjdk-17-slim AS build
This starts the build stage using a lightweight Docker image with Maven 3.8.5 and OpenJDK 17. Labeling it as build allows us to reference this stage in later steps.
### WORKDIR /app
Sets the working directory to /app within the container. Any subsequent commands run within this directory.
### COPY . .
Copies all files from the current directory on the host machine into the /app directory in the container. This includes the source code, pom.xml, and any other files required for building the application.
### RUN ./mvnw package -DskipTests
Runs the Maven build command to compile the code and package it as a JAR file, skipping any tests.
The package phase includes all necessary steps to compile the code and produce the final JAR file in the target/ directory.
### Stage-2 (Run Stage)
### FROM openjdk:17-jdk-slim AS runtime
Starts a new stage, labeled runtime, which uses a lightweight OpenJDK 17 image. This image includes the JDK required to run the Java application.
### WORKDIR /app
Sets the working directory to /app again for consistency in the runtime stage
### COPY --from=build /app/target/*.jar app.jar
Copies the JAR file generated in the build stage into the runtime stage’s /app directory, naming it app.jar. This command references the first stage (build) and grabs the built JAR from the target directory.
### EXPOSE 8080:
Tells Docker that this container will listen on port 8080, which is the default port for many Java web applications.
### CMD ["java", "-jar", "app.jar"]
This will start your Java application, running the main method defined in the app.jar file.

### You can reduce the image size of the application by just changing the runtime image in the second stage
Modify runtime image of the above Dockerfile and build the image again. You can see that the size of the image is reduced. This is because, first we used JDK at runtime, later we used JRE which have runtime only unlike JDK which which has additional tools for developing Java applications.

### Modified Dockerfile 
# Stage 1: Build stage 284
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /app

COPY . .

RUN ./mvnw package -DskipTests

# Stage 2: Runtime stage
FROM openjdk:11-jre-slim

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
