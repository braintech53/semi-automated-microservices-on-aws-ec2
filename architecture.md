# 🏗️ Architecture Overview

This document describes the full infrastructure design and logic behind the **Semi-Automated Microservices on AWS EC2** project. The architecture enables horizontal scalability, security, and modular service deployment across multiple environments — without using traditional CI/CD tools.

---

## 🎯 Purpose

To demonstrate how a real-world, production-grade microservice-based SaaS can be hosted and scaled on AWS EC2 using infrastructure best practices — while keeping full control of deployments without any third-party automation tools like Jenkins or GitHub Actions.

---

## 🧱 Environments

The infrastructure is divided into **three isolated environments**:

- **Test**
- **Stage**
- **Production**

Each environment includes:

- Its own ALB, security groups, and ASG per service
- Separate subnets and routing logic
- Git branch mapping (`test`, `stage`, `main`)
- Custom domain/subdomain routing via Route53

---

## ⚙️ Microservices

Each service runs on its own EC2 instance (inside an ASG):

1. `Admin Panel (React)`
2. `Auth Service`
3. `Gmail Service`
4. `Calendar Service`
5. `MCP (Main Controller)`
6. `AI Proxy`
7. `Reminder Service`
8. `Telegram Bot`

**Each EC2 is independently scalable** via its Auto Scaling Group, using CPU metrics or request-based scaling policies.

---

## 🌐 Networking Design

- **1 VPC** with segregated subnets per environment
- **Public subnets** for ALBs and NAT
- **Private subnets** for EC2 services
- **Internet Gateway** for inbound access
- **NAT Gateway** for EC2 outbound communication

---

## 🔐 Security

- **Security Groups**:
  - ALB → EC2 (via specific port rules)
  - EC2 → MongoDB (private access only)
- **IAM Roles**:
  - Per-service permissions
  - Least-privilege model
- **Secrets Manager**:
  - GitHub token for private repo pulls
  - MongoDB connection strings
  - OpenAI / Ollama API keys

---

## 📦 Deployment Flow

Each environment maps to a GitHub branch:
- `test` → Test environment
- `stage` → Stage environment
- `main` → Production environment

### Deployment Strategy:

- EC2 instances are launched with a **Launch Template** that includes a **User Data script**
- That script:
  - Pulls code from GitHub (branch-based)
  - Installs dependencies
  - Uses environment variables pulled from Secrets Manager
- When deploying updates:
  - Build new AMI (optional)
  - Replace EC2 via ASG refresh OR manually terminate instance

> ❗ No Jenkins or CI/CD tools are used — this is a semi-automated, fully controlled manual deployment process.

---

## 📊 Monitoring & Scaling

- **Horizontal Auto Scaling** for each service based on:
  - CPUUtilization
  - RequestCountPerTarget (via ALB)
- **CloudWatch** monitors all EC2 and ASG metrics
- **MongoDB** is monitored separately (self-hosted or via external tools)

---

## 🧠 Request & Service Routing

1. **User** interacts with Telegram bot
2. **Telegram Service** triggers MCP (controller)
3. **MCP** uses AI Proxy to understand intent
4. **MCP** routes to the correct service (e.g., Gmail or Calendar)
5. **Service** communicates with MongoDB and responds
6. **Reminder & Gmail services** also run background workers

---

## 💽 Database

- A shared **MongoDB** EC2 instance is used across all services
- Auto snapshot backup (can integrate with S3)
- Accessible only via internal network

---

## 🛡️ Load Balancing

Each environment has its own **Application Load Balancer** (ALB):

- ALB handles SSL termination and request routing
- ALB forwards based on path to corresponding microservice ASG

Example:
