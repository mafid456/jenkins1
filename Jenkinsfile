pipeline {
    agent any

    environment {
        IMAGE_NAME = "simple-python-app"
        REPO_URL = "https://github.com/mafid456/jenkins1.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${REPO_URL}", branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    docker rm -f $IMAGE_NAME || true
                    docker run -d -p 5000:5000 --name $IMAGE_NAME $IMAGE_NAME
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
