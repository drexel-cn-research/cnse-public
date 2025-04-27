skyshark-iq> What resources are exposed to the public internet?

I've analyzed the AWS infrastructure to identify resources exposed to the public internet and traced them back to the underlying workloads. Here's a comprehensive breakdown:

## INTERNET GATEWAYS

Two internet gateways are attached to VPCs, providing direct internet connectivity:
- igw-00bb5e086f (attached to vpc-d12ce02de4ee27f94)
- igw-0ef1734c0 (attached to vpc-1f2c48700f581c155, tagged as "prod.vpc")

## PUBLIC LOAD BALANCERS

1) APPLICATION LOAD BALANCER:
   - DNS: k8s-prod-tangerineapi-da7016db6b-7772816005.us-east-2.elb.amazonaws.com
   - Type: internet-facing
   - Exposes: prod.default.tangerineapi
   - Security Groups: sg-6826077e5055b55b4, sg-435295f32007d8396
   - Runs in public subnets across 3 AZs

2) NETWORK LOAD BALANCERS (internet-facing):
   - 12 NLBs configured as internet-facing, including:
     * k8s-prod-publicap-**********-7195f30.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-14113ba6.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-a52e7415e.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-978fa15.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-d1c43151.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-53eba1bd451.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-1a1624e26.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-a9313ac9.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-203d4be8.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-1c28e9feb.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-cc6fc1cec.elb.us-east-2.amazonaws.com
     * k8s-prod-transact-**********-47442748.elb.us-east-2.amazonaws.com

## CLOUDFRONT DISTRIBUTIONS

Several CloudFront distributions are exposed to the internet:
- 6yn7.cloudfront.net (connected to datahub.prod.jointangerine.com)
- vfw0on.cloudfront.net
- e2nruth10
- dejmm6y.cloudfront.net
- dwtc87.cloudfront.net
- er451b9v.cloudfront.net
- 446igbg.cloudfront.net
- jy9sac.cloudfront.net
- 92uzoj70

## PUBLIC DOMAINS AND WORKLOADS

1) API ENDPOINTS:
   - api.prod.jointangerine.com & api2.prod.jointangerine.com
     * Maps to k8s-prod-tangerineapi-ddb6b-77705.us-east-2.elb.amazonaws.com
     * Protected by WAF (arn:aws:wafv2:us-east-2:741880676002:regional/webacl/regional-waf)
     * TLS certificate: arn:aws:acm:us-east-2:741880676002 x8:certificate/d58a-47b8-8677-42a0d85b55ead8f4b35f
     * Exposes the following Kubernetes services:
       - internal-selector-data-service (port 6001)
       - internal-service (port 6002)
       - internal-card-service (port 6003)
       - internal-category-service (port 6004)
       - internal-users-service (port 6005)
       - internal-cookbook-service (port 6006)
       - internal-marketing-service (port 6007)
       - internal-chat-service (port 6008)
       - internal-data-collection-service (port 7000)
       - internal-transactions-api-service (port 7001)
       - transactions-consumer-service (port 7002)
       - transactions-http-service (port 8080)
       - public-api-service (port 5999)
       - internal-event-bus-service (port 6009)
       - internal-notification-service (port 6010)
       - internal-insights-service (port 6011)
       - internal-scheduled-job-service (port 8001)

2) WEBHOOK ENDPOINT:
   - hook.prod.jointangerine.com
     * Maps to the same ALB as the API endpoints
     * Exposes transactions-http-service (port 8080)
     * Paths: /transactions and /transactions/

3) DATA HUB:
   - datahub.prod.jointangerine.com
     * Maps to CloudFront distribution d7zb6yn7.cloudfront.net
     * Likely serves static content or data visualization

4) CDN:
   - cdn.prod.jointangerine.com
     * Has ACM validation record pointing to _fe9ad3edfad381413f.nxntxfsdbd.acm-validations.aws

## SECURITY CONCERNS

1) SECURITY GROUPS WITH OPEN ACCESS:
   - sg-01d0b30760d: Allows HTTP/HTTPS (80/443) from 0.0.0.0/0
   - sg-0c53528c421: Allows outbound HTTP/HTTPS to 0.0.0.0/0
   - sg-04f895dfe9: Allows SSH (22) from 0.0.0.0/0 (HIGH RISK)
   - sg-090cacea4cc4: Allows Redis (6379) from 0.0.0.0/0 (HIGH RISK)
   - Multiple load balancer security groups allow specific ports from 0.0.0.0/0

2) NAMING CONCERNS:
   - Several "internal" services are exposed through public endpoints
   - NLBs with "internal" in their name are configured as internet-facing

3) POSITIVE SECURITY MEASURES:
   - WAF protection on the API endpoints
   - TLS 1.3 policy (ELBSecurityPolicy-TLS13-1-2-2021-06)
   - Access logging enabled to S3 bucket aws-elb-logs-13488345345

## RECOMMENDATIONS

1) IMMEDIATE ACTIONS:
   - Review and restrict sg-04fac795dfe9 (SSH) and sg-090c2cea4cc4 (Redis)
   - Verify if "internal" services should be publicly accessible
   - Consider moving internal services behind a VPN or AWS PrivateLink

2) ARCHITECTURE IMPROVEMENTS:
   - Implement proper network segmentation
   - Use API Gateway for public API