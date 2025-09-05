# AWS Config

## Overview

AWS Config provides a detailed view of the configuration of AWS resources in your account. It continuously monitors and records AWS resource configurations, allowing you to audit configuration history, assess compliance with configurations, and trigger automated remediation actions when resources deviate from expected states.

**Regional Service**: AWS Config is primarily a regional service. Configuration recorders, rules, and conformance packs are created and applied within specific regions. However, some aspects like aggregators can provide multi-region and multi-account visibility. For comprehensive coverage, you need to enable AWS Config in each region where you have resources.

> **🔍 CloudTrail vs AWS Config**: While CloudTrail records *who* did *what* and *when* (API calls), AWS Config records *what your resources look like* at a point in time. CloudTrail captures the API call that created a security group, while AWS Config captures the actual configuration of that security group and tracks changes to it over time.

## Exam Relevance

This module covers these Domain 1 tasks:
- Implementing configuration monitoring and compliance strategies
- Evaluating resources against compliance rules
- Setting up automated remediation
- Creating custom Config rules for organization-specific requirements

---

## Key Concepts

### Config Fundamentals

| Concept | Description | SysOps Importance | Example |
|---------|-------------|-------------------|---------|
| **Configuration Item (CI)** | Point-in-time snapshot of resource attributes | Core unit of configuration data | EC2 instance configuration including security groups, AMI, instance type |
| **Configuration History** | Collection of CIs for a resource over time | Enables change tracking and auditing | Track security group rule changes over past 6 months |
| **Configuration Recorder** | Engine that detects and captures resource changes | Required to use AWS Config | Setting up recorder in each region |
| **Configuration Stream** | SNS notifications of configuration changes | Real-time monitoring | Alert when security group rules change |
| **Config Rules** | Conditions for resource configuration compliance | Automated compliance checking | Ensure all EBS volumes are encrypted |
| **Conformance Packs** | Collections of Config rules and remediation actions | Streamlined compliance management | PCI-DSS compliance pack with 30+ rules |
| **Remediation Actions** | Automated fixes for non-compliant resources | Self-healing infrastructure | Auto-encrypt unencrypted S3 buckets |
| **Aggregator** | Multi-account, multi-region view | Enterprise-wide visibility | Single dashboard for all accounts in an organization |

#### Deep Dive into Configuration Items

A Configuration Item contains:
- Resource metadata (ID, type, creation time)
- Configuration snapshot (all settings)
- Relationships with other resources
- Tags
@@ -43,1 +43,0 @@
-- Compliance status for applicable rules
> **🔍 Exam Alert:** The SysOps exam often tests your understanding of which resource attributes are captured in configuration items and how to query this historical data.

### AWS Config Integration Points

| AWS Service | Integration Type | Use Case |
|-------------|------------------|----------|
| **CloudTrail** | Event source | Track who made configuration changes |
| **S3** | Configuration history storage | Long-term configuration history retention |
| **SNS** | Configuration change notifications | Alert on important resource changes |
| **Lambda** | Custom rule evaluation | Build organization-specific compliance rules |
| **Systems Manager** | Remediation actions | Automated fixes for non-compliant resources |
| **Organizations** | Multi-account management | Deploy consistent rules across all accounts |

---

## Service Configuration

### Basic Config Setup

```bash
# Enable AWS Config with basic settings
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/AWS_ConfigRole \  # Define recorder name and IAM role
  --recording-group allSupported=true,includeGlobalResources=true  # Record all resource types including global ones like IAM

# Create S3 bucket for configuration history
aws s3api create-bucket \
  --bucket my-config-bucket \  # Name of bucket to store configuration snapshots
  --region us-east-1  # Region must be specified for bucket creation

# Set up delivery channel to store configuration items
aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=my-config-bucket,configSnapshotDeliveryProperties="{deliveryFrequency=Six_Hours}"  # Define how often to deliver configuration snapshots

# Start the configuration recorder
aws configservice start-configuration-recorder \
  --configuration-recorder-name default  # Enable the recorder to begin monitoring resources
```

### Enabling Managed Config Rules

