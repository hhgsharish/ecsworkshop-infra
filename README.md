# ECS Workshop Infrastructure

A comprehensive AWS infrastructure setup for deploying a 2-tier microservice application using Amazon ECS (Elastic Container Service) with Fargate, Application Load Balancer, and supporting AWS services.

## Architecture Overview

This infrastructure creates a highly available, scalable, and secure environment for containerized applications across multiple availability zones.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  Internet                                        │
└─────────────────────────────┬───────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         Internet Gateway                                        │
└─────────────────────────────┬───────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    VPC (10.0.0.0/16) - ecs-vpc                                 │
│                                                                                 │
│  ┌─────────────────────────┐              ┌─────────────────────────┐          │
│  │   Public Subnet 1       │              │   Public Subnet 2       │          │
│  │   (10.0.1.0/24)         │              │   (10.0.2.0/24)         │          │
│  │   AZ-1                  │              │   AZ-2                  │          │
│  │                         │              │                         │          │
│  │  ┌─────────────────┐    │              │  ┌─────────────────┐    │          │
│  │  │  NAT Gateway    │    │              │  │                 │    │          │
│  │  │  (EIP)          │    │              │  │                 │    │          │
│  │  └─────────────────┘    │              │  └─────────────────┘    │          │
│  └─────────────────────────┘              └─────────────────────────┘          │
│                              │                          │                       │
│                              └──────────┬───────────────┘                       │
│                                         │                                       │
│                              ┌─────────────────────────┐                       │
│                              │  Application Load       │                       │
│                              │  Balancer (ALB)         │                       │
│                              │  Port 80                │                       │
│                              │  Routes: / → Frontend   │                       │
│                              │         /api* → Backend │                       │
│                              └─────────────────────────┘                       │
│                                         │                                       │
│  ┌─────────────────────────┐              ┌─────────────────────────┐          │
│  │   Private Subnet 1      │              │   Private Subnet 2      │          │
│  │   (10.0.3.0/24)         │              │   (10.0.4.0/24)         │          │
│  │   AZ-1                  │              │   AZ-2                  │          │
│  │                         │              │                         │          │
│  │  ┌─────────────────┐    │              │  ┌─────────────────┐    │          │
│  │  │ ECS Tasks       │    │              │  │ ECS Tasks       │    │          │
│  │  │ Frontend:80     │    │              │  │ Frontend:80     │    │          │
│  │  │ Backend:3000    │    │              │  │ Backend:3000    │    │          │
│  │  │ (Fargate)       │    │              │  │ (Fargate)       │    │          │
│  │  └─────────────────┘    │              │  └─────────────────┘    │          │
│  └─────────────────────────┘              └─────────────────────────┘          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                            Supporting Services                                   │
│                                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐                │
│  │ ECR Repository  │  │ CloudWatch Logs │  │ Service         │                │
│  │ my-app-repo     │  │ /ecs/frontend   │  │ Discovery       │                │
│  │ - frontend:tag  │  │ /ecs/backend    │  │ ecsworkshop.    │                │
│  │ - backend:tag   │  │                 │  │ local           │                │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘                │
│                                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐                │
│  │ ECS Cluster     │  │ IAM Roles       │  │ Security Groups │                │
│  │ my-ecs-cluster  │  │ - Task Exec     │  │ - ALB (Port 80) │                │
│  │ - Frontend Svc  │  │ - Service Role  │  │ - ECS (80,3000) │                │
│  │ - Backend Svc   │  │                 │  │                 │                │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Infrastructure Components

### Network Layer (network-stack.cf.yml)
- **VPC**: Custom Virtual Private Cloud (10.0.0.0/16)
- **Public Subnets**: 2 subnets across different AZs for high availability
  - Public Subnet 1: 10.0.1.0/24 (hosts ALB and NAT Gateway)
  - Public Subnet 2: 10.0.2.0/24 (hosts ALB for redundancy)
- **Private Subnets**: 2 subnets for ECS tasks
  - Private Subnet 1: 10.0.3.0/24 (ECS tasks)
  - Private Subnet 2: 10.0.4.0/24 (ECS tasks)
- **Internet Gateway**: Enables internet access for public subnets
- **NAT Gateway**: Provides outbound internet access for private subnets
- **Route Tables**: Proper routing for public and private traffic

### Application Layer (ecs-stack.cf.yml)
- **ECS Cluster**: Container orchestration platform (my-ecs-cluster)
- **Application Load Balancer**: Routes traffic between frontend and backend
  - Default route (/) → Frontend service
  - API routes (/api*) → Backend service
- **ECS Services**: Manages container deployment and scaling
  - Frontend Service: Runs on port 80
  - Backend Service: Runs on port 3000
