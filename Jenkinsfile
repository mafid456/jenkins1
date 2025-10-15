pipeline {
    agent any

    environment {
        CLUSTER_NAME = "ma-cluster"
        AWS_REGION = "ap-south-1"
        ECR_REPO = "503427798981.dkr.ecr.ap-south-1.amazonaws.com/siva/app"
        DOCKERHUB_REPO = "shaikmafidbasha/myapp"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mafid456/jenkins1.git'
            }
        }

        stage('Create EKS Cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                    sh '''
                    echo "Creating Kubernetes cluster on AWS..."
                    eksctl create cluster \
                      --name $ma-cluster \
                      --region ap-south-1 \
                      --nodes 2 \
                      --node-type t3.medium \
                      --managed
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                    sh '''
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 503427798981.dkr.ecr.ap-south-1.amazonaws.com/siva/app
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'shaikmafidbasha', passwordVariable: 'c9a39a2b-6afc-4762-b2a4-fd3cf481115b')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG $DOCKERHUB_REPO:$IMAGE_TAG
                docker push $DOCKERHUB_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Cluster') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Cluster created, image built & pushed, and app deployed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
