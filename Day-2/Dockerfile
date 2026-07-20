FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    unzip \
    git && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y openjdk-17-jdk && \
    rm -rf /var/lib/apt/lists/*

RUN java -version

ARG MAVEN_VERSION=3.9.16

RUN wget https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz

RUN tar -xzf apache-maven-${MAVEN_VERSION}-bin.tar.gz

RUN mv apache-maven-${MAVEN_VERSION} /opt/maven

ENV MAVEN_HOME=/opt/maven
ENV PATH=$PATH:$MAVEN_HOME/bin

RUN mvn -version

RUN apt-get update && \
    apt-get install -y docker.io

RUN docker --version

RUN $ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"


RUN unzip awscliv2.zip

RUN sudo ./aws/install

RUN aws --version

RUN curl -LO "https://dl.k8s.io/release/v1.33.0/bin/linux/amd64/kubectl"

RUN chmod +x kubectl

RUN mv kubectl /usr/local/bin/
RUN kubectl version --client
