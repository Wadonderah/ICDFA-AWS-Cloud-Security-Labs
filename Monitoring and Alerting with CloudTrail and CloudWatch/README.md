<div align="center">

# 🛡️ AWS CLOUD SECURITY MONITORING & ALERTING

## CloudTrail • CloudWatch • EventBridge • Amazon SNS

### Lab 6.1 | Audit Logging, Threat Detection, Automated Alerting, and Log Investigation

<br>

![AWS](https://img.shields.io/badge/Platform-Amazon%20Web%20Services-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Cloud Security](https://img.shields.io/badge/Domain-Cloud%20Security-0B74DE?style=for-the-badge&logo=securityscorecard&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2EA44F?style=for-the-badge&logo=checkmarx&logoColor=white)

<br>

**Wadondera A. Collins**  
*ICDFA Trainee | Cohort 11 | Cloud Security Engineering*

<br>

> **An AWS-native security monitoring solution for auditing API activity, detecting security group changes, identifying repeated console login failures, and delivering automated email alerts.**

</div>

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Executive Summary](#-executive-summary)
- [Project Objectives](#-project-objectives)
- [Professional Value](#-professional-value)
- [Skills Demonstrated](#-skills-demonstrated)
- [AWS Services Used](#-aws-services-used)
- [Lab Environment](#-lab-environment)
- [Solution Architecture](#-solution-architecture)
- [Implementation](#-implementation)
- [Testing and Validation](#-testing-and-validation)
- [Results](#-results)
- [Screenshot Evidence Plan](#-screenshot-evidence-plan)
- [Validation Checklist](#-validation-checklist)
- [Security Considerations](#-security-considerations)
- [Limitations](#-limitations)
- [Recommended Improvements](#-recommended-improvements)
- [Conclusion](#-conclusion)
- [Disclaimer](#-disclaimer)
- [Author](#-author)

---

## 🔎 Project Overview

This project implements an AWS-native monitoring and alerting workflow using **AWS CloudTrail**, **Amazon CloudWatch**, **Amazon EventBridge**, and **Amazon Simple Notification Service (Amazon SNS)**.

The solution records account activity, detects selected security-relevant events, sends automated email notifications, and supports interactive investigation of failed AWS Management Console sign-in events.

Two complementary detection methods were implemented:

1. **Event-driven monitoring:** EventBridge detects selected Amazon EC2 API calls recorded by CloudTrail and publishes transformed event details to Amazon SNS.
2. **Metric-based monitoring:** A CloudWatch Logs metric filter converts failed console sign-in events into a custom metric. A CloudWatch alarm evaluates that metric and sends an SNS notification when the configured threshold is reached.

---

## 📌 Executive Summary

The completed project establishes a foundational AWS security detection pipeline. CloudTrail provides the audit record, CloudWatch Logs stores the events, EventBridge evaluates selected API activity, CloudWatch converts matching log events into measurable security signals, and SNS delivers notifications to a confirmed email subscriber.

The monitoring controls were validated through two controlled tests:

- An inbound SSH rule was added to an EC2 security group, generating an `AuthorizeSecurityGroupIngress` event and an EventBridge-driven SNS alert.
- At least three unsuccessful AWS Management Console sign-in attempts were generated for the lab IAM user, causing the `FailedLogins` alarm to enter the `In alarm` state and publish an SNS notification.

CloudWatch Logs Insights was then used to aggregate failed sign-in records by source IP address, failure reason, AWS Region, and IAM ARN.

---

## 🎯 Project Objectives

- Analyze management events in CloudTrail Event history.
- Review the configuration of a CloudTrail trail with CloudWatch Logs delivery enabled.
- Create an SNS topic and confirm an email subscription.
- Configure an EventBridge rule for selected EC2 security-related API calls.
- Transform EventBridge event data into a readable notification.
- Create a CloudWatch Logs metric filter for failed console sign-ins.
- Create a CloudWatch alarm from the resulting custom metric.
- Validate the alerting workflows with controlled security events.
- Query CloudTrail events using CloudWatch Logs Insights.

---

## 💼 Professional Value

This project provides practical portfolio evidence for **Cloud Security Engineer**, **Cloud Engineer**, **CloudOps**, **DevOps**, and **Security Operations** roles.

It demonstrates the ability to:

- Use AWS audit logs as a security telemetry source.
- Translate API activity into actionable alerts.
- Monitor changes that can affect network exposure.
- Detect repeated AWS Management Console authentication failures.
- Create and validate custom CloudWatch metrics and alarms.
- Investigate security events with structured log queries.
- Document monitoring controls, tests, results, risks, and limitations.

---

## 🧠 Skills Demonstrated

- AWS account activity auditing
- CloudTrail event analysis
- CloudWatch Logs administration
- CloudWatch metric filter configuration
- Custom metric and alarm configuration
- EventBridge event-pattern design
- EventBridge input transformation
- SNS topic and subscription management
- EC2 security group monitoring
- Authentication-event detection
- CloudWatch Logs Insights querying
- Security control testing and validation
- Least-privilege security awareness

---

## 🧰 AWS Services Used

| AWS Service | Purpose |
|---|---|
| AWS CloudTrail | Recorded AWS management activity and supplied API events for monitoring and investigation. |
| Amazon CloudWatch Logs | Stored CloudTrail events used by the metric filter and Logs Insights query. |
| Amazon CloudWatch Metrics | Stored the custom failed-console-login metric. |
| Amazon CloudWatch Alarms | Evaluated the custom metric against the configured threshold. |
| Amazon EventBridge | Matched selected EC2 API calls and routed transformed event data to SNS. |
| Amazon SNS | Delivered email notifications for security group changes and failed-login alarms. |
| Amazon EC2 | Supplied the monitored instance and security group used for validation. |
| AWS IAM | Supplied the test user used to generate failed console sign-in events. |
| Amazon S3 | Served as the CloudTrail trail's log-file storage destination. |
| AWS Management Console | Used to configure, test, and validate the monitoring solution. |

---

## ⚙️ Lab Environment

| Component | Configuration |
|---|---|
| Existing CloudTrail trail | `LabCloudTrail` |
| CloudWatch Logs log group | `CloudTrailLogGroup` |
| SNS topic | `MySNSTopic` |
| EventBridge rule | `MonitorSecurityGroups` |
| EC2 instance | `LabInstance` |
| EC2 security group | `LabSecurityGroup` |
| Metric filter | `ConsoleLoginErrors` |
| Metric namespace | `CloudTrailMetrics` |
| Metric name | `ConsoleLoginFailureCount` |
| CloudWatch alarm | `FailedLogins` |
| Test IAM user | `test` |
| Alarm statistic | `Sum` |
| Alarm threshold | Greater than or equal to `3` |
| Metric period | `5 minutes` |
| Notification destination | Confirmed email subscription to `MySNSTopic` |

> [!IMPORTANT]
> The restricted lab identity could not complete the creation of a new CloudTrail trail with CloudWatch Logs enabled. The configuration workflow was reviewed and cancelled as instructed. The preconfigured `LabCloudTrail` trail and `CloudTrailLogGroup` were used for the remaining tasks.

---

## 🏗️ Solution Architecture

### Security Group Change Alert

```text
EC2 Security Group Change
          │
          ▼
AWS CloudTrail Management Event
          │
          ▼
Amazon EventBridge Rule Match
          │
          ▼
EventBridge Input Transformer
          │
          ▼
Amazon SNS Topic: MySNSTopic
          │
          ▼
Confirmed Email Subscriber
```

### Failed Console Login Alert

```text
Failed AWS Console Sign-In Attempts
          │
          ▼
AWS CloudTrail Management Events
          │
          ▼
CloudWatch Logs: CloudTrailLogGroup
          │
          ▼
Metric Filter: ConsoleLoginErrors
          │
          ▼
Custom Metric: ConsoleLoginFailureCount
          │
          ▼
CloudWatch Alarm: FailedLogins
          │
          ▼
Amazon SNS Topic: MySNSTopic
          │
          ▼
Confirmed Email Subscriber
```

---

## 🚀 Implementation

### Task 1: Analyze CloudTrail Events and Review Trail Configuration

CloudTrail Event history was filtered by the event source `cloudformation.amazonaws.com`. The most recent `CreateStack` event was reviewed to identify audit fields such as `userIdentity`, `eventTime`, `awsRegion`, `eventSource`, and `eventName`.

The new-trail workflow was reviewed using the following intended configuration:

- **Trail name:** `MyLabCloudTrail`
- **Storage:** New S3 bucket using the generated CloudTrail bucket name
- **Log-file SSE-KMS encryption:** Disabled for this lab
- **CloudWatch Logs:** Enabled
- **Log group:** New log group using the console default name
- **IAM role:** Existing `LabCloudTrailRole`
- **Event type:** Management events
- **API activity:** Read and Write
- **Data events:** Not selected
- **Insights events:** Not selected

The trail was not created because the lab identity lacked the required permission. The existing `LabCloudTrail` configuration was reviewed and used for the remainder of the lab.

### Task 2: Create an SNS Topic and Email Subscription

A standard SNS topic named `MySNSTopic` was created. An email endpoint was subscribed to the topic, and the subscription was confirmed through the AWS Notifications confirmation email.

The confirmed subscription became the notification destination for both the EventBridge rule and the CloudWatch alarm.

> [!NOTE]
> The broad publish and subscribe settings used in this task were specific to the guided lab. A production SNS topic should use a least-privilege resource policy.

### Task 3: Detect Security Group Changes with EventBridge

An EventBridge rule named `MonitorSecurityGroups` was configured on the default event bus.

#### Event Pattern

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["ec2.amazonaws.com"],
    "eventName": [
      "AuthorizeSecurityGroupIngress",
      "ModifyNetworkInterfaceAttribute"
    ]
  }
}
```

The target was configured as `MySNSTopic`. An input transformer extracted the security group ID, event name, event time, and complete event detail.

#### Input Path

```json
{
  "name": "$.detail.requestParameters.groupId",
  "source": "$.detail.eventName",
  "time": "$.time",
  "value": "$.detail"
}
```

#### Input Template

```text
"The <source> API call was made against the <name> security group on <time> with the following details:"
" <value> "
```

The rule was tested by adding an inbound SSH rule to `LabSecurityGroup` with `Anywhere-IPv4` as the source. CloudTrail recorded the resulting `AuthorizeSecurityGroupIngress` event. The event record confirmed that `fromPort` and `toPort` were both `22`, and the subscribed mailbox received the EventBridge-generated SNS notification.

> [!CAUTION]
> The lab procedure specifies `MonitorSecurityGroups`, while the embedded grading note refers to `MonitoringRule`. This README preserves the name configured in the procedural steps. If the grader does not recognize the rule, review the submission report for the exact expected resource name.

### Task 4: Detect Repeated Console Login Failures

A metric filter named `ConsoleLoginErrors` was created on `CloudTrailLogGroup`.

#### Metric Filter Pattern

```text
{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }
```

#### Metric Transformation

```text
Namespace: CloudTrailMetrics
Metric:    ConsoleLoginFailureCount
Value:     1
```

A CloudWatch alarm named `FailedLogins` was configured as follows:

- **Statistic:** `Sum`
- **Period:** `5 minutes`
- **Comparison:** Greater than or equal to the threshold
- **Threshold:** `3`
- **Notification action:** Publish to `MySNSTopic`

The control was tested by attempting to sign in as the lab IAM user `test` with an incorrect password at least three times. The custom metric displayed matching data, the alarm entered the `In alarm` state, and the SNS notification was received.

### Task 5: Query CloudTrail Events with CloudWatch Logs Insights

CloudWatch Logs Insights was used to search `CloudTrailLogGroup` for failed AWS Management Console sign-ins.

```sql
filter eventSource="signin.amazonaws.com"
  and eventName="ConsoleLogin"
  and responseElements.ConsoleLogin="Failure"
| stats count(*) as Total_Count
  by sourceIPAddress as Source_IP,
     errorMessage as Reason,
     awsRegion as AWS_Region,
     userIdentity.arn as IAM_Arn
```

The query grouped matching events by source IP address, failure reason, AWS Region, and IAM ARN.

---

## ✅ Testing and Validation

### Security Group Change Test

1. Selected `LabInstance` in the EC2 console.
2. Opened the associated `LabSecurityGroup`.
3. Added an inbound SSH rule using `Anywhere-IPv4`.
4. Saved the inbound rule.
5. Verified `AuthorizeSecurityGroupIngress` in CloudTrail Event history.
6. Confirmed that the event referenced TCP port `22`.
7. Confirmed delivery of the transformed SNS notification.

### Failed Console Login Test

1. Opened the console sign-in page for the test IAM user.
2. Generated at least three failed authentication attempts.
3. Re-authenticated to the lab account using the authorized lab access method.
4. Verified data for `ConsoleLoginFailureCount`.
5. Verified that `FailedLogins` entered the `In alarm` state.
6. Confirmed delivery of the CloudWatch alarm notification through SNS.

### Log Investigation Test

1. Selected `CloudTrailLogGroup` in CloudWatch Logs Insights.
2. Ran the failed-console-login query.
3. Verified that matching events were aggregated into the requested investigation fields.

---

## 📊 Results

- CloudTrail Event history exposed detailed AWS API audit records.
- `LabCloudTrail` delivered management events to `CloudTrailLogGroup`.
- `MySNSTopic` delivered notifications to a confirmed email subscriber.
- EventBridge detected the controlled security group ingress change.
- The input transformer added contextual details to the SNS notification.
- `ConsoleLoginErrors` converted matching log events into custom metric data.
- `FailedLogins` entered the alarm state after the threshold was met.
- The CloudWatch alarm published a notification to `MySNSTopic`.
- CloudWatch Logs Insights returned an aggregated view of failed console sign-in activity.

---

## 🖼️ Screenshot Evidence Plan

No screenshots were supplied with the project text. Add each screenshot only when its visible content exactly matches the filename and task description below.

| Figure Filename | Required Evidence |
|---|---|
| `Fig01 CloudTrail CreateStack Event.png` | Filtered CloudTrail Event history or the selected `CreateStack` event record. |
| `Fig02 Existing LabCloudTrail Configuration.png` | Existing `LabCloudTrail` configuration and CloudWatch Logs integration. |
| `Fig03 SNS Topic and Confirmed Subscription.png` | `MySNSTopic` and the confirmed email subscription status. |
| `Fig04 EventBridge Security Group Rule.png` | Rule name, event pattern, enabled state, and SNS target. |
| `Fig05 Security Group SSH Inbound Rule.png` | Temporary inbound SSH rule using TCP port `22`. |
| `Fig06 CloudTrail Security Group Change Event.png` | `AuthorizeSecurityGroupIngress` event with the EC2 source and port `22`. |
| `Fig07 SNS Security Group Alert Email.png` | EventBridge-generated security group notification. |
| `Fig08 CloudWatch Console Login Metric Filter.png` | Metric filter pattern and custom metric transformation. |
| `Fig09 Failed Logins Alarm Configuration.png` | Alarm threshold, period, statistic, and SNS action. |
| `Fig10 Console Login Failure Metric Graph.png` | Metric data generated by the failed-login test. |
| `Fig11 Failed Logins Alarm State.png` | `FailedLogins` alarm in the `In alarm` state. |
| `Fig12 CloudWatch Alarm Notification Email.png` | SNS email generated by the CloudWatch alarm. |
| `Fig13 CloudWatch Logs Insights Query Results.png` | Executed query and returned result fields. |

> [!WARNING]
> Before publishing screenshots, redact personal email addresses, AWS account IDs, source IP addresses, IAM ARNs, sign-in links, and other sensitive identifiers that are not required as evidence.

---

## ☑️ Validation Checklist

- [x] CloudTrail Event history filtered and reviewed
- [x] Recent `CreateStack` event analyzed
- [x] New-trail configuration workflow reviewed
- [x] Existing `LabCloudTrail` configuration examined
- [x] `MySNSTopic` created
- [x] Email subscription created and confirmed
- [x] EventBridge event pattern configured
- [x] SNS target and input transformer configured
- [x] Temporary inbound SSH rule added for testing
- [x] `AuthorizeSecurityGroupIngress` verified in CloudTrail
- [x] Security group change notification received
- [x] `ConsoleLoginErrors` metric filter created
- [x] `ConsoleLoginFailureCount` custom metric configured
- [x] `FailedLogins` alarm created
- [x] At least three failed sign-in attempts generated
- [x] Custom metric data inspected
- [x] Alarm state verified as `In alarm`
- [x] Alarm notification received
- [x] CloudWatch Logs Insights query executed

---

## 🔐 Security Considerations

- The `Anywhere-IPv4` SSH rule was created only to generate the required lab event. In production, restrict SSH access to approved source ranges or use AWS Systems Manager Session Manager.
- Broad SNS publish and subscribe permissions were lab-specific. Production topics should use least-privilege topic policies.
- Protect CloudTrail log storage with appropriate S3 access controls, encryption, retention, and integrity controls.
- Protect CloudTrail, CloudWatch Logs, EventBridge, CloudWatch alarms, and SNS resources from unauthorized modification.
- Tune alert thresholds to expected activity and documented incident-response requirements.
- Do not commit credentials, test passwords, email addresses, account IDs, IAM ARNs, source IP addresses, or sensitive links to a public repository.
- Remove temporary permissive security group rules immediately after validation.

---

## ⚠️ Limitations

- The restricted lab identity could not complete creation of a new CloudTrail trail with CloudWatch Logs enabled.
- The solution depended on the preconfigured `LabCloudTrail` trail and `CloudTrailLogGroup`.
- The EventBridge rule detected only the API actions listed in its event pattern.
- The failed-login control used a static threshold and did not correlate activity by user, source IP address, or risk context.
- Email delivery depended on confirmation of the SNS subscription.
- The lab represented a single training account rather than a centralized multi-account monitoring design.
- No screenshot files were supplied for direct inclusion in this README.

---

## 📈 Recommended Improvements

1. Deploy an organization trail for centralized multi-account logging where AWS Organizations is used.
2. Centralize CloudTrail logs in a dedicated logging account with controlled access and retention.
3. Expand detection coverage to include security group rule revocation, egress changes, IAM policy changes, CloudTrail changes, root-user activity, and unauthorized API calls.
4. Use infrastructure as code to make the controls repeatable, reviewable, and version-controlled.
5. Route alerts to an incident-management workflow in addition to email.
6. Create response runbooks for triage, containment, escalation, evidence preservation, and closure.
7. Tune alert thresholds using observed account activity without suppressing meaningful security events.
8. Include cleanup verification as part of every security control test.

---

## 🏁 Conclusion

This lab demonstrated how AWS-native services can be combined to create an auditable monitoring and alerting workflow. CloudTrail captured management activity, EventBridge detected selected EC2 security changes, CloudWatch transformed failed sign-in events into a measurable security signal, SNS delivered notifications, and CloudWatch Logs Insights supported investigation.

The completed project provides practical evidence of foundational cloud security engineering skills in audit logging, detection design, alert configuration, controlled validation, and log analysis.

---

## 📄 Disclaimer

This repository documents work completed in a restricted AWS training environment. Resource names, permissions, thresholds, and configurations were selected to satisfy the lab objectives and must be reviewed before production use.

AWS service names and trademarks belong to Amazon Web Services, Inc. This README is an original project summary and is not an official AWS publication.

---

## 👤 Author

<div align="center">

### Wadondera A. Collins

**ICDFA Trainee | Cohort 11 | Cloud Security Engineering**

Cloud Security Engineering • AWS Security Monitoring • Cloud & DevOps  
Identity and Access Management • Infrastructure Security • Incident Detection

<br>

**Project:** AWS Monitoring and Alerting with CloudTrail and CloudWatch  
**Status:** Completed

</div>
