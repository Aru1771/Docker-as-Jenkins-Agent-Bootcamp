Module 6 – Docker Socket Mount
====================================
A common question is:

If the pipeline runs inside a container, how does it build another Docker image?

The answer is by mounting the Docker socket.

Host Machine
│
├── Docker Daemon
│
└── /var/run/docker.sock
          │
          ▼
Docker Agent Container

The Docker CLI inside the container communicates with the host's Docker daemon through this socket.

Typical Docker run command:

docker run \
-v /var/run/docker.sock:/var/run/docker.sock \
my-jenkins-agent:v1

This allows the agent container to execute commands like:

docker build
docker push
docker images

without running its own Docker daemon.

Note: Mounting the Docker socket gives the container significant control over the host. In production, evaluate alternatives such as Kaniko, BuildKit, or rootless build solutions depending on your security requirements.
