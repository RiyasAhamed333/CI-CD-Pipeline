pipeline {
    agent { label 'slave-1' }

    tools {
        maven 'apache-maven'   // Must match the Maven installation name in Jenkins global tools config
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
            steps {
                git "${GIT_URL}"
            }
        }

        stage('Code-Build') {
            steps {
                sh 'mvn clean install'
                sh "whoami"
            }
        }

        stage('Docker-Build') {
            steps {
                script {
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: "${AWS_REGION}", credentials: "${AWS_CREDENTIAL}") {
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
                    withAWS(region: "${AWS_REGION}", credentials: "${AWS_CREDENTIAL}") {
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
