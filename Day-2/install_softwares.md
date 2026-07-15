Installing Software Inside a Docker Image
==========================================

Goal: Learn how to install software such as Git, Java, Maven, Node.js, Docker CLI, AWS CLI, and kubectl inside a Docker image. 
By the end of today, you'll understand why each installation step exists and how to write clean Dockerfiles.

Today's Learning Objectives
----------------------------

You'll learn:

Package managers
apt vs apt-get
Why we run apt-get update
Installing multiple packages
Cleaning package cache
Best practices
Real Jenkins agent examples

Step 1: Package Managers
-------------------------
Different Linux distributions use different package managers.

| OS           | Package Manager |
| ------------ | --------------- |
| Ubuntu       | apt / apt-get   |
| Debian       | apt / apt-get   |
| Amazon Linux | yum / dnf       |
| CentOS       | yum             |
| Fedora       | dnf             |
| Alpine       | apk             |


Example:
--------
Ubuntu

RUN apt-get update

Amazon Linux

RUN yum update -y

Alpine

RUN apk add git

Interview Question

Q: Why does the package manager change?

Answer: Because different Linux distributions use different package management systems and repositories.

Step 2: Why apt-get update?
---------------------------
One of the biggest beginner mistakes is writing:

RUN apt-get install git

This may fail because the package list is outdated.

Correct approach:

RUN apt-get update

What happens?

Ubuntu Repository
       │
       ▼
Download latest package index
       │
       ▼
Store locally

Then:

RUN apt-get install git

Docker now knows where to download Git from.


Step 3: Installing Software
----------------------------
Install Git:

RUN apt-get update && \
    apt-get install -y git
Why -y?

Without -y:

Do you want to continue? (Y/N)

Docker builds are non-interactive, so the build would hang or fail.

With:

-y

Docker automatically answers Yes.


Step 4: Installing Multiple Packages
------------------------------------
Instead of:

RUN apt-get update

RUN apt-get install -y git

RUN apt-get install -y curl

RUN apt-get install -y wget

Use:

RUN apt-get update && \
    apt-get install -y \
        git \
        curl \
        wget
Why?

Because each RUN instruction creates a new image layer.

Fewer layers generally mean:

Smaller images
Faster builds
Better cache utilization


Step 5: Cleaning Cache
-----------------------
After installation:

RUN rm -rf /var/lib/apt/lists/*

Why?

During apt-get update, Ubuntu downloads package indexes into:

/var/lib/apt/lists/

These files are only needed during installation.

Deleting them:

Reduces image size
Doesn't remove the installed software

Production Example
RUN apt-get update && \
    apt-get install -y \
        git \
        curl \
        unzip \
        wget && \
    rm -rf /var/lib/apt/lists/*

This is the pattern you'll see in many production Dockerfiles.


Step 6: Installing Java
------------------------
Example:

RUN apt-get update && \
    apt-get install -y openjdk-17-jdk && \
    rm -rf /var/lib/apt/lists/*

Verify:

RUN java -version

Step 7: Installing Maven
-------------------------
Option 1: Use Ubuntu packages
------------------------------
RUN apt-get install -y maven

Easy, but the version may be old.

Option 2: Download a specific version (preferred when you need consistency)
-------------------------------------------------------------------------------
ARG MAVEN_VERSION=3.9.9

RUN wget https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz


Extract it:

RUN tar -xzf apache-maven-${MAVEN_VERSION}-bin.tar.gz

Move it:

RUN mv apache-maven-${MAVEN_VERSION} /opt/maven

Configure environment variables:

ENV MAVEN_HOME=/opt/maven
ENV PATH=$PATH:$MAVEN_HOME/bin

Verify:

RUN mvn -version

Sometimes a Jenkins agent needs to build Docker images.

Install Docker CLI:

RUN apt-get update && \
    apt-get install -y docker.io

Important: This installs the Docker client. To actually build images, the container also needs access to a Docker daemon (for example, by mounting /var/run/docker.sock or using Docker-in-Docker). We'll cover those approaches later in the bootcamp.

Verify:

RUN docker --version


Step 9: Installing AWS CLI
---------------------------
Download:

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip

Extract:

RUN unzip awscliv2.zip

Install:

RUN ./aws/install

Verify:

RUN aws --version

Step 10: Installing kubectl
----------------------------
RUN curl -LO "https://dl.k8s.io/release/v1.33.0/bin/linux/amd64/kubectl"

Make executable:

RUN chmod +x kubectl

Move:

RUN mv kubectl /usr/local/bin/

Verify:

RUN kubectl version --client

Production Jenkins Agent

A typical DevOps Jenkins agent includes:

Ubuntu
│
├── Git
├── Java
├── Maven
├── Docker CLI
├── AWS CLI
├── kubectl
├── Helm
├── Terraform
└── Trivy

Instead of installing everything during every pipeline run, companies often build a reusable agent image with these tools already installed.

Hands-on Lab
=============
Create a Dockerfile that:

Uses jenkins/inbound-agent:latest-jdk17 as the base image.
Installs:
Git
Curl
Wget
Unzip
Verifies Git installation using:
RUN git --version

Build it:

docker build -t my-jenkins-agent:v1 .

Run it:

docker run --rm my-jenkins-agent:v1 git --version


Interview Questions
---------------------
1. Why do we run apt-get update before apt-get install?

Answer:

It downloads the latest package index from the configured repositories. Without it, the package list may be outdated, causing installation failures or older package versions to be installed.

2. Why use && in a RUN instruction?

Answer:

It chains commands together and ensures that the next command runs only if the previous one succeeds. It also reduces the number of image layers.

3. Why remove /var/lib/apt/lists/*?

Answer:

Those files are cached package indexes used only during installation. Removing them reduces the final image size without affecting installed packages.

4. Why do we install the Docker CLI in a Jenkins agent?

Answer:

So the agent can execute Docker commands such as docker build and docker push. The CLI still needs access to a Docker daemon to perform those operations.

Homework
-----------
Build the Docker image from the lab.
Run the container and verify:
git --version
curl --version
Add Helm to the image.
Add Terraform to the image.
Verify both installations.


Tomorrow (Day 3)

We'll learn how to build a professional Java/Maven Jenkins agent, including:

ARG vs ENV
Downloading specific tool versions
Setting PATH
Organizing /opt
Writing maintainable Dockerfiles
Version pinning for reproducible builds

This is the foundation for building production-grade Jenkins agent images.
