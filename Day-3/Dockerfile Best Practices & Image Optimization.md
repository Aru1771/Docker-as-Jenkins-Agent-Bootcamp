
Dockerfile Best Practices & Image Optimization
==============================================

Learning Objectives
-------------------
By the end of today, you'll understand:

Why image optimization is important
Docker image layers
Layer caching
COPY vs ADD
.dockerignore
Multi-stage builds (introduction)
Running containers as a non-root user
Real-world Jenkins agent Dockerfile

Why Should We Optimize Docker Images?
--------------------------------------
Imagine you create this Dockerfile:

FROM ubuntu:24.04

RUN apt-get update

RUN apt-get install -y git

RUN apt-get install -y curl

RUN apt-get install -y wget

It works.

But is it production-ready?

❌ No


Problems
----------
Too many image layers
Larger image size
Slower builds
More network bandwidth
More security vulnerabilities


Understanding Docker Layers
----------------------------
Every Docker instruction creates a new layer (with a few exceptions like ARG).

FROM ubuntu:24.04
RUN apt-get update
RUN apt-get install -y git
RUN apt-get install -y curl
COPY app.sh /app.sh

Layers:

Layer 5 → app.sh

Layer 4 → curl

Layer 3 → git

Layer 2 → apt update

Layer 1 → ubuntu

Docker stores these layers separately.

Why Layers Matter
------------------
Suppose you modify only:

COPY app.sh /app.sh

Docker rebuilds only:

Layer 5

Layers 1–4 are reused from cache.

This is why Docker builds are fast.

Docker Layer Cache Example
----------------------------

First build:

docker build -t agent:v1 .
Step 1/5
Step 2/5
Step 3/5
Step 4/5
Step 5/5

Now modify only app.sh.

Build again:

docker build -t agent:v2 .

Output:

Step 1 → Using cache

Step 2 → Using cache

Step 3 → Using cache

Step 4 → Using cache

Step 5 → Rebuilding

Only the last layer is rebuilt.

Best Practice
--------------
Instead of this:

RUN apt-get update

RUN apt-get install -y git

RUN apt-get install -y curl

RUN apt-get install -y wget

Use:

RUN apt-get update && \
    apt-get install -y \
    git \
    curl \
    wget \
    unzip \
    && rm -rf /var/lib/apt/lists/*

Benefits:

Fewer layers
Smaller image
Faster builds

COPY vs ADD
-------------
COPY
-----

Copies local files.

Example:

COPY script.sh /usr/local/bin/

Simple and predictable.

ADD
-----
Can:

Copy local files
Extract local tar archives automatically
Download from URLs (though this is generally discouraged in favor of curl or wget)

Example:

ADD app.tar.gz /opt/

Docker automatically extracts the archive.
RUN apt-get install -y unzip

RUN apt-get install -y maven


Which Should You Use?
----------------------
Rule:

Use COPY unless you specifically need a feature of ADD.

This is a Docker best practice.


.dockerignore
-----------------
Suppose your project contains:

project/

Dockerfile

.git/

node_modules/

target/

README.md

logs/

Without a .dockerignore, Docker sends everything to the build context.

This increases build time.

Example .dockerignore
---------------------
.git
node_modules
target
*.log

Now Docker ignores these files during the build.

Why Use a Non-Root User?
--------------------------
By default:

FROM ubuntu

Container runs as:

root

If an attacker compromises the container,

they gain root privileges inside it.

Better approach:

RUN useradd -m jenkins
USER jenkins

Now the container runs with limited privileges.


Real Jenkins Agent Dockerfile
-------------------------------

FROM jenkins/inbound-agent:latest-jdk17

USER root

RUN apt-get update && \
    apt-get install -y \
        git \
        curl \
        wget \
        unzip \
        maven \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m devops

USER devops

WORKDIR /home/devops

Notice:

Single RUN instruction for package installation
Cache cleanup
Non-root user
Clean working directory


Hands-on Lab
--------------

Step 1
Create a directory:
mkdir docker-agent-day3
cd docker-agent-day3

Step 2
FROM jenkins/inbound-agent:latest-jdk17

USER root

RUN apt-get update && \
    apt-get install -y \
        git \
        curl \
        wget \
        unzip \
        maven \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m devops

USER devops

WORKDIR /home/devops



Step 3
Create a .dockerignore file:
.git
*.log
target
node_modules


Step 4
Build the image:
docker build -t my-agent:v3 .

Step 5
Run the container:
docker run -it --rm my-agent:v3 bash

Step 6
Verify the current user:
whoami
Expected output:
devops


Step 7
Verify installed tools:
git --version
mvn -version
curl --version
wget --version




Interview Questions
-----------------------
1. Why should multiple apt-get install commands be combined into one RUN instruction?
Because each RUN creates a new Docker layer. Combining them reduces the number of layers, decreases image size, and improves build performance.

2. Why do we remove /var/lib/apt/lists/*?
After package installation, the apt package index is no longer needed. Removing it reduces the final image size.

3. What is the purpose of .dockerignore?

It prevents unnecessary files (such as .git, node_modules, logs, and build artifacts) from being sent to the Docker build context,
resulting in faster builds and smaller images.

4. Why is running a container as a non-root user considered a best practice?
Running as a non-root user follows the principle of least privilege.
If the container is compromised, the attacker has fewer privileges, reducing the potential impact.
