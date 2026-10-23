<h1 align="center">Self-Healing Cloud Security Compliance Framework on AWS</h1>

<p align="center"><strong>Continuous Compliance Monitoring, Automated Incident Response, and Security Group Remediation</strong></p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Amazon%20Web%20Services-232F3E?style=for-the-badge&amp;logo=amazonaws&amp;logoColor=white" alt="Amazon Web Services platform badge">
  <img src="https://img.shields.io/badge/Domain-Cloud%20Security-0B74DE?style=for-the-badge&amp;logo=securityscorecard&amp;logoColor=white" alt="Cloud Security domain badge">
  <img src="https://img.shields.io/badge/Status-Completed-2EA44F?style=for-the-badge&amp;logo=checkmarx&amp;logoColor=white" alt="Completed status badge">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AWS%20Config-Continuous%20Compliance-146EB4?style=for-the-badge&amp;logo=amazonaws&amp;logoColor=white" alt="AWS Config continuous compliance badge">
  <img src="https://img.shields.io/badge/AWS%20Lambda-Automated%20Remediation-FF9900?style=for-the-badge&amp;logo=awslambda&amp;logoColor=white" alt="AWS Lambda automated remediation badge">
  <img src="https://img.shields.io/badge/CloudWatch-Audit%20Logging-C925D1?style=for-the-badge&amp;logo=amazoncloudwatch&amp;logoColor=white" alt="CloudWatch audit logging badge">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-Boto3-3776AB?style=for-the-badge&amp;logo=python&amp;logoColor=white" alt="Python and Boto3 badge">
  <img src="https://img.shields.io/badge/DevSecOps-Event--Driven%20Security-0F172A?style=for-the-badge" alt="DevSecOps event-driven security badge">
  <img src="https://img.shields.io/badge/Compliance-Automated-2EA44F?style=for-the-badge" alt="Automated compliance badge">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Security-Incident%20Response-D13212?style=for-the-badge" alt="Security incident response badge">
  <img src="https://img.shields.io/badge/Architecture-Serverless-FF6A00?style=for-the-badge" alt="Serverless architecture badge">
  <img src="https://img.shields.io/badge/Infrastructure-Self%20Healing-146EB4?style=for-the-badge" alt="Self-healing infrastructure badge">
</p>

<p align="center">
  <strong>Cloud Security Engineering | DevSecOps | Infrastructure Governance</strong><br>
  <strong>Author:</strong> Wadondera A. Collins<br>
  <strong>Program:</strong> ICDFA Trainee | Cohort 11<br>
  <strong>Focus:</strong> AWS Security Automation and Compliance Enforcement
</p>

---

## Table of Contents

