## PROMPT 1

skyshark-iq> several alarms have been raised for PublicAPI under the domain api.prod.jointangerine.com - can you tell me which workloads this routes to and provide details that might help our investigation? 

Based on the data provided, I can see that api.prod.jointangerine.com is experiencing alarms. Let me provide details about its routing and infrastructure to help with your investigation.

## ROUTING INFORMATION

api.prod.jointangerine.com is a CNAME record that points to:
k8s-prod-tangerineapi-cb6ca5a-6661704994.us-east-2.elb.amazonaws.com

This is an Application Load Balancer (ALB) with the following characteristics:
- Type: internet-facing
- State: active
- VPC ID: vpc-0e1b376870b044
- Availability Zones: us-east-2a, us-east-2b, us-east-2c

## SECURITY CONFIGURATION

The ALB is secured by two security groups:
- sg-571504044a44a3
- sg-324121996c7285

The ALB exposes traffic to a Kubernetes service: prod.default.tangerineapi

## INFRASTRUCTURE DETAILS

The ALB is running in three subnets:
- subnet-9b1510a0bb19b8 (us-east-2a)
- subnet-984eed4b059997 (us-east-2b)
- subnet-9208d9c7db32e2 (us-east-2c)

## RELATED ENDPOINTS

There's also api2.prod.jointangerine.com which points to the same ALB. This could be a redundant or testing endpoint.

## SSL/TLS CONFIGURATION

There's an ACM validation record:
_938dac5d6263d0bbab2876012e6.api.prod.jointangerine.com
This indicates the domain is using AWS Certificate Manager for SSL/TLS.

## POTENTIAL INVESTIGATION AREAS

1. KUBERNETES WORKLOAD ISSUES:
   - Check the Kubernetes pods/deployments in the "tangerineapi" service
   - Look for pod restarts, OOM errors, or other container issues

2. LOAD BALANCER HEALTH:
   - Check ALB target group health
   - Verify if targets are being marked unhealthy
   - Review ALB access logs for error patterns

3. NETWORK CONNECTIVITY:
   - Verify connectivity between ALB and Kubernetes nodes
   - Check security group rules for proper ingress/egress

4. RESOURCE CONSTRAINTS:
   - Check if the Kubernetes cluster is experiencing resource pressure
   - Look for CPU/memory constraints on nodes

5. APPLICATION ERRORS:
   - Check application logs for the tangerineapi service
   - Look for increased error rates or latency

6. RELATED SERVICES:
   - The environment has several Lambda functions that might interact with the API
   - Check event-processor Lambda which handles SQS events
   - Verify if post-authentication Lambda is functioning correctly

## NEXT STEPS

1. Check CloudWatch metrics for the ALB to identify patterns in the alarms
2. Review Kubernetes logs for the tangerineapi deployment
3. Check for recent deployments or changes to the API service
4. Verify network connectivity and security group configurations
5. Look for unusual traffic patterns or potential DDoS activity

This API appears to be a critical component of your infrastructure as it's publicly exposed and likely handles authentication and core business logic.

---

## PROMPT 2

skyshark-iq> can you tell me about the tangerineapi deployments in eks? 

The tangerine API runs on an Amazon EKS cluster named "tangerine_api_eks_cluster" in the us-east-2 region. This cluster hosts multiple deployments that make up the tangerine API ecosystem. Let me provide a detailed analysis of these deployments.

## CLUSTER OVERVIEW

The EKS cluster (arn:aws:eks:us-east-2:210338948:cluster/tangerine_api_eks_cluster) has the following characteristics:
- Running Kubernetes version 1.30
- Platform version: eks.29
- Created on: 2023-06-28
- Status: ACTIVE
- Authentication mode: CONFIG_MAP
- Endpoint: https://B24678333912637254EBA4.rj0.us-east-2.eks.amazonaws.com

Network configuration:
- VPC ID: vpc-0e376870b044
- Subnets: 3 subnets (subnet-994b8002ebb0dd, subnet-9aa2c114e055b8, subnet-91388d643304cb)
- Security groups: 5 security groups
- Public access: Enabled (0.0.0.0/0)
- Private access: Enabled

## PRODUCTION DEPLOYMENTS

