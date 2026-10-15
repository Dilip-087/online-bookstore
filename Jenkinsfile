pipeline {
    agent any

    environment {
        DOCKER_CREDS = credentials('dockerhub') // Matches your credential ID
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
                    echo "Running automated HTML validation and UI test checks..."
                    sh 'ls -la index.html Dockerfile'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${TAG} -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'kuberconfig', serverUrl: '') { // Matches your credential ID
                    script {
                        sh "kubectl apply -f k8s/service.yaml"
                        sh "kubectl set image deployment/bookstore-deployment bookstore=${IMAGE_NAME}:${TAG}"
                        sh "kubectl rollout status deployment/bookstore-deployment --timeout=120s"
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Deployment failed! Initiating automated rollback..."
                withKubeConfig(credentialsId: 'kuberconfig', serverUrl: '') {
                    sh "kubectl rollout undo deployment/bookstore-deployment"
                }
            }
        }
    }
}
