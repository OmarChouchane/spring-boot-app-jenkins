# 🔐 Spring Boot DevSecOps Pipeline — Jenkins · SonarQube · Docker · Argo CD

A production-style **DevSecOps** CI/CD pipeline for a Java 17 Spring Boot application, deployed on a Kubernetes cluster provisioned on AWS EC2. Security is embedded at every stage — from code commit to production rollout.

> **Collaboration:** Pipeline architecture and GitOps delivery by **Omar Chouchane** · DevSecOps layer (SonarQube reporting & quality gate enforcement) by **Houssem Bouarada**

---

## 🏗️ Architecture

<img width="4803" height="1612" alt="diagram-export-4-23-2026-10_51_19-AM" src="https://github.com/user-attachments/assets/07931eac-63cd-475c-8018-62fab68067f9" />

---

## 🔄 Pipeline Flow (Shift-Left Security)

```
Code Commit
    │
    ▼
1. Checkout source from GitHub
    │
    ▼
2. Build & Test — Maven (deterministic, reproducible builds)
    │
    ▼
3. 🔍 SAST — SonarQube analysis + Quality Gate enforcement
    │   └─ Fail-fast: Docker push & deployment skipped if gate fails
    ▼
4. Build Docker image (immutable artifact)
    │
    ▼
5. Push to Docker Hub → omarchouchane/ultimate-cicd:<build_number>
    │
    ▼
6. GitOps — Clone manifests repo, update image tag, push to main
    │
    ▼
7. Argo CD detects drift → reconciles desired vs. actual state → deploys
```

---

## 🛡️ DevSecOps Layers

### Static Application Security Testing (SAST)
- **SonarQube 10.x** integrated into the CI pipeline as a central policy enforcement point
- Scans for vulnerabilities, code smells, and security hotspots on every commit
- Quality Gate blocks downstream stages (Docker push, deployment update) on failure
- Audit-ready reports generated within the pipeline for compliance traceability
- Aligns with **Secure SDLC** and compliance-driven delivery practices

### Shift-Left Security
- Security checks run *before* containerization — vulnerabilities caught at the source, not in production
- `sonar.qualitygate.wait=true` ensures the pipeline halts synchronously on non-compliant results
- Fail-fast mechanism: no artifact is built or deployed from a codebase that fails the security gate

### Immutable Artifacts
- Docker images tagged by Jenkins build number — no mutable `latest` tags in delivery
- Images pushed to Docker Hub only after passing the Quality Gate

### GitOps & Auditability
- Kubernetes manifests versioned in a **separate repository** — full change history and auditability
- Argo CD enforces desired-state reconciliation, preventing configuration drift
- All deployments are declarative and traceable via Git commits

### Secrets Management
- Credentials managed via Jenkins credential store (never hardcoded)
- SonarQube token, Docker Hub credentials, and GitHub token stored as Jenkins secrets

---

## 🧰 Tech Stack

| Layer | Tools |
|---|---|
| Language & Runtime | Java 17, Spring Boot |
| Build | Maven 3.9+ |
| CI Orchestration | Jenkins (Pipeline as Code) |
| SAST & Quality Gate | SonarQube 10.x |
| Containerization | Docker |
| Container Registry | Docker Hub |
| GitOps CD | Argo CD |
| Orchestration | Kubernetes (AWS EC2) |

---

## 📁 Repository Structure

```
.
├── Jenkinsfile          # Pipeline as Code
├── Dockerfile
├── pom.xml
├── argocd-basic.yml
└── src/
    └── main/
        ├── java/com/abhishek/StartApplication.java
        └── resources/
```

**Separate manifests repo:** [spring-app-manifests](https://github.com/OmarChouchane/spring-app-manifests)

---

## ⚙️ Prerequisites

- JDK 17
- Maven 3.9+
- Docker
- Kubernetes cluster (`kubectl` configured)
- Jenkins server
- SonarQube server
- Argo CD installed on cluster

---

## 🚀 Run Locally

```bash
# Build
mvn clean package

# Run
java -jar target/spring-boot-web.jar
```

Open: `http://localhost:8080`

---

## 🐳 Run with Docker

```bash
docker build -t omarchouchane/ultimate-cicd:local .
docker run -d -p 8080:8080 --name spring-boot-app omarchouchane/ultimate-cicd:local
```

---

## 🔧 Jenkins Setup

**Required credentials:**

| ID | Type | Purpose |
|---|---|---|
| `sonarqube` | Secret text | SonarQube auth token |
| `docker-cred` | Username/password | Docker Hub |
| `github` | Secret text | GitHub token for manifest push |

**Recommended trigger:**
- Build Trigger: `GitHub hook trigger for GITScm polling`
- Webhook URL: `http://<jenkins-public-ip>:8080/github-webhook/`

---

## 🔍 SonarQube Quality Gate

The pipeline waits synchronously for the Quality Gate result:

```properties
sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300
```

**If the Quality Gate fails:**
- Docker image build is skipped
- Docker Hub push is skipped
- Manifest update and Argo CD sync are skipped
- Pipeline exits with a non-zero status for full traceability

> This is the fail-fast security enforcement layer contributed by **Houssem Bouarada**, including enhanced SonarQube reporting and fail-on-quality/security-test logic.

---

## 🔁 Argo CD Setup

```bash
kubectl apply -f argocd-basic.yml
```

**Application source values:**

| Field | Value |
|---|---|
| Repository URL | `https://github.com/OmarChouchane/spring-app-manifests` |
| Revision | `main` |
| Path | `.` |
| Namespace | `default` |

---

## 🩺 Troubleshooting

```bash
# Jenkins status
sudo systemctl status jenkins --no-pager

# SonarQube port
sudo netstat -tlnp | grep 9000

# Disk space
df -h
```

**Webhook not triggering?**
- Verify EC2 inbound rule allows port `8080`
- Check Jenkins URL setting under Manage Jenkins → System
- Verify GitHub webhook delivery shows HTTP `200`

---

## 🔒 Security Notes

- **Never commit hardcoded credentials** to any branch
- Use Jenkins credential store for all secrets
- Test vulnerability scenarios on isolated branches only — revert immediately after validating pipeline behavior
- SonarQube Quality Gate is the enforcement boundary: no non-compliant code reaches the container registry

---

## 📄 License

This project is for learning and demonstration purposes. Add a formal license file if deploying to production.
