# AWS KMS DATA-AT-REST ENCRYPTION
## Technical Implementation and Evidence Assessment Report

**Lab:** 5.1 – Encrypting Data at Rest by Using AWS KMS  
**Prepared by:** Wadondera A. Collins  
**Program:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Environment:** AWS restricted training account  
**Assessment status:** Completed with documented evidence limitations  
**Evidence reviewed:** 15 screenshots

---

## 1. Document Purpose

This report presents a technical assessment of the controls implemented during the AWS KMS data-at-rest encryption lab. It is intentionally structured as an **evidence dossier**, not as a repository README or step-by-step lab guide.

The report answers four questions:

1. What security control was implemented?
2. What visible evidence confirms the implementation?
3. What security conclusion can be drawn from that evidence?
4. Which completed outcomes still require stronger screenshot proof?

---

## 2. Assessment Scope

The assessment covers the following control areas:

| Control Area | Assessed Activity |
|---|---|
| Key management | Creation and use of a symmetric customer managed KMS key |
| Object encryption | Protection of `clock.png` in Amazon S3 with SSE-KMS |
| Access control | Anonymous, public-ACL, and signed-access testing |
| Audit logging | Review of `GenerateDataKey` and `Decrypt` CloudTrail events |
| Block-storage protection | Snapshot-based conversion of an EC2 root EBS volume |
| Key availability | Effect of KMS key disablement and subsequent recovery |

### Assessment Boundary

The report uses only the visible content of the supplied screenshots as direct evidence. Lab outcomes described in the completion narrative but not shown in the screenshots are clearly marked as **reported but not screenshot-verified**.

---

## 3. Environment Baseline

| Item | Observed or Lab-Defined Value |
|---|---|
| AWS Region | US East (N. Virginia), `us-east-1` |
| KMS alias | `MyKMSKey` |
| KMS key type | Symmetric |
| KMS usage | Encrypt and decrypt |
| S3 object | `clock.png` |
| S3 encryption | SSE-KMS |
| EC2 instance | `LabInstance` |
| Root device | `/dev/xvda` |
| Original EBS volume size | 8 GiB |
| Snapshot label | `Unencrypted Root Volume` |
| Audit source | AWS CloudTrail Event history |

---

## 4. Control Implementation Summary

The implementation established a customer managed encryption key and used AWS service integrations to protect stored data. Amazon S3 requested a data key from AWS KMS for object encryption. AWS CloudTrail recorded the associated cryptographic operations. For the EC2 workload, the original unencrypted root volume was captured as a snapshot to support creation of an encrypted replacement volume.

The access-control test produced three distinct outcomes:

- The initial anonymous request returned `AccessDenied`.
- After public ACL configuration, the unsigned request returned `InvalidArgument` because AWS Signature Version 4 authorization was required.
- The signed request successfully displayed the encrypted object.

These results show that S3 permission and KMS authorization are separate security controls.

---

# 5. Evidence Assessment Records

## Record 01 – Customer Managed KMS Key

### Control Objective

Establish a dedicated symmetric key for encrypting and decrypting lab resources.

### Evidence

<img width="1359" height="382" alt="Fig 01 create-kms-key" src="https://github.com/user-attachments/assets/61a5ed6d-62bf-4ea2-805d-60cad8f46327" />


**Figure 01 – Customer managed KMS key creation result**

### Visible Evidence

- A green success notification confirms key creation.
- The alias is `MyKMSKey`.
- The key status is `Enabled`.
- The key type is `Symmetric`.
- The key specification is `SYMMETRIC_DEFAULT`.
- Key usage is `Encrypt and decrypt`.

### Assessment

**Verified.** The screenshot directly establishes that a usable symmetric customer managed KMS key was created.

---

## Record 02 – S3 Object Upload

### Control Objective

Store the designated test object in the lab S3 bucket before access and encryption validation.

### Evidence

<img width="1359" height="478" alt="Fig 02 s3-upload-object" src="https://github.com/user-attachments/assets/3c6b7fae-653a-4fb2-869e-48ebd07eff3b" />


**Figure 02 – `clock.png` present in the S3 bucket**

### Visible Evidence

- The bucket contains one object.
- The object name is `clock.png`.
- The object type is PNG.
- The visible size is 505.0 KB.
- The storage class is Standard.

