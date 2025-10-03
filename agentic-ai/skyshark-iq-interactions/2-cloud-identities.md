skyshark-iq>  Which compute resources have cloud identities that allow reading from databases? Provide a mapping of which cloud identities provide data access, and which workloads use those identities.

I've analyzed the compute resources and their associated IAM roles to identify which ones have database access capabilities. Here's a comprehensive mapping:

## DATA ACCESS ROLES

Several IAM roles appear to provide database access permissions:

a) data_lambda
   - Used by multiple Lambda functions for database operations
   - Provides access to both PostgreSQL and Redshift databases

b) msk_admin
   - Specialized role for Kafka administration
   - Has access to PostgreSQL, Redis, and Redshift

c) cognito_trigger_lambda
   - Used by authentication-related Lambda functions
   - Has access to database resources for user verification

d) prod-db-activity-stream-lambda-role-9720e29
   - Specialized role for database activity monitoring
   - Has access to read database audit logs

e) prod-redshift-serverless-role
   - Provides access specifically to Redshift
   - Used by data processing workloads

## WORKLOADS USING DATABASE ACCESS ROLES

A. LAMBDA FUNCTIONS:
   * db_migration (uses data_lambda)
     - Performs database migrations
     - Accesses PostgreSQL and Redshift
     - Environment variables: POSTGRESDB_ARN, REDSHIFT_SECRET_ARN

   * msk_admin (uses msk_admin role)
     - Manages Kafka clusters
     - Accesses PostgreSQL, Redis, and Redshift
     - Environment variables: MSK_SECRET_ARN, POSTGRESDB_ARN, REDIS_SECRET_ARN, REDSHIFT_SECRET_ARN

   * gift-card-delivery-batch (uses data_lambda)
     - Batch processing for gift card delivery
     - Accesses PostgreSQL and Redshift
     - Environment variables: DB_SECRET_ARN, REDSHIFT_SECRET_ARN

   * Cantaloupe-commission-batch (uses data_lambda)
     - Processes commission data from external API
     - Accesses PostgreSQL database
     - Environment variables: DB_SECRET_ARN

   * post-authentication (uses cognito_trigger_lambda)
     - Runs after user authentication
     - Accesses PostgreSQL and Redis
     - Environment variables: DB_SECRET_ARN, REDIS_SECRET_ARN, CACHE_CLUSTER_SECRET_ARN

   * auth_custom_message (uses cognito_trigger_lambda)
     - Customizes authentication messages
     - Accesses Redis cache
     - Environment variables: REDIS_SECRET_ARN, CACHE_CLUSTER_SECRET_ARN

   * auth_custom_email_sender (uses cognito_trigger_lambda)
     - Sends custom emails for authentication
     - Accesses Redis cache
     - Environment variables: REDIS_SECRET_ARN, CACHE_CLUSTER_SECRET_ARN

   * auth_custom_sms_sender (uses cognito_trigger_lambda)
     - Sends custom SMS for authentication
     - Accesses Redis cache
     - Environment variables: REDIS_SECRET_ARN, CACHE_CLUSTER_SECRET_ARN

   * prod-db-activity-stream-processing-lambda (uses prod-db-activity-stream-lambda-role)
     - Monitors database activity streams
     - Accesses database audit logs
     - Environment variables: DATABASE_RESOURCE_ID

B. EKS CLUSTER NODES:
   * EKS nodes (i-9300e0c5428, i-9010db8b8e)
     - Run in the tangerine_api_eks_cluster
     - Use IAM instance profile: eks-55b79-b021-0e8a-552a5ed4d4d9
     - May have database access through pod service accounts (not directly visible in data)

## SECURITY OBSERVATIONS

1. Most database access is secured through AWS Secrets Manager:
   - Database credentials stored as secrets (db1-dbuser-xh1, redshift-i3Z, etc.)
   - Lambda functions retrieve credentials at runtime

2. Network security:
   - Database-accessing Lambda functions run within VPC (vpc-0e1b376870b044)
   - Security groups control network access (sg-9b42417b310, sg-901a6ddb4d8e, etc.)

3. Potential security concerns:
   - The prod-db-activity-stream-processing-lambda has a Orange token exposed in environment variables
   - Some Lambda functions use older runtimes (nodejs14.x) that may need updating
   - The EKS cluster allows public access (EndpointPublicAccess: true) with open CIDR (0.0.0.0/0)

4. Best practices observed:
   - Use of IAM roles with specific permissions
   - Secrets management for database credentials
   - VPC isolation for database-accessing workloads
   - HTTP tokens required for EC2 instance metadata (HttpTokens: "required")