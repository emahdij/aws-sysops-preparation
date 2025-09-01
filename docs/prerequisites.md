# Prerequisites for AWS SysOps Administration

Before diving into the specific AWS services, it's helpful to understand these core concepts:

## Regional vs Global Services

**Regional Services**
- Services that operate within a specific AWS region (e.g., us-east-1)
- Resources in these services exist independently in each region
- Examples: EC2, RDS, Lambda, CloudWatch Metrics/Logs/Alarms (most resources)

**Global Services** 
- Services that operate across all AWS regions
- Resources in these services are available worldwide from a single endpoint
- Examples: IAM, Route 53, CloudFront, WAF
- Note: CloudTrail can monitor both regional and global services

Understanding this distinction is critical for:
- Designing resilient architectures
- Configuring monitoring and compliance
- Understanding data sovereignty requirements
- Implementing disaster recovery strategies

## Resource ARNs and Wildcards

AWS uses Amazon Resource Names (ARNs) to uniquely identify resources. Many AWS CLI commands and configurations require understanding ARN syntax:
arn:partition:service:region:account-id:resource-type/resource-id


For example:
- S3 bucket: `arn:aws:s3:::my-bucket`
- S3 object: `arn:aws:s3:::my-bucket/my-file.txt` 
- EC2 instance: `arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef`

Wildcards (`*`) can be used in many AWS configurations to apply settings to multiple resources:
- `arn:aws:s3:::my-bucket/*` = All objects in a specific bucket
- `arn:aws:ec2:us-east-1:123456789012:*` = All EC2 resources in an account/region

## Shared Responsibility Model

Understanding what AWS is responsible for versus what you're responsible for:

**AWS Responsibility ("Security OF the Cloud")**
- Physical security of data centers
- Hardware and network infrastructure
- Virtualization infrastructure
- AWS services implementation

**Customer Responsibility ("Security IN the Cloud")**
- Data encryption and protection
- Operating system patching
- Network and firewall configuration
- IAM user management and permissions

SysOps administrators must clearly understand these boundaries when configuring, monitoring, and troubleshooting AWS resources.

## Service Quotas

AWS imposes default service limits (quotas) to prevent accidental resource over-provisioning:

- Service quotas vary by region and account
- Many quotas can be increased through support tickets
- Planning for quotas is essential for production workloads
- CloudWatch can monitor and alert on approaching quotas

## Resource Tagging

Tags are key-value pairs attached to AWS resources that provide metadata:

- **Cost Allocation**: Track spending by department, project, or environment
- **Access Control**: Grant permissions based on resource tags
- **Operations**: Filter resources for backup, monitoring, or automation
- **Compliance**: Mark resources with compliance requirements

Consistent tagging strategy is essential for effective resource management.

## AWS CLI Basics

The AWS CLI is the primary tool for SysOps administrators to manage resources programmatically:

- `aws configure` - Set up credentials
- Region selection impacts which resources are managed
- Output formats: JSON, YAML, text, table
- Most examples in this guide use AWS CLI for clarity
