skyshark-iq> What resources are exposed to the public internet?

I've analyzed the AWS infrastructure to identify resources exposed to the public internet and traced them back to the underlying workloads. Here's a comprehensive breakdown:

## INTERNET GATEWAYS

Two internet gateways are attached to VPCs, providing direct internet connectivity:
- igw-00aa4d075e (attached to vpc-c01bd01cd3dd16e83)
- igw-0de0623b9 (attached to vpc-0e1b37699e470b044, tagged as "prod.vpc")

## PUBLIC LOAD BALANCERS

1) APPLICATION LOAD BALANCER:
   - DNS: k8s-prod-tangerineapi-da7016db6b-7772816005.us-east-2.elb.amazonaws.com
   - Type: internet-facing
   - Exposes: prod.default.tangerineapi
   - Security Groups: sg-5715066d4044a44a3, sg-324184e20996c7285
   - Runs in public subnets across 3 AZs

2) NETWORK LOAD BALANCERS (internet-facing):
   - 12 NLBs configured as internet-facing, including:
     * k8s-prod-publicap-**********-6084e20.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-03002a95.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-941d6304d.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-866e904.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-c0b32040.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-42d9a0ac340.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-090513d15.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-982029b8.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-191c3ad7.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-0b17d8eda.elb.us-east-2.amazonaws.com
     * k8s-prod-internal-**********-bb5eb0bdb.elb.us-east-2.amazonaws.com
     * k8s-prod-transact-**********-36331637.elb.us-east-2.amazonaws.com

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
     * Maps to k8s-prod-tangerineapi-cca5a-66694.us-east-2.elb.amazonaws.com
     * Protected by WAF (arn:aws:wafv2:us-east-2:630770565891:regional/webacl/regional-waf)
     * TLS certificate: arn:aws:acm:us-east-2:630770565891 x8:certificate/c47a-36a7-7566-31a9c74a44d9c7e3a24f
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
     * Maps to CloudFront distribution c6ya5xm6.cloudfront.net
     * Likely serves static content or data visualization

4) CDN:
   - cdn.prod.jointangerine.com
     * Has ACM validation record pointing to _ed8c2dcb9ec270302e.mxmsxeraca.acm-validations.aws

## SECURITY CONCERNS

1) SECURITY GROUPS WITH OPEN ACCESS:
   - sg-90c0a29659c: Allows HTTP/HTTPS (80/443) from 0.0.0.0/0
   - sg-9b42417b310: Allows outbound HTTP/HTTPS to 0.0.0.0/0
   - sg-93e784ced8: Allows SSH (22) from 0.0.0.0/0 (HIGH RISK)
   - sg-981b9db93bb3: Allows Redis (6379) from 0.0.0.0/0 (HIGH RISK)
   - Multiple load balancer security groups allow specific ports from 0.0.0.0/0

2) NAMING CONCERNS:
   - Several "internal" services are exposed through public endpoints
   - NLBs with "internal" in their name are configured as internet-facing

3) POSITIVE SECURITY MEASURES:
   - WAF protection on the API endpoints
   - TLS 1.3 policy (ELBSecurityPolicy-TLS13-1-2-2021-06)
   - Access logging enabled to S3 bucket aws-elb-logs-02377234234

## RECOMMENDATIONS

1) IMMEDIATE ACTIONS:
   - Review and restrict sg-93e784ced8 (SSH) and sg-981b9db93bb3 (Redis)
   - Verify if "internal" services should be publicly accessible
   - Consider moving internal services behind a VPN or AWS PrivateLink

2) ARCHITECTURE IMPROVEMENTS:
   - Implement proper network segmentation
   - Use API Gateway for public API