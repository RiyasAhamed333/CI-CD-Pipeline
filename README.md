<h1 align="center">🚀 Jenkins CI/CD Pipeline with Maven, Docker, AWS ECR & Tomcat Deployment 🌐</h1>

<p align="center">
A complete end-to-end CI/CD pipeline using <b>GitHub</b>, <b>Jenkins</b>, <b>Maven</b>, <b>Docker</b>, and <b>AWS ECR</b> that automatically builds, packages, pushes, and deploys your Java application with every commit.
</p>

---

<p align="center">
  <img src="https://img.shields.io/badge/Built%20With-Jenkins-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Powered%20By-AWS-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Hosted%20On-Docker-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Repository-GitHub-black?style=for-the-badge"/>
</p>

---

## 🛠 Project Info

<p align="center">
  <img src="https://img.shields.io/badge/build-passing-brightgreen"/>
  <img src="https://img.shields.io/badge/license-MIT-blue"/>
  <img src="https://img.shields.io/badge/Made%20With-Jenkins-blue"/>
  <img src="https://img.shields.io/badge/Cloud-AWS%20ECR-orange"/>
</p>

---

## 📊 Repository Stats

<p align="center">
  <img src="https://img.shields.io/badge/repo%20size-3%20MB-blue"/>
  <img src="https://img.shields.io/badge/last%20commit-today-success"/>
  <img src="https://img.shields.io/badge/issues-0%20open-brightgreen"/>
  <img src="https://img.shields.io/badge/stars-⭐--1-lightgrey"/>
  <img src="https://img.shields.io/badge/forks-1-blue"/>
</p>

---

# 📦 CI/CD Pipeline Overview

This pipeline performs:

✔ GitHub checkout  
✔ Maven build (WAR file)  
✔ Docker image build  
✔ Push image to **AWS ECR**  
✔ Pull & deploy on Tomcat container  
✔ Runs on Jenkins agent (`slave-1`)

---

# 🧱 Architecture

Developer → GitHub → Jenkins → Maven Build → Docker Build → AWS ECR
→ Pull Image → Tomcat Container → Application Running


---

# 🧰 Tools & Technology

- Jenkins Declarative Pipeline  
- Maven (WAR build)  
- Docker & DockerHub/ECR  
- AWS ECR Repository  
- Tomcat 10 (JDK 17)  
- GitHub Webhooks (optional)  

# 📄 Jenkinsfile (CI/CD Pipeline)

```groovy
pipeline {
    agent { label 'slave-1' }

    tools {
        maven 'apache-maven'   // Must match Maven tool name in Jenkins
    }

    environment {
        AWS_REGION     = "us-east-1"
        AWS_CREDENTIAL = "aws-creds"
        ECR_REGISTRY   = "119994750175.dkr.ecr.us-east-1.amazonaws.com"
        ECR_REPO       = "my_repo"
        DOCKER_IMAGE   = "${ECR_REGISTRY}/${ECR_REPO}:latest"
        CONTAINER_NAME = "my-con1"
        HOST_PORT      = "80"
        APP_PORT       = "8080"
        GIT_URL        = "https://github.com/vikki-iam/My-Demo-Project.git"
    }

    stages {
        stage('CheckOut') {
            steps { git "${GIT_URL}" }
        }

        stage('Code-Build') {
            steps {
                sh 'mvn clean install'
                sh 'whoami'
            }
        }

        stage('Docker-Build') {
            steps {
                script {
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: AWS_REGION, credentials: AWS_CREDENTIAL) {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                        sh "docker build -t ${ECR_REPO} ."
                        sh "docker tag ${ECR_REPO}:latest ${DOCKER_IMAGE}"
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying container..."
                    withAWS(region: AWS_REGION, credentials: AWS_CREDENTIAL) {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                        sh "docker pull ${DOCKER_IMAGE}"
                        sh "docker stop ${CONTAINER_NAME} || true"
                        sh "docker rm ${CONTAINER_NAME} || true"
                        sh "docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${APP_PORT} ${DOCKER_IMAGE}"
                    }
                }
            }
        }
    }
}





🐳 Dockerfile (Tomcat Deployment)
FROM tomcat:10.1-jdk17-temurin
RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/java-app.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]


⚙️ How the Pipeline Works
Stage	Description
Checkout	Pulls code from GitHub
Maven Build	Compiles & packages .war
Docker Build	Creates image with Tomcat
Push to ECR	Stores image in AWS registry
Deploy	Runs updated container on Jenkins node

🌎 Deployment Access

Your app runs on:

http://<server-ip>:80
(Port 80 → Container's 8080)

🤝 Community & Contributions
<p align="center"> <img src="https://img.shields.io/badge/PRs-welcome-brightgreen"/> <img src="https://img.shields.io/badge/Contributions-welcome-blue"/> <img src="https://img.shields.io/badge/Visitors-300+-blueviolet"/> </p>
