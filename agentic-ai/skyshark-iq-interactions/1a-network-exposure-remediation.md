# Remediation prompt example 

This example builds on the output of [./2-network-exposure.md](./2-network-exposure.md). I prompted skyshark-iq to help determine _why_ the NLBs are public, and asked how to fix. It correcly identified that the annotations on the Kubernetes resource are the source of the problem, and provided the steps to take to correct the problem.

# Prompt and completion:

skyshark-iq> You have previously told me about some network load balancers that have "internal" in their name, but they are all public facing. They are managed by the AWS load balancer controller in kubernetes - you have access to the kubernetes Service resources that define these load balancers. Can you give me the required steps I must take to make them internal instead of internet facing, using the AWS load balancer controller?

# Making "Internal" Network Load Balancers Actually Internal

I've analyzed your Kubernetes Services and their corresponding AWS Network Load Balancers. You're correct - despite having "internal" in their names, these load balancers are configured as internet-facing, which creates a security concern.

## Current Configuration Analysis

All your Kubernetes Services in the `prod` namespace have these annotations:
```
service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
service.beta.kubernetes.io/aws-load-balancer-type: external
service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
```

These services are deployed in the EKS cluster `tangerine_api_eks_cluster` and are managed by Helm through the `tangerine-api-prod-release`.

## Required Steps to Make Load Balancers Internal

1) MODIFY SERVICE ANNOTATIONS:
   For each Kubernetes Service, you need to change the `aws-load-balancer-scheme` annotation from "internet-facing" to "internal".

2) APPLY THE CHANGES:
   You can do this by either:
   - Editing the Helm values file and upgrading the release
   - Directly patching each Service resource

Here's the detailed implementation:

=======================================================================

## Option 1: Update via Helm (Recommended)

1. Get your current Helm values:
   ```
   helm get values tangerine-api-prod-release -n prod > current-values.yaml
   ```

2. Edit the values file to update the service annotations:
   Look for the service definitions and change:
   ```yaml
   service:
     annotations:
       service.beta.kubernetes.io/aws-load-balancer-scheme: internal
   ```

3. Apply the updated values:
   ```
   helm upgrade tangerine-api-prod-release [chart-name] -f current-values.yaml -n prod
   ```

## Option 2: Patch Services Directly

For each service that needs to be changed, run:

```bash
kubectl patch service internal-category-lb -n prod --type=merge -p '{"metadata":{"annotations":{"service.beta.kubernetes.io/aws-load-balancer-scheme":"internal"}}}'
```

Repeat for all services:
- internal-category-lb
- internal-notification-lb
- internal-insights-lb
- internal-event-bus-lb
- internal-merchant-selector-data-lb
- internal-card-lb
- internal-data-collection-lb
- internal-marketing-lb
- internal-merchant-lb
- internal-transactions-api-lb
- internal-users-lb
- internal-cookbook-lb
- internal-scheduled-job-lb
- internal-chat-lb

=======================================================================

## Verification Steps

After applying changes:

1. Check that the services have updated annotations:
   ```
   kubectl get service internal-category-lb -n prod -o jsonpath='{.metadata.annotations}'
   ```

2. Verify the AWS Load Balancer scheme has changed:
   ```
   aws elbv2 describe-load-balancers --names k8s-prod-internal-e468c4d88d --query 'LoadBalancers[].Scheme'
   ```
   It should return "internal" instead of "internet-facing"

3. Confirm the DNS name format has changed from:
   `k8s-prod-internal-XXXX.elb.us-east-2.amazonaws.com`
   to something like:
   `internal-k8s-prod-internal-XXXX.elb.us-east-2.amazonaws.com`

## Security Considerations

1. After making these changes, the load balancers will only be accessible from within the VPC or connected networks.

2. Update any DNS records that point to these load balancers.

3. Review and update any security groups attached to these load balancers.

4. Consider implementing AWS WAF or other security controls for any services that still need external access.

5. Ensure your application code and other services can still reach these endpoints using internal DNS names.

This change will significantly improve your security posture by removing unnecessary public exposure of your internal services.
