End-to-End DevOps CI/CD Pipeline: GitHub, Jenkins, Maven, Docker & Kubernetes on AWS EKS
============================================================================================

Project Overview
=================
This project demonstrates a complete, automated CI/CD pipeline designed to provision infrastructure, build a Java web application, containerize it with Docker, and deploy it onto an Amazon EKS (Kubernetes) cluster using Jenkins.   

Architecture & Workflow
=============================
Plaintext

[ Developer ] ---> Push Code ---> [ GitHub ] ---> Clone ---> [ Jenkins CI/CD ]
                                                                |
                                             +------------------+------------------+
                                             | Build (Maven)                       | Dockerize
                                             v                                     v
                                    [ Apache Tomcat ]                     [ Docker Container ]
                                                                                   |
                                                                                   v
                                                                          [ Kubernetes Deployment ]
                                                                                   |
                                                                                   v
                                                                          [ AWS EKS Cluster ]
End-to-End Workflow Steps:
============================
Infrastructure Provisioning: Set up an AWS EKS management host and create a managed Kubernetes cluster using eksctl.

Code Commit: Developers push source code changes (.jsp files and pom.xml) to the GitHub repository.   

Checkout & Build: Jenkins pulls the code from GitHub and triggers Apache Maven to compile and package the application.   

Containerization: Jenkins uses the project Dockerfile to build a lightweight Docker container image.   

Orchestration & Deployment: The pipeline applies Kubernetes deployment manifests (k8s-deploy.yml) to roll out application pods seamlessly onto the EKS cluster.   

 Tech Stack
========================
Cloud & Infrastructure: AWS (EC2, EKS, IAM), eksctl, kubectl

Version Control: Git, GitHub   

CI/CD Automation: Jenkins (Declarative Pipelines)   

Build Tool: Apache Maven   

Containerization: Docker   

Orchestration: Kubernetes (AWS EKS)   

Runtime: Apache Tomcat, Java, JSP   

📂 Project Repository Structure
===================================
Plaintext

WEB-APP-CICD-main/
└── maven-web-app/
    ├── Declarative-JenkinsFile      # Jenkins declarative pipeline configuration[cite: 4]
    ├── Scripted-JenkinsFile         # Jenkins scripted pipeline configuration[cite: 4]
    ├── docker-k8s-jenkinsfile       # Integrated Docker & Kubernetes pipeline[cite: 4]
    ├── Dockerfile                   # Container image build instructions[cite: 4]
    ├── Jenkinsfile                  # Default execution pipeline[cite: 4]
    ├── k8s-deploy.yml               # Kubernetes deployment & service manifests[cite: 4]
    ├── pom.xml                      # Maven project object model configuration[cite: 4]
    └── src/
        └── main/
            └── webapp/              # Web application assets (JSP files)[cite: 4]
                ├── WEB-INF/
                │   └── web.xml
                ├── demo.jsp
                └── index.jsp
                
⚙️ Step-by-Step Implementation Guide
======================================
Follow these sequential steps to set up your infrastructure, tools, and CI/CD pipeline from scratch.
-----------------------------------------------------------------------------------------------------

Step 1: Create EKS Management Host in AWS
----------------------------------------------
Launch a new Ubuntu EC2 instance (t2.micro).

Connect to the machine and install kubectl:

Bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

Install the latest AWS CLI
-----------------------------

Bash
sudo apt install unzip 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

Install eksctl:
----------------------

Bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

Step 2: Create IAM Role & Attach to Hosts
------------------------------------------------
Create a new IAM role via the AWS Console selecting EC2 as the use case.

Attach necessary policies including administrator access, EC2, VPC, CloudFormation, and IAM full access permissions.

Attach this role to both your EKS Management Host and your Jenkins Server via the EC2 Console (Security -> Modify IAM Role).

Step 3: Create EKS Cluster using eksctl
---------------------------------------------------
Run the following command on your management host to provision the cluster in Mumbai (ap-south-1):

Bash
eksctl create cluster --name ashokit-cluster --region ap-south-1 --node-type t2.medium --zones ap-south-1a,ap-south-1b
Note: Cluster creation takes about 5 to 10 minutes. Once complete, verify the worker nodes using:

Bash
kubectl get nodes
Step 4: Setup Jenkins Server in Linux VM
Launch an Ubuntu EC2 instance (t2.medium) to ensure sufficient RAM (4GB) and open port 8080 in its security group inbound rules.

Install Java and Jenkins:
-------------------------------------------------------

Bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
Access Jenkins via http://<public-ip>:8080/, unlock it using the initial admin password (sudo cat /var/lib/jenkins/secrets/initialAdminPassword), and complete the initial setup.

Step 5: Configure Maven as Global Tool in Jenkins
----------------------------------------------------------
Go to Manage Jenkins -> Tools -> Maven Installations -> Add Maven (name it Maven-3.9.6 or matching your version and choose install automatically).

Step 6: Setup Docker in Jenkins Server
----------------------------------------------
Run the following commands on the Jenkins EC2 instance:

Bash
curl -fsSL get.docker.com | /bin/bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo docker version

Step 7: Install AWS CLI & Kubectl in Jenkins Server
-------------------------------------------------------
Install AWS CLI on Jenkins:

Bash
sudo apt install unzip 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

Install kubectl on Jenkins:
--------------------------------

Bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

Step 8: Configure EKS Cluster Access in Jenkins
---------------------------------------------------
On your EKS Management host, copy your kubeconfig file data (cat .kube/config).

On the Jenkins server, create the kubeconfig directory and file:

Bash
sudo mkdir -p /var/lib/jenkins/.kube
sudo vi /var/lib/jenkins/.kube/config
# (Paste your cluster config here and save)
Alternatively, update the kubeconfig directly via AWS CLI:

Bash
aws eks update-kubeconfig --region ap-south-1 --name ashokit-cluster
kubectl get nodes

Step 9: Create Jenkins CI/CD Pipeline Job
-----------------------------------------------
Create a new Pipeline job in Jenkins.

Use the following declarative pipeline script definition:
----------------------------------------------------------

Groovy
pipeline {
    agent any
    
    tools {
        maven "Maven-3.9.6"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/jagankumarpanda/WEB-APP-CICD-main.git'
            }
        }
        stage('Maven Build') {
            steps {
                dir('maven-web-app') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Image') {
            steps {
                dir('maven-web-app') {
                    sh 'docker build -t ashokit/mavenwebapp .'
                }
            }
        }
        stage('k8s deployment') {
            steps {
                dir('maven-web-app') {
                    sh 'kubectl apply -f k8s-deploy.yml'
                }
            }
        }
    }
}
Step 10: Access Application & Clean Up
----------------------------------------------
Access your application via your LoadBalancer or service endpoint URL: http://<LoadBalancer-URL>/context-path/

Important Cleanup: After practicing and testing your pipeline, delete your cluster and associated AWS resources to avoid unexpected cloud billing:

Bash
eksctl delete cluster --name ashokit-cluster --region ap-south-1
