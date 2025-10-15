pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '503427798981.dkr.ecr.ap-south-1.amazonaws.com/siva/app'
        CLUSTER_NAME = 'ma-eks-cluster'
        KUBE_NAMESPACE = 'default'
        PATH = "/usr/local/bin:$PATH" // ensure eksctl & kubectl are visible to Jenkins
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
                sh '''
                    docker tag siva/app:latest $ECR_REPO:latest
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Create EKS Cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                    if /usr/local/bin/eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                        echo "Cluster $CLUSTER_NAME already exists. Skipping creation."
                    else
                        echo "Creating EKS cluster $CLUSTER_NAME..."
                        /usr/local/bin/eksctl create cluster \
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

        stage('Configure kubectl') {
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
        always {
            echo "Cleaning up: Deleting EKS cluster $CLUSTER_NAME..."
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                sh '''
                if /usr/local/bin/eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                    /usr/local/bin/eksctl delete cluster --name $CLUSTER_NAME --region $AWS_REGION
                    echo "Cluster $CLUSTER_NAME deleted successfully."
                else
                    echo "Cluster $CLUSTER_NAME does not exist. Nothing to delete."
                fi
                '''
            }
        }
        failure { echo '❌ Pipeline failed. Check logs.' }
        success { echo '✅ Pipeline completed successfully!' }
    }
}
