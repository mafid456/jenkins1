pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "shaikmafidbasha"   // <--- change to your Docker Hub repo
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
            echo "✅ Docker image pushed successfully to Docker Hub"
        }
        failure {
            echo "❌ Build failed — check logs"
        }
    }
}
