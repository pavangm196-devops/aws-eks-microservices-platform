# AWS EKS Microservices Platform

> A production-grade microservices application deployed on AWS EKS with automated CI/CD using GitHub Actions — demonstrating end-to-end DevOps engineering on AWS.

---

## Tech Stack

![AWS](https://img.shields.io/badge/AWS-EKS-orange?logo=amazon-aws)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.31-blue?logo=kubernetes)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue?logo=docker)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-black?logo=github-actions)
![Node.js](https://img.shields.io/badge/Backend-Node.js%20%2B%20TypeScript-green?logo=node.js)
![React](https://img.shields.io/badge/Frontend-React%20%2B%20Vite-61DAFB?logo=react)
![MySQL](https://img.shields.io/badge/Database-MySQL%20%2B%20Prisma-4479A1?logo=mysql)
![Nginx](https://img.shields.io/badge/Server-Nginx-009639?logo=nginx)

---

## What Problem This Solves

Managing a monolithic application makes independent scaling, deployment, and maintenance difficult. This platform breaks the application into independently deployable microservices — each with its own database schema, Dockerfile, and Kubernetes deployment — enabling zero-downtime deploys, horizontal scaling per service, and isolated failure domains.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS EKS Cluster                       │
│                    (us-west-2, K8s 1.31)                     │
│                                                              │
│  ┌────────────┐    ┌────────────┐    ┌────────────────────┐  │
│  │  Frontend  │    │   Auth     │    │    Post Service    │  │
│  │ React+Vite │───▶│  Service  │    │  (CRUD Blog Posts) │  │
│  │   Nginx    │    │  Port 3001 │    │    Port 3002        │  │
│  └────────────┘    └─────┬──────┘    └────────────────────┘  │
│                          │                                    │
│  ┌────────────────┐  ┌───▼──────────────────────────────┐    │
│  │Comment Service │  │          MySQL (per service)      │    │
│  │  Port 3003     │  │   auth-db / post-db / comment-db  │    │
│  └────────────────┘  └──────────────────────────────────┘    │
│                                                              │
│  Node Groups: 3x t3.medium | Auto Scaler + ALB Ingress       │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
  GitHub Actions CI/CD
  Build → Push ECR → Deploy to EKS
```

> Architecture HTML diagram available at: `architecture-diagram.html`

---

## Key Features / What I Built

- **3 independent microservices** — Auth Service (JWT-based login/register), Post Service (CRUD blog posts), Comment Service (CRUD comments) — each isolated with its own MySQL schema via Prisma ORM
- **AWS EKS cluster** provisioned using `eksctl` with managed node groups (3x t3.medium), VPC CNI, CoreDNS, kube-proxy addons
- **IAM addon policies** for Auto Scaler, ALB Ingress, and CloudWatch logging enabled at cluster level
- **CloudWatch cluster logging** for API, audit, and authenticator log types
- **Dockerized services** — each service has its own `Dockerfile`; `docker-compose.yaml` for local development
- **GitHub Actions CI/CD pipeline** — automated build, Docker image push, and Kubernetes rollout on push to `main`
- **React + TypeScript + Vite frontend** served via Nginx in a container
- **Kubernetes manifests** in `/k8s` directory — Deployments, Services, Ingress for each microservice

---

## Folder Structure

```
aws-eks-microservices-platform/
├── .github/
│   └── workflows/
│       └── deploy.yaml          # GitHub Actions CI/CD pipeline
├── frontend/                    # React + TypeScript + Vite app
├── k8s/                         # Kubernetes manifests
│   ├── auth-deployment.yaml
│   ├── post-deployment.yaml
│   ├── comment-deployment.yaml
│   └── ingress.yaml
├── services/
│   ├── auth-service/            # Node.js + Express + Prisma
│   ├── post-service/
│   └── comment-service/
├── eks-cluster.yaml             # eksctl cluster config
├── docker-compose.yaml          # Local dev setup
├── init-db.sql                  # MySQL database initializer
├── architecture-diagram.html    # Visual architecture diagram
└── README.md
```

---

## How to Run

### Prerequisites

- Node.js 20+
- Docker & Docker Compose
- AWS CLI configured
- `eksctl` installed

### Local Development (Docker Compose)

```bash
# Clone the repo
git clone https://github.com/pavangm196-devops/aws-eks-microservices-platform.git
cd aws-eks-microservices-platform

# Start all services + MySQL + frontend
docker-compose up --build
```

Services available at:
- Frontend → http://localhost:3000
- Auth Service → http://localhost:3001
- Post Service → http://localhost:3002
- Comment Service → http://localhost:3003

### Run Services Individually

```bash
# Start MySQL
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mysql:8.0

# Initialize databases
mysql -h 127.0.0.1 -u root -ppassword < init-db.sql

# Start each service
cd services/auth-service && npm install && npx prisma db push && npx tsx src/index.ts
cd services/post-service && npm install && npx prisma db push && npx tsx src/index.ts
cd services/comment-service && npm install && npx prisma db push && npx tsx src/index.ts

# Start frontend
cd frontend && npm install && npm run dev
```

### Deploy to AWS EKS

```bash
# Create EKS cluster
eksctl create cluster -f eks-cluster.yaml

# Apply Kubernetes manifests
kubectl apply -f k8s/

# Verify pods are running
kubectl get pods -A
```

---

## What This Demonstrates

| Skill | How it's shown |
|---|---|
| Kubernetes | EKS cluster setup, managed node groups, K8s manifests for all services |
| AWS | EKS, IAM addon policies, ALB Ingress, CloudWatch logging |
| CI/CD | GitHub Actions pipeline for build → push → deploy |
| Containerization | Dockerfiles per service, Docker Compose for local dev |
| Microservices | 3 isolated services, independent DB schemas, inter-service communication |
| Infrastructure as Code | eksctl YAML cluster config |
| Backend Engineering | Node.js + TypeScript + Prisma ORM |

---

## Author

**Pavan G M** — DevOps Engineer | AWS Certified Solutions Architect (SAA-C03) | CKA Certified

[![GitHub](https://img.shields.io/badge/GitHub-pavangm196--devops-black?logo=github)](https://github.com/pavangm196-devops)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-pavan--gm96-blue?logo=linkedin)](https://linkedin.com/in/pavan-gm96)
