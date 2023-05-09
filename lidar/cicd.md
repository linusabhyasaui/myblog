# LIDAR: CICD

----------

[HOME]({{ site.baseurl }}/) || [LIDAR]({{ site.baseurl }}/lidar.html) 

----------

## Introduction

Continuous Integration and Continuous Deployment (CI/CD) are essential parts of modern software development practices. 
They help streamline the development process, improve code quality, and ensure faster and more efficient software delivery. 
Docker is a powerful tool for building, shipping, and running applications in a consistent and reliable manner. 
In this report, we will discuss the creation and function of a Dockerfile, docker-compose, and gitlab-ci.yml files, 
which are used in a CI/CD pipeline to build, test, and deploy a Java Spring Boot application.

## Dockerfile

The Dockerfile is a text file that contains a set of instructions for building a Docker image. 
It specifies the base image, environment variables, required packages, and commands necessary to build and run the application. 

For the LIDAR Backend project, the Dockerfile is designed as following:

~~~~~~ Dockerfile
FROM openjdk:17-alpine
ARG JAR_FILE=build/libs/lidar-spine-0.0.1-SNAPSHOT.jar
RUN apk update && apk add tzdata
ENV TZ=Asia/Jakarta
COPY ${JAR_FILE} app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
~~~~~~~~

<details>
    <summary>Explanation</summary>

* `FROM openjdk:17-alpine` - 
This specifies the base image that this Docker image will be built on. 
In this case, we are using the openjdk:17-alpine image, which is a lightweight Linux distribution with OpenJDK 17 installed.
<br>

* `ARG JAR_FILE=build/libs/lidar-spine-0.0.1-SNAPSHOT.jar` - 
This line specifies a build-time argument called JAR_FILE, which is used to specify the name and location of the application jar file. 
By default, it is set to build/libs/lidar-spine-0.0.1-SNAPSHOT.jar.
<br>

* `RUN apk update && apk add tzdata` - 
This line updates the package index and installs the tzdata package, which is required to set the timezone in the next line.
<br>

* `ENV TZ=Asia/Jakarta` - 
This sets the timezone to Asia/Jakarta.
<br>

* `COPY ${JAR_FILE} app.jar` - 
This copies the application jar file specified by the JAR_FILE argument to the container and renames it to app.jar.
<br>

* `EXPOSE 8080` - 
This exposes port 8080, which is the default port that the application runs on.
<br>

* `ENTRYPOINT ["java","-jar","/app.jar"]` - 
This specifies the command that should be run when the container starts. 
In this case, it runs the Java Virtual Machine and starts the application by running the app.jar file.

</details>
<br>

## docker-compose

The docker-compose.yml file is used to define and run multi-container Docker applications. 
It specifies the services, networks, and volumes required to run the application. 

Here is the docker-compose.yml file as designed for the LIDAR Backend project:

~~~~~~ yaml
version: '3'
services:
  web:
    image: apap-registry.cs.ui.ac.id/linus.abhyasa/lidar-spine
    restart: always
    ports:
      - "10009:8080"
    depends_on:
      - db
    networks:
      - frontend-network
      - backend-network

    environment:
      POSTGRES_HOST: db
      POSTGRES_PORT: 5432
      POSTGRES_DB: lidardb
      POSTGRES_USER: lidar
      POSTGRES_PASSWORD: Ohbaip9jahPhoh3ohcae
      
  db:
    image: 'postgres:15-alpine'
    environment:
      POSTGRES_DB: lidardb
      POSTGRES_USER: lidar
      POSTGRES_PASSWORD: Ohbaip9jahPhoh3ohcae

    volumes:
      - ./postgres-data:/var/lib/postgresql/data

    ports:
      - "5432:5432"

    networks:
      - backend-network

networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
~~~~~~~~

<details>
    <summary>Explanation</summary>

* `version: '3'`: 
specifies the version of the Docker Compose file format that this file adheres to.
<br>

* `services`: 
this section defines the services that make up the application. 
In this case, there are two services defined, web and db.
<br>

<details>
<summary>Web</summary>

* `web`: 
this is the name of the first service.
<br>

* `container_name`: 
this sets the name of the container to be created. 
In this case, it is set to lidar-spine.
<br>

* `image`: 
this specifies the image to use for the container. 
It uses the value of the $REGISTRY_SERVER environment variable to construct the image name.
In the case for the LIDAR project, the registry server stores previously compiled images of the project, simplifying deployment.
<br>

* `restart`: 
specifies the restart policy for the container. 
In this case, it is set to always, which means that the container will always be restarted unless it is explicitly stopped.
This is needed in this case due to the server on which the LIDAR project is hosted is prone to restarts.
<br>

* `ports`: 
this specifies the ports to expose on the container and the host. 
In this case, it exposes port 8080 on the container and 10009 on the host.
The port 10009:8080 is used as the Springboot project is set to deploy on port 8080 but the provided external port is 10009.
<br>

* `depends_on`:
this specifies that this service requires another service to be able to run. 
In this case, the web service requires the db service to be functional.
<br>

* `networks`: 
this specifies the networks the container should use. 
In this case, the service is assigned to use the **frontend-network** and the **backend-network**.
This is so that the web service can access both the external network and the internal network.

</details>
<br>

<details>
<summary>db</summary>

* `db`:
this is the name of the second service.
<br>

* `image`:
this sets the image for this service to postgres, this is due to the LIDAR project using postgres as the database system.
<br>

* `environment`:
Like before this defines the environment variables used.
In this case, the environment variables are those required to set up the database.
<br>

* `volumes`: 
this mounts a volume from the host into the container. 
In this case, it mounts the system's stored data to the container.
<br>

* `ports`:
In this case, it exposes port 5432 on the container and the host.
The port 5432:5432 is used as postgres by default uses port 5432.
<br>

* `networks`:
In this case, the service is assigned to only use the **backend-network**.
This is so that the db service can access both the internal network without being exposed externally.

</details>
<br>

* `networks`:
this section defines the network paths that are used within the application. 
In this case, there are two networks defined, **frontend-network** and **backend-network**.
These networks are both bridge networks. 
The use of networks in this application is so that external users do not have direct access to the database which is on
the **backend-network** only.

</details>
<br>

## Notes:

The Frontend Docker is similar to that of the Backend but set up for a next.js application instead of a java-springboot +
postgres application.

----------

[next>](cicd2.md)

----------

 © {{ site.copyright }} --- {{ site.author }} --- Version: {{ site.version }}.
