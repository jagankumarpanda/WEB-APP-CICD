# End-to-End DevOps CI/CD Pipeline

## Project Overview

This project demonstrates an end-to-end CI/CD pipeline for building, containerizing, and deploying a Java web application on **AWS EKS**.

The pipeline automates the application build using **Maven**, creates a **Docker container image**, and deploys the application to **Kubernetes running on Amazon EKS** using **Jenkins**.

## Architecture

![CI/CD Pipeline Architecture](maven-web-app/architecture.png)

## CI/CD Workflow

```text
Developer
    |
    | Push Code
    v
GitHub Repository
    |
    | Jenkins Checkout
    v
Jenkins
    |
    | Maven Build
    v
Java Web Application
    |
    | Docker Build
    v
Docker Image
    |
    | Kubernetes Deployment
    v
AWS EKS Cluster
    |
    v
Application Pods
    |
    v
LoadBalancer / Application
```

### Workflow Steps

1. **Code Commit** – Developer pushes application code to GitHub.
2. **Source Checkout** – Jenkins pulls the latest source code from GitHub.
3. **Build** – Maven compiles the Java application and creates the WAR package.
4. **Containerization** – Jenkins builds a Docker image using the project Dockerfile.
5. **Deployment** – Kubernetes manifests are applied using `kubectl`.
6. **Orchestration** – Amazon EKS manages the application pods.
7. **Application Access** – The application is exposed through a Kubernetes Service/LoadBalancer.

## Tech Stack

| Category | Technologies |
|---|---|
| Cloud | AWS EC2, EKS, IAM |
| Version Control | Git, GitHub |
| CI/CD | Jenkins |
| Build Tool | Apache Maven |
| Containerization | Docker |
| Orchestration | Kubernetes |
| Kubernetes Platform | Amazon EKS |
| Runtime | Apache Tomcat |
| Application | Java, JSP |
| CLI Tools | AWS CLI, kubectl, eksctl |

## Project Structure

```text
WEB-APP-CICD-main/
└── maven-web-app/
    ├── Declarative-JenkinsFile
    ├── Scripted-JenkinsFile
    ├── docker-k8s-jenkinsfile
    ├── Jenkinsfile
    ├── Dockerfile
    ├── k8s-deploy.yml
    ├── pom.xml
    └── src/
        └── main/
            └── webapp/
                ├── WEB-INF/
                │   └── web.xml
                ├── demo.jsp
                └── index.jsp
```

## Infrastructure

The project uses AWS resources for the CI/CD and deployment environment:

- **EC2** – Jenkins and EKS management environment
- **IAM** – Access and permissions
- **Amazon EKS** – Managed Kubernetes cluster
- **Kubernetes** – Application deployment and service management
- **eksctl** – EKS cluster provisioning
- **kubectl** – Kubernetes cluster management

## Jenkins Pipeline

The Jenkins pipeline performs the following stages:

```text
Clone Repository
       ↓
Maven Build
       ↓
Docker Image Build
       ↓
Kubernetes Deployment
```

Example Jenkins pipeline:

```groovy
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

        stage('Kubernetes Deployment') {
            steps {
                dir('maven-web-app') {
                    sh 'kubectl apply -f k8s-deploy.yml'
                }
            }
        }
    }
}
```

## Kubernetes Deployment

The Kubernetes deployment is defined in:

```text
k8s-deploy.yml
```

Jenkins applies the Kubernetes configuration using:

```bash
kubectl apply -f k8s-deploy.yml
```

This creates the required Kubernetes resources and deploys the application to the EKS cluster.

## Key Learning Outcomes

Through this project, I gained hands-on experience with:

- Building CI/CD pipelines using Jenkins
- Managing source code with Git and GitHub
- Building Java applications with Maven
- Creating Docker container images
- Deploying applications using Kubernetes
- Provisioning Amazon EKS using eksctl
- Managing Kubernetes clusters using kubectl
- Working with AWS EC2, IAM and EKS
- Automating application deployment from source code to Kubernetes

## AWS Cleanup

AWS resources can generate charges when left running. After completing testing, remove the EKS cluster and other resources that are no longer required.

Example:

```bash
eksctl delete cluster --name <cluster-name> --region <region>
```

Always verify that unused EC2 instances, load balancers, EBS volumes, and other AWS resources have also been removed.

## Conclusion

This project demonstrates a complete DevOps workflow where application code moves from **GitHub → Jenkins → Maven → Docker → Kubernetes → AWS EKS** through an automated CI/CD pipeline.