The cluster hosts several deployments in the "prod" namespace, which appear to be microservices that make up the tangerine API:

1. public-api-deployment:
   - 5 replicas
   - Image: 1902377.dkr.ecr.us-east-2.amazonaws.com/tangerine-api
   - Exposed via LoadBalancer (public-api-lb) and NodePort (public-api-service)
   - Resources: 2 CPU, 1Gi memory
   - Health checks on ports 5999

2. internal-transactions-api-deployment:
   - 3 replicas
   - Resources: 500m CPU, 1Gi memory
   - Internal service only (ClusterIP)

3. internal-scheduled-job-deployment:
   - 1 replica
   - Resources: 2 CPU, 4Gi memory
   - Runs scheduled tasks

4. internal-event-bus-deployment:
   - 1 replica
   - Resources: 2 CPU, 1Gi memory
   - Handles event messaging

5. internal-insights-deployment:
   - 2 replicas
   - Resources: 2 CPU, 1Gi memory
   - Added more recently (Jan 2025)

6. internal-category-deployment:
   - 3 replicas
   - Resources: 2 CPU, 1Gi memory

7. internal-marketing-deployment:
   - 2 replicas
   - Resources: 500m CPU, 1Gi memory

8. internal-data-collection-deployment:
   - 2 replicas
   - Resources: 500m CPU, 1Gi memory

9. internal-cookbook-deployment:
   - 3 replicas
   - Resources: 1 CPU, 1Gi memory

10. internal-merchant-selector-data-deployment:
    - 2 replicas
    - Resources: 500m CPU, 1Gi memory

11. internal-chat-deployment:
    - 3 replicas
    - Resources: 1 CPU, 1Gi memory

12. internal-notification-service:
    - Added in November 2024
    - Exposed on port 6010

## SYSTEM DEPLOYMENTS

The cluster also runs several system deployments:

1. coredns (kube-system namespace):
   - DNS service for the cluster
   - 2 replicas

2. aws-load-balancer-controller (kube-system namespace):
   - Manages AWS load balancers
   - 2 replicas

3. metrics-server (kube-system namespace):
   - Collects metrics from cluster nodes and pods
   - 1 replica

4. cert-manager, cert-manager-webhook, cert-manager-cainjector (cert-manager namespace):
   - Manages TLS certificates
   - 1 replica each

5. opentelemetry-operator (opentelemetry-operator-system namespace):
   - Manages observability components
   - 1 replica

6. arc-gha-rs-controller (arc-systems namespace):
   - buildsystem controller
   - 1 replica

## SECURITY OBSERVATIONS

1. Public Exposure:
   - The public-api-deployment is exposed via a LoadBalancer (internet-facing)
   - The EKS cluster has public endpoint access enabled (0.0.0.0/0)

2. Authentication:
   - Using CONFIG_MAP authentication mode rather than IAM
   - Service accounts are used for pod identity

3. Secrets Management:
   - Multiple secret ARNs referenced in environment variables
   - Sensitive configuration stored in ConfigMaps

4. Network Security:
   - Multiple security groups applied to the cluster
   - Private endpoint access is enabled

## BEST PRACTICE RECOMMENDATIONS

1. Authentication:
   - Consider migrating from CONFIG_MAP to IAM authentication for better security
   - Implement OIDC federation for pod identity

2. Network Security:
   - Restrict public endpoint access to specific CIDR blocks instead of 0.0.0.0/0
   - Consider using private endpoints only if feasible

3. Secrets Management:
   - Move sensitive data from ConfigMaps to Kubernetes Secrets or AWS Secrets Manager
   - Use AWS Secrets Manager CSI driver for better secrets handling

4. Resource Management:
   - Implement Pod Disruption Budgets for critical services
   - Consider using Horizontal Pod Autoscalers for services with variable load

5. Logging and Monitoring:
   - Enable EKS control plane logging (currently disabled)
   - Ensure all services have proper liveness and readiness probes

6. Upgrade Strategy:
   - Plan for regular EKS version upgrades
   - The cluster is using extended support policy

7. Container Security:
   - Implement image scanning in the ECR pipeline
   - Consider using Pod Security Policies or Pod Security Standards