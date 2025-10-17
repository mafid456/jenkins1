pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '503427798981.dkr.ecr.ap-south-1.amazonaws.com/basha/app'
        CLUSTER_NAME = 'ma-eks-cluster'
        KUBE_NAMESPACE = 'default'
        PATH = "/usr/local/bin:/usr/bin:/bin"   // ensures eksctl and kubectl work
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

        stage('Build & Push Docker Image') {
            steps {
                sh '''
                    docker build -t basha/app:latest .
                    docker tag basha/app:latest $ECR_REPO:latest
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Create EKS Cluster if Not Exists') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
                    sh '''
                        echo "Checking if EKS cluster $CLUSTER_NAME exists..."
                        if eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                            echo "✅ Cluster $CLUSTER_NAME already exists."
                        else
                            echo "⏳ Cluster not found. Creating new one..."
                            eksctl create cluster --name $CLUSTER_NAME --region $AWS_REGION --nodes 2 --node-type t3.medium --managed
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

        stage('Deploy Application to EKS') {
            steps {
                sh '''
                    echo "Creating deployment.yml and service.yml..."

                    cat <<EOF > deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  namespace: $KUBE_NAMESPACE
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
      - name: flask-app
        image: $ECR_REPO:latest
        ports:
        - containerPort: 5000
EOF

                    cat <<EOF > service.yml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
  namespace: $KUBE_NAMESPACE
spec:
  type: LoadBalancer
  selector:
    app: flask-app
  ports:
  - port: 80
    targetPort: 5000
EOF

                    echo "Deploying to EKS..."
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml

                    echo "Waiting for the service to get external IP..."
                    sleep 60
                    kubectl get svc flask-service -n $KUBE_NAMESPACE
                '''
            }
        }

        stage('Schedule Auto Delete (2 Hours)') {
            steps {
                sh '''
                    echo "Scheduling cluster deletion after 2 hours..."
                    nohup bash -c "sleep 7200 && eksctl delete cluster --name $CLUSTER_NAME --region $AWS_REGION" &
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment complete! Cluster will auto-delete after 2 hours.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for errors.'
        }
    }
}
