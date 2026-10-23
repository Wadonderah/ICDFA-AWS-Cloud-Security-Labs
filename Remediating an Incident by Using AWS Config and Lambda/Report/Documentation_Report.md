<div align="center">

# TECHNICAL IMPLEMENTATION & ASSURANCE REPORT

## Automated Security-Group Incident Remediation with AWS Config and AWS Lambda

**Lab 7.1 | Cloud Security Engineering**

---

**Prepared by:** Wadondera A. Collins  
**Programme:** ICDFA Trainee | Cohort 11  
**Environment:** Amazon Web Services Training Account  
**Region:** US East (N. Virginia)  
**Report date:** 23 October 2026  
**Classification:** Portfolio / Educational  
**Status:** Final

</div>

---

## Document Control

| Control | Record |
|---|---|
| Document owner | Wadondera A. Collins |
| Document purpose | Technical implementation record and evidence-based assurance report |
| Security domain | Configuration compliance and automated incident remediation |
| In-scope resource | Amazon EC2 Security Groups |
| Core services | AWS Config, AWS Lambda, IAM, Amazon EC2, Amazon VPC, Amazon CloudWatch, Amazon S3 |
| Evidence reviewed | Six AWS Management Console captures |
| Review basis | Configuration evidence, control objectives, and validation results |

### Revision History

| Version | Date | Description | Author |
|---|---|---|---|
| 1.0 | 23 October 2026 | Initial final report | Wadondera A. Collins |

---

## Table of Contents

