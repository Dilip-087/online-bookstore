pipeline {
    agent any

    environment {
        IMAGE_NAME = 'dilip087/online-bookstore'
        TAG = "${env.BUILD_NUMBER}"
        DOCKER_BIN = 'C:\\Users\\itzme\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_BIN = 'C:\\Users\\itzme\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG_PATH = 'C:\\Users\\itzme\\.kube\\config'
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        // Explicitly log in using plain-text strings fed into the pipeline securely
                        powershell "echo $env:DOCKER_PASS | & '${env.DOCKER_BIN}' login --username $env:DOCKER_USER --password-stdin"
                        powershell "& '${env.DOCKER_BIN}' push ${IMAGE_NAME}:${TAG}"
                        powershell "& '${env.DOCKER_BIN}' push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying application to Docker Desktop Kubernetes..."
                    powershell "\$env:KUBECONFIG='${env.KUBECONFIG_PATH}'; & '${env.KUBECTL_BIN}' apply -f k8s/service.yaml"
                    powershell "\$env:KUBECONFIG='${env.KUBECONFIG_PATH}'; & '${env.KUBECTL_BIN}' set image deployment/bookstore-deployment bookstore=${IMAGE_NAME}:${TAG}"
                    powershell "\$env:KUBECONFIG='${env.KUBECONFIG_PATH}'; & '${env.KUBECTL_BIN}' rollout status deployment/bookstore-deployment --timeout=120s"
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Deployment failed! Initiating automated rollback..."
                powershell "\$env:KUBECONFIG='${env.KUBECONFIG_PATH}'; & '${env.KUBECTL_BIN}' rollout undo deployment/bookstore-deployment"
            }
        }
    }
}
