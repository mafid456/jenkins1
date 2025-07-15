pipeline {
    agent {
        docker {
            image 'python:3.10'
            args '-u root' // Run as root to access Docker socket if needed
        }
    }

    environment {
        IMAGE_NAME = "python-docker-app"
        CONTAINER_NAME = "flask_app"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/mafid456/jenkins1.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install --upgrade pip'
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

        stage('Deploy Docker Container') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME || true
                    docker run -d --name $CONTAINER_NAME -p 5000:5000 $IMAGE_NAME
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment succeeded!"
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}