### Assessment

**Verified.** The screenshot confirms successful object storage in the target bucket.

---

## Record 03 – Bucket Encryption Configuration

### Control Objective

Configure the S3 bucket to use AWS KMS-based server-side encryption.

### Evidence

<img width="1351" height="623" alt="Fig 03 s3-edit-default-encryption" src="https://github.com/user-attachments/assets/e60422ee-085b-44ed-a88a-47f638afdd21" />


**Figure 03 – S3 default encryption configuration using SSE-KMS**

### Visible Evidence

- `Server-side encryption with AWS Key Management Service keys (SSE-KMS)` is selected.
- The option to choose from available AWS KMS keys is selected.
- A KMS key ARN is visible in the selection field.
- S3 Bucket Key is set to `Enable`.

### Assessment

**Verified as configuration evidence.** The screen shows the selected settings before the visible Save changes action is performed. Object-level confirmation is provided separately in Record 04.

---

## Record 04 – Object-Level Encryption Verification

### Control Objective

Confirm that the stored object is protected with SSE-KMS.

### Evidence

<img width="1336" height="625" alt="Fig 04 s3-object-encryption-details" src="https://github.com/user-attachments/assets/ad4aa167-97cd-4991-a1dd-664552835b13" />


**Figure 04 – `clock.png` server-side encryption properties**

### Visible Evidence

- Encryption type is `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`.
- An encryption key ARN is displayed.
- Bucket Key is enabled.
- A CRC64NVME checksum is displayed as a separate integrity property.

### Assessment

**Verified.** The screenshot is direct object-level proof that `clock.png` is protected with SSE-KMS.

---

## Record 05 – Initial Anonymous Access Test

### Control Objective

Determine whether the object is publicly accessible under the original S3 access settings.

### Evidence

<img width="1359" height="284" alt="Fig 05 s3-access-denied-error" src="https://github.com/user-attachments/assets/b778e29e-1c20-4c86-9afb-79da6ee3a8f4" />


**Figure 05 – Initial direct object request returning `AccessDenied`**

### Visible Evidence

- The direct S3 object URL is open in a browser.
- The XML response code is `AccessDenied`.
- The message states `Access Denied`.

### Assessment

**Verified.** Anonymous retrieval was blocked under the initial access configuration.

---

## Record 06 – Block Public Access Test Configuration

### Control Objective

Temporarily modify the bucket controls required for the controlled public-access test.

### Evidence

<img width="1359" height="499" alt="Fig 06 s3-block-public-access-settings" src="https://github.com/user-attachments/assets/d98adee7-10b4-47b5-b5c0-8c30d2c39ff1" />


**Figure 06 – Edit Block Public Access settings**

### Visible Evidence

- `Block all public access` is cleared.
- The related ACL- and policy-based blocking options are also cleared.
- The configuration is displayed on the edit screen.

### Assessment

**Verified as configuration evidence.** The screenshot shows the test configuration before saving. This is not a recommended production configuration.

---

## Record 07 – Object Ownership and ACL Configuration

### Control Objective

Enable ACL-based object permissions for the controlled public-access test.

### Evidence

<img width="1311" height="583" alt="Fig 07 s3-object-ownership-acls" src="https://github.com/user-attachments/assets/733ae190-52d3-4e03-96bb-15c2c87575ee" />


**Figure 07 – Object Ownership changed to ACLs enabled**

### Visible Evidence

- `ACLs enabled` is selected.
- `Bucket owner preferred` remains selected.
- The acknowledgement that ACLs will be restored is checked.

### Assessment

**Verified as configuration evidence.** The screenshot supports the ACL preparation required before making the object public.

---

## Record 08 – Public ACL Application Result

### Control Objective

Confirm that the object-level public-access change completed successfully.

### Evidence

<img width="1355" height="512" alt="Fig 08 s3-make-public-success" src="https://github.com/user-attachments/assets/6d9716cb-7615-4c62-8df6-4f6f526f8eba" />


**Figure 08 – Successful Make public operation**

### Visible Evidence

- The green notification states `Successfully edited public access`.
- One object was successfully updated.
- The visible object size is 505.0 KB.
- Zero objects failed.

### Assessment

**Verified.** The object-level public ACL operation completed successfully.

---

## Record 09 – Unsigned Access After Public ACL

### Control Objective

Determine whether public object permission alone allows retrieval of an SSE-KMS-encrypted object.

### Evidence

<img width="1359" height="221" alt="Fig 09 s3-invalid-argument-error" src="https://github.com/user-attachments/assets/7e3ee3d3-a9a5-49f2-961c-d1516e00b582" />


**Figure 09 – SSE-KMS request rejected without Signature Version 4 authorization**

### Visible Evidence

- The response code is `InvalidArgument`.
- The message states that requests using server-side encryption with AWS KMS managed keys require AWS Signature Version 4.
- The missing argument is `Authorization`.

### Assessment

**Verified.** Public object permission did not provide the authentication required for retrieval of the KMS-encrypted object.

### Security Interpretation

This result demonstrates control separation. S3 object permission and authorization to use the KMS-protected data path are not equivalent.

---

## Record 10 – Signed Object Access

### Control Objective

Validate successful retrieval through an authenticated Signature Version 4 request.

### Evidence

<img width="1358" height="639" alt="Fig 10 s3-presigned-url-image" src="https://github.com/user-attachments/assets/33688d0c-5a8b-4473-9908-22187e793f1d" />


**Figure 10 – `clock.png` displayed through a signed S3 request**

### Visible Evidence

- The image renders successfully in the browser.
- The address bar contains `X-Amz-Algorithm=AWS4-HMAC-SHA256`.
- Additional `X-Amz-` query parameters are visible.

### Assessment

**Verified.** A signed request successfully retrieved the encrypted object.

---

## Record 11 – CloudTrail `GenerateDataKey` Audit Event

### Control Objective

Confirm that KMS data-key generation associated with S3 encryption is auditable.

### Evidence

<img width="1332" height="615" alt="Fig 11 cloudtrail-generate-data-key" src="https://github.com/user-attachments/assets/d13345a2-06d0-45a9-a261-63c8ff4c89e4" />


**Figure 11 – CloudTrail record for `GenerateDataKey`**

### Visible Evidence

- Event source is `kms.amazonaws.com`.
- Event name is `GenerateDataKey`.
- AWS Region is `us-east-1`.
- Key specification is `AES_256`.
- The encryption context contains an S3 ARN.
- A KMS key ARN is visible in `keyId`.
- The assumed lab role appears in the identity context.

### Assessment

**Verified.** CloudTrail recorded the KMS operation used in the S3 encryption workflow.

---

## Record 12 – CloudTrail `Decrypt` Audit Event

### Control Objective

Confirm that KMS decryption activity associated with S3 object access is auditable.

### Evidence

<img width="1333" height="626" alt="Fig 12 cloudtrail-decrypt-event" src="https://github.com/user-attachments/assets/3f50e1f7-c246-47dd-bf00-d187c907a5b5" />


**Figure 12 – CloudTrail record for `Decrypt`**

### Visible Evidence

- Event source is `kms.amazonaws.com`.
- Event name is `Decrypt`.
- AWS Region is `us-east-1`.
- The event is invoked by the S3 service context.
- The assumed lab role appears in the identity data.

### Assessment

**Verified.** The screenshot provides audit evidence of the KMS decryption operation associated with authorized object access.

---

## Record 13 – Original EC2 Root-Volume State

### Control Objective

Establish the root volume's encryption state before conversion.

### Evidence

<img width="1339" height="616" alt="Fig 13 ec2-instance-block-devices" src="https://github.com/user-attachments/assets/1d7490d0-62ff-466e-9adc-7191ec1ff02c" />


**Figure 13 – Original `/dev/xvda` root volume shown as unencrypted**

### Visible Evidence

- The selected instance is `LabInstance`.
- Instance state is Running.
- Status checks show `2/2 checks passed`.
- Device name is `/dev/xvda`.
- Volume size is 8 GiB.
- Volume state is `In-use`.
- Attachment status is `Attached`.
- Encryption is `No`.
- No KMS key ID is displayed.

### Assessment

**Verified.** The screenshot establishes the unencrypted baseline before the EBS conversion workflow.

---

## Record 14 – Root-Volume Snapshot Preparation

### Control Objective

Prepare a snapshot of the original unencrypted root volume.

### Evidence

<img width="1353" height="615" alt="Fig 14 ec2-create-snapshot" src="https://github.com/user-attachments/assets/ef23c483-327b-4341-a960-23a1b8e237b2" />


**Figure 14 – Create snapshot configuration for the source root volume**

### Visible Evidence

- The source EBS volume is displayed.
- Availability Zone is `us-east-1c`.
- Encryption status is `Not encrypted`.
- A `Name` tag is configured with value `Unencrypted Root Volume`.

### Assessment

**Verified as configuration evidence.** The screenshot shows the correct source and tag before snapshot creation.

---

## Record 15 – Snapshot Completion and Volume-Creation Progression

### Control Objective

Confirm snapshot readiness for creation of the replacement EBS volume.

### Evidence

<img width="1343" height="597" alt="Fig 15 ec2-snapshot-completed" src="https://github.com/user-attachments/assets/4f2b8e8d-9de4-4312-a2c0-e4452b7e09fe" />


**Figure 15 – Completed snapshot of the unencrypted root volume**

### Visible Evidence

- Snapshot status is `Completed`.
- Progress is 100%.
- Snapshot name is `Unencrypted Root Volume`.
- Full snapshot size is 1.8 GiB.
- Source volume size is 8 GiB.
- Snapshot encryption is `Not encrypted`.
- A green notification states that a volume was successfully created.

### Assessment

**Partially verified for Task 6.** The screenshot directly proves snapshot completion and provides visible indication that volume creation succeeded. The screenshot does not show the new volume's encryption state, selected KMS key, or final attachment to `LabInstance`.

---

# 6. Evidence Coverage Matrix

| Requirement | Evidence | Assessment Status |
|---|---|---|
| Create an enabled symmetric customer managed key | Figure 01 | Verified |
| Upload `clock.png` to S3 | Figure 02 | Verified |
| Configure SSE-KMS | Figures 03–04 | Verified |
| Test initial anonymous access | Figure 05 | Verified |
| Modify public-access controls for the lab test | Figures 06–08 | Verified |
| Test unsigned access after public ACL | Figure 09 | Verified |
| Test signed access | Figure 10 | Verified |
| Audit `GenerateDataKey` | Figure 11 | Verified |
| Audit `Decrypt` | Figure 12 | Verified |
| Establish original unencrypted root-volume state | Figure 13 | Verified |
| Create and complete source snapshot | Figures 14–15 | Verified |
| Show new EBS volume encrypted with `MyKMSKey` | Not supplied | Evidence required |
| Show encrypted volume attached as `/dev/xvda` | Not supplied | Evidence required |
| Show KMS key disabled | Not supplied | Evidence required |
| Show S3 `KMS.DisabledException` | Not supplied | Evidence required |
| Show EC2 startup failure with disabled key | Not supplied | Evidence required |
| Show `DisableKey` and failed `CreateGrant` events | Not supplied | Evidence required |
| Show key re-enabled and EC2 recovery | Not supplied | Evidence required |

---

## 7. Key Findings

### Finding 1 – KMS Key Provisioning Was Successful

The evidence shows an enabled symmetric key configured for encryption and decryption. This satisfied the cryptographic foundation required by the later S3 and EBS activities.

### Finding 2 – S3 Object Encryption Was Directly Confirmed

The object properties identify SSE-KMS, an encryption key ARN, and Bucket Key enabled. This is stronger proof than relying only on the bucket's pre-save configuration screen.

### Finding 3 – Public Permission Did Not Bypass KMS-Backed Access Requirements

The access sequence changed from `AccessDenied` to `InvalidArgument` after the public ACL operation. The second error explicitly required Signature Version 4 authorization. Signed access then succeeded.

### Finding 4 – KMS Operations Were Auditable

CloudTrail recorded both `GenerateDataKey` and `Decrypt`, including service, Region, identity context, and KMS-related request data.

### Finding 5 – The EC2 Evidence Package Is Incomplete

The screenshots establish the original unencrypted root volume and completed snapshot. The evidence does not directly establish that the replacement volume was encrypted with `MyKMSKey` and attached as the root device.

### Finding 6 – Key-Disablement Testing Lacks Screenshot Proof

The lab narrative states that disabling the key blocked S3 decryption and EC2 startup, followed by successful recovery after re-enablement. No supplied screenshot directly verifies those outcomes.

---

## 8. Risk and Control Analysis

