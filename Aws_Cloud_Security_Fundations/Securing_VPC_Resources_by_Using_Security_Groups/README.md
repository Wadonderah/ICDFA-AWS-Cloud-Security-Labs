<div align="center">

# Securing AWS VPC Resources with Security Groups and Network ACLs

### Lab 4.1 | Amazon VPC Network Security and Private Instance Access

![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?logo=amazonwebservices&logoColor=white)
![Amazon VPC](https://img.shields.io/badge/Amazon_VPC-Network_Security-8C4FFF?logo=amazonwebservices&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon_EC2-Compute-FF9900?logo=amazonec2&logoColor=white)
![Systems Manager](https://img.shields.io/badge/AWS_Systems_Manager-Session_Manager-232F3E?logo=amazonwebservices&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2EA44F)

**Author:** Wadondera A. Collins  
**Program:** ICDFA Trainee | Cohort 11 | Cloud Security Engineering  
**Assignment:** Lab 4.1: Securing VPC Resources by Using Security Groups  
**Estimated Lab Duration:** 90 minutes

</div>

---

## Table of Contents

- [Project Overview](#project-overview)
- [Scenario](#scenario)
- [Objectives](#objectives)
- [Professional Value](#professional-value)
- [Skills Demonstrated](#skills-demonstrated)
- [AWS Services and Technologies](#aws-services-and-technologies)
- [Lab Environment](#lab-environment)
- [Architecture Overview](#architecture-overview)
- [Implementation Summary](#implementation-summary)
- [Security Controls Applied](#security-controls-applied)
- [Connectivity Validation](#connectivity-validation)
- [Screenshot Evidence](#screenshot-evidence)
- [Key Security Findings](#key-security-findings)
- [Validation Checklist](#validation-checklist)
- [Limitations and Next Steps](#limitations-and-next-steps)
- [Security Notes](#security-notes)
- [Conclusion](#conclusion)
- [Disclaimer](#disclaimer)
- [Author](#author)

---

## Project Overview

This completed lab demonstrates how layered network security controls can protect workloads deployed across public and private subnets in an Amazon Virtual Private Cloud (Amazon VPC). The implementation used Amazon EC2 security groups, a network access control list, subnet routing, a NAT gateway, an internet gateway, a bastion host, SSH agent forwarding, and AWS Systems Manager Session Manager.

The lab began with two public proxy instances and one private application server. Access was progressively tested and restricted, first by source IP address and then by security group reference. Network ACL rules were introduced to show subnet-level filtering and rule-order behavior. Administrative access to the private application server was validated through both a bastion host and Session Manager.

---

## Scenario

The environment contained a functional VPC named `LabVPC` with the CIDR range `10.0.0.0/16`. It included three subnets:

- `PublicSubnetA`, hosting `ProxyServer1`
- `PublicSubnetB`, initially hosting `ProxyServer2`, later renamed `Bastion`
- `PrivateSubnet`, hosting `AppServer`

The public instances forwarded HTTP traffic to the application server in the private subnet. The private application server had no public IPv4 address and relied on controlled internal access paths.

---

## Objectives

The completed assignment achieved the following objectives:

- Examined security groups to identify permitted inbound and outbound traffic.
- Changed the security groups attached to EC2 instances.
- Created a dedicated bastion-host security group.
- Applied least-privilege inbound rules.
- Used a security group as the source of another security group's rule.
- Configured network ACL rules to deny and allow HTTP traffic.
- Validated rule-order behavior in a network ACL.
- Connected to a private EC2 instance through a bastion host using SSH agent forwarding.
- Connected directly to a private EC2 instance using AWS Systems Manager Session Manager.
- Modified the application website through a Session Manager terminal session.

---

## Professional Value

This project reflects practical cloud security engineering responsibilities, including network segmentation, layered traffic filtering, least-privilege access, secure administrative connectivity, and validation of control effectiveness. The completed work demonstrates the ability to distinguish between instance-level and subnet-level controls and to select more scalable access patterns for cloud workloads.

---

## Skills Demonstrated

- Amazon VPC resource inspection
- Public and private subnet analysis
- Route table and gateway identification
- EC2 security group administration
- Least-privilege rule design
- Security group referencing
- Network ACL configuration and rule-order analysis
- HTTP connectivity testing
- Bastion-host configuration
- SSH agent forwarding
- Private instance administration
- AWS Systems Manager Session Manager
- Secure Linux file modification
- Layered cloud network security validation

---

## AWS Services and Technologies

| Service or Technology | Purpose in the Lab |
|---|---|
| Amazon VPC | Provided the isolated virtual network and subnet structure. |
| Amazon EC2 | Hosted the proxy, bastion, and application-server workloads. |
| Security Groups | Enforced stateful instance-level traffic controls. |
| Network ACL | Enforced stateless subnet-level traffic controls. |
| Internet Gateway | Enabled internet connectivity for public subnet resources. |
| NAT Gateway | Enabled outbound internet access from the private subnet. |
| Route Tables | Directed local, internet-bound, and NAT-bound traffic. |
| AWS Systems Manager Session Manager | Provided browser-based administrative access to the private instance without direct inbound SSH access. |
| SSH | Supported controlled terminal access through the bastion host. |
| Linux | Hosted the web server and provided the command-line environment. |

---

## Lab Environment

| Resource | Placement or Configuration | Security Role |
|---|---|---|
| `LabVPC` | `10.0.0.0/16` | Isolated network boundary for the lab. |
| `PublicSubnetA` | Public subnet | Hosted `ProxyServer1`. |
| `PublicSubnetB` | Public subnet | Hosted `ProxyServer2`, later repurposed as `Bastion`. |
| `PrivateSubnet` | Private subnet | Hosted `AppServer` without a public IPv4 address. |
| `ProxyServer1` | `PublicSubnetA` | Forwarded HTTP traffic to `AppServer`. |
| `ProxyServer2` / `Bastion` | `PublicSubnetB` | Initially acted as a second proxy, then provided controlled SSH access. |
| `AppServer` | `PrivateSubnet` | Hosted the application website. |
| `ProxySG` | Proxy instances | Allowed required HTTP access. |
| `ProxySG2` | Initial second-proxy group | Was replaced by `ProxySG` during access scaling. |
| `AppServerSG` | Application server | Restricted HTTP and SSH sources. |
| `BastionSG` | Bastion host | Allowed SSH connectivity for the lab exercise. |

> Exact resource identifiers and IP addresses are intentionally omitted because they were temporary values assigned to the lab session.

---

## Architecture Overview

```text
Internet
   |
Internet Gateway
   |
+--------------------------- LabVPC: 10.0.0.0/16 ---------------------------+
|                                                                            |
|  PublicSubnetA                         PublicSubnetB                         |
|  +----------------+                    +----------------+                    |
|  | ProxyServer1   |                    | ProxyServer2   |                    |
|  | ProxySG        |                    | later Bastion  |                    |
|  +-------+--------+                    +-------+--------+                    |
|          | HTTP                                   | SSH                      |
|          +------------------+---------------------+                          |
|                             |                                                |
|                     PrivateSubnet                                            |
|                     +----------------+                                       |
|                     | AppServer      |                                       |
|                     | AppServerSG    |                                       |
|                     | No public IPv4 |                                       |
|                     +----------------+                                       |
|                             |                                                |
|                         NAT Gateway                                          |
|                     for outbound access                                      |
+----------------------------------------------------------------------------+

Administrative alternative: AWS Systems Manager Session Manager -> AppServer
```

---

## Implementation Summary

### 1. Analyzed the VPC and Private Subnet

- Identified `LabVPC` and confirmed the `10.0.0.0/16` CIDR range.
- Reviewed the three configured subnets.
- Confirmed that the private subnet routed non-local traffic to a NAT gateway.
- Renamed the private route table from `changeme` to `Private`.
- Verified that the NAT gateway operated from `PublicSubnetA`.
- Confirmed that `AppServer` had a private IPv4 address but no public IPv4 address.
- Reviewed `AppServerSG`, including its initial HTTP rule and default outbound behavior.

### 2. Analyzed Public Subnet Resources

- Confirmed that the public subnet route table sent `0.0.0.0/0` traffic to an internet gateway.
- Verified that `ProxyServer1` and `ProxyServer2` had public IPv4 addresses.
- Reviewed the proxy security groups.
- Added an HTTP inbound rule to `ProxySG2` to permit IPv4 traffic for the initial connectivity test.

### 3. Tested Initial HTTP Connectivity

- Accessed the application website through the public IP address of `ProxyServer1`.
- Accessed the same website through the public IP address of `ProxyServer2`.
- Confirmed that both proxy servers could forward HTTP traffic to `AppServer`.

### 4. Restricted HTTP Access by Source IP

- Replaced the broad HTTP source in `AppServerSG` with the private IPv4 address of `ProxyServer1` using a `/32` prefix.
- Confirmed that access through `ProxyServer1` continued to work.
- Confirmed that access through `ProxyServer2` failed as expected.
- Demonstrated a precise but less scalable IP-based access-control approach.

### 5. Scaled Access with Security Group Referencing

- Replaced the source IP rule in `AppServerSG` with a reference to `ProxySG`.
- Removed `ProxySG2` from `ProxyServer2` and attached `ProxySG`.
- Confirmed that both proxy servers could reach `AppServer` after sharing the approved source security group.
- Eliminated the need to maintain separate source IP addresses for each approved proxy instance.

### 6. Applied Network ACL Filtering

- Added inbound network ACL rule `99` to deny HTTP traffic.
- Confirmed that website access timed out even though the relevant security group allowed HTTP.
- Added inbound network ACL rule `98` to allow HTTP traffic.
- Confirmed that website access was restored because the lower-numbered matching rule was evaluated first.

### 7. Connected Through a Bastion Host

- Renamed `ProxyServer2` to `Bastion`.
- Created `BastionSG` with an SSH inbound rule.
- Replaced `ProxySG` on the bastion instance with `BastionSG`.
- Added an SSH rule to `AppServerSG` that allowed the bastion's private IPv4 address as a `/32` source.
- Loaded the lab key into `ssh-agent` and connected to the bastion with agent forwarding.
- Connected from the bastion to `AppServer` without copying the private key to the bastion.
- Created `newfile.txt` on `AppServer` to validate successful access.

### 8. Connected with Session Manager

- Opened a Session Manager connection directly to `AppServer`.
- Accessed the private instance without first connecting through the bastion host.
- Updated `/var/www/html/index.html` from the Session Manager terminal.
- Reloaded the website through `ProxyServer1` and verified that the new Session Manager message appeared.

---

## Security Controls Applied

### Stateful Instance-Level Filtering

Security groups were used to control traffic at the EC2 network-interface level. The lab demonstrated broad source access, single-IP restriction, and security-group-based source authorization.

### Stateless Subnet-Level Filtering

The network ACL was used as a second traffic-control layer. Conflicting HTTP rules demonstrated that ACL entries are processed in ascending numerical order and that the first matching rule determines the result.

### Network Segmentation

The application server remained in a private subnet with no public IPv4 address. Public-facing proxy resources handled inbound web requests, while the NAT gateway supported private-subnet outbound connectivity.

### Controlled Administrative Access

Two private-instance administration methods were validated:

1. SSH through a bastion host with agent forwarding.
2. Direct browser-based access using Systems Manager Session Manager.

---

## Connectivity Validation

| Test | Expected Result | Observed Outcome |
|---|---|---|
| HTTP through `ProxyServer1` before restrictions | Website loads | Passed |
| HTTP through `ProxyServer2` before restrictions | Website loads | Passed |
| HTTP through `ProxyServer1` after `/32` restriction | Website loads | Passed |
| HTTP through `ProxyServer2` after `/32` restriction | Connection fails | Passed |
| HTTP through both proxies after `ProxySG` reference | Website loads | Passed |
| HTTP after network ACL rule `99` deny | Connection times out | Passed |
| HTTP after network ACL rule `98` allow | Website loads | Passed |
| SSH from bastion to `AppServer` | Private shell opens | Passed |
| Creation of `newfile.txt` | File created | Passed |
| Session Manager connection to `AppServer` | Terminal opens | Passed |
| Website after Session Manager edit | Updated message is displayed | Passed |

---

## Screenshot Evidence

### Task 8: Session Manager Website Modification

![Session Manager command used to update the application website](evidence/Fig01%20Session%20Manager%20Website%20Update.png)

**Figure 1: Session Manager command used to update the application website on AppServer.**

The evidence shows an active AWS Systems Manager Session Manager terminal connected to the `AppServer` instance. The terminal displays the completed `sudo sed -i` command used to replace text in `/var/www/html/index.html` with the message `Session manager was used to edit this file.` The returned `sh-4.2$` prompt indicates that the command completed without a visible shell error.

> Recommended evidence filename: `Fig01 Session Manager Website Update.png`

---

## Key Security Findings

1. **Private subnets reduce direct exposure.** `AppServer` did not require a public IPv4 address to provide its service through controlled proxy paths.
2. **Least privilege improves traffic control.** Replacing `0.0.0.0/0` with a specific source reduced the approved HTTP access scope.
3. **Security group references scale better than fixed IP rules.** Membership in `ProxySG` became the authorization condition for HTTP access to `AppServer`.
4. **Security groups and network ACLs must both permit traffic.** An allow rule in a security group did not override a matching network ACL deny rule.
5. **Network ACL ordering is security-critical.** Rule `98` was evaluated before rule `99`, changing the effective HTTP decision.
6. **Agent forwarding avoids placing the private key on the bastion host.** The SSH credential remained on the originating environment during the multi-hop connection.
7. **Session Manager reduces dependence on inbound SSH access.** The private application server was administered through a managed AWS connection path.

---

## Validation Checklist

- [x] `LabVPC` and subnet configuration reviewed.
- [x] Private route table renamed to `Private`.
- [x] NAT gateway path for the private subnet verified.
- [x] Public subnet internet-gateway routing verified.
- [x] Proxy security group HTTP access configured.
- [x] Initial HTTP connectivity through both proxy servers validated.
- [x] `AppServerSG` restricted to a single `/32` source.
- [x] Failed unauthorized proxy connection validated.
- [x] `AppServerSG` updated to reference `ProxySG`.
- [x] Both proxy instances validated after group-based authorization.
- [x] Network ACL deny and allow rules tested.
- [x] Bastion host and `BastionSG` configured.
- [x] SSH agent forwarding used to reach `AppServer`.
- [x] `newfile.txt` created on the private server.
- [x] Session Manager used to connect to `AppServer`.
- [x] Website content modified and validated.
- [x] Lab submitted and completed.

---

## Limitations and Next Steps

This work was completed in a restricted training environment with temporary resources and lab-provided credentials. The following improvements would be appropriate for a production-oriented extension:

- Restrict bastion SSH access to a trusted administrative CIDR rather than `Anywhere-IPv4`.
- Prefer Session Manager when inbound SSH is not operationally required.
- Remove or close unused security group rules after testing.
- Use dedicated network ACLs where subnet-specific policies are required.
- Enable centralized logging and monitoring for administrative sessions and network changes.
- Use infrastructure as code to make security controls repeatable and reviewable.
- Add high availability across multiple Availability Zones where required.
- Document cleanup procedures for temporary access paths and test resources.

---

## Security Notes

- No private keys, passwords, public IP addresses, private IP addresses, account identifiers, or temporary credentials are stored in this repository.
- Environment-specific values should be represented with placeholders in public documentation.
- A bastion host should be hardened, monitored, patched, and limited to approved administrative sources.
- Security group rules should use the narrowest source and port range that meets the workload requirement.
- Session Manager access should be governed through least-privilege IAM permissions and appropriate audit controls.

---

## Conclusion

Lab 4.1 was completed successfully by securing communication among public proxy instances, a bastion host, and a private application server. The implementation progressed from broad HTTP access to least-privilege IP-based control and then to scalable security-group referencing. Network ACL testing demonstrated layered enforcement and first-match rule processing. Finally, private administrative access was validated through both SSH agent forwarding and AWS Systems Manager Session Manager.

The completed assignment demonstrates practical understanding of AWS network segmentation, defense in depth, secure access patterns, and validation-driven cloud security engineering.

---

## Disclaimer

This repository documents a completed educational lab performed in a restricted AWS training environment. Resource names, configurations, and commands are presented for portfolio and learning purposes. Temporary identifiers, IP addresses, account details, credentials, and private keys are intentionally excluded. Follow organizational security policies and current AWS documentation before adapting any configuration for production use.

---

## Author

<div align="center">

### Wadondera A. Collins

**ICDFA Trainee | Cohort 11 | Cloud Security Engineering**

*Building practical expertise in cloud security, secure infrastructure, and DevSecOps.*

</div>
