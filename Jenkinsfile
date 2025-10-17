pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '503427798981.dkr.ecr.ap-south-1.amazonaws.com/siva/app'
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                sh '''
                    docker build -t siva/app:latest .
                    docker tag siva/app:latest $ECR_REPO:latest
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Ensure EKS Cluster Exists') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                        echo "Checking if EKS cluster $CLUSTER_NAME exists..."
                        if eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                            echo "✅ Cluster $CLUSTER_NAME already exists."
                        else
                            echo "⏳ Cluster not found. Creating new one..."
                            eksctl create cluster \
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                        aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
                        kubectl get nodes
                    '''
                }
            }
        }

        stage('Deploy Application to EKS') {
            steps {
                sh '''
                    echo "Deploying application to EKS..."

                    # Write deployment and service files dynamically
                    cat <<EOF > deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  labels:
    app: flask-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-container
        image: $ECR_REPO:latest
        ports:
        - containerPort: 5000
EOF

                    cat <<EOF > service.yml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
EOF

                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml

                    echo "✅ Flask application deployed successfully!"
                '''
            }
        }

        stage('Schedule Auto-Delete After 2 Hours') {
            steps {
                sh '''
                    echo "Scheduling automatic cleanup after 2 hours..."
                    (sleep 7200 && kubectl delete -f deployment.yml && kubectl delete -f service.yml && echo "🧹 Deployment and service deleted automatically after 2 hours.") &
                '''
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
        success {
            echo '✅ Application deployed successfully and will auto-delete after 2 hours.'
        }
    }
}
