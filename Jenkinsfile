pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-docker-app"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url:'https://github.com/mafid456/jenkins1.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest test_app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 5000:5000 --name flask_app $IMAGE_NAME'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker containers...'
            sh 'docker rm -f flask_app || true'
        }
    }
}
