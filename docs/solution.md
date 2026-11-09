
# Solution

The SLIMTA Inventory Automation Platform is an AWS-based serverless application developed to support multi-channel e-commerce operations.

It integrates Zoho Inventory, marketplace APIs, AWS automation services, and a centralized React dashboard to improve inventory consistency, order visibility, and operational monitoring.

The solution extends the existing inventory management system rather than replacing it.

## 1. Solution Overview

Zoho Inventory serves as the primary inventory authority, while Amazon DynamoDB provides an operational data layer for the AWS application.

AWS Lambda functions handle inventory synchronization, marketplace integrations, order processing, and API requests.

Amazon EventBridge supports scheduled automation, while Amazon API Gateway exposes backend functionality to the management dashboard.

The dashboard provides consolidated visibility into inventory, orders, sales performance, integration status, and system alerts.

## 2. Core AWS Architecture

The platform uses the following AWS services:

| Service | Responsibility |
|---------|----------------|
| AWS Lambda | Serverless execution of integration and business logic |
| Amazon API Gateway | HTTP API endpoints for the dashboard and supported workflows |
| Amazon DynamoDB | Operational storage for products, inventory, orders, and system alerts |
| Amazon EventBridge | Scheduled execution of synchronization workflows |
| Amazon SNS | Email notifications for supported operational and inventory events |
| Amazon CloudWatch | Application logging and operational troubleshooting |
| AWS Secrets Manager | Secure storage and retrieval of supported integration credentials |
| Amazon S3 | Hosting frontend static assets |
| Amazon CloudFront | Delivery of the React dashboard |
| AWS IAM | Access control between AWS services |

Terraform is used to manage AWS infrastructure as code.

GitHub Actions supports automated validation, build, and deployment workflows.

## 3. Inventory Synchronization

Zoho Inventory is the central source of inventory quantities.

The inventory synchronization workflow retrieves stock information from Zoho and updates the operational inventory records stored in DynamoDB.

The platform also includes marketplace-specific stock synchronization and reconciliation logic for supported integrations, including Trendyol and Noon.

### Inventory Data Flow

1. Zoho Inventory maintains the primary inventory records.
2. A scheduled or manually initiated Lambda workflow retrieves inventory information.
3. The application processes and maps inventory records to supported SKUs.
4. DynamoDB stores inventory information used by the AWS application.
5. Marketplace integration logic updates or reconciles supported marketplace stock quantities.
6. Synchronization results and errors are recorded for monitoring.

This approach provides a centralized operational view while retaining Zoho Inventory as the primary inventory authority.

## 4. Order Processing and Integration

The platform supports different order ingestion workflows depending on the marketplace.

### Amazon and Shopify

Amazon and Shopify sales orders are synchronized from Zoho into DynamoDB through the Zoho order synchronization workflow.

These records are used by the dashboard for order visibility, reporting, and business intelligence.

### Trendyol

A dedicated Trendyol polling Lambda retrieves marketplace orders and processes supported order information.

The integration includes order normalization, SKU mapping, DynamoDB order processing, and Zoho Sales Order creation.

It also contains logic for handling supported order status changes, including cancellations.

### Other Integrations

The platform contains Noon API integration functionality, including inventory synchronization.

Additional order integration code exists, but implemented code is not necessarily equivalent to an enabled production workflow.

## 5. Dashboard and Business Intelligence

The React dashboard provides a centralized interface for business operations.

Its capabilities include:

- Inventory management and stock visibility
- Low-stock monitoring
- Order management and filtering
- Sales and revenue summaries
- Marketplace performance comparisons
- Business intelligence and sales trends
- Integration monitoring
- Persistent operational alerts

The frontend communicates with the AWS backend through Amazon API Gateway.

DynamoDB supplies the operational data used by the dashboard.

## 6. Reliability and Error Handling

Marketplace integrations depend on external APIs, which may experience rate limits, authentication errors, temporary failures, or inconsistent data.

The platform includes reliability mechanisms such as:

- Retry and error-handling logic in supported integration workflows
- HTTP 429 rate-limit handling
- Duplicate order detection
- SKU validation and mapping
- Structured application logging
- Persistent system alerts
- Alert deduplication and occurrence tracking
- Alert resolution and expiration
- Inventory reconciliation checks

Some failures require manual investigation or corrective action.

These mechanisms improve operational visibility and recovery but do not guarantee that all synchronization failures are resolved automatically.

## 7. Infrastructure as Code

Terraform is used to define and manage AWS infrastructure.

Infrastructure as Code helps make resource configuration repeatable, reviewable, and easier to maintain.

The infrastructure includes components such as:

- Lambda functions
- DynamoDB tables
- IAM roles and permissions
- API Gateway configuration
- Scheduled automation resources
- Supporting AWS services

Terraform configuration is maintained in the private implementation repository.

## 8. Continuous Integration and Deployment

GitHub Actions supports CI/CD for backend and frontend components.

### Backend Pipeline

The backend workflow includes:

1. Python code compilation checks
2. Black formatting validation
3. Ruff linting
4. MyPy static analysis
5. Pytest execution and coverage reporting
6. Deployment of supported Lambda functions after successful validation on the main branch

### Frontend Pipeline

The frontend workflow includes:

1. Dependency installation using `npm ci`
2. React application build using `npm run build`
3. Deployment of generated static assets to Amazon S3
4. Amazon CloudFront cache invalidation

These workflows help reduce manual deployment steps and provide automated validation before backend releases.

## 9. Security Approach

The solution uses AWS IAM for service permissions and AWS Secrets Manager for supported integration credentials.

The application separates business data, integration logic, and frontend delivery across AWS components.

The public GitHub showcase intentionally excludes:

- Production source code
- Credentials and authentication tokens
- AWS account identifiers
- Live private API endpoints
- Customer information
- Confidential business and sales records

The public repository is intended to demonstrate architecture, engineering decisions, and implementation experience without exposing the operational environment.

## 10. Amazon SQS Implementation

An earlier Amazon order-processing workflow implemented Amazon SQS and a dead-letter queue.

The SQS queue and its Lambda event source mapping remain deployed.

However, recent queue monitoring did not show message-processing activity during the inspected period.

The currently described Amazon order synchronization workflow uses Zoho as its upstream source.

For this reason, Amazon SQS is documented as an implemented component rather than presented as the active Amazon order ingestion path.

## 11. Business and Engineering Benefits

The platform provides:

- Reduced dependence on manual inventory updates
- Improved inventory consistency through synchronization and reconciliation
- Consolidated operational data across supported sales channels
- Centralized business intelligence and reporting
- Better visibility into integration failures and system health
- Repeatable infrastructure management with Terraform
- Automated validation and deployment through GitHub Actions
- A serverless architecture that avoids managing application servers

These benefits describe the platform's capabilities and intended operational improvements. Specific quantitative outcomes have not been independently established.

## Conclusion

The SLIMTA Inventory Automation Platform demonstrates how AWS serverless services, external API integrations, Infrastructure as Code, CI/CD, and operational monitoring can be combined to address real e-commerce business challenges.

It is a practical implementation of cloud engineering and DevOps concepts in an operating business environment.

For detailed system components and data flows, see [Architecture](architecture.md).
