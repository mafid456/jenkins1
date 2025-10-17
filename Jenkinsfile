pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:/usr/bin:/bin"
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '503427798981.dkr.ecr.ap-south-1.amazonaws.com/basha/app'
        CLUSTER_NAME = 'ma-eks-cluster'
        KUBE_NAMESPACE = 'default'
        DEPLOYMENT_NAME = 'flask-app-deployment'
        SERVICE_NAME = 'flask-app-service'
        IMAGE_TAG = 'latest'
        IMAGE_PULL_SECRET = 'ecr-secret'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mafid456/jenkins1.git'
            }
        }

        stage('Login to AWS ECR & Create Pull Secret') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
                    sh '''
# Login to ECR
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO

# Create Kubernetes secret for pulling images
kubectl delete secret $IMAGE_PULL_SECRET --ignore-not-found
kubectl create secret docker-registry $IMAGE_PULL_SECRET \
  --docker-server=$ECR_REPO \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region $AWS_REGION) \
  --namespace $KUBE_NAMESPACE
'''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t basha/app:$IMAGE_TAG ."
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
docker tag basha/app:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG
docker push $ECR_REPO:$IMAGE_TAG
"""
            }
        }

        stage('Create EKS Cluster if Needed') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
                    sh '''
echo "Checking if EKS cluster $CLUSTER_NAME exists..."
if eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
    echo "✅ Cluster $CLUSTER_NAME already exists. Skipping creation."
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

        stage('Configure kubectl') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '25503878-b6ba-410e-9bf4-cba116399ff5']]) {
                    sh '''
aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
kubectl get nodes
'''
                }
            }
        }

        stage('Deploy Flask App') {
            steps {
                sh '''
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${DEPLOYMENT_NAME}
  namespace: ${KUBE_NAMESPACE}
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
      - name: flask-app-container
        image: ${ECR_REPO}:${IMAGE_TAG}
        ports:
        - containerPort: 5000
      imagePullSecrets:
      - name: ${IMAGE_PULL_SECRET}
---
apiVersion: v1
kind: Service
metadata:
  name: ${SERVICE_NAME}
  namespace: ${KUBE_NAMESPACE}
spec:
  type: LoadBalancer
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
EOF
'''
            }
        }

        stage('Schedule Auto Deletion of App Only (2 hours)') {
            steps {
                sh '''
echo "Flask app will be deleted automatically after 2 hours..."
nohup bash -c '
sleep 7200
echo "Deleting Flask app deployment and service..."
kubectl delete deployment ${DEPLOYMENT_NAME} -n ${KUBE_NAMESPACE}
kubectl delete service ${SERVICE_NAME} -n ${KUBE_NAMESPACE}
echo "✅ Flask app removed. EKS cluster remains intact."
' >/dev/null 2>&1 &
'''
            }
        }
    }

    post {
        failure { echo '❌ Pipeline failed. Check logs.' }
        success { echo '✅ Docker image deployed to EKS and Flask app scheduled for auto-deletion (EKS cluster retained)!' }
    }
}
