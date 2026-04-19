# Amazon Security Lake (amazon-security-lake)
Amazon Security Lake is a service that automatically centralizes an organization's security data from cloud, on-premises, and custom sources into a purpose-built data lake stored in your own Amazon S3. It manages the data lifecycle to help you optimize storage and supports OCSF (Open Cybersecurity Schema Framework) for normalized security data analysis.

**URL:** [Visit Amazon Security Lake](https://aws.amazon.com/security-lake/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Data Lake, Security, SIEM, Threat Detection

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### Amazon Security Lake API
The Amazon Security Lake API provides programmatic access to create and manage data lakes, data sources, subscribers, and log sources for centralizing and analyzing security data across your organization using the OCSF (Open Cybersecurity Schema Framework).

**Human URL:** [https://docs.aws.amazon.com/security-lake/latest/APIReference/Welcome.html](https://docs.aws.amazon.com/security-lake/latest/APIReference/Welcome.html)

#### Tags:

 - Data Lake, Security, Threat Detection, OCSF

#### Properties

- [Documentation](https://docs.aws.amazon.com/security-lake/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-security-lake-openapi.yml)
- [JSONSchema - DataLake](json-schema/amazon-security-lake-data-lake-schema.json)
- [JSONSchema - LogSource](json-schema/amazon-security-lake-log-source-schema.json)
- [JSONSchema - Subscriber](json-schema/amazon-security-lake-subscriber-schema.json)

## Common Properties

- [Portal](https://aws.amazon.com/security-lake/)
- [GettingStarted](https://aws.amazon.com/security-lake/getting-started/)
- [Documentation](https://docs.aws.amazon.com/security-lake/)
- [APIReference](https://docs.aws.amazon.com/security-lake/latest/APIReference/)
- [Console](https://console.aws.amazon.com/securitylake/)
- [Pricing](https://aws.amazon.com/security-lake/pricing/)
- [FAQ](https://aws.amazon.com/security-lake/faqs/)
- [Blog](https://aws.amazon.com/blogs/security/tag/amazon-security-lake/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Support](https://aws.amazon.com/premiumsupport/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [GitHubOrganization](https://github.com/aws)
- [YouTube](https://www.youtube.com/user/AmazonWebServices)
- [StackOverflow](https://stackoverflow.com/questions/tagged/amazon-security-lake)

## Features

| Name | Description |
|------|-------------|
| Automatic Data Centralization | Automatically centralizes security data from AWS services, third-party tools, and custom sources into a single data lake. |
| OCSF Normalization | Converts security data to the Open Cybersecurity Schema Framework (OCSF) for standardized analysis across tools. |
| Apache Parquet Format | Stores all security data in Apache Parquet format optimized for analytical query performance. |
| Multi-Account Support | Centralizes security data across an entire AWS Organization from all accounts and regions. |
| Lifecycle Management | Automatically manages storage lifecycle with configurable retention and tiering policies. |
| Subscriber Access | Grant third-party SIEMs and analytics tools direct query access to your security data lake. |
| Native AWS Integration | Native connectors for CloudTrail, VPC Flow Logs, Route 53, Security Hub, and EKS audit logs. |
| Custom Log Sources | Ingest custom and third-party security data sources in OCSF format. |

## Use Cases

| Name | Description |
|------|-------------|
| Security Data Centralization | Aggregate all security data from across a multi-account AWS environment into one queryable data lake. |
| SIEM Integration | Provide SIEM platforms like Splunk, Sumo Logic, and Microsoft Sentinel direct access to normalized security data. |
| Threat Hunting | Enable security analysts to query normalized OCSF data for threat hunting and forensic investigation. |
| Compliance Data Retention | Retain security logs in a cost-optimized data lake for compliance audit requirements. |
| Security Analytics | Run advanced analytics and ML models against normalized security data for anomaly detection. |
| Multi-Cloud Security Data | Centralize security data from on-premises and other cloud providers alongside AWS security data. |

## Integrations

| Name | Description |
|------|-------------|
| AWS CloudTrail | Native connector for management event and data event logs from CloudTrail. |
| Amazon VPC Flow Logs | Ingest VPC network flow logs for network traffic analysis. |
| Amazon Route 53 | Collect DNS query logs for domain analysis and threat detection. |
| AWS Security Hub | Aggregate Security Hub findings into the security data lake. |
| Amazon EKS | Ingest Kubernetes audit logs from Amazon EKS clusters. |
| Amazon S3 | All security data is stored in S3 buckets within your own AWS account. |
| AWS Lake Formation | Control fine-grained subscriber access using AWS Lake Formation permissions. |
| Splunk | SIEM subscriber integration for Splunk to query Security Lake data directly. |
| Microsoft Sentinel | Connect Microsoft Sentinel as a subscriber to consume OCSF-normalized data. |
| CrowdStrike | Ingest CrowdStrike endpoint detection findings as a custom log source. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon Security Lake API](openapi/amazon-security-lake-openapi.yml)

### JSON Schema

- [DataLake](json-schema/amazon-security-lake-data-lake-schema.json)
- [LogSource](json-schema/amazon-security-lake-log-source-schema.json)
- [Subscriber](json-schema/amazon-security-lake-subscriber-schema.json)

### JSON Structure

- [DataLake](json-structure/amazon-security-lake-data-lake-structure.json)
- [LogSource](json-structure/amazon-security-lake-log-source-structure.json)
- [Subscriber](json-structure/amazon-security-lake-subscriber-structure.json)

### JSON-LD

- [Amazon Security Lake Context](json-ld/amazon-security-lake-context.jsonld)

### Examples

- [DataLake Example](examples/amazon-security-lake-data-lake-example.json)
- [LogSource Example](examples/amazon-security-lake-log-source-example.json)
- [Subscriber Example](examples/amazon-security-lake-subscriber-example.json)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Amazon Security Lake API](capabilities/shared/amazon-security-lake.yaml) — 9 operations for data lake, log source, and subscriber management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Security Data Lake](capabilities/security-data-lake.yaml) | Amazon Security Lake | 10 | Security Data Engineer, CISO |

## Vocabulary

- [Amazon Security Lake Vocabulary](vocabulary/amazon-security-lake-vocabulary.yaml) — Unified taxonomy mapping 5 resources, 5 actions, 1 workflow, and 2 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon Security Lake Spectral Rules](rules/amazon-security-lake-spectral-rules.yml) — 21 rules across 9 categories enforcing Amazon Security Lake API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
