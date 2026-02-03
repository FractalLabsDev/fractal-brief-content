# Talkwise AWS Infrastructure Analysis

**Region:** us-east-2
**Repository:** [FractalLabsDev/talkwise-infra](https://github.com/FractalLabsDev/talkwise-infra)
**Analysis Date:** 2026-02-03

---

## Executive Summary

The `talkwise-infra` repository defines a modular Terraform infrastructure supporting **two architecture patterns**:

1. **Scheduled Lambda** - Event-driven tasks (practice-interviews, ruk-scheduler, project-x)
2. **ECS Fargate** - Long-running containerized services (agent-factory)

---

## Core AWS Services

| Category | Services |
|----------|----------|
| **Compute** | Lambda, ECS Fargate |
| **Networking** | VPC, ALB, Security Groups |
| **Scheduling** | EventBridge Scheduler |
| **Containers** | ECR |
| **Secrets** | Secrets Manager, SSM Parameter Store |
| **Security** | WAF, IAM |
| **CI/CD** | CodeDeploy (Blue/Green), GitHub Actions OIDC |
| **Monitoring** | CloudWatch Logs & Metrics |
| **State** | S3 + DynamoDB (Terraform state) |

---

## Network Architecture

### VPC Configuration
- **CIDR:** `10.0.0.0/16`
- **Availability Zones:** 2 (high availability)
- **DNS:** Hostnames and support enabled

### Subnet Layout
| Type | Count | CIDR Size | Features |
|------|-------|-----------|----------|
| Public | 2 | /20 | Auto-assign public IP, IGW route |
| Private | 2 | /20 | NAT Gateway route (optional) |

### Internet Connectivity
- **Internet Gateway:** Public subnet egress
- **NAT Gateway:** Optional, enables private subnet outbound access
- **Elastic IP:** Static addressing for NAT

### Security Groups

**ALB Security Group:**
- Inbound: HTTP (80), HTTPS (443) from anywhere
- Outbound: All traffic

**ECS Security Group:**
- Inbound: Container port (3000) from ALB only
- Outbound: All traffic

---

## Compute Resources

### Lambda Functions (Scheduled Tasks)

| Setting | Value |
|---------|-------|
| Runtime | Node.js 22.x |
| Memory | 128 MB (configurable) |
| Timeout | 30-60 seconds |
| Build | esbuild (TypeScript → JS) |

**Projects:**
- `practice-interviews` - Customer.io trial event processing
- `ruk-scheduler` - Message hub scheduling
- `project-x` - Template project

**Environment Variables (auto-injected):**
- `PROJECT_NAME`
- `ENVIRONMENT`
- `FUNCTION_NAME`

### ECS Fargate (Long-running Services)

| Setting | Value |
|---------|-------|
| Launch Type | Fargate |
| CPU | 512 units |
| Memory | 1024 MB |
| Network Mode | awsvpc |
| Container Port | 3000 |

**Health Check:**
- Endpoint: `/health`
- Interval: 30s
- Unhealthy threshold: 3 failures
- Start period: 60s grace

**Auto-scaling (optional):**
- Min: 1 task
- Max: 4 tasks
- Metrics: CPU/Memory

---

## Load Balancing (ALB)

### Listeners
| Port | Action |
|------|--------|
| 80 (HTTP) | Redirect to HTTPS or forward |
| 443 (HTTPS) | TLS 1.2+ termination |

### Target Groups
- **Blue:** Primary traffic target
- **Green:** Deployment target (blue/green only)

### Health Checks
- Path: `/health`
- Healthy threshold: 2
- Unhealthy threshold: 3
- Interval: 30s

---

## EventBridge Scheduler

### Schedule Types
| Schedule | Expression |
|----------|------------|
| every_3_min | Testing only |
| every_5_min | Testing |
| every_10_min | Testing |
| every_15_min | Production |
| every_30_min | Production |
| hourly | `rate(1 hour)` |
| daily | `cron(0 9 * * ? *)` (9 AM UTC) |
| weekly | `cron(0 9 ? * MON *)` (Monday 9 AM) |

Custom cron expressions and timezone configuration supported.

---

## Security

### WAF Rules (when enabled)
1. **AWS Managed - Common Rule Set** (OWASP Top 10)
2. **AWS Managed - Known Bad Inputs**
3. **AWS Managed - SQL Injection**
4. **Rate Limiting** - 2000 req/5min per IP
5. **Custom Blocks:** `.php`, `phpunit`, `eval-stdin`, directory traversal

### IAM Roles
| Role | Purpose |
|------|---------|
| Lambda Execution | CloudWatch Logs, SSM Parameter Store |
| EventBridge Scheduler | Lambda InvokeFunction |
| ECS Task Execution | ECR pull, Secrets Manager, CloudWatch |
| ECS Task | Runtime secrets access |
| CodeDeploy | ECS deployment management |
| GitHub Actions OIDC | CI/CD without static credentials |

---

## Secrets Management

### AWS Secrets Manager (agent-factory)
- `anthropic-api-key`
- `github-access-token`

### SSM Parameter Store (Lambda projects)
Pattern: `/{project}/{environment}/{key}`

Examples:
- `/practice-interviews/dev/backend-url`
- `/ruk-scheduler/dev/message-hub-url`
- `/project-x/dev/api-key`

---

## CI/CD: CodeDeploy Blue/Green

| Setting | Value |
|---------|-------|
| Deployment Type | BLUE_GREEN |
| Traffic Routing | Immediate after green ready |
| Auto-rollback | On deployment failure |
| Blue Termination | After success (configurable wait) |

---

## Deployed Projects

### 1. practice-interviews
- **Type:** Scheduled Lambda
- **Function:** `pi-customerio-trial-ended-lambda`
- **Purpose:** Customer.io trial event processing

### 2. ruk-scheduler
- **Type:** Scheduled Lambda
- **Purpose:** Message hub scheduling tasks

### 3. agent-factory
- **Type:** ECS Fargate + Full Stack
- **Components:** VPC, ALB, ECS, ECR, CodeDeploy, WAF, GitHub OIDC
- **Secrets:** Anthropic API key, GitHub token

### 4. project-x
- **Type:** Scheduled Lambda (template)
- **Purpose:** New project scaffolding

---

## Storage

### Terraform State
- **Bucket:** `infra-platform-tfstate-427479925030`
- **Encryption:** AES256
- **Versioning:** Enabled
- **Locking:** DynamoDB

### Container Images
- **ECR Repository:** `{project}-{environment}-api`
- **Scanning:** On push
- **Encryption:** AES256

---

## Module Architecture

```
modules/
├── lambda/          # Lambda + auto-build
├── scheduler/       # EventBridge Scheduler
├── iam/            # IAM roles & policies
├── ssm/            # Parameter Store
├── secrets/        # Secrets Manager
├── networking/     # VPC, subnets, SGs
├── ecs/           # ECS Fargate
├── ecr/           # Container registry
├── alb/           # Load balancer
├── codedeploy/    # Blue/Green deployments
└── waf/           # Web Application Firewall
```

---

## Cost Considerations

| Service | Pricing Model |
|---------|---------------|
| Lambda | Pay-per-invocation |
| ECS Fargate | vCPU/memory seconds |
| EventBridge | $0.10/million API calls |
| ALB | Hourly + per LCU |
| NAT Gateway | Hourly + per GB |

---

## Summary

The infrastructure provides a production-ready platform with:

- **Modularity:** Reusable Terraform modules
- **Scalability:** Auto-scaling ECS, serverless Lambda
- **Security:** WAF, least-privilege IAM, private subnets
- **Reliability:** Multi-AZ deployment, health checks
- **CI/CD:** Blue/green deployments, GitHub Actions OIDC
- **Observability:** CloudWatch logging and metrics

---

*Note: This analysis is based on the Terraform configuration in the repository. Actual deployed resources may vary based on environment-specific variables and manual changes.*
