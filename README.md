# Spring Boot CI/CD with Jenkins, SonarQube, Docker, and Argo CD

A production-style Spring Boot project with an end-to-end DevOps workflow:

- Build and test with Maven
- Static analysis with SonarQube
- Container image build and push to Docker Hub
- GitOps-style deployment updates for Kubernetes manifests
- Argo CD sync to cluster

## Overview

This repository contains the application source and Jenkins pipeline logic.
The deployment manifests are managed in a separate repository:

- App source: https://github.com/OmarChouchane/spring-boot-app-jenkins
- Manifests: https://github.com/OmarChouchane/spring-app-manifests

## Architecture

Add your architecture diagram image to this repository (example path shown below), then it will render automatically.

![CI/CD Architecture](docs/pipeline-architecture.png)

## Pipeline Flow

1. Checkout source from GitHub
2. Build and test with Maven
3. Run SonarQube analysis and wait for Quality Gate
4. Build Docker image
5. Push image to Docker Hub (`omarchouchane/ultimate-cicd:<build_number>`)
6. Clone manifests repo and update image tag
7. Push manifests change to `main`
8. Argo CD detects change and deploys

## Tech Stack

- Java 17
- Spring Boot
- Maven
- Jenkins (Pipeline as Code)
- SonarQube 10.x
- Docker
- Kubernetes
- Argo CD

## Repository Structure

```
.
|- Jenkinsfile
|- Dockerfile
|- pom.xml
|- argocd-basic.yml
|- src/
|  |- main/
|     |- java/com/abhishek/StartApplication.java
|     |- resources/
```

## Prerequisites

- JDK 17
- Maven 3.9+
- Docker
- Kubernetes cluster access (`kubectl` configured)
- Jenkins server
- SonarQube server
- Argo CD installed on cluster

## Run Locally

Build package:

```bash
mvn clean package
```

Run jar:

```bash
java -jar target/spring-boot-web.jar
```

Open:

```text
http://localhost:8080
```

## Run with Docker

Build image:

```bash
docker build -t omarchouchane/ultimate-cicd:local .
```

Run container:

```bash
docker run -d -p 8080:8080 --name spring-boot-app omarchouchane/ultimate-cicd:local
```

## Jenkins Setup Notes

Expected Jenkins credentials:

- `sonarqube`: Secret text token for SonarQube
- `docker-cred`: Docker Hub credentials
- `github`: GitHub token for pushing manifests

Recommended Jenkins trigger for auto-build on push:

- Build Trigger: `GitHub hook trigger for GITScm polling`
- GitHub webhook URL: `http://<jenkins-public-ip>:8080/github-webhook/`

## SonarQube Quality Gate

Pipeline waits for Quality Gate result via Maven properties:

- `sonar.qualitygate.wait=true`
- `sonar.qualitygate.timeout=300`

If Quality Gate fails, Docker push and deployment update stages are skipped.

## Argo CD Setup

Base Argo CD CR example:

```bash
kubectl apply -f argocd-basic.yml
```

Application source values:

- Repository URL: `https://github.com/OmarChouchane/spring-app-manifests`
- Revision: `main`
- Path: `.`
- Namespace: `default`

## Troubleshooting

Common checks:

```bash
sudo systemctl status jenkins --no-pager
sudo netstat -tlnp | grep 9000
df -h
```

If webhook is not triggering:

- Verify EC2 inbound rule for port 8080
- Verify Jenkins URL setting
- Verify GitHub webhook delivery status is HTTP 200

## Security Note

Do not commit test vulnerabilities or hardcoded credentials to production branches.
Use temporary test commits only, then revert immediately after validating pipeline behavior.

## License

This project is for learning and demonstration. Add a formal license file if needed.
