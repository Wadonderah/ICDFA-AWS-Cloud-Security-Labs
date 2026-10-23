<div align="center">

# Automated Security Group Compliance Enforcement on AWS

### AWS Config • AWS Lambda • Amazon EC2 • IAM • CloudWatch

### Detecting, Auditing, and Automatically Remediating Unauthorized Security Group Changes

**Cloud Security Engineering | DevSecOps | Infrastructure Governance**

---

**Author:** Wadondera A. Collins  
**Role:** ICDFA Trainee | Cloud Security Engineering

</div>

---

## Overview

Maintaining secure network access controls is a critical requirement in modern cloud environments. Unauthorized modifications to Amazon EC2 Security Groups can expose workloads to unnecessary risk, create compliance violations, and increase the attack surface of production infrastructure.

This project demonstrates the implementation of an automated security remediation solution using AWS Config and AWS Lambda. The solution continuously monitors Security Group configurations, detects unauthorized inbound rule changes, and automatically restores approved configurations whenever policy violations occur.

By integrating compliance monitoring, event-driven automation, and serverless remediation, the solution provides continuous governance of network security controls while reducing manual intervention. Security events and remediation activities are recorded in Amazon CloudWatch Logs, providing a complete audit trail for compliance verification.

## Key Capabilities

- Continuous EC2 Security Group monitoring
- Automated configuration drift detection
- Event-driven remediation using AWS Lambda
- Enforcement of approved inbound access policies
- Centralized auditing with CloudWatch Logs
- Real-time compliance validation using AWS Config
- Serverless security operations and incident response

## Business Value

This solution automatically enforces network security standards by ensuring that only approved inbound ports remain accessible. It reduces exposure to security risks, strengthens governance, improves compliance, and minimizes operational overhead.

---

## Solution Architecture

```text
EC2 Security Group
        |
        v
AWS Config
(Continuous Monitoring)
        |
Configuration Change Detected
        |
        v
AWS Config Rule
        |
        v
AWS Lambda Function
        |
Automatic Remediation
        |
        v
Compliant Security Group Configuration
```

---

## Technologies Used

- AWS Config
- AWS Lambda
- AWS Identity and Access Management (IAM)
- Amazon EC2 Security Groups
- Amazon VPC
- Amazon CloudWatch
- Amazon S3
- Python (Boto3)

---

## Implementation Summary

### Task 1: IAM Configuration

- Reviewed the AwsConfigLambdaSGRole permissions.
- Validated permissions for security group management and logging.
- Attached the AWS_ConfigRole managed policy to AwsConfigRole.

### Task 2: AWS Config Setup

- Enabled AWS Config.
- Configured continuous recording.
- Monitored AWS EC2 SecurityGroup resources.
- Configured an S3 delivery channel.

### Task 3: Security Incident Simulation

The LabSG1 security group was intentionally modified by adding:

- HTTP (80)
- HTTPS (443)
- SMTPS (465)
- IMAPS (993)

This simulated configuration drift and a potential security incident.

### Task 4: Config Rule Creation

Created a custom AWS Config rule:

- Rule Name: EC2SecurityGroup
- Trigger: Configuration Changes
- Scope: EC2 Security Groups
- Remediation Engine: AWS Lambda

### Task 5: Automated Remediation Validation

AWS Config detected the unauthorized changes and invoked the Lambda function.

Remediation results:

- SMTPS rule removed
- IMAPS rule removed
- Approved configuration restored
- Security group returned to a compliant state

### Task 6: CloudWatch Verification

Verified remediation actions through CloudWatch Logs.

Audit records confirmed:

- Lambda invocation
- Detection of non-compliant rules
- Automatic removal of unauthorized ports
- Successful remediation workflow execution

---

## Security Benefits

- Continuous compliance monitoring
- Configuration drift prevention
- Automated incident response
- Reduced manual intervention
- Enhanced governance and auditing
- Improved cloud security posture

---

## Skills Demonstrated

### Cloud Security

- Compliance Monitoring
- Security Governance
- Incident Response
- Security Automation

### AWS Services

- AWS Config
- AWS Lambda
- IAM
- Amazon EC2
- Amazon VPC
- Amazon CloudWatch

### DevSecOps

- Event-Driven Architecture
- Automated Remediation
- Infrastructure Security
- Serverless Operations

---

## Validation Checklist

- [x] IAM roles reviewed and configured
- [x] AWS Config enabled
- [x] Security Groups monitored
- [x] Security incident simulated
- [x] Custom AWS Config rule created
- [x] Lambda remediation integrated
- [x] Automatic remediation verified
- [x] CloudWatch logs analyzed
- [x] Compliance successfully restored

---

## Key Takeaways

This project demonstrates how AWS Config and AWS Lambda can be integrated to build an automated compliance and remediation framework. By continuously monitoring Security Group configurations and automatically correcting unauthorized changes, organizations can strengthen cloud security, improve governance, and reduce operational risk.

---

## Disclaimer

This project was completed in a controlled AWS training environment for cloud security education and hands-on practice. Configurations and resource names may differ from production implementations.

---

## Author

**Wadondera A. Collins**  
ICDFA Trainee | Cloud Security Engineering
