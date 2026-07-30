Module 2 – Jenkins Pipeline Flow
======================================

A production CI pipeline typically looks like this:

Developer Pushes Code
          │
          ▼
      GitHub Webhook
          │
          ▼
 Jenkins Controller
          │
          ▼
Start Docker Agent
          │
          ▼
Checkout Source Code
          │
          ▼
Build Application
          │
          ▼
Run Unit Tests
          │
          ▼
SonarQube Analysis
          │
          ▼
Trivy Scan
          │
          ▼
Build Docker Image
          │
          ▼
Push Image to Registry

Notice that every stage runs inside the Docker agent.









          │
          ▼
Deploy to Kubernetes