```bash
# Add a managed rule to check if EBS volumes are encrypted
aws configservice put-config-rule \
  --config-rule name=encrypted-volumes,description="Checks if EBS volumes are encrypted" \  # Rule name and description
  --source="{owner=AWS,sourceIdentifier=ENCRYPTED_VOLUMES}" \  # AWS-managed rule identifier
  --scope="{complianceResourceTypes=[AWS::EC2::Volume]}"  # Only apply to EBS volumes

# Add a managed rule to check for unrestricted SSH access
aws configservice put-config-rule \
  --config-rule name=restricted-ssh,description="Checks if security groups allow unrestricted SSH access" \
  --source="{owner=AWS,sourceIdentifier=RESTRICTED_INCOMING_TRAFFIC}" \
  --inputParameters="{blockedPort1=22}"

---

## CloudTrail and AWS Config: Understanding the Difference

| Feature | CloudTrail | AWS Config |
|---------|-----------|------------|
| **Primary Purpose** | Track API activity and user actions | Track resource configurations and compliance |
| **What it Records** | API calls (who did what and when) | Resource states and configurations (what resources look like) |
| **S3 Storage Content** | JSON log files of API calls | Configuration snapshots, configuration history files |
| **Example** | Record that "user Bob created security group sg-123" | Record the actual configuration of sg-123 (rules, ports, etc.) |
| **Use Case** | Security analysis, audit, compliance | Resource inventory, configuration compliance, change management |

### How They Work Together

CloudTrail and AWS Config complement each other:
1. CloudTrail records the API call that creates or modifies a resource
2. AWS Config records the resulting configuration of that resource
3. Together, they provide a complete picture of what changed, who changed it, and the resulting state

```bash
# Example: Investigating a security group change
# Step 1: Use AWS Config to see what changed in the security group
aws configservice get-resource-config-history \
  --resource-type AWS::EC2::SecurityGroup \  # Resource type to investigate
  --resource-id sg-01234567890abcdef \  # Specific security group ID
  --earlier-time 2023-01-01T00:00:00Z \  # Starting point for history
  --later-time 2023-01-02T00:00:00Z  # Ending point for history

# Step 2: Use CloudTrail to find who made the change
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceName,AttributeValue=sg-01234567890abcdef  # Find events affecting this resource
```

---

## Implementation Examples

### Terraform Configuration

```hcl
# Enable AWS Config
resource "aws_config_configuration_recorder" "main" {
  name      = "aws-config"
  role_arn  = aws_iam_role.config.arn
  
  recording_group {
    all_supported                 = true  # Record all supported resource types
    include_global_resource_types = true  # Include global services like IAM
  }
}

# S3 bucket for configuration history
resource "aws_s3_bucket" "config" {
  bucket        = "config-bucket-${data.aws_caller_identity.current.account_id}"  # Create unique bucket name
  force_destroy = true  # Allow Terraform to delete even if not empty
}

# Set up delivery channel
resource "aws_config_delivery_channel" "main" {
  name           = "aws-config"
  s3_bucket_name = aws_s3_bucket.config.bucket
  
  snapshot_delivery_properties {
    delivery_frequency = "Six_Hours"  # How often to capture configuration snapshots
  }
  
  depends_on = [aws_config_configuration_recorder.main]  # Ensure recorder exists first
}

# Start the configuration recorder
resource "aws_config_configuration_recorder_status" "main" {
  name       = aws_config_configuration_recorder.main.name
  is_enabled = true  # Set to active/recording state
  
  depends_on = [aws_config_delivery_channel.main]  # Ensure delivery channel exists first
}

# Add managed rules
resource "aws_config_config_rule" "encrypted_volumes" {
  name        = "encrypted-volumes"
  description = "Checks if EBS volumes are encrypted"
  
  source {
    owner             = "AWS"  # AWS-managed rule
    source_identifier = "ENCRYPTED_VOLUMES"  # Specific rule identifier
  }
  
  scope {
    compliance_resource_types = ["AWS::EC2::Volume"]  # Only apply to volumes
  }
  
  depends_on = [aws_config_configuration_recorder_status.main]  # Ensure recorder is active
}
```

### Creating a Custom Config Rule with Lambda

import json
import boto3
from typing import Tuple

config = boto3.client("config")
REQUIRED_TAGS = ["Environment", "Project", "Owner"]

def evaluate_compliance(ci: dict) -> Tuple[str, str]:
    if ci["resourceType"] != "AWS::EC2::Instance":
        return "NOT_APPLICABLE", "Rule targets EC2 instances only"
    # Tags may appear in ci['tags'] (map) or ci['configuration']['tags'] (list)
    tag_map = ci.get("tags") or {}
    if isinstance(tag_map, dict):
        keys = set(tag_map.keys())
    else:
        tags = ci.get("configuration", {}).get("tags", []) or []
        keys = {t.get("key") or t.get("Key") for t in tags}
    missing = [t for t in REQUIRED_TAGS if t not in keys]
    if missing:
        return "NON_COMPLIANT", f"Missing tags: {', '.join(missing)}"
    return "COMPLIANT", "All required tags present"

def lambda_handler(event, _context):
    invoking_event = json.loads(event["invokingEvent"])
    ci = invoking_event["configurationItem"]
    compliance_type, annotation = evaluate_compliance(ci)
    config.put_evaluations(
        Evaluations=[{
            "ComplianceResourceType": ci["resourceType"],
            "ComplianceResourceId": ci["resourceId"],
            "ComplianceType": compliance_type,
            "Annotation": (annotation or "")[:256],
            "OrderingTimestamp": ci["configurationItemCaptureTime"],
        }],
        ResultToken=event["resultToken"],
    )
    return {"compliance_type": compliance_type}

---

## Advanced Features

### Conformance Packs

Conformance packs are collections of AWS Config rules and remediation actions deployed as a single entity:

```bash
# Deploy an operational best practices conformance pack
aws configservice put-conformance-pack \
  --conformance-pack-name "operational-best-practices" \  # Name for the conformance pack
  --template-s3-uri "s3://aws-configservice-conformancepack-us-east-1/operational-best-practices-for-aws-identity-and-access-management.yaml" \  # Pre-built template location
  --delivery-s3-bucket "my-config-bucket"  # Where to store results
