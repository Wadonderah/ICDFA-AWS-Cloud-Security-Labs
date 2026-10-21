# LAB 6.1 TECHNICAL CONTROL VALIDATION REPORT

## AWS CloudTrail, EventBridge, CloudWatch, and SNS Monitoring Controls

---

**Report owner:** Wadondera A. Collins  
**Programme:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Assessment type:** Implementation verification and control-validation report  
**Implementation date:** 21 October 2026  
**Environment:** Restricted AWS training account  
**Primary console Region:** US East (N. Virginia), `us-east-1`  
**Document status:** Final  
**Version:** 1.0

---

## DOCUMENT AUTHORIZATION

| Role | Name | Responsibility |
|---|---|---|
| Implementer | Wadondera A. Collins | Configured, tested, and documented the monitoring controls |
| Reviewer | Not assigned | Technical verification and assessment |
| Approver | Not assigned | Final academic or organizational approval |

### Revision History

| Version | Date | Change description | Author |
|---|---|---|---|
| 1.0 | 21 October 2026 | Initial formal control-validation report | Wadondera A. Collins |

---

# MANAGEMENT SUMMARY

This report records the implementation and validation of two AWS security-monitoring controls in a restricted lab environment.

**Control A** monitored selected Amazon EC2 security group API activity through AWS CloudTrail and Amazon EventBridge. **Control B** identified repeated failed AWS Management Console sign-in attempts by converting CloudTrail log events into a custom Amazon CloudWatch metric and evaluating that metric with a CloudWatch alarm.

The evidence confirms that the existing `LabCloudTrail` trail was logging and integrated with `CloudTrailLogGroup`; `MySNSTopic` was created; `MonitorSecurityGroups` was enabled with the intended EC2 event pattern; an inbound SSH rule on TCP port 22 was added for controlled testing; the `ConsoleLoginErrors` metric filter and `ConsoleLoginFailureCount` metric were configured; the `FailedLogins` alarm reached the `In alarm` state; the SNS alarm action executed successfully; and a CloudWatch Logs Insights query returned six failed authentication events for the displayed result grouping.

The failed-login monitoring control was validated end to end. The security-group monitoring control was partially evidenced because the supplied image set does not include the EventBridge target, input transformer, matching CloudTrail event, or delivered security-group alert email.

---

# SECTION A: ENGAGEMENT DEFINITION

## A.1 Assignment Objective

The assignment required implementation of logging, event detection, notification, threshold alarming, and log investigation capabilities within an AWS account.

## A.2 Control Objectives

| Control ID | Control objective | Success criterion |
|---|---|---|
| CT-01 | Preserve account activity for investigation | CloudTrail trail is active and events are available |
| NT-01 | Establish an operator notification channel | SNS topic and email subscription workflow are configured |
| SG-01 | Detect selected EC2 security group changes | EventBridge rule is enabled with the required API event pattern |
| AU-01 | Detect repeated failed console logins | Matching events publish a custom metric and trigger an alarm at the configured threshold |
| IR-01 | Support investigation of failed logins | Logs Insights returns grouped authentication-failure results |

## A.3 Scope Boundary

### In Scope

- CloudTrail event inspection
- CloudTrail-to-CloudWatch Logs integration review
- SNS topic and email subscription workflow
- EventBridge event-pattern rule
- EC2 security group ingress change
- CloudWatch Logs metric filter
- Custom metric and alarm
- Controlled failed IAM sign-ins
- Logs Insights investigation
- Screenshot evidence analysis

### Outside Scope

- Production deployment
- Multi-account or AWS Organizations design
- Automated remediation
- Security information and event management integration
- Infrastructure-as-code deployment
- Long-term retention design
- Organizational incident escalation

## A.4 Environmental Constraint

The lab identity did not have permission to complete creation of a new CloudTrail trail with CloudWatch Logs enabled. The required configuration workflow was reviewed and cancelled. The preconfigured `LabCloudTrail`, `CloudTrailLogGroup`, and `LabCloudTrailRole` were therefore used for the operational tasks.

