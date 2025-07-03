# Production-Grade Microservices Infrastructure on AWS EC2

This repository demonstrates a production-ready, horizontally scalable microservice architecture implemented on AWS EC2. The infrastructure leverages native AWS autoscaling capabilities, implements robust secrets management, and maintains strict environmental isolation—all without external DevOps tooling dependencies.

The architecture prioritizes operational efficiency through strategic infrastructure decisions that optimize for modularity and cost-effectiveness.

---

## Infrastructure Objectives

This infrastructure blueprint demonstrates the implementation of a production-grade cloud architecture using AWS core services, deliberately avoiding external CI/CD and IaC tooling to showcase fundamental cloud-native principles.

Key architectural aspects:

- Independent microservice deployment and scaling mechanisms
- Secure credentials and environment management protocols
- Streamlined deployment workflow utilizing branch-based strategies
- AWS-native service orchestration minimizing external automation dependencies

---

## System Architecture

The infrastructure supports a distributed microservice ecosystem where each service operates within isolated EC2 instance groups, governed by independent scaling policies. Shared components like databases and load balancers provide centralized functionality.

### Core Components

| Component            | Implementation |
|---------------------|----------------|
| Compute Layer       | EC2 (service-specific) |
| Scaling Framework   | Auto Scaling Groups (per-service) |
| Load Distribution   | AWS ALB |
| Secrets Management  | AWS Secrets Manager |
| Environment Control | Branch-based deployment (test/stage/prod) |
| Data Layer          | MongoDB (EC2-hosted, shared) |
| Frontend Layer      | React-based admin interface |
| Process Management  | PM2 for Node.js runtime |
| Version Control     | GitHub (multi-branch strategy) |

---

## Service Architecture

| Service Component   | Functional Domain |
|--------------------|-------------------|
| `auth-service`     | Authentication and session orchestration |
| `gmail-service`    | Gmail API integration and webhook handling |
| `calendar-service` | Google Calendar synchronization |
| `reminder-service` | Scheduled task execution |
| `telegram-bot`     | Telegram webhook endpoint |
| `mcp-service`      | Message routing and tool orchestration |
| `ai-proxy-service` | AI model integration layer |
| `admin-panel`      | Administrative interface |
| `mongodb`          | Centralized data store |

Each service operates within a dedicated EC2 instance or Auto Scaling Group, with independent scaling thresholds based on service-specific metrics.

---

## Deployment Architecture

The deployment workflow follows a deterministic, branch-based strategy:

1. Code deployment paths:
   - `test` → testing environment
   - `stage` → staging environment
   - `main` → production environment

2. Deployment protocol:
   - Controlled EC2 instance termination
   - ASG-triggered instance provisioning
   - Instance initialization sequence:
     - GitHub repository synchronization
     - Secrets Manager credential injection
     - Dependency resolution
     - PM2-managed process initialization

This architecture ensures deployment consistency and security through complete isolation of credentials.

📄 Detailed deployment specifications: [`deployment-strategy.md`](./deployment-strategy.md)

---

## Infrastructure Topology

The system's modular architecture and communication pathways are illustrated below:

> ![Architecture Diagram](./diagram-architecture.png)

📄 Comprehensive architecture documentation: [`architecture.md`](./architecture.md)

---

## Scaling Architecture

- Service-specific Auto Scaling Groups
- CPU and network metrics-driven scaling
- MongoDB horizontal scaling capability
- Automated instance health management

📄 Scaling specifications: [`autoscaling.md`](./autoscaling.md)

---

## Security Architecture

Implemented security controls:

- Private subnet EC2 deployment
- ALB-mediated public traffic
- Restricted EC2 access patterns
- AWS Secrets Manager credential isolation
- Granular IAM role assignments
- VPC-scoped MongoDB access control

📄 Security specifications: [`security.md`](./security.md)

---

## Environment Architecture

| Branch | Environment | Purpose |
|--------|-------------|----------|
| `test` | Testing     | Development validation |
| `stage`| Staging     | Pre-production verification |
| `main` | Production  | Production deployment |

Each environment maintains isolated EC2 resources, Secrets Manager configurations, and ALB routing rules.

---

## Core Capabilities

- ✅ Service-specific horizontal scaling
- ✅ Git-based deployment versioning
- ✅ AWS Secrets Manager integration
- ✅ Centralized MongoDB architecture
- ✅ Single-responsibility compute instances
- ✅ Service isolation principles
- ✅ ALB-managed traffic distribution
- ✅ ASG-based instance lifecycle management

---

## Documentation Index

| Document | Scope |
|----------|--------|
| `architecture.md`        | System topology and design rationale |
| `deployment-strategy.md` | Branch-based environment management |
| `autoscaling.md`        | Scaling policies and failover protocols |
| `security.md`           | IAM framework and security controls |
| `microservices.md`      | Service specifications |
| `user-data-script.md`   | Instance initialization protocols |
| `sops.md`               | Operational procedures |
| `diagram-architecture.drawio.json` | Infrastructure diagram source |
| `env-example.md`        | Environment configuration reference |

---

## Implementation Notes

This infrastructure demonstrates a production-ready deployment architecture using AWS core services. The framework provides a foundation for future enhancement through:

- CI/CD integration
- Infrastructure-as-Code implementation
- Advanced monitoring integration
- Cost optimization strategies

The current implementation maintains manual control to demonstrate infrastructure management principles without external tooling dependencies.

---

## License

MIT
