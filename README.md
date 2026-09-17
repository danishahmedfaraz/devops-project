# DevOps CI/CD Project

A production-style Java web application demonstrating an end-to-end DevOps CI/CD pipeline using AWS, Jenkins, Maven, SonarQube, Nexus, Docker, and Kubernetes.

---

## 🚀 Project Overview

This project demonstrates how a Java web application can be automatically built, tested, analyzed, packaged, containerized, and deployed through a complete CI/CD pipeline.

The pipeline performs:

- Source code checkout from GitHub
- Maven compilation and packaging
- Automated unit testing
- JaCoCo code coverage generation
- SonarQube static code analysis
- SonarQube Quality Gate validation
- Artifact publishing to Nexus Repository
- Docker image creation
- Kubernetes deployment
- Rolling updates
- Horizontal Pod Autoscaling
- Persistent storage
- Post-deployment application verification
- Kubernetes rollout history tracking
- Application rollback

---

## 🏗️ Architecture

```text
                         Developer
                             |
                             v
                          GitHub
                             |
                             v
                    +----------------+
                    |    Jenkins     |
                    |    CI / CD     |
                    +----------------+
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
           Checkout        Maven         Unit Tests
                             |
                             v
                       SonarQube
                             |
                             v
                       Quality Gate
                             |
                             v
                         Nexus
                             |
                             v
                      Docker Image
                             |
                             v
                  Kubernetes / Minikube
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
         Rolling Update     HPA       Persistent Storage
              |              |
              v              v
       Application Pods   2 → 4 Pods
              |
              v
       Kubernetes Service
              |
              v
        NGINX Ingress
              |
              v
         Application
🛠️ Technology Stack
Technology	Purpose
AWS EC2	Cloud infrastructure
Amazon Linux	Server operating system
GitHub	Source code management
Jenkins	CI/CD automation
Maven	Build and dependency management
JUnit	Unit testing
Mockito	Test mocking
JaCoCo	Code coverage
SonarQube	Static code analysis
Nexus Repository	Artifact repository
Docker	Application containerization
Kubernetes	Container orchestration
Minikube	Kubernetes cluster
NGINX Ingress	Application routing
HPA	Automatic pod scaling
PersistentVolumeClaim	Persistent application storage
Tomcat	Java application server
🔄 CI/CD Pipeline

The Jenkins pipeline consists of the following stages.

1. Checkout

Jenkins checks out the latest application source code from GitHub.

2. Build

Maven compiles the application and packages it as a WAR file.

mvn clean package
3. Test

Automated JUnit tests are executed.

mvn test

JaCoCo generates the code coverage report used during SonarQube analysis.

4. SonarQube Analysis

The project is analyzed by SonarQube for code quality and maintainability issues.

The Jenkins pipeline sends the analysis to the configured SonarQube server.

5. Quality Gate

Jenkins waits for the SonarQube Quality Gate result.

If the Quality Gate fails, the pipeline stops before deployment continues.

6. Publish to Nexus

The generated WAR artifact is published to Nexus Repository.

mvn deploy

This provides centralized artifact storage and version management.

7. Docker Deployment

Jenkins builds a Docker image using the Jenkins build number.

Example:

devops-project:35

The application container is deployed on port 8083.

8. Kubernetes Deployment

Jenkins loads the Docker image into Minikube and deploys the application into the dev namespace.

The deployment uses Kubernetes RollingUpdate strategy.

9. Post-Deployment Verification

Jenkins verifies:

Deployment status
Pod status
Rollout completion
Application HTTP response

The pipeline performs an HTTP test against the Kubernetes Service.

curl -f http://devops-project-service/devops-project/

If the application verification fails, the Jenkins pipeline fails.

☸️ Kubernetes Configuration

The application uses the following Kubernetes resources:

Namespace
   |
   +-- Deployment
   |      |
   |      +-- Application Pods
   |
   +-- Service
   |
   +-- Ingress
   |
   +-- ConfigMap
   |
   +-- Secret
   |
   +-- PersistentVolumeClaim
   |
   +-- HorizontalPodAutoscaler
Deployment

The application runs with two replicas by default.

replicas: 2

The deployment uses:

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1

This allows Kubernetes to create a replacement pod before removing an existing pod.

ConfigMap

Application configuration is managed using a Kubernetes ConfigMap.

Example configuration:

APP_NAME
APP_ENV

This separates application configuration from the container image.

Secret

Sensitive configuration such as database credentials is stored using a Kubernetes Secret.

The credentials included in this project are dummy demonstration credentials and are not production credentials.

Persistent Storage

The application uses a PersistentVolumeClaim.

Configuration:

Storage: 1Gi
Access Mode: ReadWriteOnce

Persistence was tested by writing data to /data, deleting the application pod, and verifying that the data remained available from the replacement pod.

📈 Horizontal Pod Autoscaling

The application uses Kubernetes Horizontal Pod Autoscaling.

Configuration:

Minimum replicas: 2
Maximum replicas: 4
CPU target: 70%

During testing, the HPA successfully scaled:

2 replicas → 4 replicas

when CPU utilization exceeded the configured threshold.

After the load was removed:

4 replicas → 2 replicas

This demonstrates automatic scaling based on CPU utilization.

🔄 Rolling Deployment

The project uses Kubernetes RollingUpdate deployment strategy.

Each Jenkins build creates a unique Docker image tag.

Example:

devops-project:35

The Jenkins pipeline then updates the Kubernetes deployment to the new image.

Kubernetes rollout history records the Jenkins build responsible for the deployment.

Example:

Jenkins Build 35

This provides deployment traceability and supports rollback to previous application versions.

↩️ Rollback

Kubernetes rollout history can be inspected using:

kubectl rollout history deployment/devops-project -n dev

A previous deployment can be restored using:

kubectl rollout undo deployment/devops-project -n dev

Deployment status can be checked with:

kubectl rollout status deployment/devops-project -n dev
🧪 Testing

The project uses:

JUnit 4 for unit testing
Mockito for mocking
JaCoCo for code coverage
SonarQube for static analysis
Jenkins Quality Gate enforcement

Example:

mvn test

The test suite validates the application servlet response and verifies expected application output.

🔍 Monitoring and Operations

Useful Kubernetes commands:

kubectl get pods -n dev
kubectl get deployment -n dev
kubectl get svc -n dev
kubectl get ingress -n dev
kubectl get hpa -n dev
kubectl top pods -n dev

Deployment status:

kubectl rollout status deployment/devops-project -n dev

Rollout history:

kubectl rollout history deployment/devops-project -n dev
📂 Repository Structure
devops-project/
│
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── devops/
│   │               └── App.java
│   │
│   └── test/
│       └── java/
│           └── com/
│               └── devops/
│                   └── AppTest.java
│
├── k8s-deployment.yaml
├── k8s-service.yaml
├── k8s-ingress.yaml
├── k8s-configmap.yaml
├── k8s-secret.yaml
├── k8s-pvc.yaml
├── k8s-hpa.yaml
├── Dockerfile
├── Jenkinsfile
├── pom.xml
└── README.md
🎯 Key DevOps Concepts Demonstrated

This project demonstrates practical experience with:

CI/CD pipelines
AWS EC2 infrastructure
Git-based development workflow
Automated builds
Automated testing
Code coverage
Static code analysis
Quality Gates
Artifact management
Docker containerization
Kubernetes deployments
Rolling updates
Horizontal Pod Autoscaling
ConfigMaps
Secrets
Persistent storage
Kubernetes Services
Ingress
Deployment verification
Rollback strategies
Release traceability
📊 Deployment Flow

The complete deployment workflow is:

Git Commit
    ↓
GitHub
    ↓
Jenkins
    ↓
Maven Build
    ↓
Unit Tests
    ↓
SonarQube Analysis
    ↓
Quality Gate
    ↓
Nexus Repository
    ↓
Docker Image
    ↓
Kubernetes
    ↓
Rolling Deployment
    ↓
Post-Deployment Verification
✅ Project Outcome

The project provides an automated path from source-code commit to a running Kubernetes application.

It demonstrates a complete DevOps workflow incorporating:

Automated CI/CD
Code quality validation
Artifact management
Containerization
Kubernetes orchestration
Rolling deployments
Automatic scaling
Persistent storage
Deployment verification
Rollback capabilities
Release traceability

The final application is deployed as a Java WAR application and exposed through Kubernetes Service and NGINX Ingress.
