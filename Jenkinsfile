pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:/usr/bin:/bin"
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '503427798981.dkr.ecr.ap-south-1.amazonaws.com/basha/app'
        CLUSTER_NAME = 'ma-eks-cluster'
        KUBE_NAMESPACE = 'default'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mafid456/jenkins1.git'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t basha/app:latest .'
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh 'docker tag basha/app:latest $ECR_REPO:latest'
                sh 'docker push $ECR_REPO:latest'
            }
        }

        stage('Create EKS Cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
                    sh '''
                    echo "Checking if EKS cluster $CLUSTER_NAME exists..."
                    if /usr/local/bin/eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                        echo "✅ Cluster $CLUSTER_NAME already exists. Skipping creation."
                    else
                        echo "⏳ Cluster not found. Creating new one..."
                        /usr/local/bin/eksctl create cluster \
                          --name ma-eks-cluster \
                          --region ap-south-1 \
                          --nodes 2 \
                          --node-type t3.medium \
                          --managed
                    fi
                    '''
                }
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
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
