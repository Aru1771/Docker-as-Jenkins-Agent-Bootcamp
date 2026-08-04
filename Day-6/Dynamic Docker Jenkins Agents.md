📅 Day 6 – Dynamic Docker Jenkins Agents
==============================================
🎯 Goal

By the end of today, you'll understand:

Why static agents are not ideal
What are dynamic Docker agents
How Jenkins creates containers automatically
Docker Cloud integration
Docker Agent Templates
Complete execution flow
Production architecture
Hands-on

Step 1 – Problem with Static Agents
------------------------------------
Suppose your Jenkins server has:

Jenkins Controller

↓

Agent-1

↓

Docker Installed

Today you run:
--------------
20 Pipelines

Tomorrow:

200 Pipelines

Problem:

One Agent

↓

Too many jobs

↓

Queue increases

Step 2 – Production Requirement
---------------------------------
Companies need:

Build agents only when required
Remove them after the build
Save infrastructure cost
Keep build environments clean

This is called dynamic provisioning.

Step 3 – Dynamic Docker Agent
-----------------------------
Instead of keeping an agent running all the time:

Pipeline Starts

↓

Create Docker Container

↓

Execute Build

↓

Delete Container

Every pipeline gets a fresh environment.

Step 4 – Architecture
--------------------

Developer

↓

GitHub Push

↓

Jenkins Controller

↓

Docker Engine

↓

Create Docker Container

↓

Execute Pipeline

↓

Container Removed

Notice:

The controller does not execute the build.

It asks Docker to create a temporary container.

Step 5 – Why Is This Better?
----------------------------

Suppose Build-1 installs:

Java 21

Maven 3.9

Build-2 requires:

Java 17

Gradle

With dynamic agents:

Build-1

↓

Container A

↓

Deleted

------------------

Build-2

↓

Container B

↓

Deleted

Each build is isolated.

Step 6 – Docker Cloud in Jenkins
---------------------------------

Go to:

Manage Jenkins

↓

Clouds

↓

Add a new Cloud

↓

Docker

Configure:

Docker Host

↓

unix:///var/run/docker.sock

Jenkins now knows how to communicate with Docker.

Step 7 – Docker Agent Template
--------------------------------

Create a template.

Example:

Image

↓

mycompany/jenkins-agent:latest

Remote filesystem:

/home/jenkins

Labels:

docker-agent

Launch method:

Attach Docker Container

Now Jenkins knows:

"Whenever a pipeline requests label docker-agent, create a container from this image."

Step 8 – Pipeline
---------------------
Example:

pipeline {
    agent {
        label 'docker-agent'
    }

    stages {
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
            }
        }
    }
}

Flow:

Pipeline

↓

Label

↓

docker-agent

↓

Jenkins Cloud

↓

Create Container

↓

Execute Pipeline

↓

Remove Container
↓

Builds become slow

Step 9 – Complete Production Flow
------------------------------------
Suppose a developer pushes code.

Developer

↓

GitHub

↓

Webhook

↓

Jenkins Controller

↓

Docker Cloud

↓

Start New Container

↓

Clone Repository

↓

Run Maven

↓

Run SonarQube

↓

Run OWASP

↓

Build Docker Image

↓

Run Trivy

↓

Push Image

↓

Container Deleted

This matches the CI/CD pipeline you've been building.

Step 10 – Multiple Builds
---------------------------
Suppose three developers push code.

Developer A

↓

Container-1

----------------

Developer B

↓

Container-2

----------------

Developer C

↓

Container-3

Each build gets its own container.

No conflicts.

Step 11 – Production Agent Image
---------------------------------
Instead of using a generic image, companies build their own.

Example:

mycompany/jenkins-agent

Includes:

Java

Maven

Git

Docker CLI

kubectl

Helm

AWS CLI

Trivy

Sonar Scanner

Every build uses the same standardized tools and versions.

Step 12 – Benefits
--------------------
✅ Clean environment for every build

✅ No leftover files

✅ Tool version consistency

✅ Easy scaling

✅ Lower infrastructure cost

✅ Better security