---

# SECTION B: SYSTEM UNDER TEST

## B.1 Resource Inventory

| Layer | AWS resource | Identifier used in the lab | Function |
|---|---|---|---|
| Audit | AWS CloudTrail trail | `LabCloudTrail` | Captures management events |
| Storage | Amazon S3 destination | Generated lab bucket | Stores trail log files |
| Log processing | CloudWatch Logs group | `CloudTrailLogGroup` | Receives CloudTrail events |
| Notification | SNS topic | `MySNSTopic` | Receives alert publications |
| Event detection | EventBridge rule | `MonitorSecurityGroups` | Matches selected EC2 API operations |
| Test resource | EC2 security group | Generated name containing `LabSecurityGroup` | Produces a controlled ingress-change event |
| Log detection | Metric filter | `ConsoleLoginErrors` | Matches failed console authentication events |
| Metric | Custom CloudWatch metric | `CloudTrailMetrics/ConsoleLoginFailureCount` | Counts matching log events |
| Alarm | CloudWatch alarm | `FailedLogins` | Evaluates failure count against threshold |
| Identity test | IAM user | `test` | Produces controlled failed sign-in events |

## B.2 Data and Alert Flows

### Flow 1: Security Group Change

```text
EC2 security group API call
          -> CloudTrail event
          -> EventBridge rule evaluation
          -> SNS topic
          -> Email subscription
```

### Flow 2: Repeated Failed Sign-In

```text
Failed console login
          -> CloudTrail event
          -> CloudWatch Logs
          -> Metric filter
          -> Custom metric
          -> CloudWatch alarm
          -> SNS topic
```

### Flow 3: Investigation

```text
CloudTrailLogGroup
          -> Logs Insights query
          -> Grouped failure result
```

---

# SECTION C: CONTROL SPECIFICATIONS

## C.1 Security Group Monitoring Rule

The EventBridge rule matched CloudTrail API events with the following pattern:

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

The intended event transformation used:

```json
{
  "name": "$.detail.requestParameters.groupId",
  "source": "$.detail.eventName",
  "time": "$.time",
  "value": "$.detail"
}
```

```text
"The <source> API call was made against the <name> security group on <time> with the following details:"
" <value> "
```

## C.2 Failed Login Metric Filter

```text
{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }
```

| Property | Configured value |
|---|---|
| Filter name | `ConsoleLoginErrors` |
| Metric namespace | `CloudTrailMetrics` |
| Metric name | `ConsoleLoginFailureCount` |
| Metric value | `1` |

## C.3 Alarm Logic

| Property | Configured value |
|---|---|
| Alarm name | `FailedLogins` |
| Metric | `ConsoleLoginFailureCount` |
| Statistic | `Sum` |
| Period | `5 minutes` |
| Comparison | Greater than or equal to |
| Threshold | `3` |
| Evaluation | `1 datapoint` |
| Action | Publish to `MySNSTopic` |

## C.4 Investigation Query

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

---

# SECTION D: EXECUTION LOG

## D.1 Audit Layer Verification

### Activity

A recent CloudFormation `CreateStack` event was opened in CloudTrail Event history. The event view exposed the event name, event source, time, user information, source IP, AWS Region, request ID, event ID, referenced stack, and JSON event body.

### Evidence

<img width="1176" height="563" alt="Fig01 CloudTrail CreateStack Event Details" src="https://github.com/user-attachments/assets/1e1a15ac-6b45-4256-8c29-92f48ea41be7" />


**Figure 1. CloudTrail CreateStack Event Details.** The event source is `cloudformation.amazonaws.com`, the event name is `CreateStack`, and the visible event record provides the audit context required for investigation. The AWS Config access-denied banner is a separate lab-permission limitation.

### Verification Decision