- **Task Definitions**: Container specifications using Fargate
- **ECR Repository**: Stores Docker images for both services
- **Service Discovery**: Enables service-to-service communication
- **Security Groups**: Network access control
- **CloudWatch Logs**: Centralized logging with 7-day retention
- **IAM Roles**: Secure access permissions for ECS tasks and services

## Key Features

- **High Availability**: Multi-AZ deployment across 2 availability zones
- **Security**: Private subnets for application workloads, security groups for network isolation
- **Scalability**: ECS Fargate for serverless container management
- **Monitoring**: CloudWatch integration for logs and metrics
- **Service Discovery**: Internal DNS for service communication
- **Load Balancing**: Application Load Balancer with path-based routing
- **Container Registry**: ECR for secure image storage with vulnerability scanning

## Deployment

### Prerequisites
- AWS CLI configured with appropriate permissions
- GitHub repository with AWS credentials stored as secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`

### Automated Deployment
The infrastructure deploys automatically via GitHub Actions when code is pushed to the `main` branch:

1. **Network Stack**: Creates VPC, subnets, gateways, and routing
2. **ECS Stack**: Deploys container services, load balancer, and supporting resources

### Manual Deployment
```bash
# Deploy network stack
aws cloudformation deploy \
  --template-file templates/network-stack.cf.yml \
  --stack-name ecs-network-stack \
  --region us-east-1

# Deploy ECS stack (after network stack completes)
aws cloudformation deploy \
  --template-file templates/ecs-stack.cf.yml \
  --stack-name ecs-app-stack \
  --capabilities CAPABILITY_IAM \
  --region us-east-1 \
  --parameter-overrides \
    VpcId=$(aws cloudformation describe-stacks --stack-name ecs-network-stack --query "Stacks[0].Outputs[?OutputKey=='VpcId'].OutputValue" --output text) \
    PublicSubnet1Id=$(aws cloudformation describe-stacks --stack-name ecs-network-stack --query "Stacks[0].Outputs[?OutputKey=='PublicSubnet1Id'].OutputValue" --output text) \
    PublicSubnet2Id=$(aws cloudformation describe-stacks --stack-name ecs-network-stack --query "Stacks[0].Outputs[?OutputKey=='PublicSubnet2Id'].OutputValue" --output text) \
    PrivateSubnet1Id=$(aws cloudformation describe-stacks --stack-name ecs-network-stack --query "Stacks[0].Outputs[?OutputKey=='PrivateSubnet1Id'].OutputValue" --output text) \
    PrivateSubnet2Id=$(aws cloudformation describe-stacks --stack-name ecs-network-stack --query "Stacks[0].Outputs[?OutputKey=='PrivateSubnet2Id'].OutputValue" --output text)
```

## Project Structure
```
ecsworkshop-infra/
├── templates/
│   ├── network-stack.cf.yml    # VPC and networking resources
│   └── ecs-stack.cf.yml        # ECS cluster and application resources
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD pipeline
└── README.md                   # This file
```

## Outputs
After successful deployment, the ECS stack provides:
- **ALB DNS Name**: Public endpoint for accessing the application
- **ECR Repository URI**: Location for pushing Docker images

## Cost Optimization
- **Fargate**: Pay only for running containers
- **NAT Gateway**: Single NAT Gateway for cost efficiency
- **Log Retention**: 7-day retention to minimize storage costs
- **Resource Sizing**: Minimal CPU/memory allocation (256 CPU, 512 MB memory)

## Security Best Practices
- Private subnets for application workloads
- Security groups with least privilege access
- IAM roles with minimal required permissions
- ECR vulnerability scanning enabled
- No direct internet access for ECS tasks

## Monitoring and Logging
- CloudWatch log groups for each service
- Application Load Balancer access logs
- ECS service and task metrics
- Container insights for detailed monitoring

## Troubleshooting

### Common Issues
- **Network Stack Fails**: Check CloudFormation events for CIDR conflicts or YAML syntax errors
- **ECS Stack Fails**: Ensure network stack is complete and verify IAM permissions
- **GitHub Actions Fails**: Verify AWS credentials are properly configured in repository secrets
- **Tasks Not Starting**: Check that Docker images exist in ECR repository

### Verification Steps
1. Confirm VPC and subnets are created in AWS Console
2. Verify ECS cluster exists with services defined
3. Check ALB is running and target groups are healthy
4. Validate ECR repository is accessible

## Next Steps
1. Deploy application containers to ECR repository
2. Update ECS service desired count to start tasks
3. Configure custom domain and SSL certificate
4. Set up monitoring alerts and dashboards
5. Implement auto-scaling policies