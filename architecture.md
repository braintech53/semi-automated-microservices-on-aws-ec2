# Infrastructure Architecture Specification

This document details the infrastructure architecture and implementation logic of the AWS EC2-based microservices deployment. The architecture enables horizontal scalability, security isolation, and modular service deployment across multiple environments without traditional CI/CD tooling dependencies.

---

## Architectural Intent

This infrastructure demonstrates production-grade microservice deployment architecture on AWS EC2, maintaining complete deployment control while adhering to cloud-native infrastructure principles and avoiding third-party automation dependencies.

---

## Environment Architecture

The infrastructure implements three isolated deployment environments:

- **Testing Environment**
- **Staging Environment**
- **Production Environment**

Each environment encompasses:

- Dedicated ALB configuration
- Environment-specific security groups
- Service-specific Auto Scaling Groups
- Isolated subnet architecture
- Environment-mapped Git branches
- Route53-managed domain configuration

---

## Service Architecture

Each service operates within its dedicated EC2 instance, managed by an Auto Scaling Group:

1. `Admin Panel (React)`
2. `Auth Service`
3. `Gmail Service`
4. `Calendar Service`
5. `MCP (Master Control Program)`
6. `AI Proxy`
7. `Reminder Service`
8. `Telegram Bot`

Each compute unit maintains independent scaling policies based on service-specific metrics.

---

## Network Architecture

- **VPC Configuration**: Environment-segregated subnets
- **Public Subnet Layer**: ALB and NAT infrastructure
- **Private Subnet Layer**: Service compute instances
- **Internet Gateway**: Inbound traffic management
- **NAT Gateway**: Outbound compute communication

---

## Security Architecture

### Access Control Framework

- **Security Group Configuration**:
  - ALB → EC2 (port-specific routing)
  - EC2 → MongoDB (VPC-scoped access)
- **IAM Framework**:
  - Service-specific role assignments
  - Least-privilege access model
- **Secrets Management**:
  - GitHub authentication tokens
  - MongoDB connection strings
  - External API credentials

---

## Deployment Architecture

Environment-to-Branch Mapping:
- `test` → Testing environment
- `stage` → Staging environment
- `main` → Production environment

### Deployment Protocol

- EC2 provisioning via Launch Template configuration
- Instance initialization sequence:
  - GitHub repository synchronization
  - Dependency resolution
  - Environment configuration injection
- Deployment execution:
  - AMI version control (optional)
  - ASG-managed instance rotation

> Note: This architecture deliberately excludes CI/CD automation to maintain deployment control.

---

## Monitoring Architecture

- **Horizontal Scaling**: Service-specific metrics
  - CPU utilization thresholds
  - ALB request metrics
- **CloudWatch Integration**: EC2 and ASG telemetry
- **MongoDB Monitoring**: Independent metric collection

---

## Request Flow Architecture

1. **User Interaction**: Telegram interface
2. **Request Processing**: Telegram service → MCP
3. **Intent Analysis**: MCP → AI Proxy
4. **Service Routing**: MCP → Target Service
5. **Data Operations**: Service ↔ MongoDB
6. **Background Processing**: Reminder and Gmail services

---

## Data Architecture

- **MongoDB Deployment**: Shared compute instance
- **Backup Protocol**: Automated snapshot management
- **Access Control**: VPC-scoped connectivity

---

## Load Distribution Architecture

Environment-specific Application Load Balancers:

- SSL termination handling
- Service-based request routing
- Health check management

Example Configuration:
```hcl
listener_rule {
  priority = 100
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.auth_service.arn
  }
  condition {
    path_pattern {
      values = ["/auth/*"]
    }
  }
}
```