**Pass.** CloudTrail management-event details were available for analysis.

---

## D.2 Trail Delivery Verification

### Activity

The existing `LabCloudTrail` configuration was reviewed to confirm logging and CloudWatch Logs integration.

### Evidence

<img width="1334" height="618" alt="Fig02 LabCloudTrail Logging and CloudWatch Integration" src="https://github.com/user-attachments/assets/9dc9130b-c097-47b9-94be-934e87c84849" />


**Figure 2. LabCloudTrail Logging and CloudWatch Integration.** The trail page shows `Logging`, multi-Region operation, log-file validation, an S3 log location, `CloudTrailLogGroup`, and `LabCloudTrailRole`. Log-file SSE-KMS encryption is shown as not enabled.

### Verification Decision

**Pass.** The existing trail provided the audit feed required by later control stages.

---

## D.3 Notification Channel Preparation

### Activity 1: Topic Creation

<img width="1356" height="591" alt="Fig03 MySNSTopic Created Successfully" src="https://github.com/user-attachments/assets/7a08af35-a036-4859-ad95-f67b9dd923af" />


**Figure 3. MySNSTopic Created Successfully.** The success banner confirms creation of the standard SNS topic. The screenshot shows zero subscriptions at this point and is used only as topic-creation evidence.

### Activity 2: Subscription Request

<img width="701" height="336" alt="Fig04 SNS Email Subscription Pending Confirmation" src="https://github.com/user-attachments/assets/6e6dc5be-2397-47d2-8197-ebe0bac96964" />


**Figure 4. SNS Email Subscription Pending Confirmation.** The subscription uses the `EMAIL` protocol and is visibly pending confirmation. The personal endpoint must be redacted before publication.

### Activity 3: Confirmation Initiation

<img width="1359" height="627" alt="Fig05 AWS SNS Subscription Confirmation Email" src="https://github.com/user-attachments/assets/94dc0f2f-500e-4bc4-b081-7ea39f52df2b" />


**Figure 5. AWS SNS Subscription Confirmation Email.** AWS Notifications delivered the confirmation message for `MySNSTopic`.

<img width="1353" height="607" alt="Fig06 AWS SNS Confirm Subscription Page" src="https://github.com/user-attachments/assets/77087422-dbfc-4c98-9b7c-20906948d525" />


**Figure 6. AWS SNS Confirm Subscription Page.** The confirmation page was reached. The image shows the confirmation action rather than the post-confirmation success message.

### Verification Decision

**Partial pass.** Topic creation and the subscription-confirmation workflow are evidenced. A final SNS console state of `Confirmed` is not visible in the supplied screenshots.

---

## D.4 Security Group Detection Configuration

### Activity 1: Rule Creation

<img width="1349" height="619" alt="Fig07 EventBridge MonitorSecurityGroups Rule" src="https://github.com/user-attachments/assets/078667ac-61eb-4ba1-99b0-2e2dd4b2854e" />


**Figure 7. EventBridge MonitorSecurityGroups Rule.** The rule is named `MonitorSecurityGroups`, is enabled on the default event bus, and displays the required EC2 API event pattern.

### Activity 2: Controlled Event Generation

<img width="1359" height="564" alt="Fig08 EC2 Security Group SSH Inbound Rule" src="https://github.com/user-attachments/assets/ddcd6729-32ae-4364-be21-af40bcdcc1d9" />


**Figure 8. EC2 Security Group SSH Inbound Rule.** The security group contains an SSH rule for TCP port 22 from `0.0.0.0/0`, confirming the controlled configuration change. An HTTP rule on TCP port 80 is also visible.

### Verification Decision

**Partial pass.** The enabled rule and event-generating change are evidenced. The supplied files do not show the EventBridge target, input transformer, matching `AuthorizeSecurityGroupIngress` CloudTrail record, or alert email.

---

## D.5 Failed Login Detection Configuration

### Activity 1: Metric Filter

