Multi-Stage Docker Builds & Production Jenkins Agent Images
============================================================
This day focuses on creating optimized, secure, and production-ready Docker images for Jenkins agents.

🎯 Learning Objectives

By the end of today, you'll understand:

What is a Multi-Stage Build?
Why Multi-Stage Builds are important
Builder image vs Runtime image
Image size optimization
Docker layer optimization
Security best practices
Building a production-ready Jenkins agent

Why Do We Need Multi-Stage Builds?
-----------------------------------

Imagine you're building a Java application.

Without Multi-Stage Build:

Docker Image

Ubuntu
Java
Maven
Git
Build Files
Source Code
Compiled JAR
Temporary Files

Image Size:

1.2 GB

Most of these files are not needed to run the application.


What is a Multi-Stage Build?
----------------------------
A Multi-Stage Build separates:

Build Environment
Runtime Environment

Example:
Stage 1

Maven
↓

Compile Source Code
↓

Generate JAR

----------------------------

Stage 2

JRE Only
↓

Copy JAR

↓

Run Application

Now the runtime image contains only:

JRE
Application JAR

No Maven.

No source code.

No temporary files.

Single-Stage Example
---------------------
FROM maven:3.9.6-eclipse-temurin-17

WORKDIR /app

COPY . .

RUN mvn clean package

CMD ["java","-jar","target/app.jar"]

Problems:

Maven remains in the final image
Source code remains
Build cache remains
Large image

Multi-Stage Example
----------------------
# Build Stage
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .

RUN mvn dependency:go-offline

COPY src ./src

RUN mvn clean package -DskipTests

# Runtime Stage
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=builder /app/target/app.jar app.jar

ENTRYPOINT ["java","-jar","app.jar"]

Advantages:

Smaller image
Faster deployments
Better security
No build tools in production

Multi-Stage Build Flow
----------------------
Developer
      │
Source Code
      │
Docker Build
      │
Builder Stage
(Maven)
      │
Creates JAR
      │
Runtime Stage
(JRE)
      │
Production Image

Why is This Useful for Jenkins Agents?
---------------------------------------
Suppose your Jenkins pipeline needs:

Git
Maven
Docker CLI
AWS CLI
kubectl
Helm

Instead of installing everything every time the pipeline runs, create a reusable Jenkins agent image.

Example Jenkins Agent Dockerfile
-----------------------------------
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y \
    git \
    curl \
    unzip \
    wget \
    openjdk-17-jdk \
    maven \
    python3 \
    python3-pip \
    docker.io && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
    -o awscliv2.zip && \
    unzip awscliv2.zip && \
    ./aws/install

RUN curl -LO "https://dl.k8s.io/release/v1.33.0/bin/linux/amd64/kubectl"

RUN install -m 0755 kubectl /usr/local/bin/kubectl

CMD ["bash"]


Image Optimization
--------------------
Instead of:

RUN apt update

RUN apt install git

RUN apt install curl

RUN apt install unzip

Use:

RUN apt-get update && \
    apt-get install -y \
    git \
    curl \
    unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

Benefits:

Fewer layers
Smaller image
Better build performance

Docker Layer Caching
---------------------
Bad:

COPY . .

RUN mvn package

Every code change invalidates the cache.

Better:

COPY pom.xml .

RUN mvn dependency:go-offline

COPY src ./src

RUN mvn package

Dependencies are cached separately, making rebuilds much faster.

Security Best Practices
------------------------
Avoid running containers as root.

Create a non-root user:

RUN useradd -m -u 1001 jenkins

USER jenkins

Benefits:

Reduces privilege escalation risks
Aligns with container security best practices

Real Production Pipeline
-------------------------
GitHub
     │
Webhook
     │
Jenkins
     │
Docker Agent
     │
Checkout Code
     │
Maven Build
     │
SonarQube Scan
     │
OWASP Dependency Check
     │
Docker Build
     │
Trivy Scan
     │
Push Image
     │
Deploy to Kubernetes

One pre-built Jenkins agent image can execute all these stages consistently.

🛠 Hands-on Lab
-------------------
Task 1

Create this directory structure:

jenkins-agent/
├── Dockerfile
└── README.md
Task 2

Write a Dockerfile that installs:

Git
Java 17
Maven
Python 3
pip
curl
unzip
AWS CLI
kubectl
Task 3

Build the image:

docker build -t jenkins-agent:v1 .
Task 4

Verify the installed tools:

docker run --rm jenkins-agent:v1 git --version
docker run --rm jenkins-agent:v1 mvn -version
docker run --rm jenkins-agent:v1 java -version
docker run --rm jenkins-agent:v1 python3 --version
docker run --rm jenkins-agent:v1 aws --version
docker run --rm jenkins-agent:v1 kubectl version --client
Task 5

Check the image size:

docker images

Think about which components contribute most to the image size and how multi-stage builds or smaller base images could reduce it.
