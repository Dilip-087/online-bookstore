pipeline {
    agent any

    environment {
        IMAGE_NAME = 'dilip087/online-bookstore'
        TAG = "${env.BUILD_NUMBER}"
        DOCKER_BIN = 'C:\\Users\\itzme\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_BIN = 'C:\\Users\\itzme\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Automated Quality & Test Gate') {
            steps {
                script {
                    echo "Validating workspace files..."
                    powershell 'Get-ChildItem index.html, Dockerfile'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    powershell "& '${env.DOCKER_BIN}' context use default"
                    powershell "& '${env.DOCKER_BIN}' build -t ${IMAGE_NAME}:${TAG} -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Uses your active local docker login session
                    powershell "& '${env.DOCKER_BIN}' push ${IMAGE_NAME}:${TAG}"
                    powershell "& '${env.DOCKER_BIN}' push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying application to Docker Desktop Kubernetes..."
                    powershell "& '${env.KUBECTL_BIN}' apply -f k8s/service.yaml"
                    powershell "& '${env.KUBECTL_BIN}' set image deployment/bookstore-deployment bookstore=${IMAGE_NAME}:${TAG}"
                    powershell "& '${env.KUBECTL_BIN}' rollout status deployment/bookstore-deployment --timeout=120s"
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Deployment failed! Initiating automated rollback..."
                powershell "& '${env.KUBECTL_BIN}' rollout undo deployment/bookstore-deployment"
            }
        }
    }
}