- [Project Overview](#project-overview)
- [Executive Summary](#executive-summary)
- [Professional Value](#professional-value)
- [Project Objectives](#project-objectives)
- [Solution Architecture](#solution-architecture)
- [Technology Stack](#technology-stack)
- [Lab Environment](#lab-environment)
- [Security Validation Scenario](#security-validation-scenario)
- [Technical Execution](#technical-execution)
- [Automated Remediation Workflow](#automated-remediation-workflow)
- [Results and Validation](#results-and-validation)
- [Core Competencies Demonstrated](#core-competencies-demonstrated)
- [Security Considerations](#security-considerations)
- [Limitations and Next Steps](#limitations-and-next-steps)
- [Conclusion](#conclusion)
- [Disclaimer](#disclaimer)
- [Author](#author)

---

## Project Overview

This project presents a cloud-native security automation framework that continuously monitors Amazon EC2 Security Group configurations and automatically remediates unauthorized inbound-rule changes.

The solution uses AWS Config to identify configuration drift, a custom compliance rule to evaluate monitored resources, AWS Lambda to restore approved network controls, IAM to enforce service permissions, and Amazon CloudWatch Logs to preserve an auditable record of remediation activity.

Rather than relying on manual review, the framework establishes a repeatable, event-driven control that detects security-policy violations and returns affected resources to a compliant state.

> [!IMPORTANT]
> The project demonstrates how AWS-native services can be integrated to create a self-healing security control for continuous compliance enforcement.

---

## Executive Summary

Unauthorized Security Group changes can expose cloud workloads, increase attack surface, and create compliance gaps. This project addresses that risk by implementing an automated control that monitors EC2 Security Groups and removes unapproved inbound permissions.

To validate the control, additional inbound access for SMTPS on TCP port 465 and IMAPS on TCP port 993 was intentionally introduced into a monitored Security Group. AWS Config detected the configuration change and invoked a Lambda function that removed the unauthorized rules and restored the approved HTTP and HTTPS configuration. CloudWatch Logs provided evidence of the remediation actions.

The completed solution demonstrates practical capability in cloud security monitoring, compliance automation, serverless incident response, IAM role configuration, network-control enforcement, and audit validation.

---

## Professional Value

This project demonstrates the ability to design, configure, and validate an automated security control using AWS-native services.

### Business and Security Outcomes

- Reduces the risk of unauthorized network exposure.
- Enforces approved Security Group baselines consistently.
- Detects configuration drift through continuous monitoring.
- Accelerates response to security-policy violations.
- Reduces repetitive manual remediation work.
- Improves auditability through centralized CloudWatch logging.
- Supports scalable cloud governance and DevSecOps practices.

### Role Alignment

The implementation reflects responsibilities commonly associated with:

- Cloud Security Engineering
- DevSecOps Engineering
- Cloud Governance and Compliance
- Security Operations
- AWS Infrastructure Engineering

---

## Project Objectives

The project was completed to demonstrate the ability to:

1. Evaluate IAM roles used by AWS Config and AWS Lambda.
2. Configure AWS Config to continuously monitor EC2 Security Groups.
3. Simulate a non-compliant network-security configuration.
4. Create a custom AWS Config rule linked to a Lambda function.
5. Validate automated removal of unauthorized inbound permissions.
6. Review CloudWatch Logs to confirm remediation activity.
7. Document the solution as a recruiter-ready cloud security portfolio project.

---

## Solution Architecture

```text
Amazon EC2 Security Group
            |
            v
AWS Config Configuration Recorder
            |
            v
Custom AWS Config Rule
            |
            v
AWS Lambda Remediation Function
            |
            v
EC2 Security Group API Actions
            |
            v
Approved Inbound Rules Restored
            |
            v
Amazon CloudWatch Audit Logs
```

### Architecture Responsibilities

| Component | Responsibility |
|---|---|
| AWS Config | Records Security Group configuration changes and initiates compliance evaluation. |
| Custom Config Rule | Evaluates Security Groups when their configuration changes. |
| AWS Lambda | Compares the observed configuration with the approved permissions and remediates deviations. |
| IAM | Grants AWS Config and Lambda the permissions required to perform their assigned actions. |
| Amazon EC2 | Hosts the monitored Security Group resources. |
| Amazon CloudWatch | Records Lambda execution and remediation evidence. |
| Amazon S3 | Receives AWS Config delivery-channel data in the lab environment. |

---

## Technology Stack

### AWS Services

- AWS Config
- AWS Lambda
- AWS Identity and Access Management (IAM)
- Amazon EC2 Security Groups
- Amazon Virtual Private Cloud (Amazon VPC)
- Amazon CloudWatch Logs
- Amazon Simple Storage Service (Amazon S3)

### Development and Automation

- Python
- Boto3, the AWS SDK for Python
- JSON IAM policies
- Event-driven serverless architecture

### Security Practices

- Continuous compliance monitoring
- Configuration-drift detection
- Least-privilege role assignment
- Automated incident remediation
- Network-security baseline enforcement
- Centralized logging and audit validation

---

## Lab Environment

| Item | Configuration |
|---|---|
| Environment | Controlled AWS training environment |
| Monitored Resource | Amazon EC2 Security Groups |
| Target Security Group | LabSG1 in Lab VPC |
| Recording Frequency | Continuous |
| Config Role | AwsConfigRole |
| Lambda Role | AwsConfigLambdaSGRole |
| Config Rule | EC2SecurityGroup |
| Rule Trigger | Configuration changes |
| Rule Parameter | `debug=true` |
| Approved Services | HTTP and HTTPS |
| Simulated Unauthorized Services | SMTPS and IMAPS |

> [!NOTE]
> Access to services and actions in the training environment was restricted to the resources required for the lab.

---

## Security Validation Scenario

To test the effectiveness of the control, the LabSG1 Security Group was intentionally modified to include inbound access that exceeded the approved baseline.

### Test Configuration

| Service | Protocol | Port | Test State |
|---|---:|---:|---|
| HTTP | TCP | 80 | Approved |
| HTTPS | TCP | 443 | Approved |
| SMTPS | TCP | 465 | Unauthorized test rule |
| IMAPS | TCP | 993 | Unauthorized test rule |

The SMTPS and IMAPS rules represented configuration drift that increased network exposure and required automated remediation.

---

## Technical Execution

### Phase 1: IAM Role Assessment and Configuration

The `AwsConfigLambdaSGRole` policy was reviewed to confirm that the Lambda function could:

- Create and write CloudWatch log events.
- Submit AWS Config evaluations.
- Describe EC2 Security Groups.
- Authorize Security Group ingress rules.
- Revoke Security Group ingress rules.

The AWS-managed `AWS_ConfigRole` policy was attached to `AwsConfigRole`, enabling AWS Config to read supported resource configurations and perform monitoring activities.

### Phase 2: Continuous Compliance Monitoring

AWS Config was configured to:

- Record specific resource types.
- Monitor `AWS::EC2::SecurityGroup` resources.
- Use continuous recording.
- Assume `AwsConfigRole`.
- Deliver configuration data through the configured Amazon S3 delivery channel.

The resulting inventory confirmed that AWS Config was tracking the target Security Groups and related resources.

### Phase 3: Controlled Configuration Drift

The LabSG1 inbound rules were modified to allow HTTP, HTTPS, SMTPS, and IMAPS from IPv4 sources. This provided a controlled validation case for the compliance and remediation workflow.

### Phase 4: Compliance and Remediation Control

A custom AWS Config rule named `EC2SecurityGroup` was created with the following configuration:

| Configuration | Value |
|---|---|
| Rule Type | Custom Lambda rule |
| Description | Restrict inbound ports to HTTP and HTTPS |
| Trigger | When configuration changes |
| Scope | AWS EC2 Security Groups |
| Lambda Integration | Pre-created remediation function |
| Custom Parameter | `debug=true` |

The rule evaluated monitored Security Groups and invoked the Lambda function when configuration changes occurred.

### Phase 5: Automated Remediation Validation

AWS Config identified the modified Security Group and invoked the remediation function. The Lambda logic applied the approved inbound permissions and removed rules that were outside the defined baseline.

#### Validated Outcome

- HTTP remained permitted.
- HTTPS remained permitted.
- SMTPS on TCP port 465 was removed.
- IMAPS on TCP port 993 was removed.
- Approved IPv4 and IPv6 HTTP/HTTPS permissions were restored.
- The monitored resource returned to a compliant state.

### Phase 6: Audit Verification

The `awsconfig_lambda_security_group` CloudWatch log group was reviewed using the search term `revoking`.

The resulting events provided evidence that the Lambda function revoked the unauthorized SMTPS and IMAPS permissions. The logs also supported verification that the automated control executed as intended.

---

## Automated Remediation Workflow

```text
1. A Security Group configuration changes.
                     |
                     v
2. AWS Config records the new configuration.
                     |
                     v
3. The EC2SecurityGroup rule evaluates the resource.
                     |
                     v
4. AWS Config invokes the Lambda function.
                     |
                     v
5. Lambda compares the rules with the approved baseline.
                     |
                     v
6. Unauthorized ingress permissions are revoked.
                     |
                     v
7. Approved permissions are restored.
                     |
                     v
8. CloudWatch Logs preserve remediation evidence.
```

> [!TIP]
> This event-driven design reduces the time between configuration drift and corrective action while preserving evidence for operational review.

---

## Results and Validation

### Final Results

- AWS Config continuously monitored the target Security Groups.
- The custom rule evaluated Security Group configuration changes.
- AWS Lambda automatically removed unauthorized inbound rules.
- The approved HTTP and HTTPS baseline was restored.
- CloudWatch Logs confirmed the remediation activity.
- The final Security Group configuration met the defined compliance requirement.

### Validation Checklist

- [x] Reviewed Lambda execution-role permissions.
- [x] Attached the required AWS Config managed policy.
- [x] Enabled continuous Security Group recording.
- [x] Verified the AWS Config resource inventory.
- [x] Introduced controlled configuration drift.
- [x] Created the custom AWS Config rule.
- [x] Integrated the rule with the Lambda function.
- [x] Confirmed removal of SMTPS and IMAPS rules.
- [x] Verified CloudWatch remediation logs.
- [x] Restored the approved network-security baseline.

---

## Core Competencies Demonstrated

### Cloud Security Engineering

- Configuration compliance monitoring
- Network-security control validation
- Security baseline enforcement
- Cloud risk reduction
- Automated incident response
- Audit evidence analysis

### DevSecOps and Automation

- Event-driven security architecture
- Serverless remediation workflows
- Continuous compliance
- Security automation
- Infrastructure resilience
- Operational logging and validation

### AWS Platform

- AWS Config configuration recording and custom rules
- AWS Lambda security automation
- IAM role and policy analysis
- EC2 Security Group administration
- Amazon VPC network-security controls
- Amazon CloudWatch log investigation
- Amazon S3 configuration-data delivery

---

## Security Considerations

- Production deployments should scope IAM permissions as narrowly as practical.
- Remediation logic should be tested against approved business requirements before organization-wide rollout.
- Security Group baselines should be version-controlled and reviewed through an authorized change-management process.
- CloudWatch Logs should be protected with appropriate retention, encryption, and access controls.
- Automated remediation should include alerts or ticketing integration so security teams retain operational visibility.
- Multi-account environments should consider centralized governance with AWS Organizations and delegated administration.

---

## Limitations and Next Steps

### Current Limitations

- The implementation was validated in a controlled training environment.
- The approved rule set was defined within the provided Lambda function.
- Monitoring focused on EC2 Security Groups in one AWS Region.
- The workflow did not include external notification or ticket creation.

### Recommended Enhancements

1. Store approved Security Group baselines in a centrally managed configuration source.
2. Add Amazon SNS notifications for remediation events.
3. Create operational dashboards and alarms in Amazon CloudWatch.
4. Extend coverage across multiple AWS accounts and Regions.
5. Integrate remediation events with an incident-management platform.
6. Deploy the architecture through AWS CloudFormation or Terraform.
7. Add automated tests for compliant and non-compliant rule combinations.

---

## Conclusion

This project demonstrates the implementation of a self-healing cloud security control using AWS Config, AWS Lambda, IAM, Amazon EC2, and Amazon CloudWatch.

The solution continuously monitored Security Group configurations, detected unauthorized network permissions, automatically restored the approved baseline, and produced audit evidence of the remediation process. The outcome highlights practical experience in Cloud Security Engineering, DevSecOps, compliance automation, serverless incident response, and AWS governance.

For recruiters and hiring managers, the project provides evidence of the ability to translate a security requirement into a functioning cloud-native control that reduces risk, improves consistency, and supports auditable operations.

---

## Disclaimer

This project was completed in a controlled AWS training environment for educational and portfolio-development purposes. Resource names, permissions, network rules, and implementation choices should be reviewed and adapted before use in production.

The original lab material is the property of Amazon Web Services, Inc. This README is an independent portfolio summary of the implementation and learning outcomes and does not reproduce the full lab guide.

---

## Author

<h3 align="center">Wadondera A. Collins</h3>

<p align="center">
  <strong>ICDFA Trainee | Cohort 11 | Cloud Security Engineering</strong><br>
  Cloud Security | AWS | DevSecOps | Security Automation
</p>

<p align="center"><strong>Project Outcome:</strong> Designed, configured, and validated an automated AWS security control that detected Security Group configuration drift, removed unauthorized ingress permissions, restored compliance, and preserved remediation evidence in CloudWatch Logs.</p>
