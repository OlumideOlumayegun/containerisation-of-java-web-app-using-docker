# Containerisation of a Multi-Tier Java Web Application Using Docker, Docker Compose, Multi-Stage Builds, and Docker Hub

![project banner](./images/project-banner.png)

## Project Overview

This project demonstrates how to containerise an existing Java-based multi-tier web application (VProfile) using Docker. The application consists of multiple interconnected services that were previously deployed on virtual machines. The project walks through transforming the application into a portable, reproducible, and production-ready containerised solution using Docker, multi-stage Docker builds, Docker Compose, and Docker Hub.

By the end of this project, you will have:

* Containerised a multi-tier Java web application.
* Created production-ready Docker images.
* Implemented multi-stage Docker builds.
* Orchestrated multiple containers with Docker Compose.
* Published custom images to Docker Hub.
* Produced Infrastructure-as-Code (IaC) artifacts for repeatable deployments.

---

# Project Architecture

The application consists of five services:

![project banner](./images/architecture-drawing.png)

---

# Technologies Used

* Docker Engine
* Docker Compose
* Docker Hub
* Multi-stage Docker Builds
* Apache Tomcat
* Nginx
* MySQL
* RabbitMQ
* Memcached
* Maven
* JDK 21
* Git
* Vagrant
* Ubuntu Linux

---

# Learning Objectives

After completing this project, you will be able to:

* Analyse an existing application before containerisation.
* Select appropriate Docker base images.
* Build custom Docker images.
* Implement multi-stage Docker builds.
* Customise official Docker images.
* Configure Docker networking.
* Manage Docker volumes.
* Deploy multi-container applications.
* Publish images to Docker Hub.
* Apply Infrastructure-as-Code principles.

---

# Step 1 – Analyse the Existing Application

Before writing any Dockerfiles, understand how the application currently works.

Identify:

* Services used
* Build tools
* Runtime dependencies
* Software versions
* Deployment steps
* Configuration files
* Ports
* Database setup

For the VProfile application, the services are:

| Service   | Purpose                 |
| --------- | ----------------------- |
| Tomcat    | Java application server |
| Nginx     | Reverse proxy           |
| MySQL     | Database                |
| Memcached | Cache                   |
| RabbitMQ  | Message broker          |

Development tools:

* Maven
* JDK 21

---

# Step 2 – Identify Docker Base Images

Locate official images on Docker Hub.

Example images used:

| Component | Docker Image                         |
| --------- | ------------------------------------ |
| MySQL     | mysql:8.0.33                         |
| Tomcat    | tomcat:10-jdk21                      |
| Maven     | maven:3.9.9-eclipse-temurin-21-jammy |
| Nginx     | nginx:latest                         |
| Memcached | memcached:latest                     |
| RabbitMQ  | rabbitmq:latest                      |

Always match image versions with the application's software versions whenever possible.

---

# Step 3 – Create a Docker Hub Account

Create a Docker Hub account.

(Optional)

Create an Organisation to:

* manage repositories
* collaborate with teams
* store private images

Create repositories for:

```
vprofile-app
vprofile-db
vprofile-web
```

---

# Step 4 – Prepare the Development Environment

Clone the project repository.

```
git clone https://github.com/OlumideOlumayegun/containerisation-of-java-web-app-using-docker.git
```

Switch to the **containerisation-of-java-web-app-using-docker** directory.

```
cd containerisation-of-java-web-app-using-docker
```

Provision the Ubuntu VM using Vagrant.

```
vagrant up
```

SSH into the VM.

```
vagrant ssh
```

---

# Step 5 – Install Docker Engine

