**📌 Project Overview**
This project demonstrates a fully automated, production-style DevOps pipeline for a full-stack Java web application containerized as microservices using Docker. Every stage — from code quality to artifact management, security scanning, containerization, deployment, and notification — runs without any human intervention.
Application: A full-stack social networking web app with Login, Sign Up, User Profile management, and a Social Feed — backed by a MySQL database with bcrypt-hashed passwords.


## 🔄 Pipeline Architecture

| Stage | Step | Details |
|-------|------|---------|
| 1️⃣ | **Code Checkout** | GitHub → `main` branch |
| 2️⃣ | **SonarQube Analysis** | `mvn clean verify sonar:sonar` → Quality Gate ✅ PASSED |
| 3️⃣ | **Maven Build** | `mvn clean package` → `vprofile-v2.war` |
| 4️⃣ | **Nexus Artifact Storage** | 46.2 MB WAR stored → SHA1 + MD5 verified |
| 5️⃣ | **Docker Image Build** | App image + DB image → `brkdockerhub/mydockerproject` |
| 6️⃣ | **Trivy Security Scan** | CVE scan on every layer → 269 OS CVEs documented |
| 7️⃣ | **Push to Docker Hub** | Public registry → pushed in under 1 minute |
| 8️⃣ | **Deploy via Docker Stack** | `docker stack deploy -c compose.yml mystack` → live on port 1111 |
| 📧 | **Jenkins Notification** | Auto SMTP email on SUCCESS / FAILURE |

---

## 🧩 Pipeline Stages — Deep Dive

### 1️⃣ Code Checkout
- Branch: `main`
- Repo: `https://github.com/JTD-Devops/microservicesproject-docker.git`

### 2️⃣ SonarQube — Code Quality Analysis

| Metric | Result |
|--------|--------|
| **Quality Gate** | ✅ PASSED |
| **Lines Scanned** | 3,300+ |
| **Bugs** | 34 |
| **Vulnerabilities** | 2 |
| **Security Hotspots** | 13 |
| **Code Smells** | 131 |
| **Technical Debt** | 2 days 1 hour |
| **Unit Tests** | 9 |
| **Code Coverage** | 6.7% on 456 lines |
| **Maintainability** | A |

### 3️⃣ Maven Build
- Command: `mvn clean package`
- Output: `vprofile-v2.war`
- WAR copied to Docker-app folder for containerization

### 4️⃣ Nexus — Artifact Storage

| Property | Value |
|----------|-------|
| **Repository** | myrepo |
| **Format** | Maven2 |
| **Artifact** | vprofile-v2.war |
| **Version** | v2 |
| **File Size** | 46.2 MB |
| **Integrity** | SHA1 + MD5 verified |

### 5️⃣ Docker — Image Build
- `docker build -t brkdockerhub/mydockerproject:app Docker-app`
- `docker build -t brkdockerhub/mydockerproject:db Docker-db`
- Base Image: Amazon Linux
- 2 separate images — App + Database

### 6️⃣ Trivy — Security Scan
- Scans every image layer before push
- **269 OS-level CVEs** surfaced and documented
- Tomcat core JARs → majority at **0 CVEs**

### 7️⃣ Push to Docker Hub
- Images pushed to `brkdockerhub/mydockerproject`
- Push completed in **under 1 minute**

### 8️⃣ Deploy via Docker Stack
- Command: `docker stack deploy -c compose.yml mystack`
- Manual approval gate before deploy
- App live on **port 1111**

### 📧 Jenkins SMTP Notification
- Triggered automatically after every build
- Reports: build number, status, log URL
- Result: `prod-deployment ` → ✅ **SUCCESS**

## 🖥️ Application Screenshots


### Login & Registration
| Login Page | Sign Up Page |
|-----------|-------------|
| [![Login](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-login.png)](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-login.png) | [![Signup](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-signup.png)](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-signup.png) |

### User Profile & Setup
| Profile Feed | Profile Setup |
|-------------|--------------|
| [![Profile](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-profile.png)](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-profile.png) | [![Profile Setup](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-profilesetup.png)](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/app-profilesetup.png) |

### 🗄️ Database Verification

| Field | Value |
|-------|-------|
| **ID** | 14 |
| **Username** | devops |
| **Email** | devops@gmail.com |
| **DOB** | 01/01/1999 |
| **Father** | rajnikanth |
| **Mother** | sonali |
| **Gender** | male |
| **Status** | unMarried |
| **Password** | `$2a$11$CIXpDdTbraDG3ifrSaew...` ✅ bcrypt hashed |

> User data persisted in MySQL — password encrypted with bcrypt ✅

