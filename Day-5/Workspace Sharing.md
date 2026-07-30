Module 5 – Workspace Sharing
=================================
When Jenkins starts the Docker container, it mounts the workspace.

Jenkins Workspace

/var/lib/jenkins/workspace/app

           │
           ▼

Docker Container

/workspace

So when you run:

mvn clean package

the generated JAR file is available to later stages.
