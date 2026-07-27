What is a Jenkins Agent?
-------------------------
Jenkins has two main components:

Jenkins Controller (Master)

Jenkins Agent (Worker)
Jenkins Controller Responsibilities

The controller:

Manages Jenkins UI
Schedules jobs
Stores job configurations
Stores credentials
Coordinates builds

It should not perform heavy build task

Jenkins Agent Responsibilities
---------------------------------
The agent performs the actual work:

Checkout source code
Compile applications
Run tests
Build Docker images
Scan images
Push images

                   Jenkins Controller
                           │
             Schedules Pipeline Job
                           │
                           ▼
               Docker Jenkins Agent
                           │
        ┌─────────────────────────────────┐
        │ Git Clone                       │
        │ Maven Build                     │
        │ Docker Build                    │
        │ Trivy Scan                      │
        │ Push to Registry                │
        │ kubectl Deploy                  │
        └─────────────────────────────────┘
Deploy to Kubernetes

Example architecture:
