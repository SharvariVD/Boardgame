# 🎮 Boardgame — End-to-End CI/CD Pipeline

A full-stack Java Spring Boot web application for browsing and reviewing board games, deployed using a complete DevSecOps pipeline with Jenkins, SonarQube, Nexus, Docker, and Kubernetes.

---

## 🏗️ Pipeline Architecture

![CI/CD Pipeline](screenshots/cicd-architecture.png)

---

## 🛠️ Tech Stack

**App:** `Java` `Spring Boot` `Spring MVC` `Spring Security` `Thymeleaf` `JDBC` `H2 Database` `Maven`

**Pipeline:** `Jenkins` `SonarQube` `Nexus` `Docker` `Kubernetes` `Trivy`

---

## 🔄 What I Did — Pipeline Stages

### 1. Git Checkout
- Jenkins pulls the latest code from the `main` branch of the GitHub repository using stored credentials

### 2. Compile & Test
- Runs `mvn compile` to compile the Java source code
- Runs `mvn test` to execute all JUnit unit tests

### 3. Trivy Filesystem Scan
- Scans the entire project filesystem for vulnerabilities before building
- Generates a `trivy-fs-report.html` report saved as a build artifact

### 4. SonarQube Analysis + Quality Gate
- Runs static code analysis using SonarQube Scanner
- Checks code quality metrics (bugs, vulnerabilities, code smells, coverage)
- Pipeline waits for the Quality Gate result — marked `UNSTABLE` if gate fails but continues

### 5. Build Package
- Runs `mvn package` to compile and package the app into a `.jar` file

### 6. Publish to Nexus
- Deploys the built `.jar` artifact to a **Nexus Repository Manager** using `mvn deploy`
- Uses a global Maven settings config for Nexus credentials

### 7. Docker Build & Tag
- Builds a Docker image from the `Dockerfile` using OpenJDK 17 Alpine base
- Tags it as `sharvari2004/boards:latest`

### 8. Trivy Image Scan
- Scans the Docker image for OS and library vulnerabilities
- Generates a `trivy-image-report.html` attached to the build email

### 9. Push Docker Image
- Pushes the scanned image to **DockerHub** registry under `sharvari2004/boards:latest`

### 10. Deploy to Kubernetes
- Connects to the Kubernetes cluster using stored kubeconfig credentials
- Applies `deployment-service.yaml` — creates a `Deployment` (2 replicas) and a `LoadBalancer` Service in the `webapps` namespace

### 11. Verify Deployment
- Runs `kubectl get pods -n webapps` and `kubectl get svc -n webapps` to confirm pods are running and service is exposed

### 12. Email Notification (post always)
- After every build (success or failure), sends an HTML email with build status, job name, build number, and the Trivy image report attached

---

## ☸️ Kubernetes Setup

- **Deployment:** 2 replicas of the boardgame container, image pulled from DockerHub, running on port `8080`
- **Service:** `LoadBalancer` type, maps external port `80` to container port `8080`
- **Namespace:** `webapps`

---

## 🔒 Security Scanning

Two Trivy scans are run in the pipeline:
- **Filesystem scan** — before build, catches vulnerable dependencies in source
- **Image scan** — after Docker build, catches vulnerabilities in the container layer

---

## 📦 Application Features

- View board games and reviews without logging in
- **Users** can add board games and write reviews
- **Managers** can edit and delete reviews
- Authentication via Spring Security (username + password)
- Role-based access control (non-member / user / manager)

**Default credentials to test:**
| Username | Password | Role |
|---|---|---|
| bugs | bunny | User |
| daffy | duck | Manager |

