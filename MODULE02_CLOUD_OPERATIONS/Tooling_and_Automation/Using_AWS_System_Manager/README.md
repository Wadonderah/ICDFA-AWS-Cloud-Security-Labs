<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:071A2B,45:0B5CAD,100:00A4EF&height=230&section=header&text=AWS%20Systems%20Manager&fontSize=42&fontColor=FFFFFF&fontAlignY=35&desc=Secure%20EC2%20Operations%20Without%20Inbound%20SSH&descSize=19&descAlignY=57&animation=fadeIn" width="100%" alt="AWS Systems Manager secure EC2 operations banner" />

# 🔐 AWS Systems Manager: Secure EC2 Operations

### Module 2 · Lab 1 | Cloud Security Engineering Portfolio Project

<p>
  <img src="https://img.shields.io/badge/AWS-Systems%20Manager-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS Systems Manager" />
  <img src="https://img.shields.io/badge/Amazon-EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2" />
  <img src="https://img.shields.io/badge/Security-No%20Inbound%20SSH-0B5CAD?style=for-the-badge&logo=shield&logoColor=white" alt="No inbound SSH" />
</p>

<p>
  <img src="https://img.shields.io/badge/IAM-Least%20Privilege-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="IAM least privilege" />
  <img src="https://img.shields.io/badge/CloudTrail-Audit%20Ready-7B42BC?style=flat-square&logo=amazonaws&logoColor=white" alt="CloudTrail audit ready" />
  <img src="https://img.shields.io/badge/Parameter%20Store-Secure%20Configuration-1F6FEB?style=flat-square&logo=amazonaws&logoColor=white" alt="Parameter Store secure configuration" />
  <img src="https://img.shields.io/badge/Session%20Manager-Keyless%20Access-2EA44F?style=flat-square&logo=gnometerminal&logoColor=white" alt="Session Manager keyless access" />
  <img src="https://img.shields.io/badge/Project-Completed-success?style=flat-square&logo=github&logoColor=white" alt="Project completed" />
</p>

**Inventory Visibility · Controlled Remote Execution · Centralized Configuration · Auditable Instance Access**

<br />

**Wadondera A. Collins**  
*ICDFA Trainee · Cohort 11 · Cloud Security Engineering*

</div>
---

## Table of Contents

