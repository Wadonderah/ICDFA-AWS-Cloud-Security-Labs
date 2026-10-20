<div align="center">

# 🔐 AWS KMS Data-at-Rest Encryption

### Lab 5.1: Encrypting Data at Rest by Using AWS KMS

[![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-FF9900?style=for-the-badge&logo=amazonwebservices&logoColor=white)](https://aws.amazon.com/)
[![AWS KMS](https://img.shields.io/badge/AWS%20KMS-Customer%20Managed%20Key-527FFF?style=for-the-badge&logo=amazonwebservices&logoColor=white)](https://aws.amazon.com/kms/)
[![Amazon S3](https://img.shields.io/badge/Amazon%20S3-SSE--KMS-569A31?style=for-the-badge&logo=amazons3&logoColor=white)](https://aws.amazon.com/s3/)
[![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-Encrypted%20Root%20Volume-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)](https://aws.amazon.com/ec2/)
[![AWS CloudTrail](https://img.shields.io/badge/AWS%20CloudTrail-Audit%20Logging-8A2BE2?style=for-the-badge&logo=amazonwebservices&logoColor=white)](https://aws.amazon.com/cloudtrail/)

**Author:** Wadondera A. Collins  
**Program:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Project Type:** AWS Cloud Security Lab  
**Focus Area:** Encryption, Key Management, Storage Security, and Audit Logging  
**Status:** ✅ Completed

</div>

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Executive Summary](#-executive-summary)
- [Business Scenario](#-business-scenario)
- [Project Objectives](#-project-objectives)
- [Professional Value](#-professional-value)
- [AWS Services and Technologies](#-aws-services-and-technologies)
- [Lab Environment](#-lab-environment)
- [Architecture Overview](#-architecture-overview)
- [Implementation Methodology](#-implementation-methodology)
  - [Task 1: Create a Customer Managed KMS Key](#task-1-create-a-customer-managed-kms-key)
  - [Task 2: Encrypt an Amazon S3 Object with SSE-KMS](#task-2-encrypt-an-amazon-s3-object-with-sse-kms)
  - [Task 3: Test Public Access to the Encrypted Object](#task-3-test-public-access-to-the-encrypted-object)
  - [Task 4: Test Signed and Authenticated Access](#task-4-test-signed-and-authenticated-access)
  - [Task 5: Audit KMS Activity with AWS CloudTrail](#task-5-audit-kms-activity-with-aws-cloudtrail)
  - [Task 6: Encrypt an Existing EC2 Root EBS Volume](#task-6-encrypt-an-existing-ec2-root-ebs-volume)
  - [Task 7: Disable and Re-enable the KMS Key](#task-7-disable-and-re-enable-the-kms-key)
- [Encryption Workflows](#-encryption-workflows)
- [Security Concepts Demonstrated](#-security-concepts-demonstrated)
- [Validation Checklist](#-validation-checklist)
- [Results](#-results)
- [Challenges and Operational Considerations](#-challenges-and-operational-considerations)
- [Security Recommendations](#-security-recommendations)
- [Skills Demonstrated](#-skills-demonstrated)
- [Key Takeaways](#-key-takeaways)
- [Conclusion](#-conclusion)
- [Disclaimer](#-disclaimer)
- [Author](#-author)

---

## 📖 Project Overview

This completed project demonstrates how AWS Key Management Service (AWS KMS) can be used to protect data at rest across Amazon Simple Storage Service (Amazon S3) and Amazon Elastic Block Store (Amazon EBS).

The implementation used a customer managed symmetric KMS key named `MyKMSKey` to encrypt an S3 object with server-side encryption using AWS KMS keys (SSE-KMS). The same key was then used to encrypt a replacement root EBS volume for an existing Amazon EC2 instance.

The project also examined access behavior for unsigned and signed S3 requests, reviewed KMS-related events in AWS CloudTrail, and tested the operational effect of disabling and re-enabling the encryption key.

---

## 🧾 Executive Summary

The lab began with an empty S3 bucket and an EC2 instance whose root EBS volume was not encrypted. A customer managed KMS key was created and assigned to the provided `voclabs` role for administration and cryptographic use.

The `clock.png` object was uploaded to the S3 bucket using SSE-KMS. Direct public access through the object URL remained unsuccessful because the encrypted object required an authenticated AWS Signature Version 4 request. Opening the object through the authenticated S3 console succeeded because Amazon S3 could request decryption of the encrypted data key from AWS KMS.

AWS CloudTrail was used to examine KMS activity, including `GenerateDataKey`, `Decrypt`, `DisableKey`, and failed key-dependent activity. The EC2 root volume was encrypted by creating a snapshot of the original volume, producing a new encrypted volume from the snapshot, detaching the original volume, and attaching the encrypted replacement as `/dev/xvda`.

Disabling `MyKMSKey` prevented the S3 object from being decrypted and stopped the EC2 instance from successfully starting. Re-enabling the key restored access and allowed the instance to enter the running state. The completed project confirmed that encryption key availability, permissions, and lifecycle management are critical to both security and workload continuity.

---

## 🏢 Business Scenario

An organization stores business files in Amazon S3 and operates workloads on Amazon EC2. The organization needs stronger protection for information stored in the cloud and requires centralized control over encryption keys.

The security solution must accomplish the following:

- Encrypt sensitive S3 objects at rest.
- Encrypt the root storage volume of an existing EC2 instance.
- Restrict decryption to authorized identities and services.
- Record encryption and key-management activity for auditing.
- Demonstrate that accidental public exposure does not automatically provide access to KMS-encrypted data.
- Evaluate how key disablement affects dependent services and workloads.

---

## 🎯 Project Objectives

The project objectives were to:

1. Create an AWS KMS customer managed symmetric key.
2. Configure the provided IAM role as a key administrator and key user.
3. Encrypt an Amazon S3 object using SSE-KMS.
4. Compare unsigned public access with authenticated signed access.
5. Review KMS API activity in AWS CloudTrail Event history.
6. Encrypt the root EBS volume of an existing EC2 instance.
7. Test the effect of disabling a KMS key on S3 and EC2 resources.
8. Restore normal operations by re-enabling the key.

---

## 💼 Professional Value

This project reflects practical responsibilities associated with cloud security engineering, cloud administration, DevSecOps, and security operations. It demonstrates the ability to implement encryption controls, inspect audit evidence, manage service dependencies, and validate the security impact of cryptographic key lifecycle actions.

The completed work is relevant to roles involving:

- Cloud Security Engineering
- AWS Security Administration
- Cloud Infrastructure Engineering
- DevSecOps
- Security Operations
- Governance, Risk, and Compliance
- Identity and Access Management
- Infrastructure Protection

---

## 🛠 AWS Services and Technologies

| Service or Technology | Project Use |
|---|---|
| AWS Key Management Service | Created and managed the customer managed encryption key. |
| Amazon S3 | Stored the `clock.png` object encrypted with SSE-KMS. |
| Amazon EC2 | Hosted the `LabInstance` compute workload. |
| Amazon EBS | Provided the original and replacement root block-storage volumes. |
| EBS Snapshots | Captured the original unencrypted root volume before conversion. |
| AWS CloudTrail | Recorded and exposed KMS-related API activity for audit review. |
| AWS Identity and Access Management | Supplied the `voclabs` role used for key administration and usage. |
| AWS Signature Version 4 | Authenticated the successful signed request to access the encrypted S3 object. |

---

## 🧪 Lab Environment

| Component | Configuration |
|---|---|
| KMS key alias | `MyKMSKey` |
| KMS key type | Symmetric |
| Key administrator | `voclabs` role |
| Key user | `voclabs` role |
| S3 bucket | Lab-provided bucket containing `imagebucket` in its name |
| S3 test object | `clock.png` |
| S3 encryption | SSE-KMS using `MyKMSKey` |
| EC2 instance | `LabInstance` |
| Original EBS volume | Unencrypted root volume |
| Replacement EBS volume | Encrypted root volume using `MyKMSKey` |
| Root device name | `/dev/xvda` |
| Audit source | AWS CloudTrail Event history |

> **Note:** The environment was a restricted training account. Actions outside the lab scope could be unavailable.

---

## 🏗 Architecture Overview

### Initial Architecture

```text
AWS Account
├── Amazon S3
│   └── ImageBucket
│       └── Empty
│
└── Amazon EC2
    └── LabInstance
        └── Unencrypted root EBS volume
```

### Final Architecture

```text
User / Authorized AWS Role
            │
            ├─────────────────────────────┐
            │                             │
            ▼                             ▼
      Amazon S3                      Amazon EC2
      ImageBucket                    LabInstance
      └── clock.png                  └── Encrypted root EBS volume
          └── SSE-KMS                    └── KMS key: MyKMSKey
            │                             │
            └──────────────┬──────────────┘
                           ▼
                    AWS KMS
                    └── MyKMSKey
                           │
                           ▼
                    AWS CloudTrail
                    └── KMS audit events
```

---

## 🚀 Implementation Methodology

### Task 1: Create a Customer Managed KMS Key

#### Objective

Create a symmetric customer managed key for encryption and decryption operations across Amazon S3 and Amazon EBS.

#### Configuration

| Setting | Value |
|---|---|
| Key type | Symmetric |
| Key alias | `MyKMSKey` |
| Key administrators | `voclabs` |
| Key users | `voclabs` |

#### Implementation Summary

The KMS console was used to create a symmetric customer managed key. The provided `voclabs` role was selected as both the key administrator and key user. This configuration allowed the role to manage the key and authorize supported cryptographic operations during the lab.

#### Result

`MyKMSKey` was successfully created and became available for integration with Amazon S3 and Amazon EBS.

---

### Task 2: Encrypt an Amazon S3 Object with SSE-KMS

#### Objective

Upload an object to Amazon S3 and encrypt it using the customer managed key.

#### Implementation Summary

The lab-provided S3 bucket was reviewed, and `clock.png` was uploaded with the following object-level encryption configuration:

```text
Encryption type: Server-side encryption with AWS KMS keys (SSE-KMS)
KMS key: MyKMSKey
```

The object's Properties page was then reviewed to confirm that SSE-KMS was enabled.

#### Envelope Encryption Process

1. Amazon S3 requested a data key from AWS KMS.
2. AWS KMS generated a plaintext data key and an encrypted copy of that data key.
3. Amazon S3 used the plaintext data key to encrypt `clock.png`.
4. Amazon S3 discarded the plaintext data key after encryption.
5. The encrypted data key was retained with the encrypted object metadata.

#### Result

`clock.png` was successfully stored as an SSE-KMS-encrypted object using `MyKMSKey`.

---

### Task 3: Test Public Access to the Encrypted Object

#### Objective

Determine whether public object permissions were sufficient to retrieve an SSE-KMS-encrypted object through its direct S3 object URL.

#### Test 1: Default Access Controls

The direct object URL was opened before public permissions were changed.

```text
Observed result: Access Denied
```

This showed that the bucket's default public-access protections prevented anonymous retrieval.

#### Test 2: Public ACL Configuration

For controlled testing, Block Public Access was modified, ACLs were enabled, and the object was made public using an ACL. The object URL was then tested again.

```text
Observed result: Invalid Argument
```

The request still failed because the SSE-KMS-encrypted object required an authenticated AWS Signature Version 4 request. Anonymous public access did not provide the authorization needed to use the KMS key.

#### Security Significance

The test demonstrated defense in depth. Even when object-level public access was temporarily enabled, the encrypted object was not successfully retrieved through an unsigned anonymous request.

> **Production caution:** Public S3 access should remain blocked unless a documented business requirement and an approved security design explicitly require it.

---

### Task 4: Test Signed and Authenticated Access

#### Objective

Open the encrypted S3 object through an authenticated request and confirm successful decryption.

#### Implementation Summary

The object was selected in the Amazon S3 console and opened using the console's authenticated access path. The request included AWS Signature Version 4 authentication information.

The decryption sequence was:

1. The authenticated user requested `clock.png`.
2. Amazon S3 identified the object as SSE-KMS encrypted.
3. Amazon S3 sent the encrypted data key to AWS KMS.
4. AWS KMS decrypted the data key using `MyKMSKey`.
5. AWS KMS returned the plaintext data key to Amazon S3.
6. Amazon S3 decrypted the object and returned it to the authorized requester.
7. The plaintext data key was removed from active use after the operation.

#### Result

The image opened successfully through the authenticated S3 console request.

---

### Task 5: Audit KMS Activity with AWS CloudTrail

#### Objective

Use AWS CloudTrail Event history to identify and analyze API activity associated with the KMS key.

#### Event Source Filter

```text
kms.amazonaws.com
```

#### Events Reviewed

##### `GenerateDataKey`

This event was associated with the upload and encryption of `clock.png`. The event record exposed details such as the requesting principal, event name, KMS key identifier, and the S3 resource involved in the encryption workflow.

##### `Decrypt`

This event was associated with successfully opening the encrypted object through the authenticated S3 console. The event showed that AWS KMS processed a decryption request for the encrypted data key used by the S3 object.

#### Result

CloudTrail provided audit evidence connecting identities, API operations, the selected KMS key, and affected AWS resources.

---

### Task 6: Encrypt an Existing EC2 Root EBS Volume

#### Objective

Replace the unencrypted root volume of `LabInstance` with an encrypted volume created from a snapshot.

#### Initial State

```text
LabInstance
└── Root EBS volume
    └── Encryption: Disabled
```

#### Implementation Summary

1. Stopped `LabInstance`.
2. Identified the root EBS volume and its Availability Zone.
3. Created a snapshot tagged `Unencrypted Root Volume`.
4. Waited for the snapshot to reach the completed state.
5. Created a new encrypted EBS volume from the snapshot.
6. Selected `MyKMSKey` as the encryption key.
7. Created the new volume in the same Availability Zone as the instance.
8. Named the original volume `Old unencrypted root volume`.
9. Named the replacement volume `New encrypted root volume`.
10. Detached the original root volume.
11. Attached the encrypted replacement to `LabInstance` as `/dev/xvda`.
12. Verified that the attached root volume displayed encryption and a KMS key ID.

#### Conversion Workflow

```text
Unencrypted root EBS volume
            │
            ▼
     Unencrypted snapshot
            │
            ▼
Create encrypted EBS volume using MyKMSKey
            │
            ▼
Detach original root volume
            │
            ▼
Attach encrypted volume as /dev/xvda
```

#### Result

The existing EC2 instance was successfully reconfigured to use an encrypted root EBS volume.

---

### Task 7: Disable and Re-enable the KMS Key

#### Objective

Observe the effect of KMS key availability on encrypted S3 data and an EC2 instance with an encrypted root volume.

#### Key Disablement Test

`MyKMSKey` was disabled in the AWS KMS console.

##### EC2 Observation

`LabInstance` was started while the key was disabled.

```text
Observed state transition: Pending → Stopped
```

The instance did not reach the running state because the encrypted root volume could not be made available for normal startup without access to the required KMS key.

##### S3 Observation

An attempt was made to open `clock.png` through the S3 console.

```text
Observed error: KMS.DisabledException
```

The object could not be decrypted while the associated KMS key was disabled.

##### CloudTrail Observation

The following events were reviewed:

- `DisableKey`
- `StartInstances`
- `CreateGrant`

The event sequence showed the administrative key action, the EC2 start request, and the subsequent key-related failure encountered during the encrypted-volume workflow.

#### Recovery

`MyKMSKey` was re-enabled. `LabInstance` was started again and successfully reached the running state. KMS-dependent access to the encrypted S3 object was also restored.

#### Result

The test confirmed that disabling a KMS key can immediately interrupt access to resources that depend on it.

---

## 🔄 Encryption Workflows

### S3 Encryption Workflow

```text
Upload clock.png
       │
       ▼
Amazon S3 requests a data key
       │
       ▼
AWS KMS uses MyKMSKey
       │
       ├── Plaintext data key
       └── Encrypted data key
       │
       ▼
Amazon S3 encrypts clock.png
       │
       ├── Stores encrypted object
       ├── Stores encrypted data key as metadata
       └── Discards plaintext data key
```

### S3 Decryption Workflow

```text
Authenticated request for clock.png
              │
              ▼
Amazon S3 sends encrypted data key to AWS KMS
              │
              ▼
AWS KMS decrypts data key using MyKMSKey
              │
              ▼
Amazon S3 decrypts and returns the object
```

### EBS Root-Volume Encryption Workflow

```text
Stop EC2 instance
       │
       ▼
Snapshot unencrypted root volume
       │
       ▼
Create encrypted volume from snapshot
       │
       ▼
Detach original root volume
       │
       ▼
Attach encrypted volume as /dev/xvda
       │
       ▼
Start EC2 instance
```

---

## 🔒 Security Concepts Demonstrated

### Encryption at Rest

The project protected stored S3 and EBS data by encrypting it before it remained on persistent storage.

### Envelope Encryption

A KMS key protected data keys, while the data keys performed the direct encryption and decryption of service data.

### Customer Managed Key Control

The customer managed key provided direct control over administrators, users, status, and lifecycle actions.

### Least Privilege

The lab assigned key administration and key usage to the specified role rather than providing unrestricted anonymous decryption capability.

### Defense in Depth

S3 access controls and KMS authorization operated as separate protective layers. Public object permissions alone did not produce successful anonymous access to the SSE-KMS object.

### Auditability

CloudTrail recorded KMS-related actions, supporting investigation, accountability, and compliance evidence.

### Key Availability and Operational Resilience

Disabling the key interrupted access to both the S3 object and the encrypted EC2 root volume, demonstrating that encryption-key management is also an availability responsibility.

---

## ✅ Validation Checklist

| Validation Item | Expected Outcome | Status |
|---|---|---|
| Customer managed symmetric key created | `MyKMSKey` available | ✅ Passed |
| Key administrator configured | `voclabs` assigned | ✅ Passed |
| Key user configured | `voclabs` assigned | ✅ Passed |
| S3 object uploaded | `clock.png` present | ✅ Passed |
| Object encryption verified | SSE-KMS using `MyKMSKey` | ✅ Passed |
| Default public request tested | Access denied | ✅ Passed |
| Public ACL test completed | Unsigned request still unsuccessful | ✅ Passed |
| Authenticated request tested | Image opened successfully | ✅ Passed |
| CloudTrail KMS filter applied | `kms.amazonaws.com` events displayed | ✅ Passed |
| `GenerateDataKey` reviewed | Encryption activity identified | ✅ Passed |
| `Decrypt` reviewed | Decryption activity identified | ✅ Passed |
| Root-volume snapshot created | Snapshot completed | ✅ Passed |
| Encrypted EBS volume created | `MyKMSKey` selected | ✅ Passed |
| Root volume replaced | Encrypted volume attached as `/dev/xvda` | ✅ Passed |
| KMS key disabled | Key status changed | ✅ Passed |
| EC2 dependency tested | Instance failed to reach running state | ✅ Passed |
| S3 dependency tested | `KMS.DisabledException` observed | ✅ Passed |
| Key re-enabled | KMS operations restored | ✅ Passed |
| EC2 recovery validated | `LabInstance` reached running state | ✅ Passed |

---

## 📊 Results

The completed project produced the following outcomes:

- A customer managed symmetric AWS KMS key named `MyKMSKey` was created.
- `clock.png` was encrypted in Amazon S3 using SSE-KMS.
- Anonymous access to the encrypted object remained unsuccessful.
- Authenticated signed access successfully retrieved and decrypted the object.
- CloudTrail exposed KMS encryption, decryption, and key-management activity.
- The original unencrypted EC2 root volume was replaced with an encrypted EBS volume.
- Disabling the KMS key interrupted both S3 object access and EC2 startup.
- Re-enabling the KMS key restored normal operation.

---

## ⚙ Challenges and Operational Considerations

### Availability Zone Consistency

The replacement EBS volume had to be created in the same Availability Zone as `LabInstance` before it could be attached.

### Root Device Mapping

The encrypted replacement volume had to be attached using `/dev/xvda`, matching the root-device location expected by the instance.

### Snapshot Readiness

The EBS snapshot needed to reach the completed state before it could be used to create the replacement encrypted volume.

### Key Dependency

A disabled KMS key can block dependent storage and compute operations. Key lifecycle changes must therefore be governed as carefully as access-policy changes.

### Public Access Testing

The temporary public-access configuration was appropriate only for the controlled lab test. A production S3 bucket should not be made public without a valid requirement, approved design, and compensating security controls.

---

## 🛡 Security Recommendations

For production environments, the following improvements should be considered:

1. Keep S3 Block Public Access enabled unless public access is explicitly required.
2. Use least-privilege IAM and KMS key policies.
3. Separate key administrators from application key users where operationally appropriate.
4. Enable KMS key rotation based on organizational requirements.
5. Monitor CloudTrail for unexpected `Decrypt`, `DisableKey`, `ScheduleKeyDeletion`, and key-policy modification activity.
6. Use alerts for high-risk KMS administrative actions.
7. Standardize EBS encryption for new volumes and instances to reduce manual conversion work.
8. Test key-disablement and recovery procedures in non-production environments.
9. Document resource dependencies before disabling or scheduling deletion of a KMS key.
10. Preserve audit logs according to organizational retention and compliance requirements.

---

## 🧠 Skills Demonstrated

- AWS KMS administration
- Customer managed key configuration
- Symmetric encryption concepts
- Envelope encryption
- Amazon S3 SSE-KMS configuration
- S3 access-control testing
- AWS Signature Version 4 awareness
- Amazon EBS snapshot management
- EC2 root-volume replacement
- Encrypted EBS volume deployment
- AWS CloudTrail event analysis
- IAM role and key-permission configuration
- Encryption dependency troubleshooting
- Security validation and documentation
- Cloud governance and compliance awareness

---

## 💡 Key Takeaways

- Encryption at rest protects stored cloud data against unauthorized disclosure.
- KMS permissions are separate from S3 object permissions and must also authorize decryption.
- Customer managed keys provide direct lifecycle and access control but also introduce operational responsibility.
- CloudTrail provides valuable evidence for reviewing KMS usage and administrative actions.
- Existing unencrypted EBS root volumes can be replaced through a snapshot-based encryption workflow.
- Disabling a KMS key can affect multiple AWS services simultaneously.
- Key availability, access policy, and change control are essential to workload continuity.
- Encryption is most effective when combined with least privilege, private-by-default storage, continuous monitoring, and tested recovery procedures.

---

## 🏁 Conclusion

This project successfully implemented and validated data-at-rest encryption across Amazon S3 and Amazon EBS using an AWS KMS customer managed key.

The lab confirmed that SSE-KMS protects S3 data beyond basic object permissions, that authenticated signed requests are required for authorized retrieval, and that KMS activity can be audited through AWS CloudTrail. It also demonstrated a practical method for replacing an existing unencrypted EC2 root volume with an encrypted EBS volume.

The key-disablement test provided the most important operational lesson: encryption security and service availability are closely connected. When `MyKMSKey` was unavailable, Amazon S3 could not decrypt the protected object and the EC2 instance could not successfully start from its encrypted root volume. Re-enabling the key restored both services.

Overall, the project strengthened practical skills in AWS encryption, storage security, key lifecycle management, audit analysis, and cloud-security troubleshooting.

---

## ⚠ Disclaimer

This project was completed in a controlled AWS training environment for educational and professional portfolio purposes. Resource names, permissions, account restrictions, and available actions may differ from production AWS environments.

The temporary public-access configuration was performed only to satisfy the controlled lab objective. Public access should not be enabled in a production S3 environment unless it is explicitly required and protected by an approved security architecture.

No credentials, account identifiers, signed URLs, secret values, or sensitive access information are included in this repository documentation.

---

## 👨‍💻 Author

<div align="center">

### Wadondera A. Collins

**ICDFA Trainee | Cohort 11 | Cloud Security Engineering**

Cloud Security • AWS Security • Azure Security • DevSecOps • Identity and Access Management • Infrastructure Protection

</div>

---

<div align="center">

**Completed and documented by Wadondera A. Collins**

</div>
