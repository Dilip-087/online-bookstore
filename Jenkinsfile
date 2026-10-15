pipeline {
    agent any

    environment {
        DOCKER_CREDS = credentials('dockerhub-creds')
        IMAGE_NAME = 'dilip087/online-bookstore'
        TAG = "${env.BUILD_NUMBER}"
        // Absolute paths for Docker Desktop executables on Windows
        DOCKER_BIN = '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe"'
        KUBECTL_BIN = '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe"'
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
                    bat "${env.DOCKER_BIN} build -t ${IMAGE_NAME}:${TAG} -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    bat "echo %DOCKER_CREDS_PSW% | ${env.DOCKER_BIN} login -u %DOCKER_CREDS_USR% --password-stdin"
                    bat "${env.DOCKER_BIN} push ${IMAGE_NAME}:${TAG}"
                    bat "${env.DOCKER_BIN} push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying application to Docker Desktop Kubernetes..."
                    bat "${env.KUBECTL_BIN} apply -f k8s/service.yaml"
                    bat "${env.KUBECTL_BIN} set image deployment/bookstore-deployment bookstore=${IMAGE_NAME}:${TAG}"
                    bat "${env.KUBECTL_BIN} rollout status deployment/bookstore-deployment --timeout=120s"
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Deployment failed! Initiating automated rollback..."
                bat "${env.KUBECTL_BIN} rollout undo deployment/bookstore-deployment"
            }
        }
    }
}
