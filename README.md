# 🚀 ChatSphere — Production-Grade 3-Tier Chat App on AWS EKS

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS EKS](https://img.shields.io/badge/AWS_EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Amazon ECR](https://img.shields.io/badge/Amazon_ECR-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)

A fully containerized, production-grade **real-time chat application** deployed on **AWS EKS** using a complete 3-tier architecture — React frontend, Node.js/Express backend, and MongoDB database — all running as Kubernetes workloads with ALB Ingress, EBS-backed persistent storage, Secrets management, and health probes.

---

## 📌 Project Overview

This project demonstrates deploying a real-world full-stack application on Kubernetes (EKS) with production-grade practices:

- **3-tier architecture** — frontend, backend, and database as separate containerized workloads
- **AWS ALB Ingress Controller** — single load balancer with path-based routing (`/api` → backend, `/` → frontend)
- **EBS-backed persistent storage** — MongoDB data persists across pod restarts via dynamic EBS provisioning
- **Kubernetes Secrets** — no hardcoded credentials anywhere in manifests
- **Resource requests & limits** — all pods have defined CPU/memory boundaries
- **Liveness & readiness probes** — Kubernetes knows exactly when each pod is healthy
- **ECR for image registry** — private, AWS-managed container registry for backend and frontend images

---

## 🏗️ Architecture

```
<img width="1200" height="1000" alt="chatsphere_architecture" src="https://github.com/user-attachments/assets/18145c35-3f28-4ed0-b6f7-4541745d4abb" />


IAM Role → AWS Load Balancer Controller (kube-system)
EBS CSI Driver → Dynamic EBS volume provisioning
ECR → chatapp-backend:v1, chatapp-frontend:v1
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React + TailwindCSS + DaisyUI | Real-time chat UI |
| Backend | Node.js + Express + Socket.io | REST API + WebSocket server |
| Database | MongoDB 6.0 | Persistent chat data storage |
| Auth | JWT + Cookie Parser | Secure user authentication |
| Container Runtime | Docker | Image builds for frontend & backend |
| Image Registry | AWS ECR | Private container registry |
| Orchestration | Kubernetes (AWS EKS) | Container deployment & management |
| Load Balancer | AWS ALB + Ingress Controller | Path-based external traffic routing |
| Storage | AWS EBS (gp2) + PVC | Persistent MongoDB data |
| Secrets | Kubernetes Secrets (Opaque) | Keyless credential management |
| Cluster Management | eksctl | EKS cluster provisioning |
| Helm | AWS Load Balancer Controller | ALB controller installation |

---

## 📁 Project Structure

```
full-stack_chatApp/
├── backend/                       # Node.js/Express API
│   ├── src/
│   │   ├── index.js               # App entry point + health route
│   │   ├── routes/
│   │   │   ├── auth.route.js      # Auth endpoints
│   │   │   └── message.route.js   # Message endpoints
│   │   └── lib/
│   │       ├── db.js              # MongoDB connection
│   │       └── socket.js          # Socket.io setup
│   └── Dockerfile
├── frontend/                      # React + TailwindCSS app
│   ├── src/
│   └── Dockerfile
├── k8s/                           # Kubernetes manifests
│   ├── namespace.yml              # chat-app namespace
│   ├── secrets.yml                # JWT, MongoDB credentials
│   ├── mongodb-pvc.yml            # EBS-backed PVC (gp2, 5Gi)
│   ├── mongodb-deployment.yml     # MongoDB StatefulSet-style deployment
│   ├── mongodb-service.yml        # ClusterIP — internal only
│   ├── backend-deployment.yml     # Node.js API deployment
│   ├── backend-service.yml        # ClusterIP — internal only
│   ├── frontend-deployment.yml    # React app deployment
│   ├── frontend-service.yml       # ClusterIP — routed via ALB
│   └── ingress.yml                # ALB Ingress with path-based routing
└── docker-compose.yaml            # Local development setup
```

---

## ✅ Prerequisites

- AWS Account with permissions for EKS, ECR, IAM, ELB, EBS
- AWS CLI configured (`aws configure`)
- `eksctl` installed
- `kubectl` installed
- `helm` installed
- `docker` installed
- `git` installed

---

## 🚀 Deployment Guide

### Step 1 — Clone the Repository

```bash
git clone https://github.com/appusagar077/full-stack_chatApp.git
cd full-stack_chatApp
```

### Step 2 — Set Environment Variables

```bash
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION=us-west-1
```

### Step 3 — Build and Push Images to ECR

```bash
# Create ECR repositories
aws ecr create-repository --repository-name chatapp-backend --region $AWS_REGION
aws ecr create-repository --repository-name chatapp-frontend --region $AWS_REGION

# Authenticate Docker to ECR
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS \
  --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

# Build and push backend
cd backend
docker build -t chatapp-backend:v1 .
docker tag chatapp-backend:v1 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/chatapp-backend:v1
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/chatapp-backend:v1
cd ..

# Build and push frontend
cd frontend
docker build -t chatapp-frontend:v1 .
docker tag chatapp-frontend:v1 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/chatapp-frontend:v1
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/chatapp-frontend:v1
cd ..
```

### Step 4 — Update Deployment Manifests with ECR Image URIs

```bash
# Update image references in manifests
sed -i "s|<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com|$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com|g" k8s/backend-deployment.yml
sed -i "s|<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com|$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com|g" k8s/frontend-deployment.yml
```

### Step 5 — Create EKS Cluster

```bash
eksctl create cluster \
  --name chatapp-cluster \
  --region $AWS_REGION \
  --nodegroup-name chatapp-nodes \
  --node-type m7i-flex.large \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 2 \
  --managed
```
> ⏳ This takes approximately 15-20 minutes.

### Step 6 — Install EBS CSI Driver

```bash
eksctl create addon \
  --cluster chatapp-cluster \
  --name aws-ebs-csi-driver \
  --region $AWS_REGION \
  --force
```

### Step 7 — Install AWS Load Balancer Controller

```bash
# Associate OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster chatapp-cluster \
  --region $AWS_REGION \
  --approve

# Download and create IAM policy
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.2/docs/install/iam_policy.json
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# Create IAM service account
eksctl create iamserviceaccount \
  --cluster=chatapp-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region $AWS_REGION

# Install via Helm
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=chatapp-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

### Step 8 — Deploy the Application

Apply manifests in this exact order:

```bash
cd k8s

kubectl apply -f namespace.yml
kubectl apply -f secrets.yml
kubectl apply -f mongodb-pvc.yml
kubectl apply -f mongodb-deployment.yml
kubectl apply -f mongodb-service.yml
kubectl apply -f backend-deployment.yml
kubectl apply -f backend-service.yml
kubectl apply -f frontend-deployment.yml
kubectl apply -f frontend-service.yml
kubectl apply -f ingress.yml
```

### Step 9 — Verify Deployment

```bash
# Check all pods are Running
kubectl get pods -n chat-app

# Check PVC is Bound
kubectl get pvc -n chat-app

# Check services
kubectl get svc -n chat-app

# Check Ingress and get ALB hostname
kubectl get ingress -n chat-app
```

Wait 2-3 minutes for ALB to provision, then get the DNS name:

```bash
kubectl get ingress -n chat-app \
  -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}'
```

Open in browser:
```
http://<ALB-DNS-NAME>        → Chat app frontend
http://<ALB-DNS-NAME>/api/health  → Returns "OK" (backend health check)
```

---

## 🖼️ Screenshots

| Step | Screenshot |
|---|---|
| All pods Running | *<img width="1059" height="158" alt="image1" src="https://github.com/user-attachments/assets/df3e8266-62ac-47ca-9c5f-d55d8f6e126b" />*
 |
| PVC Bound (EBS gp2) | *<img width="1052" height="133" alt="image2" src="https://github.com/user-attachments/assets/6a207271-d2a4-4c4e-967b-f098873f383f" />
*  |
| Ingress with ALB hostname | *<img width="1696" height="225" alt="image3" src="https://github.com/user-attachments/assets/91824d8e-ddfe-4538-aeb1-0af8ab6ab023" />
* |
| Chat app live in browser | *<img width="1920" height="1033" alt="image4" src="https://github.com/user-attachments/assets/a3ab9567-fa31-4ac4-8191-0bbd9bc2178b" />
* |
| `/api/health` returning OK | *<img width="1920" height="1031" alt="image5" src="https://github.com/user-attachments/assets/d957002e-637e-43c6-9e03-d5dab3aa4f8f" />
* |
| EKS cluster overview (AWS Console) | *<img width="1920" height="1080" alt="image6" src="https://github.com/user-attachments/assets/a1fdbf9c-e802-4be8-99f0-fba909b409e1" />
* |
| ECR repositories with pushed images | *<img width="1920" height="1080" alt="image7" src="https://github.com/user-attachments/assets/1afaba40-2e9c-4ded-af3a-edcefbd745b2" />
* |
| ALB Active (EC2 → Load Balancers) | *<img width="1920" height="1080" alt="image8" src="https://github.com/user-attachments/assets/040b4d70-004d-4e78-afd9-f2845172291b" />
* |

---

## 🔑 Key Concepts Demonstrated

**AWS ALB Ingress Controller** — Instead of exposing each service with its own load balancer, a single ALB handles all external traffic with path-based routing (`/api` → backend, `/` → frontend). This is the production-standard approach on EKS — fewer load balancers, lower cost, cleaner architecture.

**EBS-backed Dynamic Storage Provisioning** — MongoDB's PVC uses `storageClassName: gp2`, which triggers the EBS CSI driver to automatically create and attach a real AWS EBS volume. No manual PV creation — Kubernetes handles it declaratively.

**Kubernetes Secrets for Credential Management** — MongoDB credentials and JWT secrets are stored as base64-encoded Kubernetes Secrets and injected as environment variables at runtime. Zero hardcoded credentials in any manifest or image.

**Resource Requests & Limits** — Every pod defines CPU and memory boundaries, preventing any single workload from starving others on the node — a requirement for production Kubernetes deployments.

**Liveness & Readiness Probes** — Kubernetes actively health-checks all 3 tiers. The backend exposes a dedicated `/api/health` endpoint; MongoDB is probed via `mongosh ping`. Pods that fail health checks are automatically restarted or removed from traffic rotation.

**IAM Roles for Service Accounts (IRSA)** — The ALB controller uses an IAM service account with scoped permissions, not long-lived access keys. This is the production-safe, least-privilege approach for giving Kubernetes workloads AWS permissions.

---

## 🧹 Cleanup (Avoid AWS Charges)

> ⚠️ EKS is **not free tier** — the control plane costs ~$0.10/hour regardless of usage. Always clean up after testing.

```bash
# Step 1 — Delete Ingress first (de-provisions the ALB)
kubectl delete -f k8s/ingress.yml

# Wait 1-2 minutes, then verify ALB is gone
aws elbv2 describe-load-balancers --region $AWS_REGION

# Step 2 — Delete remaining app resources
kubectl delete -f k8s/frontend-deployment.yml -f k8s/frontend-service.yml
kubectl delete -f k8s/backend-deployment.yml -f k8s/backend-service.yml
kubectl delete -f k8s/mongodb-deployment.yml -f k8s/mongodb-service.yml
kubectl delete -f k8s/mongodb-pvc.yml
kubectl delete -f k8s/secrets.yml -f k8s/namespace.yml

# Step 3 — Delete EKS cluster (removes EC2 nodes, VPC, control plane)
eksctl delete cluster --name chatapp-cluster --region $AWS_REGION

# Step 4 — Verify nothing remains
aws eks list-clusters --region $AWS_REGION
aws ec2 describe-instances --region $AWS_REGION \
  --filters "Name=instance-state-name,Values=running"
aws elbv2 describe-load-balancers --region $AWS_REGION
```

---

## 🙋 Author

**Sagar SM**
Cloud & DevOps Engineer | AWS | Kubernetes | Terraform | Jenkins

- GitHub: [@appusagar077](https://github.com/appusagar077)
- LinkedIn: [linkedin.com/in/sagar-sm](https://linkedin.com/in/sagar-sm)
