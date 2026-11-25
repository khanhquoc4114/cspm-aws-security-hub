# Cloud Security Posture Management with AWS Security Hub

A Cloud Security Posture Management (CSPM) project for **NT534 – Advanced Network Security** at the University of Information Technology (UIT – VNU-HCM).

- Demo videos: [YouTube playlist](https://www.youtube.com/playlist?list=PL7IdJecfX87jHfO43NYd6MXL8mBYWBAIf)

## Contents

- [What is CSPM?](#what-is-cspm)
- [Scope](#scope)
- [Repository structure](#repository-structure)
- [AWS Security Hub](#aws-security-hub)
- [Malware detection on EC2 with GuardDuty](#malware-detection-on-ec2-with-guardduty)
- [Automated Security Response on AWS](#automated-security-response-on-aws)
- [References](#references)

## What is CSPM?

Cloud Security Posture Management tools find and fix misconfigurations in cloud environments (IaaS, PaaS and SaaS). They provide:

- **Visibility** into cloud resources and their configuration
- **Continuous monitoring** against security standards
- **Threat detection**
- **Remediation workflows**, manual or automated

## Scope

This project uses AWS Security Hub, and only AWS, to:

1. Run automated security checks that detect misconfigured AWS resources.
2. Remediate the resulting findings automatically with [Automated Security Response on AWS](https://github.com/aws-solutions/automated-security-response-on-aws) (ASR).

Security standards evaluated in Security Hub:

| Standard | Security checks | Automated remediation (ASR v2.0.2) |
|---|---|---|
| CIS AWS Foundations Benchmark v1.2.0 | ✅ | ✅ |
| PCI DSS v3.2.1 | ✅ | ✅ |
| NIST SP 800-53 Rev. 5 | ✅ | ❌ (no playbook in this ASR version) |

The setup uses a multi-account model: one Security Hub **administrator** account and one or more **member** accounts.

## Repository structure

```
.
├── README.md
├── aws-sharr-deploy.template        # ASR v2.0.2 – administrator stack (CloudFormation)
├── aws-sharr-member.template        # ASR v2.0.2 – member account stack
├── aws-sharr-member-roles.template  # ASR v2.0.2 – remediation IAM roles for member accounts
└── securittyHub_detectVirus.json    # Security Hub finding imported from GuardDuty (malware demo)
```

The three `.template` files are the unmodified CloudFormation templates published by AWS Solutions for ASR v2.0.2 (formerly *AWS Security Hub Automated Response & Remediation*, SHARR).

### Deploying the templates

1. **Administrator account:** deploy `aws-sharr-deploy.template` and choose which playbooks to load (AFSBP, CIS 1.2.0, CIS 1.4.0, PCI DSS 3.2.1, Security Control).
2. **Each member account** (including the administrator account if you want it to be remediated):
   - Deploy `aws-sharr-member-roles.template` once per account, and pass the administrator account ID in `SecHubAdminAccount`.
   - Deploy `aws-sharr-member.template` in each Region you want to remediate.

> **Note:** `aws-sharr-deploy.template` sends anonymous usage metrics to AWS by default (`sendAnonymousMetrics: Yes`). Change this mapping before deploying if you don't want that.

## AWS Security Hub

![Security Hub architecture used in this project](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/ad694793-4c02-4fef-bbcf-b086bd882488)

### Key terms

| Term | Meaning |
|---|---|
| **AWS Config rule** | A rule that evaluates whether an AWS resource's configuration matches the desired setting. |
| **Security control** | A Security Hub check, usually backed by an AWS Config rule, that maps to requirements in one or more security standards. |
| **Finding** | A potential security issue that a security check or an integrated service (such as GuardDuty) produces. |
| **Playbook** | A set of remediations for the controls of one security standard. |

### Example security control: EC2.13

| Field | Value |
|---|---|
| Title | Security groups should not allow ingress from 0.0.0.0/0 or ::/0 to port 22 |
| Severity | High |
| AWS Config rule | `restricted-ssh` |
| Parameters | None |

![restricted-ssh Config rule](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/6de87c8c-683a-4613-8fe3-043ec71a15c4)

The control fails if an EC2 security group allows inbound traffic from `0.0.0.0/0` or `::/0` to port 22.

Security groups filter inbound and outbound traffic to AWS resources, and they are stateful. Unrestricted SSH access exposes instances to brute-force attacks and exploitation, so no security group should allow it.

**Remediation:** remove the inbound rule that allows port 22 from `0.0.0.0/0` or `::/0` in every affected security group.

### Security control list

![Security control list](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/1eb301ef-3226-463f-b0e7-7036587cb691)

### Standalone findings

![Standalone findings](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/6ced0d81-b308-48d7-b8ac-fc62a658c64a)

### Standard security check: PCI DSS v3.2.1

![PCI DSS summary](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/acd8d6e3-5ba7-49a9-b3cc-3be057f5a0c9)
![PCI DSS controls](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/9dc4d00b-6aa6-4934-bd11-11440e017edc)
![PCI DSS control details](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/174ae4da-6652-4488-bfff-ab7a21628aa0)

## Malware detection on EC2 with GuardDuty

This demo shows how Security Hub aggregates findings from other AWS security services:

1. Malware samples (from an ELF malware-analysis training set) were placed on an EC2 instance.
2. GuardDuty **Malware Protection for EC2** scanned the instance's EBS volume.
3. GuardDuty reported the detected malware, and Security Hub imported the finding automatically.

![GuardDuty malware scan setup](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/74c783cd-8ae7-4ec2-8ad5-abe1ef2b0300)

### GuardDuty report

![GuardDuty finding](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/2b471a25-41db-4634-bdd0-c5f7f64bbaca)

### The same finding in Security Hub

![Security Hub finding from GuardDuty](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/1af955b1-06a2-4413-a443-456452c1af65)

**Console view**

![Security Hub console](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/43272332-5b7e-4824-b0db-8ea58b643e1c)

**JSON (AWS Security Finding Format)**

The full finding is in [`securittyHub_detectVirus.json`](securittyHub_detectVirus.json). It reports 8 malicious files (6 unique threats, for example `Trojan.GenericKD.43521836`) with HIGH severity.

> **Warning:** do not run real malware samples in a production account. Use an isolated sandbox account and terminate the instance after the test.

## Automated Security Response on AWS

[Automated Security Response on AWS](https://github.com/aws-solutions/automated-security-response-on-aws) is an AWS Solution that remediates Security Hub findings automatically, using AWS Step Functions and AWS Systems Manager (SSM) Automation runbooks.

![ASR architecture](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/4f7d120c-b2b1-4d4d-817b-ed6035206bdb)

### Key terms

| Term | Meaning |
|---|---|
| **Orchestrator** | A Step Functions state machine in the administrator account. It receives findings and starts remediations in the target account. |
| **Control runbook** | An SSM Automation document. It parses a finding for one control and routes it to the right remediation runbook. |
| **Remediation runbook** | An SSM Automation document that performs the steps to fix the finding. |
| **Playbook** | The set of control runbooks for one security standard. |

### Prerequisites

![Prerequisites](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/9c0f3edc-805b-40f7-b1fc-85c723a9eab0)

### Orchestrator workflow

![Orchestrator workflow](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/929db750-3807-4da9-af94-4f7bcf0a9dde)

## Demos

The project has five demos, all available in the [YouTube playlist](https://www.youtube.com/playlist?list=PL7IdJecfX87jHfO43NYd6MXL8mBYWBAIf). Only Demo 1 includes a detailed runbook walkthrough below.

| # | Control | Scenario | Account |
|---|---|---|---|
| 1 | EC2.13 – Security groups should not allow ingress from 0.0.0.0/0 to port 22 | A security group opens port 22 to all IPv4 addresses | Member |
| 2 | IAM.14 – IAM password policy should require at least one number | A weak IAM password policy | Member |
| 3 | RDS.5 – RDS DB instances should be configured with multiple Availability Zones | An RDS DB instance in a single Availability Zone | Administrator |
| 4 | EC2.7 – EBS default encryption should be enabled | EBS encryption by default is disabled and an unencrypted volume exists | Member |
| 5 | S3.1 – S3 general purpose buckets should have block public access settings enabled | Block Public Access is disabled | Member |

### Demo 1: EC2.13 – unrestricted SSH in a member account

- **Input:** a user in the member account creates a security group that allows port 22 from `0.0.0.0/0`, which violates EC2.13.
- **Action:** Security Hub detects the violation and ASR remediates it.
- **Expected results:**
  - A failed EC2.13 finding appears in Security Hub in both the administrator and the member account.
  - An email notification reports the remediation status.
  - The offending inbound rule is removed automatically.

#### Control runbook for EC2.13

<img width="30%" src="https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/9bdfabbc-1838-40e0-8ffd-9a21742e1468" alt="EC2.13 control runbook">

**Purpose:** removes public access to remote administration ports from an EC2 security group.

**Input parameters**

- `Finding`: the Security Hub finding, in JSON.
- `AutomationAssumeRole`: the ARN of the IAM role that the automation assumes.

**Output parameters**

- `Remediation.Output`: the output of the `AWS-DisablePublicAccessForSecurityGroup` runbook.

**Step 1 – ParseInput:** a Python script extracts the security group ID and other details from the finding.

![ParseInput step](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/771555d3-2bdf-4409-aa19-fa0569b97ba4)

**Step 2 – Remediation:** runs the `AWS-DisablePublicAccessForSecurityGroup` runbook with the group ID and the assume-role ARN.

![Remediation step](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/641c5c83-f836-46e0-b25c-46d7fbc2b327)

#### Remediation runbook: AWS-DisablePublicAccessForSecurityGroup

The runbook takes the security group ID and the IP permissions to revoke:

<img width="30%" src="https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/7bbc2023-d095-46cf-bb9d-fa18b7007aa7" alt="Remediation runbook inputs">

It then calls the EC2 `RevokeSecurityGroupIngress` API to remove the public SSH and RDP rules:

![RevokeSecurityGroupIngress](https://github.com/PNg-HA/CSPM-with-AWS-Security-Hub/assets/93396414/6759f7c5-ee19-41f3-8300-ba3d44d816f8)

## References

- [AWS Security Hub User Guide](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
- [Security Hub controls reference](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-controls-reference.html)
- [Automated Security Response on AWS (source)](https://github.com/aws-solutions/automated-security-response-on-aws)
- [GuardDuty Malware Protection for EC2](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection.html)