<img width="1356" height="488" alt="Fig09 CloudWatch ConsoleLoginErrors Metric Filter" src="https://github.com/user-attachments/assets/c19e9d21-385c-4282-9b49-ee088957b587" />


**Figure 9. CloudWatch ConsoleLoginErrors Metric Filter.** The review page shows the failed-authentication filter pattern, `ConsoleLoginErrors`, `CloudTrailMetrics`, `ConsoleLoginFailureCount`, and a metric value of `1`.

### Activity 2: Alarm Threshold

<img width="721" height="252" alt="Fig10 Failed Login Alarm Threshold Configuration" src="https://github.com/user-attachments/assets/eb5a814e-1269-48df-b0ab-a6fb1eb7141b" />


**Figure 10. Failed Login Alarm Threshold Configuration.** The configuration uses the `Sum` statistic, a five-minute period, a static greater-than-or-equal comparison, and a threshold of `3`.

### Activity 3: Alarm Creation

<img width="2" height="1" alt="Fig11 FailedLogins Alarm Created" src="https://github.com/user-attachments/assets/95816cab-759e-46a7-867b-156634898897" />


**Figure 11. FailedLogins Alarm Created.** The alarm initially shows `Insufficient data`, actions enabled, and the condition `ConsoleLoginFailureCount >= 3 for 1 datapoints within 5 minutes`. The page also warns that some SNS subscriptions are pending confirmation.

### Verification Decision

**Pass for configuration.** The filter, metric transformation, and alarm condition are visibly configured.

---

## D.6 Controlled Authentication Test

### Activity

Incorrect credentials were submitted for the lab IAM user to create failed console authentication events.

### Evidence

<img width="1359" height="447" alt="Fig12 IAM Test User Authentication Failed" src="https://github.com/user-attachments/assets/9d9001e9-691d-46a1-8045-48297235de9d" />


**Figure 12. IAM Test User Authentication Failed.** The AWS sign-in page displays `Authentication failed` and states that the authentication information is incorrect.

### Verification Decision

**Pass.** A failed sign-in event was generated. No password is reproduced in this report.

---

## D.7 Metric and Alarm Validation

### Activity 1: Metric Verification

<img width="1359" height="583" alt="Fig13 ConsoleLoginFailureCount Custom Metric" src="https://github.com/user-attachments/assets/caf22059-8576-4d39-a741-7b73a4072bdf" />


**Figure 13. ConsoleLoginFailureCount Custom Metric.** The metric is visible under `CloudTrailMetrics`, has graph data, and shows one associated alarm.

### Activity 2: Alarm-State Verification

<img width="1359" height="533" alt="Fig14 FailedLogins Alarm In Alarm State" src="https://github.com/user-attachments/assets/645a9096-e4b8-4a8c-a354-5fbdf8d1022b" />


**Figure 14. FailedLogins Alarm In Alarm State.** The alarm is visibly `In alarm`; the metric reaches the threshold line at three; and alarm history records both the state transition and successful SNS action execution.

### Verification Decision

**Pass.** The metric-based control operated from matching log events through alarm evaluation and SNS action invocation.

---

## D.8 Log Investigation

### Activity

The CloudWatch Logs Insights query was executed against the CloudTrail log data.

### Evidence

<img width="1359" height="562" alt="Fig15 CloudWatch Logs Insights Failed Login Query" src="https://github.com/user-attachments/assets/d09002f3-e327-475c-a3c2-b7646d4936d4" />


**Figure 15. CloudWatch Logs Insights Failed Login Query.** The query completed and displayed a grouped result with `Failed authentication`, Region `eu-north-1`, and a total count of `6`. The displayed source IP and identity fields should be redacted in a public repository.

### Verification Decision

**Pass.** Failed console login events were successfully retrieved and aggregated for investigation.

---

# SECTION E: TEST RESULTS SCHEDULE

