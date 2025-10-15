pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '503427798981.dkr.ecr.ap-south-1.amazonaws.com/siva/app'
        CLUSTER_NAME = 'ma-eks-cluster'   // replace with your actual EKS cluster name
        KUBE_NAMESPACE = 'default'        // replace if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mafid456/jenkins1.git'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t siva/app:latest .'
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh 'docker tag siva/app:latest $ECR_REPO:latest'
                sh 'docker push $ECR_REPO:latest'
            }
        }
        
        stage('Create EKS Cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                    # Check if the cluster already exists
                    if eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                        echo "Cluster $CLUSTER_NAME already exists. Skipping creation."
                    else
                        echo "Creating Kubernetes cluster $CLUSTER_NAME..."
                        eksctl create cluster \
                          --name ma-eks-cluster \
                          --region ap-south-1 \
                          --nodes 1 \
                          --node-type t3.medium \
                          --managed
                    fi
                    '''
                }
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                        aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
                        kubectl get nodes
                    '''
                }
            }
        }
    }

    post {
        failure { echo '❌ Pipeline failed. Check logs.' }
        success { echo '✅ Docker image pushed to ECR and EKS cluster configured successfully!' }
    }
}
