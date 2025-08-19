# AWS Staging Environment Infrastructure

## Overview
This repository contains CloudFormation templates for deploying a complete staging environment infrastructure on AWS, implementing a nested stack architecture.

## Architecture Components
- VPC with public/private subnets
- Web Application Firewall (WAF)
- Application Load Balancer (ALB)
- Security Groups
- EC2 Launch Template
- RDS Instance
- EC2 Instances

## Prerequisites
- AWS CLI configured
- GitHub account
- S3 bucket for CloudFormation templates
- Required IAM permissions

## Parameter Store Requirements
The following parameters must be configured in AWS Systems Manager Parameter Store:

### Database Parameters
```
/dev/database/rds/instance/identifier
/dev/database/rds/username
/dev/database/rds/password
/dev/database/rds/instance/class
/dev/database/rds/engine/version
```

### Security Parameters
```
/dev/network/vpc/cidr
/dev/network/az
/dev/network/az2
```

### EC2 Parameters
```
/dev/network/key-pair/name
/dev/network/instance/type
/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64
/dev/iam/ec2/role
/app/config/ec2Region
/app/config/accountId
```

## Required AWS credentials
```
vars.main
vars.STACK_NAME
vars.S3_BUCKET_NAME
secrets.AWS_REGION
secrets.AWS_ACCESS_KEY_ID
secrets.AWS_SECRET_ACCESS_KEY
```

## Required IAM Policies
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "cloudformation:CreateStack",
                "cloudformation:UpdateStack",
                "cloudformation:DeleteStack",
                "sns:Publish"
            ],
            "Resource": "*"
        }
    ]
}
```

## GitHub Actions Workflow
`.github/workflows/deploy.yml` handles automatic upload of nested templates to S3.

## Regional Configuration
- Primary Region: us-east-1
- Secondary Region: eu-west-1 (DR)

## Deployment
1. Configure AWS credentials
2. Update parameters in `parameters.json`
3. Deploy root stack:
```bash
aws cloudformation deploy --template-file root.yaml --stack-name staging-env
```

## Security Notes
- All passwords stored in AWS Secrets Manager
- SSL/TLS termination at ALB
- WAF rules for basic protection

## Monitoring
### CloudWatch
- EC2 CPU utilization
- RDS connections
- ALB 5XX errors

### Prometheus & Grafana
- Prometheus for metrics collection
  - Node exporter for system metrics
  - Custom exporters for application metrics
  - Service discovery via EC2 tags
- Grafana dashboards for:
  - Infrastructure metrics
  - Application performance
  - Custom alert rules