- [Project Overview](#project-overview)
- [Executive Summary](#executive-summary)
- [Security Objectives](#security-objectives)
- [Professional Value](#professional-value)
- [AWS Services and Technologies](#aws-services-and-technologies)
- [Lab Environment](#lab-environment)
- [Solution Architecture](#solution-architecture)
- [Implementation](#implementation)
- [Security Analysis](#security-analysis)
- [Validation Results](#validation-results)
- [Evidence Guide](#evidence-guide)
- [Skills Demonstrated](#skills-demonstrated)
- [Limitations and Next Steps](#limitations-and-next-steps)
- [Security Notes](#security-notes)
- [Conclusion](#conclusion)
- [Disclaimer](#disclaimer)
- [Author](#author)

---

## Project Overview

This completed lab demonstrates how **AWS Systems Manager** can be used to administer an Amazon EC2 instance through centralized, policy-controlled workflows. The project combines asset visibility, remote task execution, application configuration, and interactive instance access without relying on direct SSH connectivity.

The implementation used four Systems Manager capabilities:

1. **Inventory** to collect instance and software metadata.
2. **Run Command** to deploy a custom web application remotely.
3. **Parameter Store** to control an application feature through centralized configuration.
4. **Session Manager** to open an interactive shell without exposing inbound TCP port 22.

---

## Executive Summary

The lab replaced common manual administration patterns with AWS-native management controls. An inventory association collected configuration data from a managed EC2 instance. Run Command deployed the **Widget Manufacturing Dashboard** without an interactive server login. Parameter Store enabled a beta feature through a hierarchical application setting. Session Manager then provided command-line access through the AWS management plane rather than a publicly exposed SSH service.

From a cloud security perspective, the project demonstrates three important principles:

- **Reduce network exposure** by avoiding inbound SSH access.
- **Centralize authorization** through AWS Identity and Access Management permissions.
- **Improve traceability** by using managed AWS operational workflows that can be audited through AWS CloudTrail.

---

## Security Objectives

- Establish centralized visibility into EC2 software and configuration data.
- Execute administrative tasks without distributing or maintaining SSH keys.
- Deploy application components remotely through a managed command channel.
- Separate application configuration from application code.
- Access the instance without opening inbound port 22.
- Support least-privilege administration through IAM-controlled access.
- Improve accountability through AWS service activity logging.
- Reduce the operational risk associated with unmanaged administrative entry points.

---

## Professional Value

This project reflects practical cloud security engineering responsibilities commonly associated with:

- Secure EC2 administration
- Systems hardening
- Infrastructure operations
- Access-control design
- Configuration governance
- Remote automation
- Application feature management
- Audit-ready cloud operations

The lab is especially relevant to junior cloud security, cloud operations, DevSecOps, and AWS support roles because it demonstrates how security controls can be built directly into routine infrastructure management.

---

## AWS Services and Technologies

| Service or Technology | Role in the Project | Security Relevance |
|---|---|---|
| AWS Systems Manager Inventory | Collected managed-instance metadata | Improved asset and configuration visibility |
| AWS Systems Manager Run Command | Executed the application installation document | Reduced dependence on manual server logins |
| AWS Systems Manager Parameter Store | Stored `/dashboard/show-beta-features` | Separated configuration from application code |
| AWS Systems Manager Session Manager | Provided browser-based shell access | Avoided inbound SSH exposure and SSH key management |
| Amazon EC2 | Hosted the managed application server | Target workload administered through Systems Manager |
| AWS IAM | Authorized Systems Manager operations | Supported identity-based and least-privilege access |
| AWS CloudTrail | Recorded AWS API activity | Supported operational accountability and audit review |
| SSM Agent | Connected the EC2 instance to Systems Manager | Enabled managed control-plane communication |
| Apache HTTP Server and PHP | Hosted the dashboard application | Demonstrated automated software deployment |
| AWS SDK for PHP | Enabled application interaction with AWS services | Supported runtime retrieval of AWS-managed configuration |
| AWS CLI | Queried EC2 information from the session | Demonstrated controlled command-line administration |

---

## Lab Environment

| Component | Configuration |
|---|---|
| Managed resource | Amazon EC2 instance named `Managed Instance` |
| Management interface | AWS Management Console |
| Inventory association | `Inventory-Association` |
| Command document | Custom document containing `InstallDashboardApp` in its name |
| Deployed workload | Widget Manufacturing Dashboard |
| Parameter path | `/dashboard/show-beta-features` |
| Parameter value | `True` |
| Administrative access | AWS Systems Manager Session Manager |
| SSH exposure | Inbound port 22 not required for the completed workflow |

> **Environment note:** The AWS Region was left unchanged, as required by the lab environment.

---

## Solution Architecture

```text
Administrator
     |
     | IAM-authorized access
     v
AWS Management Console
     |
     v
AWS Systems Manager
     |-- Inventory ---------> Collects instance and software metadata
     |-- Run Command -------> Executes the dashboard installation document
     |-- Parameter Store ---> Supplies application feature configuration
     |-- Session Manager ---> Provides interactive shell access
     |
     v
Amazon EC2 Managed Instance
     |
     |-- SSM Agent
     |-- Apache + PHP
     |-- AWS SDK for PHP
     `-- Widget Manufacturing Dashboard

Audit path: Systems Manager and related API activity -> AWS CloudTrail
```

### Security Boundary

The administrative workflow is routed through AWS Systems Manager and IAM authorization. The solution does not depend on a public SSH endpoint, a bastion host, or locally managed SSH keys for the tasks completed in this lab.

---

## Implementation

### Task 1: Establish Configuration Visibility with Inventory

An inventory association named `Inventory-Association` was created for the EC2 managed instance through Systems Manager Fleet Manager.

**Actions completed:**

- Selected `Managed Instance` from Fleet Manager.
- Configured inventory collection.
- Created the inventory association.
- Reviewed installed applications and available inventory types.
- Verified that Systems Manager could collect instance metadata without an SSH session.

**Security outcome:**

Centralized inventory data improved visibility into the software and configuration state of the managed instance. This supports asset awareness and provides a foundation for identifying systems that do not meet expected configuration requirements.

---

### Task 2: Deploy the Dashboard with Run Command

A custom Systems Manager command document containing `InstallDashboardApp` in its name was selected and executed against `Managed Instance`.

**The document performed the following deployment actions:**

- Installed Apache HTTP Server.
- Installed PHP.
- Activated the web service.
- Installed the AWS SDK for PHP.
- Installed the Widget Manufacturing Dashboard and its supporting files.

**Validation performed:**

- Monitored the command until the overall status changed to `Success`.
- Opened the EC2 server IP in a browser.
- Confirmed that the Widget Manufacturing Dashboard loaded successfully.

**Security outcome:**

The application was deployed through a managed execution channel without a direct interactive login. This reduces ad hoc administrative access and supports repeatable operational workflows.

---

### Task 3: Control Application Behavior with Parameter Store

A hierarchical Parameter Store entry was created to activate a beta dashboard feature.

```text
Name: /dashboard/show-beta-features
Description: Display beta features
Value: True
```

**Validation performed:**

- Confirmed that the dashboard initially displayed two charts.
- Created the parameter.
- Refreshed the application.
- Confirmed that a third chart appeared.

**Security outcome:**

The feature setting was maintained outside the application code. This demonstrates a cleaner configuration-management pattern and allows application behavior to be changed through an AWS-managed control point.

> **Important:** The lab used a non-sensitive feature flag. Sensitive values should use the appropriate protected parameter type, encryption, and tightly scoped IAM permissions.

---

### Task 4: Administer the Instance with Session Manager

A browser-based Session Manager shell was opened for `Managed Instance`.

**Application-file validation:**

```bash
ls /var/www/html
```

This command verified that the application files were present in the Apache web root.

**Region and EC2 query:**

```bash
# Retrieve the instance Availability Zone
AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`

# Derive and export the AWS Region
export AWS_DEFAULT_REGION=${AZ::-1}

# Retrieve EC2 instance information
aws ec2 describe-instances
```

**Security outcome:**

Administrative shell access was achieved without opening inbound SSH port 22. Access could therefore be governed through IAM and recorded through AWS audit services instead of depending on traditional SSH key distribution.

---

## Security Analysis

### Threat and Control Mapping

| Security Concern | Control Demonstrated | Result |
|---|---|---|
| Exposed administrative ports | Session Manager | Port 22 was not required for instance access |
| SSH key loss, reuse, or sprawl | IAM-authorized Session Manager access | No SSH key was needed for the lab workflow |
| Untracked manual changes | Run Command | Remote execution occurred through a managed AWS operation |
| Limited software visibility | Systems Manager Inventory | Installed software and instance metadata were centrally available |
| Configuration embedded in code | Parameter Store | Feature configuration was externalized under a hierarchical path |
| Weak operational accountability | CloudTrail-compatible AWS API activity | AWS service activity could be reviewed through audit logging |
| Excessive administrative access | IAM policies | Access can be restricted to approved Systems Manager actions and resources |

### Cloud Security Principles Demonstrated

#### 1. Least Privilege

Systems Manager access should be limited to the specific actions, documents, parameters, instances, and session capabilities required by each administrator or workload.

#### 2. Reduced Attack Surface

Removing the need for inbound SSH reduces externally reachable administrative services and eliminates the need to distribute SSH credentials for the demonstrated workflow.

#### 3. Centralized Governance

Inventory, commands, parameters, and sessions are managed through AWS service interfaces, providing stronger governance than disconnected manual tools.

#### 4. Separation of Configuration and Code

The beta feature was enabled through Parameter Store rather than a code change, supporting controlled and reversible configuration updates.

#### 5. Auditability

Systems Manager operations use AWS APIs that can be monitored through CloudTrail. Production environments should also configure session and command logging according to organizational security requirements.

---

## Validation Results

| Validation Check | Expected Result | Status |
|---|---|---|
| Managed instance visible in Fleet Manager | Instance available for Systems Manager operations | Passed |
| Inventory association created | `Inventory-Association` configured | Passed |
| Inventory data collected | Applications and metadata displayed | Passed |
| Run Command execution | Overall status changed to `Success` | Passed |
| Dashboard deployment | Widget Manufacturing Dashboard loaded | Passed |
| Parameter created | `/dashboard/show-beta-features` stored with value `True` | Passed |
| Feature flag applied | Third dashboard chart displayed after refresh | Passed |
| Session Manager connectivity | Browser-based shell opened successfully | Passed |
| Application files verified | `/var/www/html` contents listed | Passed |
| AWS CLI validation | EC2 instance information returned | Passed |
| SSH dependency removed | Port 22 not required for the completed access workflow | Passed |

---

## Evidence Guide

Add only screenshots that clearly match the corresponding implementation step. Rename each file before committing it to the repository.

| Suggested Filename | Evidence It Must Show | README Placement |
|---|---|---|
| `Fig01 Managed Instance in Fleet Manager.png` | The named managed instance visible in Systems Manager Fleet Manager | Task 1 |
| `Fig02 Inventory Association Configuration.png` | `Inventory-Association` and its selected target | Task 1 |
| `Fig03 Collected Instance Inventory.png` | Inventory tab with collected application or instance metadata | Task 1 |
| `Fig04 Run Command Successful Execution.png` | `InstallDashboardApp` command with `Success` status | Task 2 |
| `Fig05 Widget Manufacturing Dashboard.png` | Successfully deployed dashboard in the browser | Task 2 |
| `Fig06 Parameter Store Feature Flag.png` | `/dashboard/show-beta-features` parameter details | Task 3 |
| `Fig07 Dashboard Beta Feature Enabled.png` | Dashboard displaying the third chart after refresh | Task 3 |
| `Fig08 Session Manager Active Session.png` | Active browser-based session for the managed instance | Task 4 |
| `Fig09 Application Files Validation.png` | Output from `ls /var/www/html` | Task 4 |
| `Fig10 EC2 Describe Instances Output.png` | Output from `aws ec2 describe-instances` | Task 4 |

> Do not add a figure unless its visible content matches the filename, caption, and documented task.

---

## Repository Structure

```text
aws-systems-manager-lab/
├── README.md
├── evidence/
│   ├── Fig01 Managed Instance in Fleet Manager.png
│   ├── Fig02 Inventory Association Configuration.png
│   ├── Fig03 Collected Instance Inventory.png
│   ├── Fig04 Run Command Successful Execution.png
│   ├── Fig05 Widget Manufacturing Dashboard.png
│   ├── Fig06 Parameter Store Feature Flag.png
│   ├── Fig07 Dashboard Beta Feature Enabled.png
│   ├── Fig08 Session Manager Active Session.png
│   ├── Fig09 Application Files Validation.png
│   └── Fig10 EC2 Describe Instances Output.png
└── LICENSE
```

> The evidence filenames above are a portfolio-ready naming plan. Include only files that exist and accurately represent the completed tasks.

---

## Skills Demonstrated

- AWS Systems Manager administration
- Amazon EC2 operational security
- Secure remote instance access
- Systems inventory and configuration visibility
- Remote command execution
- Application deployment automation
- Parameter and feature-flag management
- IAM-based authorization awareness
- CloudTrail audit awareness
- Linux command-line administration
- Reduced-exposure infrastructure design
- Cloud security documentation

---

## Limitations and Next Steps

### Lab Limitations

- The exercise used one managed EC2 instance rather than a production fleet.
- The parameter was a non-sensitive feature flag.
- Advanced session logging destinations were not configured during the documented tasks.
- Automated compliance reporting and remediation were outside the lab scope.
- The dashboard was used for validation and was not assessed as a production-grade application.

### Recommended Production Enhancements

1. **Enforce least-privilege IAM policies** for StartSession, SendCommand, GetParameter, and inventory-related operations.
2. **Restrict approved command documents** instead of allowing unrestricted Run Command usage.
3. **Enable Session Manager logging** to an approved CloudWatch Logs log group or Amazon S3 bucket.
4. **Protect sensitive parameters** with encrypted storage and a customer-managed AWS KMS key where organizational requirements call for it.
5. **Use resource tags** to scope commands and access policies across managed-instance groups.
6. **Create CloudTrail alerts** for high-risk Systems Manager actions and unexpected session activity.
7. **Apply patch and compliance controls** through Systems Manager capabilities appropriate to the production environment.
8. **Prefer IMDSv2-compatible metadata access** in production automation and validate the instance metadata configuration.
9. **Review stale sessions, associations, and parameters** as part of recurring operational governance.
10. **Integrate findings with security monitoring** so operational events can be investigated centrally.

---

## Security Notes

- Never commit credentials, session tokens, private keys, account identifiers, or sensitive command output to a public repository.
- Redact EC2 public IP addresses, account IDs, instance IDs, IAM role names, and other environment-specific values from screenshots when disclosure is unnecessary.
- Use SecureString and appropriate encryption controls for secrets rather than storing sensitive values as plaintext configuration.
- Scope `ssm:StartSession`, `ssm:SendCommand`, and parameter access to approved resources and business needs.
- Review command documents before execution because remote commands can make privileged changes to managed instances.
- Configure retention, encryption, and access controls for operational logs.
- End temporary lab sessions and delete temporary resources after validation.

---

## Conclusion

This project demonstrates a security-focused approach to EC2 administration using AWS Systems Manager. Inventory improved configuration visibility, Run Command automated application deployment, Parameter Store externalized an application feature setting, and Session Manager provided controlled shell access without inbound SSH.

The completed implementation shows how AWS-native management services can reduce attack surface, centralize authorization, improve operational consistency, and strengthen audit readiness. These capabilities are directly relevant to cloud security engineering, secure infrastructure operations, and DevSecOps practice.

---

## Disclaimer

This repository documents a completed educational lab in a temporary AWS training environment. It is intended for learning and portfolio demonstration purposes. Production implementations require organization-specific IAM design, encryption, monitoring, logging, network controls, change management, and security review.

AWS product names and trademarks belong to Amazon Web Services, Inc. The original lab material is not reproduced here; this README presents an original portfolio summary of the work completed and the security lessons demonstrated.

---

## Author

<div align="center">

### Wadondera A. Collins

**ICDFA Trainee · Cohort 11 · Cloud Security Engineering**

*Building secure, auditable, and automation-driven cloud environments.*

</div>
