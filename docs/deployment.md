
# Deployment

The SLIMTA Inventory Automation Platform uses Infrastructure as Code (IaC), GitHub Actions CI/CD pipelines, and AWS-managed services to support repeatable infrastructure management and application deployments.

The deployment approach separates infrastructure provisioning from application code delivery.

The production implementation is maintained in a private GitHub repository. This public repository documents the deployment architecture and engineering practices without exposing confidential configuration or credentials.

## 1. Deployment Strategy

The platform uses two primary deployment mechanisms:

| Component | Deployment Method |
|-----------|-------------------|
| AWS infrastructure | Terraform |
| Python Lambda application code | GitHub Actions |
| React frontend application | GitHub Actions |
| Frontend static asset storage | Amazon S3 |
| Frontend content delivery | Amazon CloudFront |

Terraform manages AWS resource definitions, while GitHub Actions automates application validation, packaging, and deployment.

This separation allows application code to be updated without unnecessarily recreating infrastructure resources.

## 2. Infrastructure as Code — Terraform

Terraform is used to define and manage AWS infrastructure through version-controlled configuration files.

The infrastructure includes resources supporting:

- AWS Lambda functions
- Amazon DynamoDB tables
- Amazon API Gateway
- Amazon EventBridge scheduling
- Amazon SNS notifications
- AWS IAM roles and permissions
- Supporting application infrastructure

### Terraform Workflow

A typical infrastructure management workflow follows:

```text
Terraform Configuration
        |
        v
terraform fmt
        |
        v
terraform validate
        |
        v
terraform plan
        |
        v
Review Proposed Changes
        |
        v
terraform apply
        |
        v
AWS Infrastructure
```

The workflow above describes the standard Terraform process used for managing infrastructure. It is not presented as a separate automated GitHub Actions infrastructure deployment pipeline.

### Why Terraform?

Terraform provides several engineering benefits:

- Infrastructure definitions are maintained as code.
- Proposed changes can be reviewed before application.
- Resource configuration becomes more repeatable.
- Infrastructure changes can be tracked through version control.
- Manual configuration drift can be reduced through consistent infrastructure management.

Terraform configuration remains in the private implementation repository.

## 3. Backend CI/CD — AWS Lambda

GitHub Actions is used to validate and deploy Python application code.

The backend workflow supports four Lambda functions:

| Lambda Function | Responsibility |
|-----------------|----------------|
| `slimta-process-order` | API requests, order processing, and dashboard backend operations |
| `slimta-trendyol-poller` | Trendyol marketplace order polling and integration |
| `slimta-inventory-sync` | Inventory synchronization and marketplace stock reconciliation |
| `slimta-zoho-order-sync` | Zoho sales order synchronization |

### Backend Deployment Flow

```text
Developer Updates Python Code
             |
             v
        Git Commit
             |
             v
      Push to GitHub
             |
             v
      GitHub Actions
             |
             v
   Python Compilation Check
             |
             v
      Black Formatting
             |
             v
       Ruff Linting
             |
             v
    MyPy Type Checking
             |
             v
   Pytest and Coverage
             |
             v
    Validation Successful?
         /         \
        No         Yes
        |           |
        v           v
   Stop Pipeline   Package Lambda Code
                    |
                    v
             Deploy to AWS Lambda
                    |
                    v
             Deployment Completed
```

### Backend Validation

The workflow includes:

1. **Python compilation checks** to detect syntax errors.
2. **Black** to validate Python formatting.
3. **Ruff** to identify supported linting and code-quality issues.
4. **MyPy** to perform static type analysis.
5. **Pytest** to run automated tests and generate coverage information.

Successful validation is required before the deployment stage proceeds on the main branch.

These checks reduce deployment risk but do not guarantee that every runtime or integration issue will be detected.

### Lambda Deployment

After successful validation, the workflow packages the required Python code and dependencies and updates the supported Lambda functions.

AWS Lambda executes the application using the Python 3.12 runtime.

The deployment process updates application code without requiring a new server or container infrastructure deployment.

## 4. Frontend CI/CD — React Dashboard

The SLIMTA management dashboard is built with React and deployed as static frontend assets.

