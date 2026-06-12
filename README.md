# 🐍 Python AWS Automation Toolkit

> Production-grade boto3 automation scripts for AWS operations — EC2 health monitoring, S3 lifecycle management, IAM privilege auditing, cost reporting, and instance scheduling. Saves 3+ hours/week of manual operational work.

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python)](https://python.org)
[![boto3](https://img.shields.io/badge/boto3-AWS_SDK-FF9900?style=flat-square&logo=amazon-aws)](https://boto3.amazonaws.com)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Last Commit](https://img.shields.io/github/last-commit/sreekar424/python-aws-automation-toolkit?style=flat-square)](https://github.com/sreekar424/python-aws-automation-toolkit)

---

## 🛠️ Scripts

### 1. `ec2_health_checker.py` — Instance Health Monitor
Checks all EC2 instances across regions, identifies unhealthy/stopped instances, sends SNS alert if any fail health checks. Run via EventBridge schedule or cron.

```bash
python ec2_health_checker.py --regions eu-west-1 us-east-1 --notify-sns arn:aws:sns:...
```

### 2. `s3_lifecycle_manager.py` — S3 Lifecycle Policy Enforcer
Scans all S3 buckets, identifies buckets without lifecycle policies, applies standard policy (30d → Infrequent Access, 90d → Glacier, 365d → delete). Prevents storage cost creep.

```bash
python s3_lifecycle_manager.py --dry-run        # Preview changes
python s3_lifecycle_manager.py --apply           # Apply to all buckets missing policy
```

### 3. `iam_privilege_auditor.py` — IAM Over-Privilege Detector
Lists all IAM roles and policies, flags roles with `*:*` actions or `Resource: *`, generates a report of over-privileged identities. Based on AWS least-privilege best practices.

```bash
python iam_privilege_auditor.py --output report_$(date +%Y%m%d).csv
```

### 4. `cost_daily_reporter.py` — AWS Cost Reporter
Pulls 30 days of AWS Cost Explorer data, calculates daily/weekly trends, identifies top spending services, sends formatted Slack report every morning.

```bash
python cost_daily_reporter.py --slack-webhook https://hooks.slack.com/...
```

### 5. `instance_scheduler.py` — Dev Environment Scheduler
Starts/stops EC2 instances based on schedule tags. Dev instances stop at 7pm, start at 8am Mon–Fri. Saves ~60% on dev environment costs.

```bash
python instance_scheduler.py --action stop --tag-key Environment --tag-value dev
```

---

## 📁 Repository Structure

```
python-aws-automation-toolkit/
├── scripts/
│   ├── ec2_health_checker.py
│   ├── s3_lifecycle_manager.py
│   ├── iam_privilege_auditor.py
│   ├── cost_daily_reporter.py
│   └── instance_scheduler.py
├── lambda/
│   ├── health_check_handler.py   # Lambda wrapper for EC2 health check
│   └── cost_report_handler.py    # Lambda wrapper for cost reporting
├── terraform/
│   └── eventbridge_schedule/     # EventBridge rules + Lambda IAM roles
├── tests/
│   ├── test_ec2_health.py
│   └── test_iam_audit.py
├── requirements.txt
└── README.md
```

## 🚀 Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Configure AWS credentials
aws configure  # or use IAM role if running on EC2

# Run health check
python scripts/ec2_health_checker.py --regions eu-west-1

# Run IAM audit
python scripts/iam_privilege_auditor.py --output iam_audit.csv

# Run cost report
python scripts/cost_daily_reporter.py --days 30
```

## 🔒 Required IAM Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {"Effect": "Allow", "Action": ["ec2:Describe*", "ec2:StartInstances", "ec2:StopInstances"], "Resource": "*"},
    {"Effect": "Allow", "Action": ["s3:GetBucketLifecycle", "s3:PutLifecycleConfiguration", "s3:ListAllMyBuckets"], "Resource": "*"},
    {"Effect": "Allow", "Action": ["iam:ListRoles", "iam:ListPolicies", "iam:GetPolicy", "iam:GetPolicyVersion"], "Resource": "*"},
    {"Effect": "Allow", "Action": ["ce:GetCostAndUsage"], "Resource": "*"},
    {"Effect": "Allow", "Action": ["sns:Publish"], "Resource": "*"}
  ]
}
```

## 📖 Lessons Learned

`iam_privilege_auditor.py` found 12 over-privileged roles in a production account that had been accumulating since the account was created. Most were old CI/CD service accounts that had been given `*:*` for "speed" and never tightened. The script now runs weekly and blocks on any new role with wildcard actions.

## 🛠️ Tech Stack

`Python 3.11` `boto3` `AWS SDK` `EC2` `S3` `IAM` `Cost Explorer` `SNS` `EventBridge` `Lambda`

---

*Part of [Sreekar KV's cloud portfolio](https://sreekar424.github.io)*
