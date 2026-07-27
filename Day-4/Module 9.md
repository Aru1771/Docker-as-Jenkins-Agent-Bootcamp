Pipeline Flow
==============

A typical pipeline looks like:

Developer

↓

Git Push

↓

Jenkins Controller

↓

Launch Docker Jenkins Agent

↓

Clone Code

↓

Compile

↓

Run Tests

↓

Sonar Scan

↓

Trivy Scan

↓

Build Docker Image

↓

Push Image

↓

Deploy to Kubernetes

The next build gets a fresh container.

↓

Destroy Agent
