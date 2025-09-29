pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "shaikmafidbasha/jenkins1"   // repo must exist in your Docker Hub
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mafid456/jenkins1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_HUB_REPO}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed successfully: ${DOCKER_HUB_REPO}:${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Failed to push Docker image — check credentials or repo name"
        }
    }
}
