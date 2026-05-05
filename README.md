# Tech Challenge 2: Continuous Deployment with Jenkins, Docker, and AWS EKS

## 🚀 Live Application
**URL:** http://a65ec2d8bea2549abab0db610afc0d52-546781412.us-east-2.elb.amazonaws.com

## 📋 Overview
This project demonstrates a full CI/CD pipeline using Jenkins, Docker, and AWS EKS. A Node.js web application is containerized with Docker, stored in Amazon ECR, deployed to an EKS Kubernetes cluster, and automatically built and deployed via a Jenkins pipeline.

## 🏗️ Architecture
- **Web App:** Node.js/Express serving "Hello, World!"
- **Containerization:** Docker (linux/amd64)
- **Container Registry:** Amazon ECR
- **Orchestration:** AWS EKS (Kubernetes 1.31)
- **Infrastructure:** Terraform (VPC, EKS cluster, node groups)
- **CI/CD:** Jenkins pipeline

## 📁 Project Structure


## 🔧 Prerequisites
- AWS CLI configured with appropriate permissions
- Terraform >= 1.0
- Docker Desktop
- kubectl
- Node.js 18+
- Jenkins server with plugins:
  - Docker Pipeline
  - Amazon ECR
  - Kubernetes CLI
  - AWS Credentials

## 🚀 Environment Setup

### 1. Clone the Repository
```bash
git clone https://github.com/cmack21/tech-challenge-2.git
cd tech-challenge-2
```

### 2. Configure AWS CLI
```bash
aws configure
# Enter your AWS Access Key ID, Secret, region (us-east-2), and output format (json)
```

### 3. Create ECR Repository
```bash
aws ecr create-repository --repository-name hello-world-app --region us-east-2
```

## 🏗️ Infrastructure Deployment (Terraform)

### Provision EKS Cluster
```bash
cd terraform
terraform init
terraform plan
terraform apply
```
This provisions:
- A VPC with public and private subnets across 2 availability zones
- NAT Gateway for private subnet internet access
- EKS cluster running Kubernetes 1.31
- Managed node group with 2 x t3.medium EC2 instances

### Connect kubectl to EKS
```bash
aws eks update-kubeconfig --region us-east-2 --name hello-world-cluster
kubectl get nodes  # Should show 2 Ready nodes
```

## 🐳 Docker

### Build and Push Image Manually
```bash
cd app

# Login to ECR
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 691652240094.dkr.ecr.us-east-2.amazonaws.com

# Build for linux/amd64 (required for EKS)
docker buildx build --platform linux/amd64 -t 691652240094.dkr.ecr.us-east-2.amazonaws.com/hello-world-app:latest --push .
```

## ☸️ Kubernetes Deployment

### Deploy Manually
```bash
cd k8s
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get service hello-world-service  # Get the external URL
```

## 🔄 Jenkins Pipeline

### Pipeline Stages
1. **Checkout** - Pulls the latest code from GitHub
2. **Build Docker Image** - Builds the app image for linux/amd64
3. **Push to ECR** - Authenticates and pushes the image to Amazon ECR
4. **Deploy to EKS** - Updates the Kubernetes deployment with the new image and waits for rollout

### Jenkins Setup
1. Launch a t3.medium EC2 instance with Ubuntu 22.04
2. Install Java 21, Docker, Jenkins, kubectl, and AWS CLI
3. Add AWS credentials in Jenkins with ID `Admin-IAM`
4. Create a Pipeline job pointing to this repository
5. Run the pipeline — it will automatically build and deploy on every run

### Triggering a Deployment
Any push to the `main` branch can trigger the pipeline. To manually trigger:
1. Go to Jenkins dashboard
2. Click `hello-world-pipeline`
3. Click `Build Now`

## 📖 Terraform Code Explanation

### main.tf
Configures the AWS provider and provisions a VPC using the `terraform-aws-modules/vpc` module. Creates public and private subnets across `us-east-2a` and `us-east-2b`, with a NAT gateway to allow private subnet nodes to reach the internet.

### eks.tf
Uses the `terraform-aws-modules/eks` module to provision an EKS cluster running Kubernetes 1.31. Configures a managed node group with t3.medium instances that auto-scale between 1 and 3 nodes.

### variables.tf
Defines the AWS region (`us-east-2`) and cluster name (`hello-world-cluster`) as input variables.

### outputs.tf
Outputs the cluster endpoint URL and cluster name after provisioning.

## 🧹 Cleanup
To avoid AWS charges, destroy all resources when done:
```bash
cd terraform
terraform destroy
```
Also delete the ECR repository:
```bash
aws ecr delete-repository --repository-name hello-world-app --region us-east-2 --force
```
