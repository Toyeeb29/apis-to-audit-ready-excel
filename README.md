# AWS Security Hub → Audit-Ready Excel Pipeline

> **GRC Engineering Lab** · Serverless security findings automation · AWS · Python · Infrastructure as Code

[![AWS](https://img.shields.io/badge/AWS-Lambda%20%7C%20S3%20%7C%20Security%20Hub-FF9900?style=flat&logo=amazon-aws)](https://aws.amazon.com/)
[![Python](https://img.shields.io/badge/Python-3.9-3776AB?style=flat&logo=python)](https://python.org)
[![IaC](https://img.shields.io/badge/IaC-CloudFormation-FF4F00?style=flat)](https://aws.amazon.com/cloudformation/)
[![Status](https://img.shields.io/badge/Lab-Completed-28a745?style=flat)](#)

---

## What This Project Does

Security and compliance teams live in spreadsheets — but raw AWS Security Hub findings are JSON blobs buried in a console. This project bridges that gap by building a **fully serverless pipeline** that pulls live Security Hub findings and outputs a formatted, audit-ready Excel workbook automatically.

Built and deployed end-to-end as a GRC engineering lab. **21 real Security Hub findings** were extracted and delivered into a multi-worksheet Excel report in a single Lambda invocation.

---

## Why This Exists — The Human Problem

GRC teams love APIs and automation. Audit teams live in Excel. That gap causes a monthly scramble: export raw findings, reformat them, paste into spreadsheets, fix errors, repeat.

This project solves the **human problem**, not just the technical one. Instead of asking auditors to adopt new tools, it automatically delivers findings in the format they already use — Excel — while letting engineers keep their automated workflows.

**For GRC engineers:**
- No more manual export and formatting cycles
- Consistent, reproducible output every time
- Serverless — pay only for what you use, scales to thousands of findings

**For audit teams:**
- Reports open directly in Excel, exactly as expected
- Executive Summary, detailed findings, and pivot analysis in one workbook
- Remediation links embedded — data is actionable, not just informational

**For the organization:**
- Auditors spend time analyzing data instead of formatting it
- Faster audit cycles, fewer back-and-forth emails
- Consistent documentation reduces audit findings related to report quality

> *The best GRC automation doesn't force people to change workflows — it meets them where they are.*

---

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────────┐
│  AWS Security   │───▶│  Lambda Function │───▶│  S3 Bucket           │
│  Hub Findings   │    │  (Python 3.9)    │    │  (Excel Reports)     │
└─────────────────┘    └──────────────────┘    └──────────────────────┘
                              ▲
                    ┌─────────────────┐
                    │  CloudFormation │
                    │  (IaC Deploy)   │
                    └─────────────────┘
```

**AWS Services:**

| Service | Role |
|---|---|
| AWS Security Hub | Centralized findings aggregation source |
| AWS Lambda | Serverless compute — processes and formats findings |
| Amazon S3 | Stores generated Excel reports and Lambda source |
| AWS IAM | Least-privilege role for Lambda execution |
| AWS CloudFormation | Full infrastructure deployed as code |

---

## Lab Walkthrough — What I Actually Did

### 1. Packaged the Lambda Source
```bash
zip -r lambda-source.zip lambda-source/
```

### 2. Provisioned S3 & Uploaded Source
```bash
export SECURITY_REPORT="security-hub-reports-$(date +%s)"
aws s3 mb s3://$SECURITY_REPORT
aws s3 cp lambda-source.zip s3://$SECURITY_REPORT/source/lambda-source.zip
```

### 3. Deployed Infrastructure via CloudFormation
```bash
aws cloudformation deploy \
  --template-file cloudformation-template.yaml \
  --stack-name security-hub-excel-pipeline \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides S3BucketName=$SECURITY_REPORT
```
> `Successfully created/updated stack - security-hub-excel-pipeline`

### 4. Invoked the Lambda & Got Results
```bash
aws lambda invoke \
  --function-name security-hub-excel-generator-cf \
  --output json \
  response.json
```

**Live response:**
```json
{
  "statusCode": 200,
  "body": {
    "message": "Security Hub Excel report generated successfully",
    "bucket": "security-hub-reports-1778547819",
    "key": "reports/security_hub_report_20260512_012424.xlsx",
    "findings_count": 21,
    "worksheets_created": ["Executive Summary", "Detailed Findings", "Pivot Analysis"]
  }
}
```

### 5. Retrieved the Excel Report
```bash
aws s3 cp s3://$SECURITY_REPORT/reports/security_hub_report_20260512_012424.xlsx \
  ./my-security-report.xlsx
```

---

## Output — What the Excel Report Contains

| Worksheet | Contents |
|---|---|
| **Executive Summary** | High-level metrics, severity breakdown, KPIs |
| **Detailed Findings** | Full finding data with resource IDs and remediation links |
| **Pivot Analysis** | Interactive tables for slicing findings by severity, service, and status |

---

## Skills Demonstrated

- **GRC Automation** — Translating raw cloud security data into compliance-ready deliverables
- **Serverless Architecture** — Lambda + S3 event-driven design with no infrastructure to manage
- **Infrastructure as Code** — Full stack provisioned via CloudFormation; zero click-ops
- **AWS CLI Proficiency** — End-to-end deployment, invocation, and retrieval from the terminal
- **Python & openpyxl** — Programmatic Excel generation with conditional formatting and multi-sheet structure
- **Security Posture Reporting** — Bridging the gap between engineering tooling and audit requirements

---

## Project Structure

```
apis-to-audit-ready-excel/
├── cloudformation-template.yaml   # Full IaC stack definition
├── lambda-source/                 # Lambda function source code
│   └── lambda_function.py        # Core findings extraction & Excel logic
├── lambda-source.zip              # Packaged deployment artifact
└── README.md
```

---

## Prerequisites

- AWS CLI installed and configured (`aws sts get-caller-identity` to verify)
- AWS Security Hub enabled with findings in your account
- IAM permissions: Lambda, S3, Security Hub, CloudFormation

---

## Cleanup

```bash
aws cloudformation delete-stack --stack-name security-hub-excel-pipeline
aws s3 rm s3://$SECURITY_REPORT --recursive
aws s3 rb s3://$SECURITY_REPORT
```

---

## The Bigger Picture — Why This Skill Matters

Pulling data from AWS APIs and transforming it into Excel isn't just a technical trick — it's becoming a core GRC engineering competency.

Executives report risk in spreadsheets. External audit firms expect Excel deliverables as standard practice. Compliance teams build control matrices, risk registers, and gap analyses in Excel. Even cyber insurance providers require detailed Excel submissions for policy applications.

GRC engineers who can bridge the technical-business divide — turning raw cloud API data into business-readable formats — become strategic contributors rather than just technical implementers. They reduce manual processing, satisfy both engineering and compliance stakeholders, and communicate fluently in both worlds.

This lab is part of a broader **GRC Engineering** portfolio demonstrating exactly that: automating the repetitive work that slows compliance teams down, and delivering output that auditors can actually use the moment it lands in their inbox.

---

*Deployed and verified on AWS account `461115678308` · Report generated: `20260512_012424` · 21 findings processed*
