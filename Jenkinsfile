pipeline {
    agent any

    environment {
        DOCKER_CREDS = credentials('dockerhub-creds')
        IMAGE_NAME = 'dilip087/online-bookstore'
        TAG = "${env.BUILD_NUMBER}"
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
                    // Safely check if files exist using PowerShell or dir
                    powershell 'Get-ChildItem index.html, Dockerfile'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${IMAGE_NAME}:${TAG} -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    bat 'echo %DOCKER_CREDS_PSW% | docker login -u %DOCKER_CREDS_USR% --password-stdin'
                    bat "docker push ${IMAGE_NAME}:${TAG}"
                    bat "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying application to Docker Desktop Kubernetes..."
                    bat "kubectl apply -f k8s/service.yaml"
                    bat "kubectl set image deployment/bookstore-deployment bookstore=${IMAGE_NAME}:${TAG}"
                    bat "kubectl rollout status deployment/bookstore-deployment --timeout=120s"
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Deployment failed! Initiating automated rollback..."
                bat "kubectl rollout undo deployment/bookstore-deployment"
            }
        }
    }
}
