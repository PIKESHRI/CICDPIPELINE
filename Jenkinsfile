pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        CLUSTER_NAME = 'my-project-eks-cluster'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/PIKESHRI/CICDPIPELINE.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWSCred']]) {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $CLUSTER_NAME
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
