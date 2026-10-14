# PROFESSIONAL LAB DOCUMENTATION REPORT

<div align="center">

## Lab 4.1: Securing VPC Resources by Using Security Groups

### Evidence-Based Implementation, Validation, and Security Analysis

**Prepared by:** Wadondera A. Collins  
**Program:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Platform:** Amazon Web Services  
**Lab Status:** Completed  
**Lab Date:** 14 October 2026  
**Estimated Duration:** 90 minutes

</div>

---

## Document Control

| Field | Details |
|---|---|
| Document type | Professional assignment documentation report |
| Assignment | Lab 4.1: Securing VPC Resources by Using Security Groups |
| Author | Wadondera A. Collins |
| Evidence reviewed | 19 supplied screenshots |
| Evidence method | Visual screenshot-to-task matching and cross-validation |
| Intended audience | Instructor, assessor, recruiter, and cloud security reviewer |
| Document purpose | Detailed evidence report, separate from the GitHub `README.md` |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Project Context](#2-project-context)
3. [Objectives](#3-objectives)
4. [Professional Value](#4-professional-value)
5. [Skills Demonstrated](#5-skills-demonstrated)
6. [AWS Services and Tools](#6-aws-services-and-tools)
7. [Lab Environment](#7-lab-environment)
8. [Architecture and Traffic Flow](#8-architecture-and-traffic-flow)
9. [Corrected Evidence Naming Plan](#9-corrected-evidence-naming-plan)
10. [Detailed Methodology, Evidence, and Analysis](#10-detailed-methodology-evidence-and-analysis)
11. [Corrected Screenshot Traceability Matrix](#11-corrected-screenshot-traceability-matrix)
12. [Results and Validation](#12-results-and-validation)
13. [Security Analysis](#13-security-analysis)
14. [Evidence Gaps and Limitations](#14-evidence-gaps-and-limitations)
15. [Recommended Next Steps](#15-recommended-next-steps)
16. [Security and Privacy Notes](#16-security-and-privacy-notes)
17. [Conclusion](#17-conclusion)
18. [Disclaimer](#18-disclaimer)
19. [Author](#19-author)

---

## 1. Executive Summary

This report documents the implementation and validation activities completed during Lab 4.1, **Securing VPC Resources by Using Security Groups**. The assignment focused on protecting Amazon EC2 resources distributed across public and private subnets in an Amazon Virtual Private Cloud.

The lab used security groups for stateful instance-level traffic control and a network access control list for subnet-level filtering. The environment included two public proxy instances and one application server in a private subnet. The work progressed from examining the initial network design to restricting HTTP traffic, sharing permissions through security group references, applying a network ACL rule, and exploring secure administrative access methods.

The corrected evidence review maps all 19 supplied screenshots according to their visible content and attachment order. Tasks 1, 2, and 3 have strong visual coverage. Tasks 4, 5, and 6 have partial evidence because configuration editors are visible, but some save confirmations and functional tests are absent. The current 19-image package does not visibly document Tasks 7 and 8.

This report is intentionally more detailed than the GitHub `README.md`. It provides task-level methodology, figure analysis, control validation, traceability, evidence strength, limitations, and security findings.

---

## 2. Project Context

The lab environment contained an AWS VPC named `LabVPC` with public and private network segments. `ProxyServer1` and `ProxyServer2` operated from public subnets and forwarded HTTP traffic to `AppServer`, which operated from a private subnet.

The private application server had no public IPv4 address visible in the supplied evidence. Internet-facing access was therefore provided through the public proxy instances. A NAT gateway supported outbound connectivity from private resources, while an internet gateway supported public subnet routing.

---

## 3. Objectives

The completed assignment was designed to demonstrate the ability to:

- Examine security groups and interpret allowed traffic.
- Analyze VPCs, subnets, route tables, gateways, and EC2 placement.
- Modify the security groups attached to EC2 instances.
- Apply least-privilege inbound rules.
- Restrict HTTP traffic to a single IPv4 source using `/32` notation.
- Scale authorized access through a security group reference.
- Configure and interpret network ACL rules.
- Test HTTP connectivity through public proxy instances.
- Connect to a private instance through a bastion host and SSH agent forwarding.
- Connect directly to a private instance with AWS Systems Manager Session Manager.
- Modify application content from a managed terminal session.

---

## 4. Professional Value

This assignment demonstrates practical cloud security engineering skills in network segmentation, access control, defense in depth, least privilege, routing analysis, and connectivity validation.

The evidence-based approach also demonstrates professional documentation discipline. Each screenshot is associated only with a step supported by visible content. Configuration intent is distinguished from confirmed completion when the Save action or post-change result is not shown.

---

## 5. Skills Demonstrated

- Amazon VPC inspection
- IPv4 CIDR analysis
- Public and private subnet identification
- Route table validation
- Internet gateway inspection
- NAT gateway inspection
- EC2 network placement analysis
- Security group rule inspection
- Security group reassignment
- Least-privilege source restriction
- Network ACL configuration
- HTTP connectivity testing
- Evidence validation and traceability
- Cloud security documentation
- Defense-in-depth analysis

---

## 6. AWS Services and Tools

| Service or tool | Purpose in the assignment |
|---|---|
| Amazon VPC | Provided the isolated network, subnets, route tables, gateways, and network ACL. |
| Amazon EC2 | Hosted the proxy servers and private application server. |
| Security groups | Enforced stateful traffic controls at the EC2 network-interface level. |
| Network ACL | Provided subnet-level stateless traffic filtering. |
| Internet gateway | Supported internet routing for public subnet resources. |
| NAT gateway | Supported outbound connections initiated by resources in the private subnet. |
| Route tables | Controlled local, internet-bound, and NAT-bound traffic paths. |
| Web browser | Validated HTTP access through the public proxy instances. |
| SSH and `ssh-agent` | Supported the bastion access method described in Task 7. |
| Systems Manager Session Manager | Supported managed private-instance access described in Task 8. |
| Linux shell | Supported file creation and website modification in the lab procedure. |

---

## 7. Lab Environment

### 7.1 VPC and Subnets

| Resource | Visible configuration | Intended role |
|---|---|---|
| `LabVPC` | `10.0.0.0/16` | Network boundary for the lab. |
| `PublicSubnetA` | `10.0.1.0/24` | Hosted `ProxyServer1` and the NAT gateway. |
| `PublicSubnetB` | `10.0.2.0/24` | Hosted `ProxyServer2`, later intended for the bastion role. |
| `PrivateSubnet` | `10.0.11.0/24` | Hosted `AppServer`. |

### 7.2 EC2 Resources

| Instance | Visible placement | Visible addressing |
|---|---|---|
| `AppServer` | `PrivateSubnet` | Private IPv4 shown; no public IPv4 shown. |
| `ProxyServer1` | `PublicSubnetA` | Public IPv4 `54.146.255.143`; private IPv4 `10.0.1.7`. |
| `ProxyServer2` | `PublicSubnetB` | Public IPv4 `35.171.23.37`; private IPv4 `10.0.2.217`. |

### 7.3 Visible Network Controls

| Control | Visible evidence |
|---|---|
| `AppServerSG` inbound | Initial HTTP rule on TCP port 80 from `0.0.0.0/0`. |
| `AppServerSG` outbound | All traffic allowed to `0.0.0.0/0`. |
| `ProxySG` | Attached to `ProxyServer1` with inbound HTTP access visible. |
| `ProxySG2` | Successfully modified to allow inbound HTTP from `0.0.0.0/0`. |
| Network ACL rule 99 | HTTP on TCP port 80 configured as Deny from `0.0.0.0/0`. |

---

## 8. Architecture and Traffic Flow

```text
Internet
   |
Internet Gateway attached to LabVPC
   |
   +------------------------+------------------------+
   |                                                 |
PublicSubnetA                                   PublicSubnetB
10.0.1.0/24                                    10.0.2.0/24
   |                                                 |
ProxyServer1                                     ProxyServer2
ProxySG                                          ProxySG2 / ProxySG
   |                                                 |
   +---------------------- HTTP ---------------------+
                             |
                             v
                       PrivateSubnet
                       10.0.11.0/24
                             |
                         AppServer
                        AppServerSG
                   No public IPv4 shown

Private outbound path:
AppServer -> private route table -> NAT gateway in PublicSubnetA -> Internet Gateway
```

---

## 9. Corrected Evidence Naming Plan

The following corrected mapping replaces the earlier inaccurate filename assignments. Rename each supplied screenshot exactly as indicated and place the renamed images in an `evidence` directory beside the Markdown report.

| Original screenshot | Correct evidence filename | Lab task |
|---|---|---:|
| `Screenshot 2026-10-14 081048.png` | `Fig01 LabVPC Subnet Inventory.png` | 1 |
| `Screenshot 2026-10-14 081133.png` | `Fig02 AppServerSG Initial HTTP Inbound Rule.png` | 1 |
| `Screenshot 2026-10-14 081353.png` | `Fig03 NAT Gateway Summary.png` | 1 |
| `Screenshot 2026-10-14 082223.png` | `Fig04 ProxyServer2 Instance Details.png` | 2 |
| `Screenshot 2026-10-14 082506.png` | `Fig05 Private Route Table Name Tag.png` | 1 |
| `Screenshot 2026-10-14 082705.png` | `Fig06 ProxyServer1 Security Group HTTP Rule.png` | 2 |
| `Screenshot 2026-10-14 082915.png` | `Fig07 ProxyServer1 Website Connectivity Test.png` | 3 |
| `Screenshot 2026-10-14 083012.png` | `Fig08 AppServer Private Subnet Placement.png` | 1 |
| `Screenshot 2026-10-14 083232.png` | `Fig09 ProxySG2 HTTP Rule Confirmation.png` | 2 |
| `Screenshot 2026-10-14 083328.png` | `Fig10 AppServerSG ProxyServer1 CIDR Rule.png` | 4 |
| `Screenshot 2026-10-14 083608.png` | `Fig11 Network ACL Rule 99 HTTP Deny.png` | 6 |
| `Screenshot 2026-10-14 083702.png` | `Fig12 AppServerSG Outbound Rule.png` | 1 |
| `Screenshot 2026-10-14 083758.png` | `Fig13 ProxyServer2 ProxySG Reassignment.png` | 5 |
| `Screenshot 2026-10-14 084045.png` | `Fig14 LabVPC Internet Gateway.png` | 2 |
| `Screenshot 2026-10-14 090259.png` | `Fig15 ProxyServer2 Website Connectivity Test.png` | 3 |
| `Screenshot 2026-10-14 090450.png` | `Fig16 ProxyServer1 Instance Details.png` | 2 |
| `Screenshot 2026-10-14 091902.png` | `Fig17 LabVPC CIDR Confirmation.png` | 1 |
| `Screenshot 2026-10-14 092016.png` | `Fig18 NAT Gateway Detailed Configuration.png` | 1 |
| `Screenshot 2026-10-14 092719.png` | `Fig19 PublicSubnetA Internet Gateway Route.png` | 2 |

### Evidence Interpretation Standard

- **Strong evidence:** The screenshot visibly shows the completed state or a successful functional result.
- **Partial evidence:** The screenshot shows the intended configuration, but a Save button or another incomplete-state indicator remains visible.
- **Missing evidence:** No supplied screenshot visibly supports the required outcome.

---

## 10. Detailed Methodology, Evidence, and Analysis

## Task 1: Analyzing the VPC and Private Subnet Resource Settings

### 10.1.1 LabVPC Identification

The VPC inventory was reviewed to distinguish the lab network from the default VPC.

<img width="1211" height="225" alt="Fig01 LabVPC Subnet Inventory" src="https://github.com/user-attachments/assets/ada9c0c0-cdd8-43db-906b-4aafa6e894ce" />


**Figure 17: LabVPC CIDR confirmation.**  
The VPC list shows two available VPCs. The row named `LabVPC` displays IPv4 CIDR `10.0.0.0/16`. The other visible VPC uses the default CIDR `172.31.0.0/16`.

**Evidence assessment:** Strong. The VPC name and IPv4 CIDR are directly visible.

### 10.1.2 Subnet Inventory

The subnet inventory was examined to identify the public and private network segments used by the lab.

<img width="1203" height="422" alt="Fig02 AppServerSG Initial HTTP Rule" src="https://github.com/user-attachments/assets/adf06b3f-fc5f-442a-9564-9a50480605b4" />


**Figure 1: LabVPC subnet inventory.**  
The screenshot lists `PublicSubnetA` with `10.0.1.0/24`, `PublicSubnetB` with `10.0.2.0/24`, and `PrivateSubnet` with `10.0.11.0/24`. Each named lab subnet shows an available state and association with `LabVPC`.

**Evidence assessment:** Strong. The subnet names, CIDRs, states, and VPC association are visible.

### 10.1.3 Private Route Table Name

The Name tag of the route table associated with the private subnet was changed from the placeholder value to `Private`.

<img width="1210" height="262" alt="Fig03 ProxyServer2 Initial Website Test" src="https://github.com/user-attachments/assets/c3f7a02e-69e9-4b16-ab99-0a95ea267521" />


**Figure 5: Private route table name tag.**  
The Manage tags page shows the `Name` key with the value `Private`. The Save button remains visible.

**Evidence assessment:** Partial. The intended tag value is visible, but the screenshot does not independently confirm that Save was selected.

### 10.1.4 NAT Gateway Summary

The NAT gateway was inspected as part of the private-subnet outbound path.

<img width="1359" height="410" alt="Fig04 NAT Gateway Summary" src="https://github.com/user-attachments/assets/ae6e8eb6-50b2-4e2d-92d2-c2fee73f4e2c" />


**Figure 3: NAT gateway summary.**  
The NAT gateway list shows one public NAT gateway in the available state. The visible row includes public IPv4 `32.192.132.176` and private IPv4 `10.0.1.15`.

**Evidence assessment:** Strong. The gateway connectivity type, state, and addresses are visible.

### 10.1.5 NAT Gateway Detailed Configuration

<img width="1210" height="490" alt="Fig05 LabVPC Internet Gateway" src="https://github.com/user-attachments/assets/0c4b1c18-a61e-4942-a278-518f35bab9e8" />


**Figure 18: NAT gateway detailed configuration.**  
The details pane shows the public NAT gateway associated with `LabVPC` and `PublicSubnetA`. The state is Available, and the visible private IPv4 value is `10.0.1.15`.

**Evidence assessment:** Strong. The VPC, subnet, connectivity type, state, and addressing are visible.

### 10.1.6 AppServer Placement

`AppServer` was inspected to verify its placement in the private subnet.

<img width="1188" height="552" alt="Fig06 Private Route Table Name Tag" src="https://github.com/user-attachments/assets/420429d6-a427-4f34-b5a1-a0fa82c1afd5" />


**Figure 8: AppServer private subnet placement.**  
The selected `AppServer` instance is running and shows private IPv4 `10.0.11.135`. The public IPv4 field is empty, and the subnet field identifies `PrivateSubnet`.

**Evidence assessment:** Strong. The instance state, private address, absence of a public address, VPC, and subnet are visible.

### 10.1.7 Initial AppServerSG Inbound Rule

<img width="1211" height="546" alt="Fig07 AppServerSG ProxyServer1 CIDR Rule" src="https://github.com/user-attachments/assets/a39c38cf-71af-469d-91d7-34bd10b744b3" />


**Figure 2: AppServerSG initial HTTP inbound rule.**  
The `AppServerSG` inbound rules view shows IPv4 HTTP traffic using TCP port 80 with source `0.0.0.0/0`.

**Evidence assessment:** Strong. The saved security group rule is visible in the rules table.

### 10.1.8 AppServerSG Outbound Rule

<img width="1199" height="527" alt="Fig08 AppServer Private Subnet Placement" src="https://github.com/user-attachments/assets/6355b80b-0df1-4d63-a421-0c3675bdb20d" />


**Figure 12: AppServerSG outbound rule.**  
The outbound rules view shows one IPv4 rule permitting all traffic, protocols, and port ranges to destination `0.0.0.0/0`.

**Evidence assessment:** Strong. The outbound rule is visible in the saved rules table.

### Task 1 Result

The supplied screenshots strongly support identification of `LabVPC`, the three named subnets, the NAT gateway, private application-server placement, and the initial application security group rules. The screenshot of the private route table tag is partial because Save remains visible. The actual private route table route to the NAT gateway is not included in the supplied evidence.

---

## Task 2: Analyzing the Public Subnet Resource Settings

### 10.2.1 ProxyServer1 Instance Details

<img width="1198" height="570" alt="Fig09 Network ACL HTTP Deny Rule" src="https://github.com/user-attachments/assets/e19695cf-ea3f-4287-8756-4d2561c61955" />


**Figure 16: ProxyServer1 instance details.**  
The selected `ProxyServer1` instance is running in `PublicSubnetA`. The screen shows public IPv4 `54.146.255.143` and private IPv4 `10.0.1.7`.

**Evidence assessment:** Strong. The instance name, state, VPC, subnet, and addressing are visible.

### 10.2.2 ProxyServer1 Security Group

<img width="1206" height="550" alt="Fig10 PublicSubnetA Internet Gateway Route" src="https://github.com/user-attachments/assets/4cf50f27-ea65-4e42-9b68-57525989a99e" />


**Figure 6: ProxyServer1 security group and HTTP rule.**  
The Security tab for `ProxyServer1` displays the attached proxy security group. The inbound rule table shows TCP port 80 from source `0.0.0.0/0`.

**Evidence assessment:** Strong. The security group association and inbound HTTP rule are visible.

### 10.2.3 ProxyServer2 Instance Details

<img width="1198" height="577" alt="Fig11 ProxySG2 HTTP Rule Confirmation" src="https://github.com/user-attachments/assets/99b9b911-3e3b-46f3-ba37-cf0718431c99" />


**Figure 4: ProxyServer2 instance details.**  
The selected `ProxyServer2` instance is running in `PublicSubnetB`. Public IPv4 `35.171.23.37` and private IPv4 `10.0.2.217` are visible.

**Evidence assessment:** Strong. The instance placement and addressing are directly visible.

### 10.2.4 ProxySG2 HTTP Rule

<img width="1177" height="542" alt="Fig12 ProxyServer1 Network Details" src="https://github.com/user-attachments/assets/264c3e2c-b4a5-41af-a7a5-19a8de306684" />


**Figure 9: ProxySG2 HTTP rule confirmation.**  
A green success banner states that inbound security group rules were successfully modified. The rule table shows IPv4 HTTP traffic over TCP port 80 from `0.0.0.0/0`.

**Evidence assessment:** Strong. The rule and successful modification banner are visible.

### 10.2.5 Internet Gateway

<img width="1183" height="539" alt="Fig13 ProxyServer2 Network Details" src="https://github.com/user-attachments/assets/e2ce44ba-9f88-44e0-b082-e92745ca9422" />


**Figure 14: LabVPC internet gateway.**  
The internet gateway named `IGW` is shown in the Attached state. The details pane associates the gateway with `LabVPC`.

**Evidence assessment:** Strong. The gateway name, state, and VPC association are directly visible.

### 10.2.6 PublicSubnetA Routing

<img width="1201" height="481" alt="Fig14 ProxyServer1 Initial Website Test" src="https://github.com/user-attachments/assets/d805139d-6c56-4f08-82de-c70ceb014965" />


**Figure 19: PublicSubnetA internet gateway route.**  
`PublicSubnetA` is selected. Its route table displays local routing for `10.0.0.0/16` and default route `0.0.0.0/0` targeting the internet gateway.

**Evidence assessment:** Strong. The subnet selection, route destinations, and internet gateway target are visible.

### Task 2 Result

The evidence confirms the placement and addressing of both public proxy instances, their relevant inbound HTTP access, the attached internet gateway, and the public default route from `PublicSubnetA`.

---

## Task 3: Testing HTTP Connectivity from Public EC2 Instances

### 10.3.1 ProxyServer1 Website Test

<img width="1205" height="310" alt="Fig15 LabVPC CIDR Confirmation" src="https://github.com/user-attachments/assets/211ca88d-1476-40c7-828c-a6d1866745c4" />


**Figure 7: ProxyServer1 website connectivity test.**  
The browser address bar displays `54.146.255.143`, which matches the public IPv4 address shown for `ProxyServer1` in Figure 16. The webpage displays `Hello from the web server running on the AppServer instance!`

**Evidence assessment:** Strong. The browser provides a functional result that matches the documented ProxyServer1 endpoint.

### 10.3.2 ProxyServer2 Website Test

<img width="1325" height="216" alt="Fig16 NAT Gateway Detailed Configuration" src="https://github.com/user-attachments/assets/265a46b9-5ab1-4bf1-8acc-30d751ef4f84" />


**Figure 15: ProxyServer2 website connectivity test.**  
The browser address bar displays `35.171.23.37`, which matches the public IPv4 address shown for `ProxyServer2` in Figure 4. The same AppServer webpage message is visible.

**Evidence assessment:** Strong. The browser provides a functional result that matches the documented ProxyServer2 endpoint.

### Task 3 Result

The two browser tests confirm that both public proxy endpoints initially forwarded HTTP traffic to the private application server.

---

## Task 4: Restricting HTTP Access by Using an IP Address

### 10.4.1 ProxyServer1 `/32` Source Rule

<img width="1141" height="442" alt="Fig17 AppServerSG Outbound Rule" src="https://github.com/user-attachments/assets/d42cabe1-f772-4bb0-a30f-fca2fc04e599" />


**Figure 10: AppServerSG ProxyServer1 CIDR rule.**  
The inbound rule editor shows HTTP, TCP port 80, and custom source `10.0.1.7/32`. Figure 16 identifies `10.0.1.7` as the private IPv4 address of `ProxyServer1`.

**Evidence assessment:** Partial. The intended least-privilege source is correct and cross-validated, but the Save rules button remains visible.

### Task 4 Result

The evidence supports preparation of the `/32` rule for `ProxyServer1`. The supplied image set does not contain a saved-rule confirmation, the successful post-change test through `ProxyServer1`, or the expected failure through `ProxyServer2`.

---

## Task 5: Scaling Restricted HTTP Access by Referencing a Security Group

### 10.5.1 ProxyServer2 ProxySG Reassignment

<img width="1359" height="210" alt="Fig18 ProxyServer1 Security Group Attachment" src="https://github.com/user-attachments/assets/8098e24d-b71d-484b-abc0-457d40a732f8" />


**Figure 13: ProxyServer2 ProxySG reassignment.**  
The Change security groups page is open for the instance ID shown for `ProxyServer2`. The associated security group list displays the same ProxySG identifier visible on `ProxyServer1` in Figure 6.

**Evidence assessment:** Partial. The intended shared proxy security group is selected, but the Save button remains visible.

### Task 5 Result

The screenshot supports the intended security group reassignment. The screenshot set does not show the saved reassignment, the `AppServerSG` HTTP rule referencing `ProxySG`, or the final website tests through both proxy servers.

---

## Task 6: Restricting HTTP Access by Using a Network ACL

### 10.6.1 Network ACL Rule 99

<img width="1140" height="279" alt="Fig19 ProxyServer2 ProxySG Reassignment" src="https://github.com/user-attachments/assets/e7423fb7-a461-495e-9468-bfbd8ea63d4c" />


**Figure 11: Network ACL rule 99 HTTP deny.**  
The inbound network ACL editor shows rule number `99`, type HTTP, TCP port 80, source `0.0.0.0/0`, and action Deny. Rule `100` allows all traffic.

**Evidence assessment:** Partial. The rule is configured correctly in the editor, but Save changes remains visible.

### Task 6 Result

The screenshot supports configuration of rule `99`. The supplied evidence does not show the saved ACL state, the expected browser timeout, rule `98` allowing HTTP, or the restored website response.

---

## Task 7: Connecting to AppServer Through a Bastion Host

The lab procedure required:

- Renaming `ProxyServer2` to `Bastion`.
- Creating `BastionSG`.
- Attaching `BastionSG` to the bastion instance.
- Allowing SSH to `AppServer` from the bastion private address.
- Starting `ssh-agent` and loading the lab key.
- Connecting to the bastion with agent forwarding.
- Connecting from the bastion to `AppServer`.
- Creating `newfile.txt` on `AppServer`.

### Task 7 Evidence Status

No screenshot in the supplied 19-image package visibly shows these Task 7 outcomes. Figure 4 shows `ProxyServer2` before the bastion rename, and Figure 13 belongs to the earlier proxy security group reassignment workflow. Neither image should be presented as bastion evidence.

### Recommended Missing Evidence

- `Fig20 ProxyServer2 Renamed Bastion.png`
- `Fig21 BastionSG SSH Inbound Rule.png`
- `Fig22 BastionSG Attached to Bastion.png`
- `Fig23 AppServerSG Bastion SSH Rule.png`
- `Fig24 SSH Connection to Bastion.png`
- `Fig25 SSH Connection to AppServer.png`
- `Fig26 AppServer Newfile Validation.png`

---

## Task 8: Connecting to AppServer with Session Manager

The lab procedure required:

- Opening a Session Manager connection to `AppServer`.
- Modifying `/var/www/html/index.html` with the supplied `sed` command.
- Reloading the application through `ProxyServer1`.
- Verifying the updated Session Manager message on the webpage.

### Task 8 Evidence Status

The current 19-image attachment package does not visibly contain the Session Manager terminal, the website-edit command, or the final updated webpage. These outcomes should not be claimed as visually verified using unrelated screenshots.

### Recommended Missing Evidence

- `Fig27 AppServer Session Manager Connection.png`
- `Fig28 Session Manager Website Update Command.png`
- `Fig29 Updated AppServer Website Validation.png`

---

## 11. Corrected Screenshot Traceability Matrix

| Figure | Correct filename | Task | Visible evidence | Strength |
|---:|---|---:|---|---|
| 01 | `Fig01 LabVPC Subnet Inventory.png` | 1 | Lab subnets and CIDR ranges | Strong |
| 02 | `Fig02 AppServerSG Initial HTTP Inbound Rule.png` | 1 | HTTP port 80 from `0.0.0.0/0` | Strong |
| 03 | `Fig03 NAT Gateway Summary.png` | 1 | Public NAT gateway in Available state | Strong |
| 04 | `Fig04 ProxyServer2 Instance Details.png` | 2 | ProxyServer2 public/private addressing and subnet | Strong |
| 05 | `Fig05 Private Route Table Name Tag.png` | 1 | `Name=Private` entered in tag editor | Partial |
| 06 | `Fig06 ProxyServer1 Security Group HTTP Rule.png` | 2 | ProxySG attachment and HTTP rule | Strong |
| 07 | `Fig07 ProxyServer1 Website Connectivity Test.png` | 3 | AppServer page reached through ProxyServer1 | Strong |
| 08 | `Fig08 AppServer Private Subnet Placement.png` | 1 | AppServer in PrivateSubnet without public IPv4 | Strong |
| 09 | `Fig09 ProxySG2 HTTP Rule Confirmation.png` | 2 | Saved HTTP rule with success banner | Strong |
| 10 | `Fig10 AppServerSG ProxyServer1 CIDR Rule.png` | 4 | HTTP source configured as `10.0.1.7/32` | Partial |
| 11 | `Fig11 Network ACL Rule 99 HTTP Deny.png` | 6 | Rule 99 denies HTTP from `0.0.0.0/0` | Partial |
| 12 | `Fig12 AppServerSG Outbound Rule.png` | 1 | All outbound IPv4 traffic permitted | Strong |
| 13 | `Fig13 ProxyServer2 ProxySG Reassignment.png` | 5 | ProxySG selected for ProxyServer2 | Partial |
| 14 | `Fig14 LabVPC Internet Gateway.png` | 2 | IGW attached to LabVPC | Strong |
| 15 | `Fig15 ProxyServer2 Website Connectivity Test.png` | 3 | AppServer page reached through ProxyServer2 | Strong |
| 16 | `Fig16 ProxyServer1 Instance Details.png` | 2 | ProxyServer1 public/private addressing and subnet | Strong |
| 17 | `Fig17 LabVPC CIDR Confirmation.png` | 1 | LabVPC uses `10.0.0.0/16` | Strong |
| 18 | `Fig18 NAT Gateway Detailed Configuration.png` | 1 | NAT gateway associated with LabVPC and PublicSubnetA | Strong |
| 19 | `Fig19 PublicSubnetA Internet Gateway Route.png` | 2 | `0.0.0.0/0` route targets the IGW | Strong |

---

## 12. Results and Validation

### 12.1 Visually Confirmed Results

- `LabVPC` uses CIDR `10.0.0.0/16`.
- The environment contains `PublicSubnetA`, `PublicSubnetB`, and `PrivateSubnet`.
- The NAT gateway is public, available, and associated with `PublicSubnetA`.
- `AppServer` operates in `PrivateSubnet` without a public IPv4 address shown.
- The initial `AppServerSG` rule allows HTTP from `0.0.0.0/0`.
- The visible `AppServerSG` outbound rule allows all IPv4 traffic.
- `ProxyServer1` operates in `PublicSubnetA` with public and private IPv4 addresses.
- `ProxyServer2` operates in `PublicSubnetB` with public and private IPv4 addresses.
- `ProxySG2` was successfully modified to allow HTTP.
- The internet gateway is attached to `LabVPC`.
- `PublicSubnetA` has a default route to the internet gateway.
- Initial browser access succeeded through both proxy public IP addresses.
- The `AppServerSG` editor contains a correctly targeted `10.0.1.7/32` HTTP rule.
- The network ACL editor contains rule `99` denying HTTP.
- The Change security groups screen contains ProxySG for `ProxyServer2`.

### 12.2 Partially Evidenced Results

- The private route table Name tag was entered as `Private`, but the saved state is not shown.
- The `/32` rule was configured, but its saved state is not shown.
- ProxySG was selected for `ProxyServer2`, but the final save confirmation is not shown.
- Network ACL rule `99` was configured, but its saved state is not shown.

### 12.3 Outcomes Not Visually Proven by the Current Package

- Private route table default routing to the NAT gateway.
- Successful ProxyServer1 access after the `/32` restriction.
- Failed ProxyServer2 access after the `/32` restriction.
- `AppServerSG` referencing ProxySG as the HTTP source.
- Successful access through both proxy servers after shared group authorization.
- Browser timeout after network ACL rule `99`.
- Network ACL rule `98` permitting HTTP.
- Restored access after rule `98`.
- Bastion host configuration and SSH workflow.
- Creation of `newfile.txt` on `AppServer`.
- Session Manager connection and website modification.

---

## 13. Security Analysis

### 13.1 Network Segmentation

The evidence shows a clear separation between public proxy resources and the private application server. `AppServer` has no public IPv4 address shown, which reduces direct internet exposure.

### 13.2 Least-Privilege Access

The initial application security group accepts HTTP from all IPv4 sources. The later `/32` configuration narrows the source to `10.0.1.7`, the visible private IPv4 address of `ProxyServer1`. This is a meaningful least-privilege improvement.

### 13.3 Scalable Authorization

Using ProxySG as the source for an application security group rule is more maintainable than adding an individual IP rule for every proxy instance. The current evidence shows ProxySG selected for `ProxyServer2`, but does not include the corresponding AppServerSG source-reference screen.

### 13.4 Layered Traffic Control

The security group evidence and network ACL editor demonstrate two distinct traffic-control layers. The security group applies to an EC2 network interface, while the network ACL provides subnet-level filtering. The functional ACL test is not included in the current screenshots.

### 13.5 Validation Discipline

The two initial browser tests provide stronger evidence than configuration views alone because the browser results verify an end-to-end application path. Similar before-and-after screenshots would strengthen Tasks 4, 5, and 6.

---

## 14. Evidence Gaps and Limitations

1. The private route table default route to the NAT gateway is not shown.
2. The route table tag image does not show the completed save state.
3. The `/32` security group rule image does not show the completed save state.
4. Post-restriction browser success and failure tests are absent.
5. The AppServerSG rule referencing ProxySG is absent.
6. The ProxyServer2 security group reassignment image does not show the completed save state.
7. Final browser tests after shared security group authorization are absent.
8. The network ACL deny-rule image does not show the completed save state.
9. Network ACL rule `98` and the restored website test are absent.
10. Task 7 bastion and SSH evidence is absent.
11. Task 8 Session Manager and final webpage evidence is absent from the current package.
12. Several screenshots expose temporary account IDs, resource IDs, and IP addresses that should be reviewed before public publication.

These gaps describe the limits of the screenshot package. They do not by themselves prove that the associated lab actions were not completed.

---

## 15. Recommended Next Steps

- Add a saved private route table view showing the NAT gateway target.
- Add the saved AppServerSG `/32` rule.
- Capture the successful ProxyServer1 and failed ProxyServer2 tests after the restriction.
- Capture AppServerSG with ProxySG as the HTTP source.
- Capture the saved ProxyServer2 security group association.
- Add both website tests after shared group authorization.
- Capture the saved network ACL rule `99`, browser timeout, rule `98`, and restored access.
- Add the complete bastion and SSH evidence sequence.
- Add the Session Manager terminal, edit command, and updated webpage.
- Redact identifiers that are not required for assessment before publishing the report publicly.

---

## 16. Security and Privacy Notes

- Do not publish private keys or the contents of `labsuser.pem`.
- Do not expose passwords, access keys, session tokens, or browser cookies.
- Review screenshots for account IDs, instance IDs, subnet IDs, security group IDs, network interface IDs, and temporary IP addresses before public publication.
- Keep unredacted evidence only in an approved private assessment location when required.
- Production SSH rules should use approved administrative sources rather than unrestricted IPv4 access.
- Apply least privilege to security group and IAM permissions.

---

## 17. Conclusion

The corrected evidence mapping provides strong visual support for Tasks 1, 2, and 3 of Lab 4.1. The screenshots confirm the VPC and subnet design, gateway placement, EC2 network placement, initial security group rules, public routing, and functional proxy access to the application server.

Tasks 4, 5, and 6 are partially supported because the relevant configurations are visible, but some images remain at the editor stage and the corresponding functional tests are not included. Tasks 7 and 8 require additional screenshots for full visual verification.

By separating confirmed outcomes from partial and missing evidence, this report provides an accurate, professional, and defensible record of the completed AWS network security assignment.

---

## 18. Disclaimer

This report documents an educational AWS lab performed in a restricted training environment. The configurations are presented for learning, assessment, and portfolio documentation. Production implementations should be reviewed against current organizational policies, threat models, compliance requirements, and official AWS guidance. Temporary identifiers and IP addresses visible in screenshots should be redacted when they are not required for the intended audience.

---

## 19. Author

<div align="center">

### Wadondera A. Collins

**ICDFA Trainee | Cohort 11 | Cloud Security Engineering**

Cloud Security | AWS Networking | Infrastructure Security | DevSecOps

</div>
