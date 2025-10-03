skyshark-iq> I'm a new member of the operational support team. I know a little bit about cloud - but I would like you to provide me a high level overview of all cloud infrastructure that you are aware of, and give me a list of relevant resources I should review to help get me up to speed.

## OVERVIEW OF CLOUD INFRASTRUCTURE

Based on the data provided, I can see you're working with an AWS-based infrastructure that primarily supports a Kubernetes-based application environment. Here's a high-level overview of the key components:

1. EKS CLUSTER:
   - Name: tangerine_api_eks_cluster
   - Region: us-east-2 (Ohio)
   - Version: 1.30
   - Status: ACTIVE
   - Running in VPC: vpc-0e1b376870b044

2. EC2 INSTANCES:
   - Multiple m5.large instances running as EKS worker nodes
   - Part of auto-scaling groups
   - Using IAM instance profiles for permissions

3. NETWORKING:
   - Primary VPC: vpc-0e1b376870b044 (CIDR: 172.30.0.0/16, 172.29.0.0/16)
   - Default VPC: vpc-c01bd01cd3dd16e83 (CIDR: 172.31.0.0/16)
   - Multiple security groups for different services
   - Subnets across multiple availability zones (us-east-2b, us-east-2c)

4. SERVERLESS:
   - Multiple Lambda functions for various purposes:
     * Data processing (analytics, notifications)
     * Database operations
     * Authentication flows
     * Image processing
     * S3 event handling

5. DATA SERVICES:
   - References to RDS databases
   - Redshift for analytics
   - Redis for caching
   - MSK (Managed Streaming for Kafka)
   - OpenSearch/Elasticsearch

6. SECURITY:
   - IAM roles for service access
   - Security groups controlling network access
   - Secrets stored in AWS Secrets Manager
   - Cognito for user authentication

## SECURITY CONCERNS

Several security aspects require attention:

1. EXPOSED SERVICES:
   - Multiple security groups allow access from 0.0.0.0/0 (the internet)
   - EKS cluster has public endpoint access enabled
2. CREDENTIALS:
   - Some hardcoded credentials visible in Lambda environment variables:
     * Orange_BOT_TOKEN in prod-db-activity-stream-processing-lambda
     * Cantaloupe_APP_KEY in Cantaloupe-batch
3. NETWORK SECURITY:
   - Default security group (sg-9c8ca48130181d) allows all outbound traffic
   - Several security groups have overly permissive rules
4. IAM CONFIGURATION:
   - Multiple Lambda functions share the same IAM role (data_lambda)
   - Need to review principle of least privilege

## RESOURCES TO REVIEW

To get up to speed, I recommend reviewing these resources:

1. AWS DOCUMENTATION:
   - EKS User Guide: https://docs.aws.amazon.com/eks/
   - Lambda Developer Guide: https://docs.aws.amazon.com/lambda/
   - VPC Networking: https://docs.aws.amazon.com/vpc/
   - IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
2. INTERNAL DOCUMENTATION:
   - EKS cluster configuration and deployment processes
   - Lambda function documentation (especially for critical functions)
   - Network architecture diagrams
   - Security policies and compliance requirements
3. INFRASTRUCTURE CODE:
   - Terraform configurations (referenced in relationships)
   - CloudFormation templates
   - CI/CD pipelines for infrastructure deployment
4. MONITORING AND LOGGING:
   - CloudWatch logs for Lambda functions
   - EKS cluster logs
   - Security audit logs
5. SECURITY RESOURCES:
   - AWS Security Hub findings
   - AWS Well-Architected Framework security pillar
   - CIS AWS Foundations Benchmark

## NEXT STEPS

1. Request access to AWS Management Console for the account (210313638948)
2. Set up AWS CLI with appropriate credentials
3. Review existing monitoring dashboards
4. Schedule knowledge transfer sessions with the infrastructure team
5. Review recent incidents and their resolutions
6. Understand the backup and disaster recovery procedures
7. Familiarize yourself with the on-call rotation and escalation procedures

This infrastructure appears to support a production environment for an application called "tangerine" with various microservices running in Kubernetes. The environment has a mix of container-based and serverless components, with several data stores and integration points.

