pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "shaikmafidbasha/jenkins1"   // make sure this repo exists in Docker Hub
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
                    docker.withRegistry('https://index.docker.io/v1/', '30c6d24f-197b-401e-8fbd-ec15666d717b') {
                        dockerImage.push("${BUILD_NUMBER}")  // version tag
                        dockerImage.push("latest")           // latest tag
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully pushed ${DOCKER_HUB_REPO}:${BUILD_NUMBER} and :latest to Docker Hub"
        }
        failure {
            echo "❌ Failed to push Docker image. Check repo/credentials."
        }
    }
}
