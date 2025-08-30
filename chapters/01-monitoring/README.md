# Chapter 1: Monitoring and Observability

## Overview

Effective monitoring is essential for AWS environments to ensure reliability, performance, and security. This chapter covers AWS monitoring services with a focus on practical implementation.

## 📊 Exam Weight

**Domain 1 (Monitoring, Logging, and Remediation): 20% of exam (≈13 questions)**

Expected question breakdown by service:
- **CloudWatch Metrics**: 4-5 questions (dashboards, custom metrics, agent setup)
- **CloudWatch Logs**: 3-4 questions (log groups, filters, insights)
- **CloudWatch Alarms**: 2-3 questions (thresholds, composite alarms, actions)
- **CloudTrail**: 2-3 questions (API logging, compliance, analysis)
- **AWS Config**: 1-2 questions (configuration compliance, rules)
- **EventBridge**: 1 question (event-driven automation)

## AWS Services Covered

| Service | Primary Use | Status |
|---------|-------------|---------|
| [**01-CloudWatch Metrics**](./notes/01-cloudwatch-metrics.md)| Collection and tracking of metrics, dashboards, agent setup | ✅ Available |
| [**02-CloudWatch Logs**](./notes/02-cloudwatch-logs.md)| Log collection, analysis, and insights | ✅ Available |
| [**03-CloudWatch Alarms**](./notes/03-cloudwatch-alarms.md)| Alerting and automated responses | ✅ Available |
| [**04-CloudTrail**](./notes/04-cloudtrail.md)| AWS API call recording and auditing | 🚧 Coming Soon |
| [**05-AWS Config**](./notes/05-aws-config.md)| Resource inventory, configuration history, and compliance | 🚧 Coming Soon |
| [**06-EventBridge**](./notes/06-eventbridge.md)| Event-driven architectures and automation | 🚧 Coming Soon |
| [**07-Systems Manager**](./notes/07-systems-manager.md)| Operational insights and automation | 🚧 Coming Soon |
| [**08-X-Ray**](./notes/08-x-ray.md)| Application tracing and performance analysis | 🚧 Coming Soon |


### Coming Soon 🚧
- 04-CloudTrail - API auditing and compliance
- 05-AWS Config - Configuration management and compliance
- 06-EventBridge - Event-driven automation
- 07-Systems Manager - Operational insights
- 08-X-Ray - Application performance monitoring
- Labs - Hands-on exercises for practical learning
- Cheatsheets - Quick reference guides

<!-- ## Labs
- [CloudWatch Alarms & Dashboard](./labs/basic/01-alarms-dashboard.md) - Core monitoring setup
- [CloudWatch Logs & Insights](./labs/intermediate/02-logs-insights.md) - Log analysis and filtering -->

<!-- ## Quick Reference
- [CloudWatch Cheatsheet](./cheatsheets/cloudwatch_cheatsheet.md) - CLI commands and thresholds -->

<!-- ## Monitoring Best Practices

- **Full-Stack Monitoring**: Monitor all layers from infrastructure to application
- **Proactive Alerts**: Set thresholds based on business impact
- **Centralized Logging**: Aggregate logs for comprehensive analysis
- **Automated Remediation**: Create automated responses to common issues
- **Cost-Optimized Monitoring**: Balance monitoring depth with cost implications
- **Resource Tagging**: Use consistent tagging to organize and filter monitoring resources -->