With the Ubuntu virtual machine running, install the **Docker Engine** using the [official Docker installation instructions](https://docs.docker.com/engine/install/).

Inside the Ubuntu VM:

* Update packages
* Install prerequisites
* Add Docker repository
* Install Docker Engine
* Verify Docker service
* Add your user to the docker group

Verify installation:

```
docker images
```

---

# Step 6 – Create the Application Docker Image

The application image uses a **multi-stage Docker build**.

## Stage 1

Build the application using Maven.

Tasks:

* Clone source code
* Checkout branch
* Execute Maven build
* Produce WAR file

## Stage 2

Start from the Tomcat image.

Tasks:

* Remove default webapps
* Copy WAR file
* Rename to ROOT.war
* Deploy application

Benefits:

* Smaller image
* Faster deployment
* Better security
* No Maven in production image

---

# Step 7 – Create the Database Docker Image

Use the official MySQL image.

Configure:

* Root password
* Database name
* SQL initialization

Copy

```
db_backup.sql
```

into

```
/docker-entrypoint-initdb.d/
```

When the container starts, MySQL automatically:

* creates the database
* imports schema
* loads initial data

---

# Step 8 – Create the Nginx Docker Image

Use the official Nginx image.

Replace the default configuration with your own reverse proxy configuration.

Configure Nginx to:

* listen on port 80
* forward traffic to Tomcat on port 8080

---

# Step 9 – Decide Which Images Need Customisation

Create custom images for:

* Application
* Database
* Web server

Use official images directly for:

* Memcached
* RabbitMQ

This reduces maintenance and build time.

---

# Step 10 – Create the Docker Compose File

Create

```
compose.yaml
```

Define five services:

* database
* memcached
* rabbitmq
* application
* web

Configure:

* image/build
* container name
* ports
* environment variables
* volumes
* networking

---

# Step 11 – Configure Persistent Storage

Create named Docker volumes.

Example:

```
vprofiledbdata
vprofileappdata
```

Use volumes to persist:

* database data
* application files (optional)

---

# Step 12 – Configure Container Networking

Docker Compose automatically creates an internal network.

Configure containers so they communicate using service names.

Example:

```
MySQL
↓

vprodb
```

```
Tomcat
↓

vproapp
```

```
RabbitMQ
↓

vpromq01
```

These names must match the application's configuration.

---

# Step 13 – Build the Images

Build all custom images.

```
docker compose build
```

Docker Compose will:

* locate Dockerfiles
* build images
* tag images

Verify:

```
docker images
```

---

# Step 14 – Deploy the Application

Start all services.

```
docker compose up -d
```

Docker Compose automatically:

* creates network
* creates volumes
* pulls official images
* starts all containers

---

# Step 15 – Verify Running Containers

Check containers.

```
docker ps
```

Verify:

* MySQL
* Tomcat
* Nginx
* RabbitMQ
* Memcached

All containers should be running.

---

# Step 16 – Test the Application

Determine the VM IP.

Example:

```
ip addr show
```

Open:

```
http://<VM-IP>
```

Verify:

* Home page loads
* Login succeeds
* Database connectivity
* RabbitMQ functionality
* Memcached caching

Successful login confirms:

* Nginx → Tomcat
* Tomcat → MySQL
* Tomcat → RabbitMQ
* Tomcat → Memcached

are all functioning correctly.

---

# Step 17 – Publish Images to Docker Hub

Authenticate.

```
docker login
```

Push images.

```
docker push username/vprofileapp

docker push username/vprofiledb

docker push username/vprofileweb
```

These images can later be deployed to Kubernetes.

---

# Step 18 – Clean Up

Stop the application.

```
docker compose down
```

Remove unused volumes.

```
docker volume prune
```

Remove unused resources.

```
docker system prune -a
```

Shutdown the VM.

```
vagrant halt
```

---

# Project Workflow Summary

```text
Analyse Existing Application
            │
            ▼
Gather Services & Versions
            │
            ▼
Select Docker Base Images
            │
            ▼
Write Dockerfiles
(App • DB • Web)
            │
            ▼
Implement Multi-Stage Build
            │
            ▼
Create Docker Compose File
            │
            ▼
Build Images
            │
            ▼
Deploy Containers
            │
            ▼
Validate Application
            │
            ▼
Push Images to Docker Hub
            │
            ▼
Infrastructure as Code
```

---

# Key Takeaways

* Containerisation begins with understanding the application's architecture and deployment process.
* Multi-stage Docker builds produce smaller, more secure production images by separating build and runtime environments.
* Official Docker images reduce maintenance effort, while custom images are reserved for services requiring application-specific configuration.
* Docker Compose simplifies multi-container deployments by defining services, networking, volumes, and environment variables in a single declarative file.
* Publishing images to Docker Hub enables consistent deployments and provides reusable artifacts for CI/CD pipelines and Kubernetes deployments.
* The complete environment—including Dockerfiles, Docker Compose configuration, and Vagrant setup—is defined as code, making the application portable, reproducible, and ready for cloud-native deployment.