| Test ID | Test objective | Expected result | Actual result | Evidence | Verdict |
|---|---|---|---|---|---|
| T-01 | Inspect an AWS management event | CloudTrail displays event context and JSON | Event details were visible | Fig. 1 | Pass |
| T-02 | Verify trail delivery configuration | Trail is logging to the required destinations | Logging and CloudWatch integration were visible | Fig. 2 | Pass |
| T-03 | Establish SNS notification topic | Topic exists | `MySNSTopic` created successfully | Fig. 3 | Pass |
| T-04 | Establish email subscription | Subscription progresses to confirmation | Pending state, email, and confirmation page visible | Figs. 4–6 | Partial |
| T-05 | Configure EC2 change detection | Enabled EventBridge rule contains required pattern | Rule and pattern visible | Fig. 7 | Pass |
| T-06 | Generate monitored EC2 change | SSH ingress rule is added | TCP 22 from `0.0.0.0/0` visible | Fig. 8 | Pass |
| T-07 | Convert failed logins into a metric | Filter maps each match to value 1 | Filter and transformation visible | Fig. 9 | Pass |
| T-08 | Configure alarm threshold | Alarm evaluates three failures in five minutes | Threshold and period visible | Fig. 10 | Pass |
| T-09 | Create the alarm | Alarm exists with actions enabled | `FailedLogins` created | Fig. 11 | Pass |
| T-10 | Generate authentication failures | AWS rejects incorrect sign-in | Authentication failure visible | Fig. 12 | Pass |
| T-11 | Publish metric data | Custom metric receives values | Metric and graph data visible | Fig. 13 | Pass |
| T-12 | Activate alarm and action | Alarm changes to alarm state and invokes SNS | State and successful action recorded | Fig. 14 | Pass |
| T-13 | Investigate failed logins | Query returns grouped failures | Six failures displayed for the result grouping | Fig. 15 | Pass |

---

# SECTION F: CONTROL EFFECTIVENESS ASSESSMENT

## F.1 Effective Controls

- CloudTrail management-event visibility
- Active CloudTrail logging
- CloudWatch Logs integration
- EventBridge event-pattern detection
- Failed-login metric filtering
- Custom metric publication
- Static threshold evaluation
- CloudWatch alarm-state transition
- SNS alarm action invocation
- Logs Insights investigation

## F.2 Partially Evidenced Controls

### SNS Email Subscription

The evidence shows the pending subscription, the confirmation email, and the confirmation page. It does not show a final `Confirmed` status.

### Security Group Alert Delivery

The rule and security group change are visible. The target configuration, transformed payload, matching CloudTrail event, and delivered email are absent from the evidence set.

## F.3 Evidence Gaps

- Final SNS subscription status
- EventBridge target tab
- EventBridge input transformer
- CloudTrail `AuthorizeSecurityGroupIngress` record
- Security group alert email
- CloudWatch alarm notification email

---

# SECTION G: RISK AND TREATMENT REGISTER

| Risk ID | Observation | Security impact | Recommended treatment | Priority |
|---|---|---|---|---|
| RSK-01 | SSH open to `0.0.0.0/0` | Broad network exposure on TCP port 22 | Remove after testing; restrict to approved sources or use Systems Manager Session Manager | High |
| RSK-02 | Broad SNS publish and subscribe settings used in the lab | Unauthorized publication or subscription if copied to production | Apply a least-privilege SNS topic policy | High |
| RSK-03 | Subscription pending warning visible | Email endpoint may not receive alerts | Confirm subscription and perform a delivery test | High |
| RSK-04 | Static threshold of three failures | Possible false positives or missed distributed attempts | Tune with production authentication patterns and risk context | Medium |
| RSK-05 | Sensitive identifiers visible in screenshots | Account and identity data exposure in a public repository | Redact account IDs, ARNs, email addresses, source IPs, and resource IDs | High |
| RSK-06 | CloudTrail log-file SSE-KMS encryption not enabled in the visible lab trail | Encryption configuration may not meet production requirements | Apply the organization's approved S3 and KMS protection design | Medium |