```

### Automated Remediation

AWS Config can automatically remediate non-compliant resources:

```hcl
# Terraform configuration for auto-remediation
resource "aws_config_remediation_configuration" "s3_bucket_public_read_prohibited" {
  config_rule_name = aws_config_config_rule.s3_bucket_public_read_prohibited.name  # Rule to attach remediation to
  target_type      = "SSM_DOCUMENT"  # Use Systems Manager for remediation
  target_id        = "AWS-DisableS3BucketPublicReadWrite"  # Pre-built remediation action
  
  parameter {
    name           = "AutomationAssumeRole"  # Role SSM will assume
    static_value   = aws_iam_role.remediation.arn
  }
  
  parameter {
    name           = "S3BucketName"  # Target bucket from the non-compliant resource
    resource_value = "RESOURCE_ID"  # Dynamically filled with non-compliant resource
  }
  
  automatic        = true  # Run automatically when non-compliance detected
  maximum_automatic_attempts = 3  # Retry up to 3 times
  retry_attempt_seconds      = 60  # Wait 60 seconds between retries
}
```

### Multi-Account, Multi-Region Aggregation

```hcl
# Set up AWS Config Aggregator
resource "aws_config_configuration_aggregator" "organization" {
  name = "organization-aggregator"  # Name for the aggregator
  
  organization_aggregation_source {
    role_arn    = aws_iam_role.config_aggregator.arn  # Role with permissions to collect data
    all_regions = true  # Collect from all regions
  }
}
```

---

## Best Practices

### Implementation Guidance
- Enable Config in all regions where you have resources
- Include global resources in at least one region
- Use conformance packs for industry-specific compliance requirements
- Implement automated remediation for critical non-compliant resources
- Set up aggregators for organization-wide visibility

### Cost Optimization

AWS Config charges for:
- Configuration items recorded ($0.003 per item)
- Config rules evaluations ($0.001 per evaluation)
- Conformance pack rules ($0.0045 per conformance pack rule)

**Real-world cost example:**
- 100 AWS resources being monitored
- 20 resources change daily (600 configuration items per month)
- 5 Config rules evaluating all resources daily
  
**Monthly cost breakdown:**
- Configuration items: 600 × $0.003 = $1.80
- Rule evaluations: 5 rules × 100 resources × 30 days × $0.001 = $15.00
- Total: $16.80/month

> **Note:** First 100,000 configuration items per month are free in each account/region.

Optimize costs by:
- Being selective about which resource types to record
- Using appropriate evaluation frequencies for rules
---

## Monitoring and Troubleshooting

### Common Issues and Solutions

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Missing configuration items** | Resources not showing up in inventory | Verify resource types are supported and included in recording group |
| **Rules showing "Insufficient Data"** | Rules not evaluating | Check IAM permissions, Lambda function for custom rules |
| **Recording stops working** | No new configuration items | Check IAM role permissions, S3 bucket permissions |
| **Remediation failures** | Auto-remediation not working | Check remediation role permissions, SSM document compatibility |
| **High AWS Config costs** | Unexpected charges | Review recording group settings, rule evaluation frequency |

### Debugging Steps

```bash
# Check configuration recorder status
aws configservice describe-configuration-recorder-status  # Verify recorder is active and healthy

# List existing rules and their compliance status
aws configservice describe-compliance-by-config-rule  # See compliance results across all rules

# Check why a specific resource is non-compliant
aws configservice get-compliance-details-by-resource \
  --resource-type AWS::EC2::SecurityGroup \  # Type of resource to check
  --resource-id sg-01234567890abcdef  # ID of the specific resource
```

---

## Exam Tips

> **🎯 Exam Focus Areas:**
> - Understanding configuration items and their contents
> - Deploying and managing Config rules
> - Implementing auto-remediation
> - Integrating Config with other AWS services
> - Multi-account Config strategies

### Key Exam Scenarios

1. **Compliance Monitoring**: Identify which AWS resources are non-compliant with organizational policies
2. **Change Tracking**: Determine who made a specific configuration change and when
3. **Resource Relationships**: Understand dependencies between resources for impact analysis
4. **Remediation**: Automatically fix non-compliant resources
5. **Multi-Account Management**: Implement Config across multiple accounts in an organization

---

## References

- [AWS Config User Guide](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
- [AWS Config Managed Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
- [AWS Config API Reference](https://docs.aws.amazon.com/config/latest/APIReference/Welcome.html)