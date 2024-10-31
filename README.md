# 1.-Infrastructure-Provisioning-with-Terraform- 

To implement this project on AWS EC2 using Jenkins, Docker, and Kubernetes, here’s a detailed step-by-step process along with key commands.

Step 1: Set Up EC2 Instances
Launch EC2 Instance: Start an EC2 instance to act as your Jenkins server and Kubernetes node.
OS: Amazon Linux 2 or Ubuntu.
Instance Type: t2.micro (for test purposes).
Open ports 22 (SSH), 8080 (Jenkins), 30000-32767 (Kubernetes NodePort range), and 80 or 443 (for web access).

SSH into the EC2 Instance:
bash

ssh -i "your-key.pem" ec2-user@<EC2-IP>
Step 2: Install Docker
Update and Install Docker:
bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
Add ec2-user to Docker Group:
bash

sudo usermod -aG docker ec2-user
Step 3: Install Jenkins
Install Java:
bash

sudo yum install -y java-11-openjdk
Add Jenkins Repo and Install Jenkins:
bash

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
Access Jenkins: Go to http://<EC2-IP>:8080 and unlock Jenkins using the password located at:
bash

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Step 4: Configure Jenkins with Docker and Kubernetes Plugins
Install Plugins: In Jenkins, go to Manage Jenkins > Manage Plugins and install:
Docker Pipeline
Kubernetes CLI
Configure Docker: Add Docker as a tool in Jenkins under Manage Jenkins > Global Tool Configuration.
Step 5: Set Up a Simple Kubernetes Cluster on EC2
Using Minikube for a local Kubernetes cluster:

Install Minikube and Kubectl:
bash

curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
Start Minikube:
bash

minikube start --driver=none
Note: The --driver=none flag runs Minikube as root. You might need elevated permissions.
Step 6: Dockerize the Application (Build Docker Images)
Clone the Repository:
bash

git clone https://github.com/TechVerito-Software-Solutions-LLP/devops-fullstack-app
cd devops-fullstack-app
Create Dockerfiles for Backend and Frontend and add them to the respective directories.
Build and Push Images:
bash

docker build -t <your-docker-username>/backend:latest ./backend
docker build -t <your-docker-username>/frontend:latest ./frontend
docker push <your-docker-username>/backend:latest
docker push <your-docker-username>/frontend:latest
Step 7: Create Kubernetes Manifests
Deployment YAML Files:
Create backend-deployment.yaml and frontend-deployment.yaml.
Service YAML Files:
Create backend-service.yaml and frontend-service.yaml to expose your services.
Example backend-deployment.yaml:

yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: <your-docker-username>/backend:latest
        ports:
        - containerPort: 5000

        
Step 8: Deploy with Kubernetes
Apply Kubernetes Files:
bash

kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-service.yaml
Step 9: CI/CD Pipeline in Jenkins
Create a Pipeline Job in Jenkins:

In Jenkins, create a Pipeline job and configure it to point to the repository.
Jenkinsfile (Pipeline Script): Add a Jenkinsfile in the root of your repository with the following stages:

groovy

pipeline {
    agent any
    stages {
        stage('Clone repository') {
            steps {
                git 'https://github.com/TechVerito-Software-Solutions-LLP/devops-fullstack-app'
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    docker.build("your-docker-username/backend", "./backend")
                    docker.build("your-docker-username/frontend", "./frontend")
                }
            }
        }
        stage('Push Docker Images') {
            steps {
                script {
                    docker.image("your-docker-username/backend").push()
                    docker.image("your-docker-username/frontend").push()
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f backend-deployment.yaml'
                sh 'kubectl apply -f frontend-deployment.yaml'
                sh 'kubectl apply -f backend-service.yaml'
                sh 'kubectl apply -f frontend-service.yaml'
            }
        }
    }
}
Step 10: Access the Application
Get Service URLs:
bash

minikube service frontend --url
minikube service backend --url
Use these URLs to access your application.
This setup outlines the essential commands and configurations for your CI/CD pipeline and Kubernetes deployment on AWS EC2 using Jenkins, Docker, and Kubernetes. Let me know if you need further customization or clarification on any step.
