# Amazon GuardDuty (amazon-guardduty)
Amazon GuardDuty is an intelligent threat detection service that continuously monitors your AWS accounts, workloads, and data for malicious activity. It uses machine learning, anomaly detection, and integrated threat intelligence to identify and prioritize potential threats to your AWS environment.

**URL:** [https://aws.amazon.com/guardduty/](https://aws.amazon.com/guardduty/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Anomaly Detection, AWS, Compliance, Machine Learning, Monitoring, Security, Threat Detection

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-04-19

## APIs

### Amazon GuardDuty API
The Amazon GuardDuty API provides programmatic access to manage detectors, findings, filters, trusted IP sets, and threat intelligence for continuous threat detection across AWS accounts and workloads.

**Human URL:** [https://aws.amazon.com/guardduty/](https://aws.amazon.com/guardduty/)

#### Tags:

 - Security, Threat Detection, Machine Learning

#### Properties

- [Documentation](https://docs.aws.amazon.com/guardduty/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-guardduty-openapi.yml)
- [GettingStarted](https://aws.amazon.com/guardduty/getting-started/)
- [Pricing](https://aws.amazon.com/guardduty/pricing/)
- [FAQ](https://aws.amazon.com/guardduty/faqs/)
- [APIReference](https://docs.aws.amazon.com/guardduty/latest/APIReference/Welcome.html)
- [JSONSchema](json-schema/guardduty-finding-schema.json)
- [JSONLD](json-ld/amazon-guardduty-context.jsonld)

## Common Properties

- [Portal](https://aws.amazon.com/guardduty/)
- [Documentation](https://docs.aws.amazon.com/guardduty/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/security/tag/amazon-guardduty/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/guardduty/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| Intelligent Threat Detection | Uses ML and anomaly detection to identify threats without manual rule management. |
| Integrated Threat Intelligence | Incorporates curated threat feeds from AWS, CrowdStrike, and Proofpoint. |
| Multi-Account Support | Monitor all accounts in an AWS Organization from a central administrator account. |
| Continuous Monitoring | Analyzes CloudTrail, VPC Flow Logs, DNS logs, and S3 access logs 24/7. |
| Finding Prioritization | Automatically prioritizes findings by severity for efficient response. |
| Malware Protection | Scans EC2 volumes and S3 objects for malware and known threats. |

## Use Cases

| Name | Description |
|------|-------------|
| Account Compromise Detection | Detect compromised AWS credentials and unauthorized API calls. |
| Insider Threat Monitoring | Identify suspicious behavior from privileged or compromised accounts. |
| Cryptocurrency Mining Detection | Detect unauthorized cryptocurrency mining using EC2 or Lambda. |
| Malware Detection | Scan workloads and data for malware and ransomware threats. |
| Data Exfiltration Prevention | Identify unusual data access patterns from S3 buckets. |

## Integrations

| Name | Description |
|------|-------------|
| AWS Security Hub | Send findings to Security Hub for centralized security management. |
| Amazon EventBridge | Trigger automated responses to findings. |
| AWS Organizations | Enable organization-wide for centralized multi-account monitoring. |
| Amazon Detective | Investigate findings in depth for root cause analysis. |
| Amazon Macie | Combine with Macie for comprehensive data security. |

## Artifacts

### OpenAPI

- [Amazon GuardDuty OpenAPI](openapi/amazon-guardduty-openapi.yml)

### JSON Schema

438 schema files in [json-schema/](json-schema/)

### JSON Structure

438 structure files in [json-structure/](json-structure/)

### JSON-LD

- [Amazon GuardDuty Context](json-ld/amazon-guardduty-context.jsonld)

### Examples

438 example files in [examples/](examples/)

## Capabilities

### Shared Per-API Definitions

- [Amazon GuardDuty](capabilities/shared/amazon-guardduty.yaml) — 11 operations for threat detection and finding management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Amazon GuardDuty Threat Detection](capabilities/amazon-guardduty-threat-detection.yaml) | Amazon GuardDuty | 12 | Security Analyst, SOC Engineer, Cloud Security Engineer |

## Vocabulary

- [Amazon GuardDuty Vocabulary](vocabulary/amazon-guardduty-vocabulary.yaml) — Unified taxonomy mapping 6 resources, 7 actions, 1 workflow, and 3 personas

## Rules

- [Amazon GuardDuty Spectral Rules](rules/amazon-guardduty-spectral-rules.yml) — 8 rules enforcing Amazon GuardDuty API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