| Risk | Observed Control | Assessment |
|---|---|---|
| Unauthorized reading of stored objects | SSE-KMS applied to `clock.png` | Control verified |
| Accidental public exposure | Initial Block Public Access plus KMS authorization requirement | Layered protection observed |
| Untraceable cryptographic activity | CloudTrail KMS events | Auditability verified |
| Unencrypted EC2 boot storage | Snapshot-based replacement workflow | Initial and snapshot stages verified |
| Service outage caused by key unavailability | Disable/re-enable exercise | Reported, not screenshot-verified |
| Disclosure through published evidence | Screenshots contain identifiers and signed-request details | Redaction required before public release |

---

## 9. Evidence Gaps and Required Additions

For complete proof of Tasks 6 and 7, the evidence package should add:

1. `Fig16 EC2 Encrypted Root Volume.png`  
   Show `Encrypted: Yes`, the KMS key ID, device `/dev/xvda`, and attached or in-use state.

2. `Fig17 KMS Key Disabled.png`  
   Show `MyKMSKey` in the disabled state.

3. `Fig18 S3 KMS Disabled Exception.png`  
   Show the failed object-open operation and `KMS.DisabledException`.

4. `Fig19 EC2 Start Failure.png`  
   Show `LabInstance` returning to Stopped after the failed start.

5. `Fig20 CloudTrail DisableKey Event.png`  
   Show the `DisableKey` event record.

6. `Fig21 CloudTrail CreateGrant Failure.png`  
   Show the CreateGrant error associated with the disabled key.

7. `Fig22 KMS Key Reenabled.png`  
   Show `MyKMSKey` restored to Enabled.

8. `Fig23 EC2 Recovery Running.png`  
   Show `LabInstance` running with the encrypted root volume attached.

---

## 10. Security Recommendations

1. Keep S3 Block Public Access enabled for private workloads.
2. Prefer Object Ownership with ACLs disabled unless ACLs are specifically required.
3. Apply least-privilege IAM and KMS key policies.
4. Separate key administration from routine key use where the operating model permits.
5. Monitor `DisableKey`, `PutKeyPolicy`, `ScheduleKeyDeletion`, `Decrypt`, and `CreateGrant` activity.
6. Enable EBS encryption by default for new workloads to avoid later root-volume conversion.
7. Maintain a dependency register before disabling or scheduling deletion of a KMS key.
8. Test key recovery and workload restoration in a non-production environment.
9. Retain CloudTrail records according to organizational audit requirements.
10. Redact identifiers and signed URL parameters before publishing screenshots.

---

## 11. Validation Statement

Based on the supplied screenshots, Tasks 1 through 5 are strongly supported by direct visual evidence. Task 6 is supported through the original-volume, snapshot-creation, and snapshot-completion stages. The final encrypted-volume attachment and all Task 7 disablement and recovery outcomes require additional screenshots for complete evidence coverage.

No screenshot was assigned to a task that the screenshot does not visibly support.

---

## 12. Conclusion

The evidence demonstrates successful creation of a customer managed KMS key, SSE-KMS protection of an S3 object, layered access-control behavior, authenticated retrieval, and CloudTrail visibility of KMS operations.

The report also establishes the unencrypted EC2 root-volume baseline and the completed snapshot required for conversion. Because the current evidence set does not show the final encrypted root volume or key-disablement testing, those outcomes are documented separately from the directly verified findings.

The implementation demonstrates an important cloud security principle: encryption, authorization, audit logging, and availability must be managed together. A secure design depends not only on encrypting data, but also on governing who may use the key, monitoring every sensitive operation, and protecting the lifecycle of the key itself.

---

## 13. Publication and Confidentiality Notice

Before publishing the screenshots to GitHub, redact visible AWS account IDs, role and session identifiers, access-key identifiers, request IDs, host IDs, public IP addresses, unnecessary ARNs, and presigned URL query parameters.

The public-access configuration was performed only for the controlled lab test and is not a recommended production configuration.

---

## 14. Author Declaration

**Wadondera A. Collins**  
ICDFA Trainee | Cohort 11 | Cloud Security Engineering

This report documents the implementation evidence supplied for the completed AWS KMS data-at-rest encryption lab. Evidence limitations are disclosed to preserve technical accuracy and portfolio credibility.