1. [Management Summary](#1-management-summary)
2. [Engagement Context](#2-engagement-context)
3. [Control Objectives and Acceptance Criteria](#3-control-objectives-and-acceptance-criteria)
4. [Solution Design](#4-solution-design)
5. [Implementation Record and Evidence](#5-implementation-record-and-evidence)
6. [Validation Results](#6-validation-results)
7. [Security and Operational Assessment](#7-security-and-operational-assessment)
8. [Evidence Gaps and Limitations](#8-evidence-gaps-and-limitations)
9. [Recommendations](#9-recommendations)
10. [Conclusion](#10-conclusion)
11. [Appendices](#11-appendices)

---

# 1. Management Summary

## 1.1 Engagement Outcome

This engagement implemented an AWS-native control for monitoring Amazon EC2 Security Group changes and invoking automated remediation when a monitored configuration deviated from an approved inbound-access baseline.

The control combined:

- **AWS Config** for configuration recording and rule evaluation;
- **AWS Lambda** for programmed evaluation and corrective action;
- **AWS Identity and Access Management** for service authorization;
- **Amazon EC2 and Amazon VPC** for the protected Security Group resource; and
- **Amazon CloudWatch Logs** for execution evidence.

A controlled validation condition was created on `LabSG1` by permitting HTTP, HTTPS, SMTPS, and IMAPS from `0.0.0.0/0`. Within the lab baseline, HTTP and HTTPS were approved, while SMTPS and IMAPS represented unauthorized additions that the Lambda logic was intended to remove.

## 1.2 Assurance Statement

The evidence supplied for this report verifies the following:

- The AWS Config service role received the required AWS-managed policy.
- The four-rule Security Group validation state was created successfully.
- A custom Lambda-backed AWS Config rule named `EC2SecurityGroup` was added.
- The rule targeted EC2 Security Groups, used detective evaluation, responded to configuration changes, and received `debug=true`.
- The associated Lambda log group existed and contained execution streams.

The evidence supplied does **not** display the contents of a CloudWatch event proving revocation of ports 465 and 993, and it does **not** show the final post-remediation `LabSG1` rule set. Those outcomes are therefore recorded as part of the completed workflow, but not classified as visually proven in this report.

## 1.3 Professional Relevance

The implementation demonstrates practical experience in cloud configuration monitoring, IAM role analysis, network-control validation, event-driven security automation, compliance assessment, audit logging, and evidence-led reporting.

---

# 2. Engagement Context

## 2.1 Security Problem

Security Groups are a primary network-control layer for workloads hosted in Amazon VPC. Unapproved ingress changes can expose services, create configuration drift, and weaken the intended security posture.

A manual-only response model introduces two material risks:

1. A configuration change may remain undetected until a scheduled review.
2. Restoration of the approved baseline depends on timely human intervention.

The implemented control addressed these risks by connecting configuration-change detection to Lambda-based corrective logic.

## 2.2 Scope

The engagement included:

- Review and update of IAM roles used by AWS Config and Lambda;
- continuous monitoring of EC2 Security Groups with AWS Config;
- controlled modification of `LabSG1` inbound permissions;
- creation of a custom Lambda-backed Config rule;
- review of the rule's scope, trigger, parameter, and evaluation state; and
- inspection of the associated CloudWatch log group and log streams.

## 2.3 Exclusions

The following were outside the demonstrated scope:

- Multi-account governance;
- multi-Region monitoring;
- production change approval;
- ticketing or notification integration;
- source-controlled infrastructure deployment; and
- testing against application availability requirements.

---

# 3. Control Objectives and Acceptance Criteria

## 3.1 Control Objectives

| ID | Control objective |
|---|---|
| CO-01 | AWS Config operates with an authorized service role. |
| CO-02 | EC2 Security Groups are evaluated when their configuration changes. |
| CO-03 | The custom rule invokes the designated Lambda logic. |
| CO-04 | HTTP and HTTPS remain within the approved inbound baseline. |
| CO-05 | SMTPS and IMAPS are identified as deviations from the lab baseline. |
| CO-06 | Corrective execution produces auditable CloudWatch records. |

## 3.2 Acceptance Criteria

| Criterion | Required evidence |
|---|---|
| IAM readiness | AWS Config role displays the required attached policy. |
| Test condition | `LabSG1` displays HTTP, HTTPS, SMTPS, and IMAPS. |
| Rule deployment | `EC2SecurityGroup` is listed as a Custom Lambda rule. |
| Rule configuration | Scope, trigger, description, and `debug=true` are visible. |
| Execution record | Lambda CloudWatch log group contains execution streams. |
| Remediation proof | Event records identify revoked permissions. |
| Final-state proof | `LabSG1` displays only the approved rules after remediation. |

---

# 4. Solution Design

## 4.1 Logical Architecture

```text
┌──────────────────────────────┐
│ Amazon EC2 Security Group    │
│ LabSG1                       │
└──────────────┬───────────────┘
               │ Configuration change
               ▼
┌──────────────────────────────┐
│ AWS Config                   │
│ Continuous resource record  │
└──────────────┬───────────────┘
               │ Detective evaluation
               ▼
┌──────────────────────────────┐
│ Custom Config Rule           │
│ EC2SecurityGroup             │
└──────────────┬───────────────┘
               │ Invocation
               ▼
┌──────────────────────────────┐
│ AWS Lambda                   │
│ Rule evaluation/remediation  │
└──────────────┬───────────────┘
               │ EC2 ingress API actions
               ▼
┌──────────────────────────────┐
│ Approved security baseline   │
│ HTTP and HTTPS               │
└──────────────┬───────────────┘
               │ Execution records
               ▼
┌──────────────────────────────┐
│ Amazon CloudWatch Logs       │
└──────────────────────────────┘
```

## 4.2 Component Responsibilities

| Component | Responsibility |
|---|---|
| `AwsConfigRole` | Authorizes AWS Config to inspect supported resource configurations and use the delivery channel. |
| `AwsConfigLambdaSGRole` | Authorizes Lambda logging, AWS Config evaluation submission, and Security Group inspection or modification. |
| AWS Config recorder | Captures relevant resource-configuration changes. |
| `EC2SecurityGroup` rule | Defines resource scope and invokes the Lambda function when configuration changes occur. |
| Lambda function | Compares observed permissions with the programmed baseline and performs corrective actions. |
| CloudWatch Logs | Retains Lambda execution records for operational investigation. |

## 4.3 Approved Lab Baseline

| Service | Protocol | Port | Baseline decision |
|---|---:|---:|---|
| HTTP | TCP | 80 | Retain |
| HTTPS | TCP | 443 | Retain |
| SMTPS | TCP | 465 | Remove |
| IMAPS | TCP | 993 | Remove |

---

# 5. Implementation Record and Evidence

## 5.1 IAM Service-Role Preparation

The `AwsConfigRole` permissions were updated by attaching the AWS-managed `AWS_ConfigRole` policy. The customer-inline `S3Access` policy remained attached for the lab delivery-channel requirement.

<img width="1348" height="575" alt="Fig01 IAM Role Policies" src="https://github.com/user-attachments/assets/735b5c14-781b-4ee4-b076-c2cad903b090" />


**Figure 1. AWS Config service-role permissions after policy attachment.**

**Observed evidence**

- The console identifies the role as `AwsConfigRole`.
- A green notification states that a policy was successfully attached.
- `AWS_ConfigRole` appears as an AWS-managed policy.
- `S3Access` appears as a customer-inline policy.

**Control assessment:** **Verified.** The screenshot directly supports control objective CO-01.

---

## 5.2 Controlled Security-Group Deviation

The `LabSG1` inbound rule set was deliberately modified to introduce a known non-baseline state for control validation.

<img width="1356" height="626" alt="Fig02 Security Group Incident Rules" src="https://github.com/user-attachments/assets/deed9f41-0f59-47cc-b294-a34ed3fb7ca9" />


**Figure 2. LabSG1 inbound rules used for the controlled configuration-drift scenario.**

**Observed evidence**

- The selected Security Group is `LabSG1`.
- The console displays four inbound rules.
- HTTP uses TCP port 80.
- HTTPS uses TCP port 443.
- SMTPS uses TCP port 465.
- IMAPS uses TCP port 993.
- All four visible rules use IPv4 source `0.0.0.0/0`.
- A green notification confirms that the inbound rules were modified successfully.

**Control assessment:** **Verified.** The screenshot establishes the pre-remediation test state and supports CO-04 and CO-05.

---

## 5.3 Custom Rule Deployment

A custom AWS Config rule named `EC2SecurityGroup` was created to evaluate monitored Security Group changes through Lambda.

<img width="1356" height="617" alt="Fig03 AWS Config Rule Created" src="https://github.com/user-attachments/assets/7f6ac05b-402c-48e0-968b-8915469b6ea1" />


**Figure 3. Custom Lambda-backed EC2SecurityGroup rule listed in AWS Config.**

**Observed evidence**

- A success notification states that `EC2SecurityGroup` was added.
- The rule type is `Custom Lambda`.
- The enabled evaluation mode is `DETECTIVE`.
- The remediation-action column displays `Not set`.

**Interpretation:** The absence of a configured AWS Config remediation action is consistent with corrective behavior being implemented inside the invoked Lambda function.

**Control assessment:** **Verified.** The screenshot directly demonstrates custom-rule deployment.

---

## 5.4 Rule Scope, Trigger, and Parameter

The custom rule was reviewed to confirm the intended description, target resource type, evaluation mode, trigger, and input parameter.

<img width="1359" height="459" alt="Fig04 AWS Config Rule Configuration" src="https://github.com/user-attachments/assets/765303f4-1712-4b5f-abe5-ded220390fe5" />


**Figure 4. EC2SecurityGroup rule details and evaluation configuration.**

**Observed evidence**

- The description is `Restrict inbound ports to HTTP and HTTPS`.
- The enabled evaluation mode is `DETECTIVE`.
- Configuration-change trigger types are visible.
- The scope is `Resources`.
- The resource type is `EC2 SecurityGroup`.
- The parameter `debug` has value `true`.
- A last successful detective-evaluation timestamp is displayed.
- The resource view is filtered to `Noncompliant` and displays `No evaluation results`.

**Control assessment:** **Partially verified.** Scope, trigger, parameter, and evaluation activity are visible. A final compliant resource result is not shown.

---

## 5.5 AWS Config Dashboard Review

The dashboard was inspected to review rule, resource, inventory, and service metrics.

<img width="1359" height="592" alt="Fig05 AWS Config Dashboard Status" src="https://github.com/user-attachments/assets/df370a68-9a87-4f19-9839-b6038589fffe" />


**Figure 5. AWS Config dashboard state at the time of evidence capture.**

**Observed evidence**

- Zero noncompliant rules and resources are shown.
- Zero compliant rules and resources are shown.
- Resource inventory displays zero.
- Several metrics panels state that no data is available for the selected range.

**Control assessment:** **Inconclusive for final compliance.** The screenshot records the dashboard state but does not establish that the monitored Security Group reached a compliant result.

---

## 5.6 Lambda Logging Review

The Lambda log group was inspected to determine whether execution records had been generated.

<img width="1344" height="633" alt="Fig06 CloudWatch Lambda Log Streams" src="https://github.com/user-attachments/assets/e77fa880-fb6e-4f39-9006-a7306cb865d8" />


**Figure 6. CloudWatch log group and Lambda execution streams.**

**Observed evidence**

- The log group is `/aws/lambda/awsconfig_lambda_security_group`.
- Five log streams are visible.
- Last-event times are displayed for each visible stream.
- Retention is shown as `Never expire`.

**Control assessment:** **Partially verified.** The existence of Lambda logging activity is proven. The screenshot does not display event-level text confirming the revocation of SMTPS and IMAPS permissions.

---

# 6. Validation Results

## 6.1 Control Validation Matrix

| Control | Expected condition | Evidence | Result |
|---|---|---|---|
| CO-01 | Required policy attached to `AwsConfigRole` | Figure 1 | **Verified** |
| CO-02 | Security Group changes evaluated by custom rule | Figures 3 and 4 | **Verified** |
| CO-03 | Lambda-backed rule deployed | Figure 3 | **Verified** |
| CO-04 | HTTP and HTTPS form part of the test baseline | Figure 2 | **Verified** |
| CO-05 | SMTPS and IMAPS introduced as deviations | Figure 2 | **Verified** |
| CO-06 | Lambda execution produces CloudWatch records | Figure 6 | **Partially verified** |
| Remediation | Ports 465 and 993 revoked | Event details not supplied | **Not visually proven** |
| Final state | Only approved rules remain | Post-remediation capture not supplied | **Not visually proven** |
| Compliance | Target resource displayed as compliant | Final result not supplied | **Not visually proven** |

## 6.2 Confirmed Findings

1. IAM preparation for AWS Config was completed successfully.
2. The controlled four-rule Security Group state was created successfully.
3. The custom Lambda-backed AWS Config rule was deployed successfully.
4. The rule configuration matched the intended EC2 Security Group monitoring scope.
5. Lambda execution generated CloudWatch log streams.

## 6.3 Assurance Qualification

The technical workflow supports the intended automatic-remediation outcome. However, the existing evidence package does not independently prove the exact revocation actions or final Security Group state. This qualification is included to keep the report accurate and suitable for technical review.

---

# 7. Security and Operational Assessment

## 7.1 Control Strengths

- **Native service integration:** The design uses AWS-managed control-plane services.
- **Event-driven response:** Configuration changes can trigger evaluation without waiting for manual review.
- **Role separation:** AWS Config and Lambda use distinct IAM roles.
- **Audit support:** CloudWatch provides a central execution record.
- **Repeatable baseline:** The Lambda logic applies a defined set of required permissions.

## 7.2 Security Considerations

### IAM Scope

The supplied lab policy includes actions that use `Resource: *`. In a production design, permissions should be reduced to the narrowest scope supported by the affected APIs and deployment model.

### Automated Change Risk

Automatic revocation can disrupt an approved business service if the baseline is incomplete. Production adoption should include baseline ownership, change approval, testing, and exception handling.

### Logging and Retention

The evidence shows CloudWatch retention as `Never expire`. A formal production policy should define retention duration, encryption, access control, and archival requirements.

### Regional Coverage

The implementation monitored resources in one AWS Region. Organizations operating across Regions should establish equivalent coverage wherever protected resources are deployed.

### Operational Awareness

Automatic remediation should be accompanied by notification or ticket creation so that security and service teams retain visibility of each control action.

---

# 8. Evidence Gaps and Limitations

## 8.1 Evidence Gaps

The following captures were not included:

1. `LabSG1` after remediation, showing only HTTP and HTTPS.
2. An expanded CloudWatch event showing the exact permissions revoked.
3. The target Security Group listed as compliant in AWS Config.
4. The Lambda source code or relevant required-permissions definition.
5. The AWS Config recording-settings page.

## 8.2 Implementation Limitations

- The work was completed in a controlled training account.
- The rule baseline was embedded in the provided Lambda implementation.
- Monitoring was limited to one Region.
- No notification or incident-ticket integration was demonstrated.
- No infrastructure-as-code deployment was demonstrated.
- No application-impact test was included.

---

# 9. Recommendations

## 9.1 Immediate Evidence Improvements

| Priority | Recommendation | Expected benefit |
|---|---|---|
| High | Capture `LabSG1` after remediation | Proves restoration of the approved baseline |
| High | Capture the expanded `revoking` CloudWatch events | Proves the precise corrective actions |
| High | Capture the target resource as compliant | Completes the detect-remediate-validate evidence chain |

## 9.2 Production Hardening

| Priority | Recommendation | Expected benefit |
|---|---|---|
| High | Review and reduce IAM permissions | Limits the impact of role misuse |
| High | Introduce baseline ownership and exception handling | Prevents legitimate services from being removed incorrectly |
| Medium | Set a defined CloudWatch retention policy | Aligns logs with governance and cost requirements |
| Medium | Add Amazon SNS or EventBridge notification | Improves operational awareness |
| Medium | Encrypt and restrict access to logs and delivery data | Protects security telemetry |

## 9.3 Engineering Maturity

| Priority | Recommendation | Expected benefit |
|---|---|---|
| Medium | Deploy with Terraform, AWS CloudFormation, or AWS CDK | Makes the control repeatable and reviewable |
| Medium | Store the approved baseline in version control | Improves traceability and change governance |
| Medium | Extend deployment across required accounts and Regions | Expands control coverage |
| Medium | Add automated positive, negative, and failure tests | Improves control reliability |

---

# 10. Conclusion

The project established the key components of an automated AWS Security Group compliance control. The evidence confirms IAM role preparation, creation of a controlled deviation, deployment and configuration of the custom Config rule, and generation of Lambda log streams.

From a Cloud Security Engineering perspective, the implementation demonstrates the ability to connect configuration monitoring, policy evaluation, automated corrective logic, and audit logging. The report also identifies a clear evidence-quality issue: the current screenshots prove the control setup and execution activity, but not the exact revocation events or final compliant resource state.

Adding the three high-priority evidence captures identified in this report would complete the assurance chain and make the project substantially stronger for technical assessment, portfolio review, and audit-style validation.

---

# 11. Appendices

## Appendix A: Evidence Register

| Figure | Source filename | Evidence represented |
|---|---|---|
| Figure 1 | `Fig02 name.png` | IAM policies attached to `AwsConfigRole` |
| Figure 2 | `Fig01 name.png` | Four-rule `LabSG1` test configuration |
| Figure 3 | `Fig03 name.png` | Custom Config rule creation |
| Figure 4 | `Fig04 name.png` | Rule details, scope, trigger, and parameter |
| Figure 5 | `Fig06 name.png` | AWS Config dashboard state |
| Figure 6 | `Fig05 name.png` | CloudWatch Lambda log streams |

## Appendix B: Evidence Naming Plan

For repository use, rename the evidence files as follows:

```text
Fig01 IAM Role Policies.png
Fig02 Security Group Incident Rules.png
Fig03 AWS Config Rule Created.png
Fig04 AWS Config Rule Configuration.png
Fig05 AWS Config Dashboard Status.png
Fig06 CloudWatch Lambda Log Streams.png
```

## Appendix C: Validation Status Definitions

| Status | Meaning |
|---|---|
| **Verified** | The supplied screenshot directly shows the expected condition. |
| **Partially verified** | The screenshot shows supporting activity but not the complete expected condition. |
| **Not visually proven** | The expected condition is not displayed in the supplied evidence. |
| **Inconclusive** | The captured state does not support a reliable pass or fail conclusion. |

## Appendix D: Acronyms

| Acronym | Meaning |
|---|---|
| AWS | Amazon Web Services |
| EC2 | Amazon Elastic Compute Cloud |
| IAM | AWS Identity and Access Management |
| VPC | Amazon Virtual Private Cloud |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | Hypertext Transfer Protocol Secure |
| SMTPS | Simple Mail Transfer Protocol Secure |
| IMAPS | Internet Message Access Protocol Secure |

---

## Disclaimer

This report documents work completed in a controlled AWS training environment. All configurations, IAM permissions, automated actions, logging settings, and baselines should be reviewed, tested, and approved before production use. The underlying lab material remains the property of Amazon Web Services, Inc. This document is an independent technical implementation and assurance report prepared for educational and portfolio purposes.

---

<div align="center">

### Wadondera A. Collins

**ICDFA Trainee | Cohort 11 | Cloud Security Engineering**  
Cloud Security | AWS | DevSecOps | Security Automation

</div>
