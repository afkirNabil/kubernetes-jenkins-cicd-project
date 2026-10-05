FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update && \
    apt-get install -y curl && \
    curl -LO "https://dl.k8s.io/release/v1.34.1/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl && \
    rm -rf /var/lib/apt/lists/*

USER jenkins