---

# SECTION H: EXCEPTIONS AND DISCREPANCIES

## H.1 Trail-Creation Exception

The restricted identity could not create the new trail requested by the exercise. This was an expected lab constraint, and the preconfigured trail supplied the required event feed.

## H.2 Rule-Name Discrepancy

The procedural steps specify `MonitorSecurityGroups`, which is the name shown in the evidence. An embedded grading note refers to `MonitoringRule`. The submission report should be reviewed if automated grading did not award the rule criterion.

## H.3 Regional Observation

The AWS console resources were visibly managed in `us-east-1`. The Logs Insights result displayed `eu-north-1` in the event's `AWS_Region` output field. Both values are recorded as shown without treating the event field as a claim about the deployment Region.

## H.4 Blank Attachment

`Screenshot 2026-10-21 185908.png` contains no readable technical evidence. The file is excluded from the numbered figure sequence.

---

# SECTION I: EVIDENCE CATALOGUE

## I.1 Approved Figure Names

| Figure | Rename screenshot to | Evidence purpose |
|---:|---|---|
| 01 | `Fig01 CloudTrail CreateStack Event Details.png` | CloudTrail event analysis |
| 02 | `Fig02 LabCloudTrail Logging and CloudWatch Integration.png` | Trail and log-group integration |
| 03 | `Fig03 MySNSTopic Created Successfully.png` | SNS topic creation |
| 04 | `Fig04 SNS Email Subscription Pending Confirmation.png` | Subscription request state |
| 05 | `Fig05 AWS SNS Subscription Confirmation Email.png` | Confirmation message delivery |
| 06 | `Fig06 AWS SNS Confirm Subscription Page.png` | Confirmation workflow |
| 07 | `Fig07 EventBridge MonitorSecurityGroups Rule.png` | Enabled rule and event pattern |
| 08 | `Fig08 EC2 Security Group SSH Inbound Rule.png` | Controlled ingress change |
| 09 | `Fig09 CloudWatch ConsoleLoginErrors Metric Filter.png` | Filter and metric transformation |
| 10 | `Fig10 Failed Login Alarm Threshold Configuration.png` | Alarm logic |
| 11 | `Fig11 FailedLogins Alarm Created.png` | Alarm creation and initial state |
| 12 | `Fig12 IAM Test User Authentication Failed.png` | Failed sign-in test |
| 13 | `Fig13 ConsoleLoginFailureCount Custom Metric.png` | Metric publication |
| 14 | `Fig14 FailedLogins Alarm In Alarm State.png` | Alarm activation and SNS action |
| 15 | `Fig15 CloudWatch Logs Insights Failed Login Query.png` | Authentication investigation |

## I.2 Evidence Directory

```text
evidence/
├── Fig01 CloudTrail CreateStack Event Details.png
├── Fig02 LabCloudTrail Logging and CloudWatch Integration.png
├── Fig03 MySNSTopic Created Successfully.png
├── Fig04 SNS Email Subscription Pending Confirmation.png
├── Fig05 AWS SNS Subscription Confirmation Email.png
├── Fig06 AWS SNS Confirm Subscription Page.png
├── Fig07 EventBridge MonitorSecurityGroups Rule.png
├── Fig08 EC2 Security Group SSH Inbound Rule.png
├── Fig09 CloudWatch ConsoleLoginErrors Metric Filter.png
├── Fig10 Failed Login Alarm Threshold Configuration.png
├── Fig11 FailedLogins Alarm Created.png
├── Fig12 IAM Test User Authentication Failed.png
├── Fig13 ConsoleLoginFailureCount Custom Metric.png
├── Fig14 FailedLogins Alarm In Alarm State.png
└── Fig15 CloudWatch Logs Insights Failed Login Query.png
```

---

# SECTION J: REMEDIATION AND IMPROVEMENT PLAN

