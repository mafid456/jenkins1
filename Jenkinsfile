pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        CLUSTER_NAME = 'ma-eks-cluster' // the EKS cluster you want to delete
    }

    stages {
        stage('Delete EKS Cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '566c9043-c535-4eb2-b23f-f2301eb08962']]) {
                    sh '''
                        # Ensure eksctl is installed
                        if ! command -v eksctl &> /dev/null; then
                            echo "eksctl is not installed. Install eksctl first."
                            exit 1
                        fi

                        # Check if the cluster exists
                        if eksctl get cluster --name $CLUSTER_NAME --region $AWS_REGION >/dev/null 2>&1; then
                            echo "Deleting EKS cluster $CLUSTER_NAME..."
                            eksctl delete cluster --name $CLUSTER_NAME --region $AWS_REGION
                            echo "Cluster $CLUSTER_NAME deleted successfully."
                        else
                            echo "Cluster $CLUSTER_NAME does not exist. Nothing to delete."
                        fi
                    '''
                }
            }
        }
    }

    post {
        failure { echo '❌ Failed to delete the EKS cluster. Check logs.' }
        success { echo '✅ EKS cluster deletion pipeline finished successfully!' }
    }
}
