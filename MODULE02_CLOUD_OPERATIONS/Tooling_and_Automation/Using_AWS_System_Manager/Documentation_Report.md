<div align="center">

# AWS Systems Manager Operations and Secure Instance Management

## Professional Technical Documentation Report

**Module 2: Lab 1 - Using AWS Systems Manager**

**Prepared by:** Wadondera A. Collins  
**Program:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Cloud Platform:** Amazon Web Services (AWS)  
**Region:** US East (N. Virginia) | `us-east-1`  
**Completion Date:** 29 October 2026  
**Project Status:** Completed

</div>

---

## Document Control

| Field | Details |
|---|---|
| Document type | Technical implementation and validation report |
| Project | AWS Systems Manager Operations and Secure Instance Management |
| Module | Module 2: Lab 1 |
| Author | Wadondera A. Collins |
| Environment | AWS lab environment |
| Primary service | AWS Systems Manager |
| Managed workload | Amazon EC2 managed instance |
| Evidence set | 10 task-aligned screenshots |
| Intended audience | Technical reviewers, recruiters, cloud engineers, and security practitioners |

> **Portfolio note:** This documentation report is intentionally structured as a formal implementation record. It is separate from, and should not be presented as, the repository `README.md`. A README provides a concise repository overview; this report records the methodology, evidence, validation, security observations, limitations, and technical outcomes.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Project Context](#2-project-context)
3. [Objectives and Success Criteria](#3-objectives-and-success-criteria)
4. [Technical Environment](#4-technical-environment)
5. [Architecture and Service Relationships](#5-architecture-and-service-relationships)
6. [Implementation Methodology](#6-implementation-methodology)
7. [Task 1 - Systems Manager Inventory](#7-task-1---systems-manager-inventory)
8. [Task 2 - Application Deployment with Run Command](#8-task-2---application-deployment-with-run-command)
9. [Task 3 - Application Configuration with Parameter Store](#9-task-3---application-configuration-with-parameter-store)
10. [Task 4 - Secure Access with Session Manager](#10-task-4---secure-access-with-session-manager)
11. [Results and Validation Matrix](#11-results-and-validation-matrix)
12. [Security Analysis](#12-security-analysis)
13. [Operational Value](#13-operational-value)
14. [Limitations and Evidence Gaps](#14-limitations-and-evidence-gaps)
15. [Recommended Enhancements](#15-recommended-enhancements)
16. [Screenshot Register and Rename Plan](#16-screenshot-register-and-rename-plan)
17. [Conclusion](#17-conclusion)
18. [Disclaimer](#18-disclaimer)
19. [Author](#19-author)

---

## 1. Executive Summary

This project demonstrated centralized administration of an Amazon EC2 instance through four AWS Systems Manager capabilities: Inventory, Run Command, Parameter Store, and Session Manager. The implementation gathered instance software metadata, deployed a web-based manufacturing dashboard without an interactive SSH login, controlled an application feature through a centrally stored parameter, and established a browser-based administrative session to inspect the instance and query EC2 information.

The completed workflow shows how AWS Systems Manager can reduce dependence on direct server access while supporting repeatable operations, centralized configuration, operational visibility, and auditable administration. The strongest security outcome was the use of Session Manager for shell access without relying on an exposed SSH service or locally managed SSH keys.

The supplied screenshots provide evidence for all four tasks. One evidence item shows Run Command while the execution was still **In Progress**, rather than its final **Success** status. The successfully rendered Widget Manufacturing Dashboard provides outcome-based confirmation that the deployment completed, but a dedicated final command-success screenshot would strengthen the audit trail.

---

## 2. Project Context

AWS Systems Manager provides centralized operational capabilities for managed compute resources. In this lab, a preconfigured Amazon EC2 instance registered as **Managed Instance** was administered through the AWS Management Console.

The implementation covered the following operational lifecycle:

1. Discover the managed node and inspect collected software inventory.
2. Select a custom Systems Manager document for application installation.
3. invoke the document through Run Command against the managed instance.
4. Validate the deployed Widget Manufacturing Dashboard in a browser.
5. Create an application feature flag in Parameter Store.
6. Compare the dashboard before and after activation of the beta feature.
7. Connect through Session Manager and validate instance-level access.

---

## 3. Objectives and Success Criteria

### 3.1 Project Objectives

- Establish visibility into the configuration and installed software of a managed EC2 instance.
- Use Systems Manager Run Command to deploy an application without manually opening an SSH session.
- Store and manage an application configuration value through Parameter Store.
- Demonstrate feature activation through centrally managed configuration.
- Access the instance through Session Manager using a browser-based shell.
- Validate the deployed files and retrieve EC2 instance information from the command line.
- Document each implementation stage with accurately matched screenshot evidence.

### 3.2 Success Criteria

| Objective | Validation requirement | Result |
|---|---|---|
| Managed-node visibility | Managed Instance is online and visible in Fleet Manager | Met |
| Inventory collection | AWS application inventory is displayed for the instance | Met |
| Command document selection | InstallDashboardApp document is identified | Met |
| Remote deployment | Run Command is submitted to the managed instance | Met with evidence caveat |
| Application validation | Widget Manufacturing Dashboard loads in a browser | Met |
| Parameter creation | `/dashboard/show-beta-features` appears in Parameter Store | Met |
| Feature-flag behavior | Dashboard changes from two visualizations to three | Met |
| Secure shell access | Session Manager opens an interactive session | Met |
| Instance inspection | Application files and EC2 details are returned | Met |

---

## 4. Technical Environment

| Component | Implementation |
|---|---|
| Cloud provider | Amazon Web Services |
| AWS Region | US East (N. Virginia), `us-east-1` |
| Compute | Amazon EC2 |
| Operating system | Amazon Linux |
| Management plane | AWS Systems Manager |
| Managed-node interface | Fleet Manager |
| Configuration inventory | Systems Manager Inventory |
| Remote execution | Systems Manager Run Command |
| Configuration storage | Systems Manager Parameter Store |
| Interactive access | Systems Manager Session Manager |
| Deployment artifact | Custom Systems Manager document containing `InstallDashboardApp` in its name |
| Application | Widget Manufacturing Dashboard |
| Command-line tooling | Linux shell and AWS CLI |
| Session channel | Browser-based Session Manager shell |

### 4.1 Commands Used

```bash
ls /var/www/html
```

```bash
# Determine the Availability Zone and derive the AWS Region
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
export AWS_DEFAULT_REGION=${AZ::-1}

# Retrieve EC2 instance information
aws ec2 describe-instances
```

> **Security improvement:** In production, use Instance Metadata Service Version 2 (IMDSv2) and its session-token workflow rather than an unauthenticated metadata request.

---

## 5. Architecture and Service Relationships

```text
Administrator
    |
    v
AWS Management Console
    |
    +--> Fleet Manager ------> Managed EC2 instance and SSM Agent status
    |
    +--> Inventory ----------> Installed software and instance metadata
    |
    +--> Run Command --------> Custom InstallDashboardApp document
    |                              |
    |                              v
    |                        Web application deployment
    |
    +--> Parameter Store ----> /dashboard/show-beta-features = True
    |                              |
    |                              v
    |                        Beta chart enabled in dashboard
    |
    +--> Session Manager ----> Browser shell on managed instance
                                   |
                                   v
                             File and EC2 validation
```

The SSM Agent on the EC2 instance acts as the management channel between the managed node and AWS Systems Manager. Run Command performs non-interactive execution, Parameter Store supplies configuration data, Inventory provides metadata visibility, and Session Manager provides interactive administration.

---

## 6. Implementation Methodology

The project followed an evidence-driven validation method. Each task was completed in the AWS Management Console, followed by a check that confirmed either the configured resource, the command state, the application behavior, or the interactive shell output.

The evidence was reviewed according to three rules:

1. A screenshot must be placed only under the task visibly represented by that screenshot.
2. A filename, Markdown link, caption, and analysis must describe the same visible state.
3. A screenshot of an intermediate state must not be described as proof of a final state.

---

## 7. Task 1 - Systems Manager Inventory

### 7.1 Purpose

The first task established that the EC2 instance was registered as a managed node and that Systems Manager could collect installed-application metadata without requiring a direct login to the instance.

### 7.2 Procedure Performed

1. Opened AWS Systems Manager.
2. Navigated to Fleet Manager.
3. Confirmed that **Managed Instance** was running and online.
4. Configured an inventory association named `Inventory-Association`.
5. Targeted the managed instance manually.
6. Opened the instance inventory and reviewed the `AWS:Application` inventory type.

### 7.3 Evidence A - Managed Node Registration

![Fleet Manager showing the online managed instance](screenshots/Fig01%20Fleet%20Manager%20Managed%20Instance%20Online.png)

**Figure 1: Fleet Manager showing the managed EC2 node in an online state.**

**Figure analysis:** The Fleet Manager view contains one managed node named **Managed Instance**. The node is displayed as running, the platform is Linux, the operating system is Amazon Linux, and the ping status is online. The displayed SSM Agent version confirms that the management agent is installed and communicating with Systems Manager. This evidence establishes the prerequisite for Inventory, Run Command, and Session Manager operations.

### 7.4 Evidence B - Collected Application Inventory

![Systems Manager application inventory for the managed instance](screenshots/Fig02%20Systems%20Manager%20Application%20Inventory.png)

**Figure 2: `AWS:Application` inventory collected from the managed instance.**

**Figure analysis:** The instance Inventory tab displays the `AWS:Application` inventory type and a populated software list. Visible entries include system libraries, base packages, development libraries, and shells. The interface reports hundreds of application records, confirming that the inventory association collected software metadata from the managed node.

### 7.5 Task Outcome

Systems Manager successfully provided centralized software visibility for the managed instance. This supports configuration review and software discovery without requiring an individual administrator to connect to the operating system.

---

## 8. Task 2 - Application Deployment with Run Command

### 8.1 Purpose

The second task used Run Command to install the Widget Manufacturing Dashboard on the managed EC2 instance without opening an interactive SSH connection.

### 8.2 Procedure Performed

1. Opened Systems Manager Run Command.
2. Filtered command documents by **Owned by me**.
3. Located the custom document containing `InstallDashboardApp` in the name.
4. Reviewed the document details.
5. Selected **Managed Instance** as the target.
6. Submitted the command.
7. Opened the instance address in a browser to validate the deployed application.

### 8.3 Evidence A - Command Document Selection

![Run Command page showing the custom dashboard installation document](screenshots/Fig03%20Run%20Command%20InstallDashboardApp%20Selection.png)

**Figure 3: Custom `InstallDashboardApp` command document selected in Run Command.**

**Figure analysis:** The Run Command workflow is filtered to documents owned by the lab account and displays a custom document whose name includes `InstallDashboardApp`. The page also shows command-parameter fields, indicating that the document is ready for configuration and targeting.

### 8.4 Evidence B - Document Details

![Systems Manager document details for InstallDashboardApp](screenshots/Fig04%20InstallDashboardApp%20Document%20Details.png)

**Figure 4: Systems Manager document metadata for the dashboard installer.**

**Figure analysis:** The document details page shows the description **Install Dashboard App**, supported platforms of Linux and macOS, an active status, and a default document version. This confirms that the selected deployment artifact was the intended custom installation document.

### 8.5 Evidence C - Command Submission

![Run Command execution in progress for the managed instance](screenshots/Fig05%20Run%20Command%20Execution%20In%20Progress.png)

**Figure 5: Run Command submitted to the managed instance and shown in progress.**

**Figure analysis:** A green notification confirms that the command was successfully sent. The command status and target status are both visibly **In Progress**, with one target listed. This image proves command submission and target association, but does not independently prove final successful completion.

### 8.6 Evidence D - Deployed Application

![Widget Manufacturing Dashboard running after deployment](screenshots/Fig06%20Dashboard%20Before%20Beta%20Feature%20Activation.png)

**Figure 6: Widget Manufacturing Dashboard rendered in the browser.**

**Figure analysis:** The browser displays the Widget Manufacturing Dashboard with the output gauge and manufacturing-stage chart. At this point, only the two standard visualizations are visible, which is consistent with the beta feature not yet being active. The responsive web page provides outcome-based evidence that the application and its supporting web components were installed and served by the instance.

### 8.7 Task Outcome

The application became accessible through a web browser after remote execution of the custom Systems Manager document. The result demonstrates agent-based application deployment without a manual SSH login.

---

## 9. Task 3 - Application Configuration with Parameter Store

### 9.1 Purpose

The third task demonstrated externalized application configuration. A Parameter Store value was used as a feature flag to control whether the dashboard displayed an additional beta visualization.

### 9.2 Configuration Applied

| Field | Value |
|---|---|
| Parameter name | `/dashboard/show-beta-features` |
| Description | Display beta features |
| Value | `True` |
| Visible parameter type | String |
| Visible tier | Standard |

### 9.3 Evidence A - Baseline Application State

![Dashboard before the beta feature was enabled](screenshots/Fig06%20Dashboard%20Before%20Beta%20Feature%20Activation.png)

**Figure 6: Baseline dashboard displaying two visualizations before beta-feature activation.**

**Figure analysis:** The dashboard displays the output gauge and production-stage chart only. No shipping-throughput beta chart is visible. This screenshot is the required baseline for comparing the application's behavior after the parameter is created.

### 9.4 Evidence B - Parameter Creation

![Parameter Store showing the beta feature parameter](screenshots/Fig07%20Parameter%20Store%20Beta%20Feature%20Flag.png)

**Figure 7: Parameter Store entry for `/dashboard/show-beta-features`.**

**Figure analysis:** A success notification indicates that the create-parameter request succeeded. The parameter list contains `/dashboard/show-beta-features`, with Standard tier and String type visible. This confirms that the feature-flag configuration was centrally stored in Systems Manager Parameter Store.

### 9.5 Evidence C - Feature Activation

![Dashboard after the beta feature was enabled](screenshots/Fig08%20Dashboard%20After%20Beta%20Feature%20Activation.png)

**Figure 8: Dashboard displaying the additional shipping-throughput beta chart.**

**Figure analysis:** Following refresh, the application displays a third visualization titled **Shipping Throughput (Beta feature)** alongside the two baseline charts. The before-and-after comparison demonstrates that the application consumed the Parameter Store setting and changed its presentation without requiring application redeployment.

### 9.6 Task Outcome

The feature flag successfully separated deployment from feature activation. This pattern supports staged releases and operational control, although production use should add formal change approval, parameter naming standards, and access controls.

---

## 10. Task 4 - Secure Access with Session Manager

### 10.1 Purpose

The final task established browser-based access to the managed instance through Session Manager and used the shell to validate application files and retrieve EC2 information.

### 10.2 Procedure Performed

1. Opened Session Manager.
2. Started a session against **Managed Instance**.
3. Listed the contents of `/var/www/html`.
4. Derived the AWS Region from instance metadata.
5. Exported the Region as `AWS_DEFAULT_REGION`.
6. Ran `aws ec2 describe-instances`.
7. Reviewed the returned EC2 data.

### 10.3 Evidence A - Session Initiation

![Session Manager start-session workflow for the managed instance](screenshots/Fig09%20Session%20Manager%20Managed%20Instance%20Selection.png)

**Figure 9: Session Manager workflow used to select the managed instance and initiate browser-based access.**

**Figure analysis:** This screenshot records the Session Manager connection stage before command execution. It belongs under Task 4 because it documents the selection and initiation of the managed session, rather than Inventory, Run Command, or Parameter Store.

### 10.4 Evidence B - Browser-Based Session

![Session Manager shell showing application files and EC2 instance output](screenshots/Fig10%20Session%20Manager%20Instance%20Validation.png)

**Figure 10: Session Manager shell used to validate deployed files and query EC2 information.**

**Figure analysis:** The browser-based session shows the contents of `/var/www/html`, including dashboard application files. The shell subsequently runs the regional setup commands and `aws ec2 describe-instances`, which returns structured EC2 information. This verifies that Session Manager provided interactive operating-system access and that the instance role or lab credentials permitted the EC2 describe operation.

### 10.5 Task Outcome

The instance was administered through a Systems Manager session rather than a traditional SSH workflow. This approach removes the need to distribute SSH keys to lab users and can support centralized access control and activity logging when the required IAM and logging configuration is enabled.

---

## 11. Results and Validation Matrix

| Validation area | Evidence | Observed result | Status |
|---|---|---|---|
| SSM-managed node | Figure 1 | Managed Instance is running and online | Passed |
| Software inventory | Figure 2 | `AWS:Application` records are populated | Passed |
| Custom command document | Figures 3-4 | Correct dashboard installation document identified | Passed |
| Command targeting | Figure 5 | Command sent to one managed target | Passed |
| Final command status | Figure 5 | Screenshot remains at **In Progress** | Evidence gap |
| Web deployment and baseline state | Figure 6 | Dashboard loads with two standard visualizations | Passed |
| Parameter creation | Figure 7 | Feature parameter appears after successful creation | Passed |
| Dynamic feature activation | Figure 8 | Third beta visualization appears | Passed |
| Session initiation | Figure 9 | Managed-instance session workflow is documented | Passed |
| Session-based access | Figure 10 | Interactive browser shell is active | Passed |
| Application-file validation | Figure 10 | `/var/www/html` content is listed | Passed |
| AWS CLI validation | Figure 10 | EC2 description output is returned | Passed |

### Overall Assessment

The implementation objectives were achieved. The application rendered successfully, the feature flag altered application behavior, managed-node inventory was available, and Session Manager provided command-line access. For a stronger audit-quality submission, the evidence set should include a screenshot showing the Run Command **Overall status: Success**.

---

## 12. Security Analysis

### 12.1 Security Strengths Demonstrated

- **Reduced direct-access dependency:** Run Command performed the installation without an interactive administrator login.
- **No SSH-key workflow required:** Session Manager provided shell access through the AWS management plane.
- **Centralized access control:** Systems Manager actions can be governed through AWS Identity and Access Management policies.
- **Configuration externalization:** Parameter Store kept the feature flag outside the deployed application files.
- **Operational visibility:** Inventory exposed installed-package metadata from a central interface.
- **API-based administration:** AWS CLI commands were executed through an authenticated managed session.

### 12.2 Risks Visible in the Evidence

- The application was accessed over plain HTTP and the browser displayed **Not secure**.
- The screenshots expose cloud identifiers, including an AWS account identifier, instance identifiers, a public IP address, and session details.
- The Session Manager output contains infrastructure metadata that should not be published without review.
- The parameter is shown as a String. String is acceptable for a non-secret feature flag, but secrets should use an encrypted secret-management approach rather than plain text.
- The metadata command uses an IMDSv1-style request. Production environments should enforce IMDSv2.

### 12.3 Required GitHub Sanitization

Before publishing the screenshots in a public repository:

1. Redact the AWS account identifier.
2. Redact public and private IP addresses.
3. Redact EC2 instance IDs, AMI IDs, subnet IDs, VPC IDs, security-group IDs, and session IDs.
4. Remove or mask generated document and command identifiers where disclosure is unnecessary.
5. Crop browser chrome when it exposes environment-specific account or network information.
6. Verify that no credentials, session tokens, cookies, or secret parameter values are visible.
7. Preserve the technical evidence after redaction so each image still proves the intended task.

> **Important:** Renaming screenshots does not remove sensitive information embedded in the image. Redaction must be completed before public upload.

---

## 13. Operational Value

### 13.1 Inventory

Inventory creates centralized visibility into managed-node software and configuration metadata. This can support asset discovery, configuration review, and investigation of software presence across larger fleets.

### 13.2 Run Command

Run Command provides repeatable remote execution against one or more managed targets. Selecting targets through instance attributes or tags can reduce manual administration and help standardize operational actions.

### 13.3 Parameter Store

Parameter Store separates configuration from application deployment. In this project, the application changed behavior after reading a centrally managed feature flag, illustrating how operational teams can enable or disable functionality independently of the deployment package.

### 13.4 Session Manager

Session Manager provides interactive access through the Systems Manager control plane. With appropriately configured IAM permissions and logging, the approach can provide stronger governance than unmanaged SSH access.

---

## 14. Limitations and Evidence Gaps

- The supplied Run Command screenshot captures **In Progress**, not **Success**.
- No screenshot shows the inventory-association configuration page or the association name `Inventory-Association`.
- No screenshot confirms the instance security group has no inbound TCP port 22 rule.
- No screenshot confirms Session Manager logs are being delivered to Amazon CloudWatch Logs or Amazon S3.
- No screenshot shows an AWS CloudTrail event for Session Manager or Run Command activity.
- No IAM policy evidence is included to demonstrate least-privilege permissions.
- The screenshots do not show TLS/HTTPS configuration for the application.
- The screenshots are not yet visibly redacted for safe public publication.
- The evidence set confirms a lab implementation, not a production-hardened deployment.

These limitations do not invalidate the completed lab. They identify areas where a production implementation or audit-ready portfolio record would require additional controls and evidence.

---

## 15. Recommended Enhancements

### 15.1 Evidence Improvements

- Capture the Run Command page after **Overall status** changes to **Success**.
- Capture the command output panel to document successful package and application installation.
- Capture the instance security group showing that inbound port 22 is absent.
- Capture the Session Manager preferences page if CloudWatch Logs or S3 session logging is configured.
- Redact all environment identifiers before publishing the repository.

### 15.2 Security Improvements

- Place the workload behind an Application Load Balancer configured with HTTPS and an AWS Certificate Manager certificate.
- Restrict network access using security groups and private subnets where the use case permits.
- Apply least-privilege IAM policies to Run Command, Parameter Store, Session Manager, and EC2 read operations.
- Use Parameter Store `SecureString` with AWS Key Management Service for sensitive configuration values.
- Enforce IMDSv2 on the EC2 instance.
- Send Session Manager and Run Command activity to centralized logging and monitoring services.
- Use resource tags and approved Systems Manager documents to control which instances can receive commands.
- Add AWS Config rules or Security Hub controls to detect configuration drift and insecure exposure.

### 15.3 Automation Improvements

- Define the instance profile, IAM policies, security groups, parameters, and logging configuration in infrastructure as code.
- Use version-controlled Systems Manager documents and controlled document promotion.
- Replace manual feature-flag changes with an approved deployment or change-management pipeline.
- Add operational alarms for failed commands, offline managed nodes, and web-application health.

---

## 16. Screenshot Register and Rename Plan

The original screenshots should be renamed exactly as follows and stored in a repository folder named `screenshots/`.

| Figure | Original filename | Required new filename | Correct report placement |
|---:|---|---|---|
| 01 | `Screenshot 2026-10-29 122741.png` | `Fig01 Fleet Manager Managed Instance Online.png` | Task 1, managed-node prerequisite |
| 02 | `Screenshot 2026-10-29 121231.png` | `Fig02 Systems Manager Application Inventory.png` | Task 1, collected inventory |
| 03 | `Screenshot 2026-10-29 121027.png` | `Fig03 Run Command InstallDashboardApp Selection.png` | Task 2, command-document selection |
| 04 | `Screenshot 2026-10-29 121435.png` | `Fig04 InstallDashboardApp Document Details.png` | Task 2, document verification |
| 05 | `Screenshot 2026-10-29 121549.png` | `Fig05 Run Command Execution In Progress.png` | Task 2, command submission state |
| 06 | `Screenshot 2026-10-29 122446.png` | `Fig06 Dashboard Before Beta Feature Activation.png` | Tasks 2-3, deployed application and baseline state |
| 07 | `Screenshot 2026-10-29 122454.png` | `Fig07 Parameter Store Beta Feature Flag.png` | Task 3, parameter creation |
| 08 | `Screenshot 2026-10-29 122153.png` | `Fig08 Dashboard After Beta Feature Activation.png` | Task 3, three-chart feature validation |
| 09 | `Screenshot 2026-10-29 122852.png` | `Fig09 Session Manager Managed Instance Selection.png` | Task 4, session initiation |
| 10 | `Screenshot 2026-10-29 123131.png` | `Fig10 Session Manager Instance Validation.png` | Task 4, shell and AWS CLI validation |

### 16.1 Corrected One-to-One File Set

Use the following ten-file evidence set. Each source screenshot maps to one descriptive filename:

```text
screenshots/
├── Fig01 Fleet Manager Managed Instance Online.png
├── Fig02 Systems Manager Application Inventory.png
├── Fig03 Run Command InstallDashboardApp Selection.png
├── Fig04 InstallDashboardApp Document Details.png
├── Fig05 Run Command Execution In Progress.png
├── Fig06 Dashboard Before Beta Feature Activation.png
├── Fig07 Parameter Store Beta Feature Flag.png
├── Fig08 Dashboard After Beta Feature Activation.png
├── Fig09 Session Manager Managed Instance Selection.png
└── Fig10 Session Manager Instance Validation.png
```

### 16.2 Visual Sequence Rationale

The uploaded images were not ordered by task chronology. The corrected report order is:

1. Fleet Manager managed-node registration.
2. Collected application inventory.
3. Run Command document selection.
4. Installation-document details.
5. Command submission in progress.
6. Deployed dashboard in its two-chart baseline state.
7. Parameter Store feature flag.
8. Dashboard after the third beta chart becomes visible.
9. Session Manager managed-instance selection.
10. Session Manager shell and AWS CLI validation.

This sequence ensures that each screenshot appears directly under the implementation step it visibly supports.

---

## 17. Conclusion

The project successfully demonstrated operational management of an Amazon EC2 instance through AWS Systems Manager. Inventory provided centralized software visibility, Run Command deployed the Widget Manufacturing Dashboard, Parameter Store controlled a beta feature, and Session Manager enabled browser-based command-line access.

The result shows practical understanding of agent-based cloud operations, centralized configuration, controlled remote execution, and reduced reliance on traditional SSH administration. The implementation is suitable as a cloud-security portfolio project after sensitive infrastructure identifiers are redacted and the final Run Command success state is added to the evidence set.

---

## 18. Disclaimer

This report documents a completed educational lab in a temporary AWS environment. It is not a production deployment guide, AWS security certification, or guarantee of production readiness. Service behavior, console layouts, pricing, and recommended controls may change. Any public repository must exclude credentials, secret values, tokens, account identifiers, and unnecessary infrastructure metadata.

Amazon Web Services, AWS, Amazon EC2, AWS Systems Manager, AWS Identity and Access Management, AWS CloudTrail, Amazon CloudWatch, AWS Key Management Service, AWS Config, and AWS Security Hub are trademarks or service names of Amazon.com, Inc. or its affiliates. This independent portfolio report is not affiliated with or endorsed by Amazon Web Services.

---

## 19. Author

**Wadondera A. Collins**  
ICDFA Trainee | Cohort 11 | Cloud Security Engineering  

**Focus areas:** Cloud security engineering, secure cloud operations, identity and access management, infrastructure automation, monitoring, and incident-ready documentation.

---

<div align="center">

**AWS Systems Manager Operations and Secure Instance Management**  
**Technical Documentation Report | Completed 29 October 2026**  
**Wadondera A. Collins**

</div>
