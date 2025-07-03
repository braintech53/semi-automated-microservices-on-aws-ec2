# Environment Configuration Architecture

This document specifies the environment isolation strategy implemented across test, stage, and production deployments within the AWS EC2-based microservices infrastructure. Each environment operates with dedicated Git branches, AWS Secrets Manager configurations, isolated compute resources, and distinct domain endpoints.

## Environment Topology

| Environment | Operational Context | Branch | Secrets Pattern | Endpoint Pattern |
|------------|-------------------|---------|-----------------|------------------|
| Test | Development validation | test | {service}_env_test | test.domain.com/{service} |
| Stage | Pre-production verification | stage | {service}_env_stage | stage.domain.com/{service} |
| Production | Live deployment | prod | {service}_env_prod | domain.com/{service} |

This pattern maintains consistency across all microservices (e.g., calendar_env_test, ai_proxy_env_stage).

## Secrets Management Architecture

Service initialization sequence retrieves environment-specific variables from AWS Secrets Manager via user-data scripts.

Example mapping:
```
test → calendar_env_test
stage → calendar_env_stage
prod → calendar_env_prod
```

Secrets scope includes:
- OAuth2 credentials (Gmail, Calendar)
- API authentication tokens
- Environment configuration
- Runtime parameters

IAM policies restrict services to environment-specific secret access.

## Version Control Architecture

Source code versioning implements branch-based environment mapping:

| Branch | Deployment Context |
|--------|-------------------|
| test | Rapid iteration environment |
| stage | Quality assurance environment |
| prod | Production environment |

Code promotion flow: feature → test → stage → prod

EC2 instance provisioning automatically synchronizes with environment-appropriate branches.

## Domain Architecture

Environment-specific domain configuration:

- test.domain.com (development traffic)
- stage.domain.com (QA workload)
- domain.com (production traffic)

SSL Implementation:
- AWS Certificate Manager integration
- Environment-specific or shared certificates
- Route 53 or ALB path-based routing

## Compute Resource Architecture

| Component | Test | Stage | Production |
|-----------|------|--------|------------|
| Instance Type | t2.micro | t3.medium | t3.medium + ASG |
| Auto Scaling | Disabled | Optional | Required |
| Launch Template | Static configuration | Stage-specific | Production-optimized |
| Monitoring | Basic | Partial | Full observability |
| Alerting | Disabled | Email notifications | Multi-channel alerts |

## Environment Isolation Framework

Each environment maintains:

- Dedicated source control branches
- Environment-specific secrets
- Independent compute resources
- Isolated domain endpoints
- Segregated network access

Production environment implements additional isolation:
- Private VPC deployment
- Enhanced security controls
- Restricted access patterns