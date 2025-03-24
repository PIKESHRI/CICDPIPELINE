CI/CD Pipeline using Jenkins for deploying NGINX image from Docker using EKS
Step1: Set Up Your Ubuntu Machine
ssh -i keypair.pem ubuntu@public_IP
Ensure your Ubuntu machine is updated:
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk curl git unzip wget                    (Java installation)
install Git 
sudo apt update
sudo apt install git
git --version
Step 2: Install Jenkins
Add Jenkins Repository and Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y Jenkins
Start and Enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable Jenkins
sudo systemctl status Jenkins    (to check the status of Jenkins running or not)
Retrieve Jenkins Initial Admin Password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
•  Open http://your-public-server-ip:8080 in a browser.
•  Enter the initial admin password.
•  Install suggested plugins.
•  Create an admin user.
Step 3: Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
Add Jenkins to the Docker group:
sudo usermod -aG docker Jenkins
sudo systemctl restart Jenkins

Step 4: Install AWS CLI and Kubectl
Install AWS CLI(command-line tool used to interact with AWS resources)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws –version
Configure AWS CLI 
aws configure
Enter your AWS credentials.
Install Kubectl(command-line tool used to interact with Kubernetes clusters)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
Install eksctl   (command-line tool that helps you to manage Amazon EKS clusters.)
curl -LO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz"
tar -xvzf eksctl_Linux_amd64.tar.gz
sudo mv eksctl /usr/local/bin
eksctl version

Step 5: Set Up AWS EKS Cluster
Create an EKS cluster:
eksctl create cluster --name my-project-eks-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.medium --nodes 2

Retrieve kubeconfig:
aws eks update-kubeconfig --region us-east-1 --name my- project-eks-cluster
kubectl get nodes

Step 6: Create Kubernetes Deployment YAML
Create a deployment.yaml file: and save this in github
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
----------
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer


Apply the deployment:
kubectl apply -f deployment.yaml
kubectl get svc
Step 7: Configure Jenkins Pipeline
Install Required Plugins
•  Go to Jenkins Dashboard → Manage Jenkins → Manage Plugins.
•  Install:
•	Pipeline
•	Kubernetes CLI
•	AWS Credentials
•	Git
Create a New Jenkins Pipeline Job
1.	Go to Jenkins → New Item → Pipeline.
2.	In the Pipeline Definition→ Job Configuration →Pipeline script from Source Code Management → Git, and ensure that the branch specified matches the branch in your GitHub repository.
3.	Branches to build -> refs/remotes/origin/master
4.	Script Path-> Jenkinsfile
Step 8:  Save this  Jenkins Pipeline Script in Github
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials-id']]) {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $CLUSTER_NAME
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
Step 9: Trigger the Pipeline
•	Save the pipeline and click Build Now.
•	Check logs to ensure the deployment is successful.
Step 10: Verify Deployment
kubectl get deployments
kubectl get pods
kubectl get svc
Get the EXTERNAL-IP of the LoadBalancer and open it in a browser to see the "Hello World" Nginx page.
ubuntu@ip-172-31-90-211:~$ kubectl get svc
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)        AGE
kubernetes      ClusterIP      10.100.0.1       <none>                                                                    443/TCP        25h
nginx-service   LoadBalancer   10.100.147.182   a1792d45512d5448090f63bdb0552be2-2080476798.us-east-1.elb.amazonaws.com   80:30516/TCP   

Browse :
http://a1792d45512d5448090f63bdb0552be2-2080476798.us-east-1.elb.amazonaws.com/

