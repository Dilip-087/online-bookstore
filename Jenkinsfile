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
                    powershell "& '${env.DOCKER_BIN}' build -t ${IMAGE_NAME}:${TAG} -
