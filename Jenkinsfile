pipeline {
    agent any

    environment {
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
                sh 'docker build -t basha/app:latest .'
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker tag basha/app:latest $ECR_REPO:latest
                    docker push $ECR_REPO:latest
                '''
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
                    echo "Creating deployment.yml..."
                    cat <<EOF > deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app-deployment
  labels:
    app: flask-app
spec:
  replicas: 2
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

                    echo "Creating service.yml..."
                    cat <<EOF > service.yml
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
EOF

                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml

                    echo "Waiting for LoadBalancer to be ready..."
                    sleep 60
                    kubectl get svc flask-app-service
                '''
            }
        }

        stage('Wait and Delete Application') {
            steps {
                script {
                    echo "✅ Flask application deployed successfully."
                    echo "🕒 It will be automatically deleted after 2 hours..."
                    sleep(time: 2, unit: 'HOURS')  // <--- Changed from 1 to 2 hours
                    sh '''
                        echo "Deleting the deployment and service..."
                        kubectl delete -f deployment.yml || true
                        kubectl delete -f service.yml || true
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
        success {
            echo '✅ Flask app deployed, exposed, and scheduled for auto-deletion after 2 hours.'
        }
    }
}
