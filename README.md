# BIGRS - Cloud-Native Task Manager Platform

A complete production-grade cloud-native application demonstrating modern DevOps practices, GitOps workflows, and AWS EKS deployment.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Technology Stack](#technology-stack)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Detailed Setup](#detailed-setup)
- [Application Components](#application-components)
- [Infrastructure Details](#infrastructure-details)
- [GitOps Workflow](#gitops-workflow)
- [CI/CD Pipeline](#cicd-pipeline)
- [Monitoring & Operations](#monitoring--operations)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## 🎯 Overview

BIGRS is a modern task management application built with a microservices architecture, deployed on AWS EKS using GitOps principles. The project demonstrates industry best practices for cloud-native applications including:

- Infrastructure as Code (Terraform)
- GitOps deployment (ArgoCD)
- CI/CD automation (Jenkins)
- Container orchestration (Kubernetes)
- Secret management (External Secrets Operator)
- Automated image updates (ArgoCD Image Updater)
- TLS certificate management (cert-manager)
- High availability and auto-scaling

**Live Application:**
- **Frontend**: https://bigrs.app
- **Backend API**: https://api.bigrs.app
- **ArgoCD**: https://argocd.bigrs.app
- **Jenkins**: https://jenkins.bigrs.app

---

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              AWS Cloud                                   │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                         VPC (Multi-AZ)                             │ │
│  │                                                                    │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │ │
│  │  │ Public Subnet│  │ Public Subnet│  │ Public Subnet│            │ │
│  │  │   (AZ-1)     │  │   (AZ-2)     │  │   (AZ-3)     │            │ │
│  │  │              │  │              │  │              │            │ │
│  │  │  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────┐  │            │ │
│  │  │  │NAT GW  │  │  │  │NAT GW  │  │  │  │NAT GW  │  │            │ │
│  │  │  └────────┘  │  │  └────────┘  │  │  └────────┘  │            │ │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘            │ │
│  │         │                 │                 │                     │ │
│  │  ┌──────▼──────┐  ┌───────▼──────┐  ┌──────▼───────┐            │ │
│  │  │Private Sub  │  │Private Sub   │  │Private Sub   │            │ │
│  │  │  (AZ-1)     │  │  (AZ-2)      │  │  (AZ-3)      │            │ │
│  │  │             │  │              │  │              │            │ │
│  │  │ ┌─────────┐ │  │ ┌─────────┐  │  │ ┌─────────┐  │            │ │
│  │  │ │EKS Nodes│ │  │ │EKS Nodes│  │  │ │EKS Nodes│  │            │ │
│  │  │ └─────────┘ │  │ └─────────┘  │  │ └─────────┘  │            │ │
│  │  └─────────────┘  └──────────────┘  └──────────────┘            │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    EKS Cluster Components                       │    │
│  │                                                                 │    │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐ │    │
│  │  │  ArgoCD    │  │  Jenkins   │  │   Nginx    │  │  Cert    │ │    │
│  │  │            │  │            │  │  Ingress   │  │ Manager  │ │    │
│  │  └────────────┘  └────────────┘  └────────────┘  └──────────┘ │    │
│  │                                                                 │    │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐               │    │
│  │  │  External  │  │   Image    │  │    ECR     │               │    │
│  │  │  Secrets   │  │  Updater   │  │   Token    │               │    │
│  │  └────────────┘  └────────────┘  └────────────┘               │    │
│  │                                                                 │    │
│  │  ┌─────────────────────────────────────────────────────────┐   │    │
│  │  │           Task Manager Application                      │   │    │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │   │    │
│  │  │  │ Frontend │  │ Backend  │  │  Redis   │  │ MySQL  │  │   │    │
│  │  │  │  (Nginx) │  │(Node.js) │  │ (Cache)  │  │ (RDS)  │  │   │    │
│  │  │  └──────────┘  └──────────┘  └──────────┘  └────────┘  │   │    │
│  │  └─────────────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│  │     ECR      │  │  RDS MySQL   │  │   Secrets    │                  │
│  │  Repositories│  │   Database   │  │   Manager    │                  │
│  └──────────────┘  └──────────────┘  └──────────────┘                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Data Flow

```
┌──────────┐
│  User    │
└────┬─────┘
     │
     ▼
┌──────────────────────────┐
│   Route53 / CloudFlare   │
│   DNS Resolution         │
└────┬─────────────────────┘
     │
     ▼
┌──────────────────────────┐
│   Network Load Balancer  │
│   (AWS NLB)              │
└────┬─────────────────────┘
     │
     ▼
┌──────────────────────────┐
│   NGINX Ingress          │
│   Controller             │
│   - TLS Termination      │
│   - Routing              │
└────┬─────────────────────┘
     │
     ├──────────────────────────────┬──────────────────────┐
     ▼                              ▼                      ▼
┌──────────┐              ┌──────────────┐      ┌────────────────┐
│ Frontend │              │   Backend    │      │    ArgoCD      │
│ (Nginx)  │──────────────│   (API)      │      │    (GitOps)    │
│          │     HTTP     │              │      │                │
│ bigrs.app│              │api.bigrs.app │      │argocd.bigrs.app│
└──────────┘              └──────┬───────┘      └────────────────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
              ┌─────────┐  ┌─────────┐  ┌─────────┐
              │  Redis  │  │  MySQL  │  │ Secrets │
              │ (Cache) │  │  (RDS)  │  │ Manager │
              └─────────┘  └─────────┘  └─────────┘
```

---

## 📁 Repository Structure

The project is organized into three main repositories:

### 1. Infrastructure Repository

Contains all Terraform code for provisioning AWS infrastructure:

```
Infrastructure/
├── terraform/
│   ├── environment/
│   │   ├── dev/              # Development environment
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   ├── terraform.tfvars
│   │   │   ├── outputs.tf
│   │   │   ├── provider.tf
│   │   │   └── creds         # AWS credentials (gitignored)
│   │   └── prod/             # Production environment
│   │       └── (same structure as dev)
│   └── modules/
│       ├── vpc/              # Network infrastructure
│       ├── eks/              # Kubernetes cluster
│       ├── ecr/              # Container registry
│       ├── iam/              # IAM roles & policies
│       ├── rds/              # MySQL database
│       ├── bastion/          # Bastion host (prod only)
│       └── argocd/           # ArgoCD bootstrap
├── scripts/
│   ├── bootstrap-argocd.sh  # ArgoCD installation
│   ├── cleanup-argocd.sh    # ArgoCD cleanup
│   └── delete-nlb.sh        # Load balancer cleanup
├── argocd/
│   └── bootstrap-app.yaml   # App of Apps config
└── helm-values/
    └── argocd-values.yaml   # ArgoCD customization
```

**Key Features:**
- Multi-AZ high availability
- Auto-scaling node groups
- Private subnet deployment
- Pod Identity (modern IRSA)
- S3 backend state management
- Automated cleanup scripts

### 2. Platform Repository

Contains Kubernetes manifests and ArgoCD applications:

```
Platform/
├── argo-apps/                          # ArgoCD Application definitions
│   ├── pre-apps.yaml                   # Prerequisites (wave -1)
│   ├── cert-manager.yaml               # TLS certificates (wave 0)
│   ├── cert-manager-issuers.yaml       # Let's Encrypt issuer (wave 1)
│   ├── nginx-ingress-controller.yaml   # Ingress controller (wave 2)
│   ├── external-secrets-operator.yaml  # Secret manager (wave 3)
│   ├── external-secrets-app.yaml       # Secret store config (wave 4)
│   ├── jenkins-app.yaml                # CI/CD server (wave 5)
│   ├── image-updater-app.yaml          # Auto updates (wave 6)
│   └── nodejs-app.yaml                 # Main application (wave 7)
│
├── helm-values/                        # Helm chart values
│   ├── cert-manager-values.yaml
│   ├── cluster_issuer.yaml
│   ├── nginx-values.yaml
│   ├── nginx-values-tf.yaml            # Terraform-generated
│   ├── jenkins-values.yaml
│   ├── external-secrets-values.yaml
│   └── image-updater-values.yaml
│
└── apps/                               # Application manifests
    ├── pre-apps/                       # Bootstrap resources
    │   ├── nginx-ingress-contrller/
    │   ├── pre-jenkins/
    │   ├── pre-ESO/
    │   ├── ecr-token-refresher/
    │   └── argocd-ingress.yaml
    │
    ├── external-secrets/               # Secret management
    │   ├── cluster-secretstore.yaml
    │   └── secrets/
    │       └── db-credentials.yaml
    │
    └── nodejs-app/                     # Task Manager app
        ├── Backend/
        │   ├── backend-deployment.yaml
        │   ├── configmap.yaml
        │   └── backend-ingress.yaml    # api.bigrs.app
        ├── Frontend/
        │   ├── frontend-deployment.yaml
        │   └── ingress.yaml            # bigrs.app
        ├── Redis/
        │   └── redis-deployment.yaml
        ├── serviceaccount.yaml
        └── kustomization.yaml
```

**Key Features:**
- App of Apps pattern
- Sync wave ordering
- Multi-source applications
- Automated image updates
- Secret synchronization
- Network policies

### 3. nodejs_app Repository

Contains application source code:

```
nodejs_app/
├── server.js                    # Express server
├── package.json
├── Dockerfile.backend
├── Dockerfile.frontend
├── Jenkinsfile                  # CI/CD pipeline
├── docker-compose.yml           # Local development
├── docker-entrypoint.sh
├── docker-entrypoint-frontend.sh
│
├── config/                      # Configuration
│   ├── database.js              # MySQL connection pool
│   └── redis.js                 # Redis client
│
├── controllers/                 # Business logic
│   └── taskController.js
│
├── models/                      # Data models
│   └── Task.js
│
├── routes/                      # API routes
│   └── tasks.js
│
├── public/                      # Frontend (SPA)
│   ├── index.html
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
│
├── scripts/                     # Utilities
│   └── init-db.js               # Database initialization
│
├── k8s/                         # Kubernetes manifests
│   ├── namespace.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   ├── redis-deployment.yaml
│   ├── mysql-deployment.yaml
│   ├── configmaps.yaml
│   ├── secrets.yaml
│   ├── persistent-volumes.yaml
│   ├── ingress.yaml
│   └── deploy.sh
│
└── nginx.conf                   # Frontend web server config
```

**Key Features:**
- RESTful API
- Redis caching
- MySQL persistence
- Health checks
- Docker multi-stage builds
- Horizontal pod autoscaling

---

## 🛠️ Technology Stack

### Infrastructure Layer
- **Cloud Provider**: AWS
- **IaC Tool**: Terraform 1.5+
- **Kubernetes**: AWS EKS 1.31
- **Container Registry**: Amazon ECR
- **Database**: Amazon RDS MySQL 8.0
- **Cache**: Redis 7.0
- **Load Balancer**: AWS Network Load Balancer
- **DNS**: Route53 / CloudFlare
- **Secrets**: AWS Secrets Manager

### Platform Layer
- **GitOps**: ArgoCD 8.1.0
- **CI/CD**: Jenkins 2.528.1-lts-jdk21
- **Ingress**: NGINX Ingress Controller 4.14.0
- **TLS**: cert-manager 1.16.1 + Let's Encrypt
- **Secret Management**: External Secrets Operator 1.0.0
- **Image Updates**: ArgoCD Image Updater 0.14.0
- **Package Manager**: Helm 3.x

### Application Layer
- **Backend**: Node.js 18 + Express.js 4.18
- **Frontend**: HTML5, CSS3, JavaScript (Vanilla) + Tailwind CSS
- **Database ORM**: mysql2 (Promise-based)
- **Cache Client**: redis 4.6
- **Web Server**: Nginx (Alpine)

### Development Tools
- **Version Control**: Git + GitHub
- **Container Runtime**: Docker 25.x
- **Container Orchestration**: Docker Compose 2.x
- **Code Editor**: VS Code (recommended)
- **CLI Tools**: kubectl, helm, aws-cli, terraform

---

## ✨ Features

### Application Features
- ✅ **Task Management**: Create, read, update, delete tasks
- ✅ **Status Tracking**: Pending, In Progress, Completed
- ✅ **Priority Levels**: Low, Medium, High
- ✅ **Due Dates**: Track task deadlines
- ✅ **Statistics Dashboard**: Real-time task metrics
- ✅ **Redis Caching**: ~80% faster response times
- ✅ **Responsive UI**: Mobile-friendly design
- ✅ **Real-time Updates**: Live task synchronization

### Infrastructure Features
- 🏗️ **Multi-AZ Deployment**: High availability across 3 AZs
- 🔄 **Auto-scaling**: HPA for both frontend and backend
- 🔐 **Pod Identity**: Secure AWS service access
- 🌐 **Private Networking**: Services in private subnets
- 💾 **Persistent Storage**: EBS volumes for stateful apps
- 🔒 **Network Policies**: Restricted pod-to-pod communication
- 🚀 **Blue-Green Deployments**: Zero-downtime updates
- 📊 **Resource Limits**: CPU and memory constraints

### DevOps Features
- 🔄 **GitOps Workflow**: Declarative infrastructure
- 🤖 **Automated CI/CD**: Build, test, deploy pipeline
- 🖼️ **Automated Image Updates**: Latest images auto-deployed
- 🔐 **Secret Management**: External Secrets Operator
- 📜 **TLS Certificates**: Automated cert issuance/renewal
- 🔍 **Health Monitoring**: Liveness and readiness probes
- 📦 **Container Scanning**: ECR image vulnerability scans
- 🧹 **Automated Cleanup**: Resource lifecycle management

---

## 📋 Prerequisites

### Required Software

**For Infrastructure Deployment:**
- AWS CLI 2.x
- Terraform 1.5+
- kubectl 1.28+
- helm 3.x
- Git 2.x

**For Application Development:**
- Node.js 18+
- Docker 25.x
- Docker Compose 2.x
- MySQL 8.0 (for local development)
- Redis 7.0 (for local development)

### AWS Requirements

**AWS Account Setup:**
1. AWS account with appropriate permissions
2. IAM user with programmatic access
3. AWS credentials configured (`aws configure`)

**Required IAM Permissions:**
- EC2 (VPC, Security Groups, etc.)
- EKS (Cluster management)
- ECR (Container registry)
- RDS (Database)
- IAM (Role management)
- S3 (Terraform state)
- Secrets Manager (Secret storage)
- EBS (Persistent volumes)
- ELB (Load balancers)

**Estimated AWS Costs:**
- EKS Control Plane: ~$73/month
- EC2 Instances (3x t3.medium): ~$90/month
- RDS MySQL (db.t3.micro): ~$15/month
- NAT Gateways (3x): ~$100/month
- Load Balancers: ~$20/month
- **Total**: ~$300/month (varies by usage)

### Domain Requirements

- Domain name (e.g., bigrs.app)
- DNS management access (Route53 or CloudFlare)
- SSL/TLS certificate (automated via Let's Encrypt)

### GitHub Requirements

- GitHub account
- Personal Access Token (PAT) with repo permissions
- Three repositories created:
  - Infrastructure
  - Platform
  - nodejs_app

---

## 🚀 Quick Start

### 1. Clone Repositories

```bash
# Clone all three repositories
git clone https://github.com/BIGRS-ITI/Infrastructure.git
git clone https://github.com/BIGRS-ITI/Platform.git
git clone https://github.com/BIGRS-ITI/nodejs_app.git
```

### 2. Configure AWS Credentials

```bash
cd Infrastructure/terraform/environment/dev

# Create credentials file
cat > creds << EOF
[default]
aws_access_key_id = YOUR_AWS_ACCESS_KEY
aws_secret_access_key = YOUR_AWS_SECRET_KEY
EOF

chmod 600 creds
```

### 3. Configure Terraform Variables

```bash
# Edit terraform.tfvars
vim terraform.tfvars
```

**Minimum required variables:**
```hcl
cluster_name         = "bigrs-cluster"
aws_region           = "us-east-1"
github_platform_repo = "BIGRS-ITI/Platform"
github_token         = "ghp_your_github_token"
```

### 4. Deploy Infrastructure

```bash
# Initialize Terraform
terraform init

# Review plan
terraform plan

# Deploy (takes ~15-20 minutes)
terraform apply
```

### 5. Access Cluster

```bash
# Configure kubectl
aws eks update-kubeconfig --name bigrs-cluster --region us-east-1

# Verify access
kubectl get nodes
kubectl get pods -A
```

### 6. Verify Deployment

```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Wait for all apps to be healthy
kubectl wait --for=condition=Healthy application --all -n argocd --timeout=600s

# Get ingress URLs
kubectl get ingress -A
```

### 7. Access Applications

```bash
# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d

# Get Jenkins admin password
kubectl exec -n jenkins deployment/jenkins -- \
  cat /run/secrets/additional/chart-admin-password
```

**Access URLs:**
- **Task Manager**: https://bigrs.app
- **Backend API**: https://api.bigrs.app
- **ArgoCD UI**: https://argocd.bigrs.app
- **Jenkins**: https://jenkins.bigrs.app

---

## 📚 Detailed Setup

### Infrastructure Deployment

#### Development Environment

```bash
cd Infrastructure/terraform/environment/dev

# 1. Create AWS credentials
cat > creds << EOF
[default]
aws_access_key_id = YOUR_KEY
aws_secret_access_key = YOUR_SECRET
EOF

# 2. Configure variables
cat > terraform.tfvars << EOF
# Cluster Configuration
cluster_name = "bigrs-cluster"
eks_version  = "1.31"
aws_region   = "us-east-1"

# VPC Configuration
vpc_cidr = "10.0.0.0/16"
vpc_name = "bigrs-vpc"

subnets_config = {
  public-1 = {
    cidr_block        = "10.0.1.0/24"
    availability_zone = "us-east-1a"
    map_public_ip     = true
  }
  public-2 = {
    cidr_block        = "10.0.2.0/24"
    availability_zone = "us-east-1b"
    map_public_ip     = true
  }
  public-3 = {
    cidr_block        = "10.0.3.0/24"
    availability_zone = "us-east-1c"
    map_public_ip     = true
  }
  private-1 = {
    cidr_block        = "10.0.11.0/24"
    availability_zone = "us-east-1a"
    map_public_ip     = false
  }
  private-2 = {
    cidr_block        = "10.0.12.0/24"
    availability_zone = "us-east-1b"
    map_public_ip     = false
  }
  private-3 = {
    cidr_block        = "10.0.13.0/24"
    availability_zone = "us-east-1c"
    map_public_ip     = false
  }
}

# ECR Configuration
repositories = {
  backend = {
    name        = "bigrs-nodejs-app-backend"
    description = "Backend API container"
  }
  frontend = {
    name        = "bigrs-nodejs-app-frontend"
    description = "Frontend web application"
  }
}

image_retention_count = 5
scan_on_push          = true
image_tag_mutability  = "MUTABLE"

# GitHub Configuration
github_platform_repo = "BIGRS-ITI/Platform"
github_token         = "ghp_your_token_here"

# EKS Configuration
endpoint_private_access_var = true
endpoint_public_access_var  = false
node_group_instance_type    = "t3.medium"

# Environment
use_bastion                = false
bastion_private_key_path   = ""
environment                = "dev"
EOF

# 3. Initialize and deploy
terraform init
terraform plan
terraform apply -auto-approve
```

#### Production Environment

Production includes a bastion host for secure access:

```bash
cd Infrastructure/terraform/environment/prod

# Additional production variables:
bastion_key_name         = "bigrs-bastion-key"
bastion_instance_type    = "t3.micro"
bastion_allowed_cidrs    = ["YOUR_IP/32"]
enable_bastion_elastic_ip = true
use_bastion              = true
bastion_private_key_path = "/path/to/bastion-key.pem"
```

### Application Deployment

#### Local Development

```bash
cd nodejs_app

# 1. Copy environment template
cp .env.example .env

# 2. Edit environment variables
vim .env

# 3. Start with Docker Compose
docker-compose up -d

# 4. Initialize database
docker-compose exec backend npm run init-db

# 5. Access application
open http://localhost:8080
```

**Environment Variables:**
```env
# Server
NODE_ENV=development
PORT=3000

# Database
DB_HOST=mysql
DB_PORT=3306
DB_USER=taskuser
DB_PASSWORD=taskpassword
DB_NAME=taskmanager

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=

# Cache
CACHE_TTL=300

# Frontend
FRONTEND_URL=http://localhost:8080
```

#### Kubernetes Deployment (Manual)

```bash
cd nodejs_app/k8s

# 1. Update image references
vim backend-deployment.yaml
vim frontend-deployment.yaml

# 2. Deploy resources
kubectl apply -f namespace.yaml
kubectl apply -f secrets.yaml
kubectl apply -f configmaps.yaml
kubectl apply -f persistent-volumes.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f redis-deployment.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f ingress.yaml

# 3. Verify deployment
kubectl get all -n taskmanager
```

#### GitOps Deployment (Recommended)

Applications are automatically deployed via ArgoCD after infrastructure provisioning.

**Deployment Flow:**
1. Terraform deploys ArgoCD
2. ArgoCD syncs from Platform repository
3. Applications deployed in sync wave order:
   - Wave -1: Pre-requisites (namespaces, etc.)
   - Wave 0: cert-manager
   - Wave 1: Cluster issuers
   - Wave 2: NGINX Ingress
   - Wave 3: External Secrets Operator
   - Wave 4: External Secrets app
   - Wave 5: Jenkins
   - Wave 6: Image Updater
   - Wave 7: Task Manager app

---

## 🔧 Application Components

### Backend API (Node.js + Express)

**Location**: `nodejs_app/`

**Key Features:**
- RESTful API endpoints
- MySQL database integration
- Redis caching layer
- Health check endpoints
- Request validation
- Error handling middleware

**API Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/tasks` | Get all tasks (with filters) |
| GET | `/api/tasks/stats` | Get task statistics |
| GET | `/api/tasks/:id` | Get task by ID |
| POST | `/api/tasks` | Create new task |
| PUT | `/api/tasks/:id` | Update task |
| DELETE | `/api/tasks/:id` | Delete task |
| GET | `/api/redis-stats` | Redis cache statistics |
| POST | `/api/redis-reset` | Reset Redis cache |

**Request Examples:**

```bash
# Create task
curl -X POST https://api.bigrs.app/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Deploy to production",
    "description": "Deploy v2.0 to EKS cluster",
    "status": "pending",
    "priority": "high",
    "due_date": "2025-12-01"
  }'

# Get all tasks
curl https://api.bigrs.app/api/tasks

# Filter by status
curl https://api.bigrs.app/api/tasks?status=in-progress

# Get statistics
curl https://api.bigrs.app/api/tasks/stats

# Health check
curl https://api.bigrs.app/api/health
```

**Response Format:**

```json
{
  "success": true,
  "count": 5,
  "data": [
    {
      "id": 1,
      "title": "Deploy to production",
      "description": "Deploy v2.0 to EKS cluster",
      "status": "pending",
      "priority": "high",
      "due_date": "2025-12-01",
      "created_at": "2025-11-18T10:00:00.000Z",
      "updated_at": "2025-11-18T10:00:00.000Z"
    }
  ]
}
```

### Frontend (Nginx + Vanilla JS)

**Location**: `nodejs_app/public/`

**Key Features:**
- Single Page Application (SPA)
- Responsive design (Tailwind CSS)
- Real-time task updates
- Redis cache statistics
- Toast notifications
- Task filtering
- Dark mode support

**UI Components:**
- Task cards with drag-and-drop (planned)
- Statistics dashboard
- Modal forms
- Filter buttons
- Redis stats viewer

### Database Schema

**MySQL (RDS)**

```sql
CREATE TABLE tasks (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  status ENUM('pending', 'in-progress',
