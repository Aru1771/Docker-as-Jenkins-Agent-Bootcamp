Why Do We Need Jenkins Agents
==============================


Recall the Jenkins architecture:
                Developer
                     │
                     ▼
             GitHub Repository
                     │
                     ▼
           Jenkins Controller
                     │
      Schedules the Job
                     │
         ┌───────────┴───────────┐
         │                       │
         ▼                       ▼
 Docker Agent 1            Docker Agent 2
(Java Build)              (DevOps Tools)


The Jenkins Controller:
-----------------------
Stores job configurations
Manages plugins
Schedules builds
Does not execute heavy workloads

The Jenkins Agent:
------------------------
Executes the pipeline
Has all required tools installed
Can be created on demand

Why Use a Docker Agent?
-------------------------

Imagine your pipeline needs:

Git
Java
Maven
Docker
AWS CLI
kubectl
Helm
Terraform
Trivy

If these tools are installed directly on the Jenkins controller:

Plugin conflicts
Version conflicts
Hard to upgrade
Difficult to maintain

Instead, package them into a Docker image.

my-jenkins-agent:v1
│
├── Git
├── Java
├── Maven
├── Docker CLI
├── AWS CLI
├── kubectl
├── Helm
├── Terraform
├── Trivy
└── Sonar Scanner

Every pipeline gets the exact same environment.
