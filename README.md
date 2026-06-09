# VProfile Containerization Project

Welcome to the **VProfile** containerization project! This project demonstrates the modern deployment of a multi-tier Java Spring MVC web application using containerization technologies (Docker, Docker Compose) as well as continuous integration workflows.

## 📌 Project Overview
VProfile is a social networking web application. The core objective of this project is to take a traditional multi-tier architectural stack and containerize all of its components for streamlined deployment, scaling, and platform independence.

## 🏗️ Architecture Stack & Container Design
The application is structured into a total of **5 containers**, operating together to deliver the web platform:

### Custom Built Images (from Dockerfiles)
We have created DockerHub repositories for the following custom components (`app`, `db`, `web`) which are built locally using provided `Dockerfiles`:
- **Nginx (`web` image)**: Reverse proxy and load balancer built through a Dockerfile, routing incoming user requests to the backend application server.
- **Tomcat (`app` image)**: The application server hosting the Java Spring MVC application `.war` file, built through a Dockerfile.
- **MySQL (`db` image)**: Relational database storing user profiles, credentials, and app data, populated via a custom Dockerfile with initial schema SQL.

### Base Images (fetched from DockerHub)
- **Memcached**: In-memory caching system used to cache database queries and session data, improving performance. Used directly from the base image.
- **RabbitMQ**: Message broker for handling queues and asynchronous message passing between application components. Used directly from the base image.

## 🚀 Technologies Used
- **Application Logic:** Java 17, Spring MVC 6, Spring Boot 3, Hibernate
- **Containerization:** Docker, Docker Compose
- **Build Tool:** Maven
- **CI/CD Pipeline:** Jenkins
- **Code Analysis:** SonarQube, Checkstyle
- **Artifact Repository:** Sonatype Nexus
- **Configuration Management:** Ansible
- **Virtualization & Provisioning:** Vagrant (used to setup Docker in isolated environments) and AWS EC2.

## 📂 Project Structure
```text
.
├── Docker-files/           # Custom Dockerfiles for app, web, and db tiers
├── ansible/                # Ansible playbooks for VM-based provisioning
├── src/                    # Source code of the Java Spring MVC application
├── vagrant/                # Vagrantfiles for local Docker VM provisioning
├── Jenkinsfile             # Declarative Jenkins pipeline for CI/CD
├── docker-compose.yml      # Multi-container Docker application orchestration
└── pom.xml                 # Maven project configuration and dependencies
```

## 🛠️ Infrastructure Setup & Deployment Flow
This project follows a distinct deployment pipeline to containerize and run the platform:
1. **VM Provisioning**: Setup an isolated environment by installing Docker on a Virtual Machine using Vagrant.
2. **Build Custom Images**: Through the `Dockerfile`s stored in `Docker-files/`, build the `app` (Tomcat), `db` (MySQL), and `web` (Nginx) images.
3. **Artifact Registry**: Push the customized application images to designated DockerHub repositories.
4. **AWS EC2 Provisioning**: Set up an AWS EC2 instance as the production/staging docker host.
5. **Docker Compose**: A cohesive `docker-compose.yml` file is crafted linking our 3 custom images (app, db, web) along with the 2 standard base images (Memcached, RabbitMQ).
6. **Deployment**: Finally, the stack is continuously deployed on the EC2 instance using the Docker Compose definition.

## 🛠️ Prerequisites
To run this project locally, ensure you have the following installed on your host machine:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Java 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html) & [Maven](https://maven.apache.org/) (If you wish to build the code locally)

## 🐳 Running Locally with Docker Compose

1. **Clone the repository**
   ```bash
   git clone
   cd vprofile-project-docker
   ```

2. **Build the Application** (Optional)
   If you have made changes to the Java source code, build the artifact first:
   ```bash
   mvn clean install -DskipTests
   ```
   *(Note: The `docker-compose.yml` uses pre-built images or builds from `./Docker-files/app` which should contain the `.war` file.)*

3. **Spin up the Containers**
   Run the following command in the root directory to build custom images and start the services:
   ```bash
   docker-compose up -d --build
   ```

4. **Verify the Services**
   Check that all 5 containers are up and running:
   ```bash
   docker-compose ps
   ```

5. **Access the Application**
   Once containers are running, navigate to:
   ```text
   http://localhost
   ```
   *(Nginx automatically routes traffic from port 80 to the Tomcat application on port 8080)*

6. **Tear Down**
   To stop and remove networks, containers, and volumes:
   ```bash
   docker-compose down
   ```

## ⚙️ CI/CD Pipeline
The project includes a `Jenkinsfile` for continuous integration. The pipeline performs the following stages:
1. **BUILD**: Compiles the source code and packages it into a `.war` file using Maven.
2. **UNIT TEST**: Runs unit tests (`mvn test`).
3. **INTEGRATION TEST**: Runs integration tests.
4. **CODE ANALYSIS WITH CHECKSTYLE**: Checks code structure against predefined rules.
5. **CODE ANALYSIS with SONARQUBE**: Scans code for bugs, vulnerabilities, and code smells.
6. **Publish to Nexus Repository Manager**: Uploads the vetted `.war` artifact securely to a Nexus artifact repository.