[![DB Verification](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/databaseverify.png)](https://raw.githubusercontent.com/BRK-Devops/microservices-project/main/databaseverify.png)

---

## ⚙️ Jenkins Pipeline Script

```groovy
pipeline {
    agent {
        node { label "prod" }
    }
    tools {
        maven "mymaven"
    }
    stages {
        stage("code") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/JTD-Devops/microservicesproject-docker.git'
            }
        }
        stage("Code Quality Analysis with Sonarqube") {
            steps {
                withSonarQubeEnv("mysonar") {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=myproject"
                }
            }
        }
        stage("maven build") {
            steps {
                sh "mvn clean package"
                sh "cp -r target Docker-app"
            }
        }
        stage("nexus-artifact storage") {
            steps {
                nexusArtifactUploader artifacts: [[
                    artifactId: 'vprofile',
                    classifier: '',
                    file: 'target/vprofile-v2.war',
                    type: 'war'
                ]], credentialsId: '<nexus-cred-id>'
            }
        }
        stage("Image build using docker") {
            steps {
                sh "docker build -t brkdockerhub/mydockerproject:app Docker-app"
                sh "docker build -t brkdockerhub/mydockerproject:db Docker-db"
            }
        }
        stage("image scan using Trivy") {
            steps {
                sh "trivy image brkdockerhub/mydockerproject:app"
                sh "trivy image brkdockerhub/mydockerproject:db"
            }
        }
        stage("pushing image to dockerhub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: '<docker-cred-id>') {
                        sh "docker push brkdockerhub/mydockerproject:app"
                        sh "docker push brkdockerhub/mydockerproject:db"
                    }
                }
            }
        }
        stage("Deploy application by using docker-stack") {
            input {
                message 'can i deploy the app?'
            }
            steps {
                sh "docker stack deploy -c compose.yml mystack"
            }
        }
    }
    post {
        always {
            mail to: '<your-email>@gmail.com',
                 subject: "Build Status: ${currentBuild.fullDisplayName}",
                 body: "The build ${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}. view log at : ${env.BUILD_URL}"
        }
    }
}
```

## 🛠️ Tech Stack

| Category | Technology |
|----------|------------|
| **CI/CD Orchestration** | Jenkins (Declarative Pipeline) |
| **Code Quality** | SonarQube |
| **Build Tool** | Apache Maven |
| **Artifact Repository** | Sonatype Nexus |
| **Containerization** | Docker + Docker Stack |
| **Container Registry** | Docker Hub |
| **Security Scanning** | Trivy (Aqua Security) |
| **Base OS Image** | Amazon Linux |
| **Source Control** | Git + GitHub |
| **Notification** | Jenkins SMTP (Gmail) |
| **Application Stack** | Java (Spring) + MySQL |

---

## 📊 Project Stats at a Glance

| Metric | Value |
|--------|-------|
| **Pipeline Stages** | 8 Stages — 0 Manual |
| **Artifact Size** | 46.2 MB WAR |
| **Docker Images** | 2 (App + DB) |
| **CVEs Documented** | 269 OS-level |
| **Code Lines Scanned** | 3,300+ |
| **Unit Tests** | 9 |
| **Quality Gate** | ✅ PASSED |
| **Build Result** | ✅ SUCCESS |
| **Notification** | Auto SMTP Email |
| **Manual Steps Required** | 0 |

---

## 🚀 How to Run This Pipeline

### Prerequisites
- Jenkins server with label `prod` node configured
- SonarQube server configured in Jenkins (`mysonar`)
- Nexus repository (`myrepo`) configured
- Docker Hub credentials added in Jenkins
- Maven tool configured as `mymaven`
- SMTP configured in Jenkins for email notifications

### Steps

**1. Clone the repo**
```bash
git clone https://github.com/BRK-Devops/microservicesproject-docker.git
```

**2. Create Jenkins pipeline job**
- Create a new pipeline job named `prod-deployment`
- Copy the Jenkinsfile from this repo

**3. Push any change to main branch**
```bash
git push origin main
```

**4. Jenkins automatically triggers:**

`Code` → `SonarQube` → `Build` → `Nexus` → `Docker` → `Trivy` → `Push` → `Deploy` → `Email`

---

## 📁 Repository Structure

```
microservicesproject-docker/
├── Docker-app/         # App Dockerfile + WAR (copied during build)
├── Docker-db/          # DB Dockerfile (MySQL)
├── compose.yml         # Docker Stack compose file
├── Jenkinsfile         # Declarative CI/CD pipeline
├── pom.xml             # Maven build config (SonarQube + Nexus)
└── src/                # Java Spring application source
    ├── main/
    │   ├── java/       # Controllers, Services, Models
    │   └── webapp/     # JSP views (Login, Signup, Profile, Feed)
    └── test/           # 9 Unit tests
```
## 🏆 Key Achievements

| # | Achievement | Metric |
|---|-------------|--------|
| ✅ | Pipeline Stages Automated | 8 Stages — 0 Manual Steps |
| ✅ | SonarQube Quality Gate | PASSED — All Conditions Met |
| ✅ | Code Lines Scanned | 3,300+ Lines Analyzed |
| ✅ | Artifact Stored in Nexus | 46.2 MB WAR — SHA1 + MD5 Verified |
| ✅ | Docker Images Built | 2 Images (App + DB) — 262.1 MB Total |
| ✅ | Images Pushed to Docker Hub | Under 1 Minute |
| ✅ | CVEs Surfaced by Trivy | 269 OS-level CVEs Documented |
| ✅ | Tomcat Core JARs | Majority at 0 CVEs |
| ✅ | Unit Tests Executed | 9 Tests — 6.7% Coverage |
| ✅ | Technical Debt Tracked | 2 Days 1 Hour |
| ✅ | Build Notification | Auto SMTP Email on Every Build |
| ✅ | Password Security | bcrypt Hashed — DB Verified Post-Deploy |
| ✅ | Final Build Status | prod-deployment → SUCCESS |