| Action | Desired outcome | Priority |
|---|---|---|
| Remove the unrestricted SSH test rule | Restore least network exposure | Immediate |
| Confirm SNS endpoint and test delivery | Ensure notifications reach the operator | Immediate |
| Capture missing EventBridge target and transformer evidence | Complete control traceability | High |
| Capture the matching CloudTrail security group event | Prove event-pattern matching input | High |
| Capture notification emails after redaction | Complete end-to-end evidence | High |
| Restrict SNS topic policy | Enforce least privilege | High |
| Expand monitored API operations | Improve change-detection coverage | Medium |
| Add incident-response runbooks | Convert alerts into operational action | Medium |
| Deploy with infrastructure as code | Improve repeatability and review | Medium |
| Centralize multi-account CloudTrail logs | Support enterprise-scale visibility | Future design |

---

# SECTION K: COMPETENCY EVIDENCE

| Competency | Evidence demonstrated |
|---|---|
| AWS audit logging | Reviewed CloudTrail event records and trail settings |
| Detection engineering | Implemented event-pattern and metric-filter logic |
| Network security monitoring | Generated and monitored an EC2 ingress change |
| Identity monitoring | Generated and detected failed console sign-ins |
| Alert engineering | Configured metric, threshold, alarm, and SNS action |
| Log analytics | Queried and grouped failed authentication events |
| Validation | Compared expected behavior with observed evidence |
| Security analysis | Identified exposure, policy, delivery, and evidence-handling risks |
| Technical reporting | Produced a traceable formal control-validation record |

---

# SECTION L: FINAL ASSURANCE STATEMENT

The available evidence supports the conclusion that the failed-console-login monitoring control operated successfully from log matching through metric publication, threshold evaluation, alarm activation, and SNS action execution.

The security-group monitoring control was configured and a matching type of EC2 change was generated. Full end-to-end assurance for that path requires the missing target configuration, matching CloudTrail event, and delivered notification evidence.

The implementation met the central learning goals of the lab: audit-event analysis, event-driven detection, metric-based alerting, notification integration, and interactive log investigation.

---

# APPENDIX A: COMPLETION ATTESTATION

## Configuration Status

- [x] CloudTrail event inspected
- [x] Existing trail verified
- [x] CloudWatch Logs integration verified
- [x] SNS topic created
- [x] SNS email subscription initiated
- [x] EventBridge rule enabled
- [x] Security group change generated
- [x] Metric filter configured
- [x] Custom metric published
- [x] Failed-login alarm configured
- [x] Authentication failures generated
- [x] Alarm state validated
- [x] SNS alarm action execution validated
- [x] Logs Insights query executed

## Evidence Status

- [x] Fifteen usable screenshots mapped to matching controls
- [x] Figure names follow `Fig01 Descriptive Name.png`
- [x] Each figure includes an analytical caption
- [x] Unsupported claims are identified as evidence gaps
- [x] Blank attachment excluded
- [ ] Sensitive screenshot fields redacted before public release
- [ ] Final SNS-confirmed state captured
- [ ] Missing notification evidence captured

---

# APPENDIX B: AUTHOR DECLARATION

I, **Wadondera A. Collins**, prepared this report as a factual record of the configuration, testing, evidence, results, limitations, and security observations associated with Lab 6.1.

**Name:** Wadondera A. Collins  
**Programme:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Date:** 21 October 2026  
**Signature:** ______________________________

---

# APPENDIX C: PUBLICATION NOTICE

This report documents a restricted AWS training environment. Before publication, redact personal email addresses, AWS account identifiers, ARNs, source IP addresses, resource IDs, and sign-in details from the screenshot files.

The Markdown image references assume that renamed screenshots are stored in an `evidence` directory beside `Documentation_Report.md`.

AWS service names and trademarks belong to Amazon Web Services, Inc. This is an independent educational control-validation report and is not an official AWS publication.