GitHub Actions automates the frontend build and deployment process.

### Frontend Deployment Flow

```text
Developer Updates React Code
             |
             v
        Git Commit
             |
             v
      Push to GitHub
             |
             v
      GitHub Actions
             |
             v
          npm ci
             |
             v
       npm run build
             |
             v
       Build Successful?
         /         \
        No         Yes
        |           |
        v           v
   Stop Pipeline   Upload Build Assets
                    |
                    v
                  Amazon S3
                    |
                    v
          CloudFront Invalidation
                    |
                    v
             Updated Dashboard
```

### Frontend Deployment Steps

1. GitHub Actions installs project dependencies using `npm ci`.
2. The React production build is generated using `npm run build`.
3. Generated static files are synchronized to the configured Amazon S3 bucket.
4. A CloudFront invalidation is requested so updated assets can be served through the CDN.

The frontend is delivered through Amazon CloudFront and communicates with backend services through Amazon API Gateway.

Frontend linting or automated frontend tests are not claimed as part of this workflow because they have not been verified in the reviewed deployment configuration.

## 5. Infrastructure vs. Application Deployment

Infrastructure provisioning and application deployment are managed as separate concerns.

| Infrastructure Deployment | Application Deployment |
|---------------------------|------------------------|
| Managed with Terraform | Managed with GitHub Actions |
| Creates or modifies AWS resources | Updates Lambda code or frontend assets |
| Uses Terraform plans and state | Uses application build and validation workflows |
| Requires review of infrastructure changes | Requires successful application validation |
| Changes resource configuration | Delivers application updates |

For example, modifying a Lambda function's Python business logic typically requires an application deployment.

Creating a new DynamoDB table or modifying an IAM role generally requires an infrastructure configuration change.

This separation supports more controlled and maintainable cloud operations.

## 6. Configuration and Secrets

The application uses environment-specific configuration and AWS-managed access controls.

AWS Secrets Manager is used for supported integration credentials.

AWS IAM policies control access to AWS resources and services.

Production credentials, tokens, private keys, and sensitive configuration are not included in the public showcase repository.

The deployment documentation intentionally excludes live account identifiers, private endpoints, and confidential operational settings.

## 7. Production Verification and Troubleshooting

Successful CI/CD execution confirms that the configured pipeline steps completed, but it does not automatically prove that every production integration is healthy.

Operational verification can include:

- Checking GitHub Actions workflow results
- Reviewing AWS Lambda execution logs
- Inspecting Amazon CloudWatch logs for runtime errors
- Checking API Gateway request behavior
- Confirming expected records in DynamoDB
- Reviewing dashboard data and integration status
- Investigating persistent system alerts
- Verifying marketplace synchronization when relevant

Production failures may require manual investigation and corrective action.

The platform includes structured logging and operational alerts to support troubleshooting.

## 8. Source Control and Repository Separation

Git is used for source control, and GitHub hosts the application repositories and CI/CD workflows.

The private implementation repository contains:

- Production application source code
- Terraform configuration
- GitHub Actions workflow definitions
- Integration logic
- Automated tests
- Deployment-related configuration

The public showcase repository contains:

- Architecture documentation
- Business problem and solution descriptions
- Sanitized dashboard screenshots
- Deployment and monitoring documentation
- Engineering decisions and project roadmap

This separation allows the project to demonstrate practical AWS and DevOps engineering experience while protecting proprietary business systems.

## 9. Engineering Benefits

The deployment approach provides:

- Repeatable infrastructure management
- Version-controlled application changes
- Automated backend validation
- Reduced manual application deployment steps
- Separation of infrastructure and application responsibilities
- Easier troubleshooting through centralized logs
- A documented deployment process suitable for ongoing maintenance

These practices support operational reliability, although they do not eliminate deployment failures or the need for production verification.

## Conclusion

The SLIMTA deployment approach combines Terraform for infrastructure management with GitHub Actions for backend and frontend application delivery.

It demonstrates practical experience with Infrastructure as Code, Python validation, automated testing, AWS Lambda deployment, React frontend builds, Amazon S3, CloudFront, and production troubleshooting.

For the overall system design, see [Architecture](architecture.md).
