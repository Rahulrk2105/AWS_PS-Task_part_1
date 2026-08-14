# 🔐 Lab 07 – Secure Amazon EC2 Instances

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Security](https://img.shields.io/badge/Security-EC2%20Security-blue)
![IAM](https://img.shields.io/badge/AWS-IAM-purple)
![IMDSv2](https://img.shields.io/badge/EC2-IMDSv2-green)
![EBS](https://img.shields.io/badge/EBS-Encryption-blue)
![KMS](https://img.shields.io/badge/AWS-KMS-yellow)

---

# 📖 Project Overview

In this hands-on task, I used my **existing Bastion EC2 instance** and secured it using four AWS security controls:

1. Configure Security Groups
2. Configure IAM Roles
3. Enable / Verify IMDSv2
4. Configure EBS Encryption

The objective was to understand how an EC2 instance can be secured at different layers.

```text
                         EC2 Security
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
   Security Group          IAM Role            IMDSv2
          |                   |                   |
   Network Security      AWS Permissions    Metadata Security
                              |
                              v
                       EBS Encryption
                        Data at Rest
```

---

# 🎯 Objective

The objective was to secure my existing **Bastion EC2 instance** without creating a new EC2 instance.

```text
                         Internet
                            |
                         SSH :22
                      My IP only
                            |
                            v
                  +------------------+
                  |  Bastion Server  |
                  |                  |
                  |   Bastion-SG     |
                  |   Bastion-role   |
                  |   IMDSv2         |
                  |   Required       |
                  +--------+---------+
                           |
                       /dev/xvda
                           |
                           v
                  +------------------+
                  |   Encrypted EBS  |
                  |   gp3 / aws/ebs  |
                  +------------------+
```

---

# 🖥️ Existing Environment

| Resource | Configuration |
|---|---|
| EC2 Instance | `Bastion-Server` |
| Security Group | `Bastion-SG` |
| SSH Port | `22` |
| SSH Source | My Public IP `/32` |
| IAM Role | `Bastion-role` |
| IAM Policy | `AmazonSSMManagedInstanceCore` |
| IMDSv2 | `Required` |
| Original Root Device | `/dev/xvda` |
| Original EBS | Not Encrypted |
| Encrypted Snapshot | `snap-0f18778bb116c9a5e` |
| Final Encrypted EBS | `vol-0613d87f538d2163c` |
| Volume Type | `gp3` |
| Volume Size | `8 GiB` |
| Availability Zone | `us-east-1d` |
| KMS Key | `aws/ebs` |

---

# 🧰 Prerequisites

Before starting this task, I already had:

- AWS Account
- Existing EC2 Bastion Server
- Existing Bastion Security Group
- Existing IAM Role
- `AmazonSSMManagedInstanceCore`
- SSH key pair
- Amazon Linux 2023
- EBS volume
- AWS Console access

---

# 1️⃣ Configure Security Group

## 🎯 Problem We Are Solving

The Bastion EC2 should not allow SSH access from everyone on the internet.

We should avoid:

```text
0.0.0.0/0
     |
     | SSH :22
     v
Bastion
```

Instead, SSH should be restricted to my trusted public IP:

```text
My Public IP
     |
     | SSH :22
     v
Bastion-SG
     |
     v
Bastion-Server
```

## 💡 Practical Understanding

A **Security Group is a virtual firewall for an EC2 instance**.

It controls:

- Inbound traffic
- Outbound traffic
- Ports
- Protocols
- Sources
- Destinations

For administrative SSH access, the source should be restricted to a trusted IP.

## 🛠️ What I Did

I checked the Security Group attached to my existing Bastion.

The Security Group was:

```text
Bastion-SG
```

The inbound SSH rule was:

```text
Type: SSH
Protocol: TCP
Port: 22
Source: My Public IP /32
```

My public IP was configured as a `/32` source.

Therefore:

```text
SSH :22
   |
   +---- My Public IP /32
```

This means only my public IP was allowed to connect through SSH.

## 🔐 Outbound Rule

The Bastion Security Group already had:

```text
Type: All Traffic
Protocol: All
Port: All
Destination: 0.0.0.0/0
```

I left the outbound rule unchanged because the Bastion was already working and there was no requirement to modify its outbound connectivity for this task.

## 📸 Security Group Screenshot

![Bastion Security Group](images/security-group.png)

## 🧠 What I Learned

A Security Group controls **network-level access** to the EC2 instance.

For my Bastion:

```text
SSH :22
   |
   v
My Public IP Only
```

### ✅ Result

Security Group configuration was successfully verified.

---

# 2️⃣ Configure IAM Role

## 🎯 Problem We Are Solving

The EC2 instance may need permission to communicate with AWS services.

We should not store permanent AWS access keys and secret keys directly inside the EC2 instance.

### ❌ Bad Approach

```text
EC2
 |
 +---- Access Key
 |
 +---- Secret Key
```

### ✅ Better Approach

```text
EC2
 |
 v
IAM Role
 |
 v
AWS Permissions
 |
 v
AWS Services
```

## 💡 Practical Understanding

An **IAM Role provides AWS permissions to an EC2 instance**.

The EC2 can use the permissions provided by the role without storing long-term AWS credentials inside the server.

## 🛠️ What I Did

My Bastion already had an IAM Role attached:

```text
Bastion-role
```

The role already had:

```text
AmazonSSMManagedInstanceCore
```

attached.

Therefore, I did **not create a new IAM Role**, because the required IAM configuration was already present.

## 📸 IAM Role Screenshot

![Bastion IAM Role](images/iam-role.png)

## 🔄 IAM Permission Flow

```text
Bastion-Server
       |
       v
Bastion-role
       |
       v
AmazonSSMManagedInstanceCore
       |
       v
AWS Systems Manager
```

## 🧠 What I Learned

An IAM Role controls **what the EC2 instance is allowed to do in AWS**.

### ✅ Result

IAM Role configuration was successfully verified.

---

# 3️⃣ Enable / Verify IMDSv2

## 🎯 Problem We Are Solving

EC2 provides an **Instance Metadata Service (IMDS)**.

The metadata service is available through:

```text
169.254.169.254
```

For better security, we want:

```text
IMDSv2 = Required
```

## 💡 Practical Understanding

IMDSv2 uses a token-based mechanism for accessing EC2 instance metadata.

The important configuration is:

```text
IMDSv2
   ↓
Required
```

## 🛠️ What I Did

I checked the metadata configuration of my existing Bastion.

The Bastion already had:

```text
IMDSv2: Required
```

Therefore, I did not need to make any additional changes.

## 📸 IMDSv2 Screenshot

![IMDSv2 Required](images/imdsv2.png)

## 🧠 What I Learned

IMDSv2 provides a more secure method for accessing EC2 instance metadata.

Since my Bastion already had:

```text
IMDSv2: Required
```

this part of the task was already correctly configured.

### ✅ Result

IMDSv2 was already enabled and required.

---

# 4️⃣ Configure EBS Encryption

## 🎯 Problem We Are Solving

The existing root EBS volume attached to my Bastion was **not encrypted**.

The original configuration was:

```text
Bastion-Server
      |
      v
/dev/xvda
      |
      v
Original EBS Volume
      |
      v
❌ Not Encrypted
```

The goal was to replace the unencrypted root volume with an encrypted EBS volume.

---

# 4.1 Check Existing EBS Volume

I checked the EBS volume attached to my Bastion.

The original root volume was not encrypted and was attached as:

```text
Device:
/dev/xvda

Encryption:
Not Encrypted
```

---

# 4.2 Create Snapshot

Because the original EBS volume was not encrypted, I first created a snapshot from the existing volume.

```text
Existing EBS Volume
       |
       | Create Snapshot
       v
    Snapshot
```

A snapshot is a point-in-time copy of an EBS volume.

It can be used to create another EBS volume.

## 🧠 Practical Understanding

The snapshot is not another disk directly attached to the EC2.

It is a stored point-in-time copy that can be used to create a new EBS volume.

---

# 4.3 Create Encrypted Snapshot

I copied the snapshot and enabled encryption.

The encrypted snapshot was:

```text
snap-0f18778bb116c9a5e
```

The configuration was:

```text
Status:
Completed

Encryption:
Encrypted

KMS Key:
aws/ebs
```

## 📸 Encrypted Snapshot

![Encrypted Snapshot](images/encrypted-snapshot.png)

## 🔄 Snapshot Encryption Flow

```text
Original EBS
     |
     | Unencrypted
     v
Original Snapshot
     |
     | Copy + Enable Encryption
     v
Encrypted Snapshot
     |
     v
KMS: aws/ebs
```

---

# 4.4 Create Encrypted EBS Volume

I created a new EBS volume from the encrypted snapshot.

The first volume was created in:

```text
us-east-1a
```

However, my Bastion was in:

```text
us-east-1d
```

An EBS volume must be in the same Availability Zone as the EC2 instance in order to attach it.

Therefore, I created another encrypted volume in:

```text
us-east-1d
```

The correct encrypted volume was:

```text
Volume ID:
vol-0613d87f538d2163c

Size:
8 GiB

Volume Type:
gp3

Availability Zone:
us-east-1d

Encryption:
Encrypted

KMS Key:
aws/ebs
```

## 📸 Encrypted EBS Volume

![Encrypted EBS Volume](images/encrypted-volume-created.png)

---

# 4.5 Stop Bastion

Before replacing the root volume, I stopped the Bastion instance.

The instance state became:

```text
Stopped
```

This was necessary because I was going to replace the root EBS volume.

---

# 4.6 Detach Original Unencrypted Volume

The original root volume was attached as:

```text
/dev/xvda
```

I detached the old unencrypted root volume.

I did not immediately delete the old volume because it could be kept as a rollback option.

---

# 4.7 Attach Encrypted Volume

I selected the new encrypted EBS volume:

```text
vol-0613d87f538d2163c
```

I attached it to:

```text
Bastion-Server
```

using:

```text
/dev/xvda
```

The final configuration became:

```text
Bastion-Server
      |
      v
/dev/xvda
      |
      v
vol-0613d87f538d2163c
      |
      +---- 8 GiB
      +---- gp3
      +---- us-east-1d
      +---- Encrypted
      +---- KMS: aws/ebs
```

## 📸 Encrypted Root Volume Attached

![Encrypted Root Volume Attached](images/encrypted-root-volume-attached.png)

---

# 4.8 Start Bastion

After attaching the encrypted root volume, I started the Bastion.

I waited for:

```text
Instance State:
Running
```

and:

```text
Status Checks:
3/3 Passed
```

The Bastion started successfully.

---

# 4.9 Verify Root EBS

I verified that the encrypted EBS volume was being used as the root device.

The final configuration was:

```text
Bastion-Server
       |
       v
/dev/xvda
       |
       v
vol-0613d87f538d2163c
       |
       +---- Encryption: Encrypted
       +---- KMS: aws/ebs
       +---- Type: gp3
       +---- Size: 8 GiB
       +---- AZ: us-east-1d
```

---

# 🔄 Complete EBS Encryption Process

```text
Original EBS Volume
        |
        | ❌ Unencrypted
        v
     Snapshot
        |
        v
Encrypted Snapshot
        |
        | KMS: aws/ebs
        v
Encrypted EBS Volume
        |
        | Correct Availability Zone
        v
Attach to Bastion
        |
        | /dev/xvda
        v
Encrypted Root Volume
        |
        v
Start Bastion
        |
        v
Verify EC2
```

---

# 🧠 Why Did I Use a Snapshot?

The original EBS volume was already unencrypted.

I used the snapshot method to create an encrypted replacement:

```text
Unencrypted EBS
      |
      v
Snapshot
      |
      v
Encrypted Snapshot
      |
      v
Encrypted EBS Volume
      |
      v
Replace Root Volume
```

This allowed me to create an encrypted version of the existing volume while keeping the original volume available during the migration.

---

# ⚠️ Important Lesson – Availability Zone

During the hands-on process, I initially created an encrypted volume in:

```text
us-east-1a
```

However, my Bastion was in:

```text
us-east-1d
```

The first volume could not be attached to the Bastion because it was in a different Availability Zone.

So I created another encrypted volume in:

```text
us-east-1d
```

This was an important hands-on troubleshooting point.

---

# 🔍 Troubleshooting

## Issue 1 – Original EBS Volume Was Not Encrypted

### Problem

The original root EBS volume showed:

```text
Encryption:
Not Encrypted
```

### Solution

I created:

```text
Snapshot
    |
    v
Encrypted Snapshot
    |
    v
Encrypted EBS Volume
```

Then I replaced the root volume.

---

## Issue 2 – Encrypted Volume Was Created in Wrong Availability Zone

### Problem

The first encrypted volume was created in:

```text
us-east-1a
```

The Bastion was in:

```text
us-east-1d
```

### Cause

EBS volumes are Availability Zone specific.

### Solution

I created another encrypted volume in:

```text
us-east-1d
```

Then attached it to the Bastion.

---

# 🧪 Important AWS Concepts Used

## Security Group

A Security Group controls network traffic to and from EC2.

```text
Security Group
      |
      +---- Inbound
      |
      +---- Outbound
```

## IAM Role

An IAM Role provides AWS permissions to an EC2 instance.

```text
EC2
 |
 v
IAM Role
 |
 v
AWS Permissions
```

## IMDSv2

IMDSv2 provides a more secure method for accessing EC2 instance metadata.

```text
EC2
 |
 v
IMDSv2
 |
 v
Instance Metadata
```

## EBS

EBS provides block storage for EC2.

```text
EC2
 |
 v
EBS Volume
```

## Snapshot

A snapshot is a point-in-time copy of an EBS volume.

```text
EBS Volume
    |
    v
Snapshot
```

## KMS

AWS Key Management Service is used to manage encryption keys.

In this task, EBS encryption used:

```text
aws/ebs
```

---

# 📊 Final Verification

| Component | Configuration | Status |
|---|---|---|
| Existing Bastion EC2 | `Bastion-Server` | ✅ |
| Security Group | `Bastion-SG` | ✅ |
| SSH Port | `22` | ✅ |
| SSH Source | My IP `/32` | ✅ |
| IAM Role | `Bastion-role` | ✅ |
| IAM Policy | `AmazonSSMManagedInstanceCore` | ✅ |
| IMDSv2 | Required | ✅ |
| Original EBS | Not Encrypted | ❌ Before |
| Snapshot | Created | ✅ |
| Encrypted Snapshot | `snap-0f18778bb116c9a5e` | ✅ |
| Encrypted EBS | `vol-0613d87f538d2163c` | ✅ |
| EBS Type | `gp3` | ✅ |
| EBS Size | `8 GiB` | ✅ |
| Availability Zone | `us-east-1d` | ✅ |
| KMS Key | `aws/ebs` | ✅ |
| Root Device | `/dev/xvda` | ✅ |
| Bastion Startup | Successful | ✅ |

---

# 🧠 What I Learned

## 1. Security Groups

A Security Group acts as a virtual firewall for an EC2 instance.

In my case:

```text
SSH :22
   |
   v
My Public IP Only
```

I learned that administrative ports should not unnecessarily be exposed to:

```text
0.0.0.0/0
```

## 2. IAM Roles

An IAM Role provides AWS permissions to an EC2 instance.

My Bastion already had:

```text
Bastion-role
```

with:

```text
AmazonSSMManagedInstanceCore
```

I learned that IAM Roles allow EC2 instances to use AWS permissions without storing long-term AWS credentials directly on the server.

## 3. IMDSv2

IMDSv2 provides a more secure method for accessing EC2 instance metadata.

My Bastion already had:

```text
IMDSv2: Required
```

So no additional configuration was required.

## 4. EBS Encryption

The original root EBS volume was not encrypted.

I learned how to create an encrypted replacement:

```text
Unencrypted Volume
       |
       v
Snapshot
       |
       v
Encrypted Snapshot
       |
       v
Encrypted EBS Volume
       |
       v
Root Volume
```

The final encrypted volume was:

```text
vol-0613d87f538d2163c
```

with:

```text
Encryption:
Encrypted

KMS:
aws/ebs
```

---

# 🔐 Complete Security Model

```text
                 EC2 Security
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
   Security        IAM Role      IMDSv2
     Group            |             |
        |              |             |
   Network         AWS Access    Metadata
   Security        Permissions   Security
                      |
                      |
                      v
                EBS Encryption
                      |
                      v
                 Data at Rest
```

---

# 🎯 What I Actually Did

I used my **existing Bastion EC2** and completed the four security requirements.

## 1. Security Group

I checked my existing:

```text
Bastion-SG
```

It had:

```text
SSH
TCP
22
My Public IP /32
```

So SSH access was restricted to my IP.

## 2. IAM Role

The Bastion already had:

```text
Bastion-role
```

with:

```text
AmazonSSMManagedInstanceCore
```

So I did not create another IAM Role.

## 3. IMDSv2

The Bastion already had:

```text
IMDSv2:
Required
```

So no additional change was required.

## 4. EBS Encryption

The existing root EBS volume was not encrypted.

I performed:

```text
Existing Unencrypted Volume
          |
          v
Created Snapshot
          |
          v
Created Encrypted Snapshot
          |
          v
Created Encrypted Volume
          |
          v
Selected Correct Availability Zone
          |
          v
Detached Old Root Volume
          |
          v
Attached Encrypted Volume
          |
          v
Used /dev/xvda
          |
          v
Started Bastion
          |
          v
Verified Successfully
```

---

# ⭐ Interview Explanation

> I secured my existing Bastion EC2 by restricting SSH access through its Security Group, verifying the IAM Role with `AmazonSSMManagedInstanceCore`, confirming that IMDSv2 was required, and replacing the unencrypted root EBS volume with an encrypted EBS volume created from an encrypted snapshot using the `aws/ebs` KMS key.

---

# 📝 Quick Revision

```text
1. Security Group
   |
   +---- Controls Network Access
   |
   +---- SSH :22
   |
   +---- My IP Only

2. IAM Role
   |
   +---- Controls AWS Permissions
   |
   +---- Bastion-role
   |
   +---- AmazonSSMManagedInstanceCore

3. IMDSv2
   |
   +---- Secures Metadata Access
   |
   +---- Required

4. EBS Encryption
   |
   +---- Protects Data at Rest
   |
   +---- Old Volume: Unencrypted
   |
   +---- Snapshot Created
   |
   +---- Encrypted Snapshot
   |
   +---- Encrypted EBS Volume
   |
   +---- Attached as /dev/xvda
   |
   +---- KMS: aws/ebs
```

---

# 📌 Final Security Checklist

```text
Security Group
     |
     v
SSH restricted to trusted IP
     |
     v
✅

IAM Role
     |
     v
Bastion-role
     |
     v
AmazonSSMManagedInstanceCore
     |
     v
✅

IMDSv2
     |
     v
Required
     |
     v
✅

EBS Encryption
     |
     v
Encrypted Snapshot
     |
     v
Encrypted EBS
     |
     v
Encrypted Root Volume
     |
     v
✅
```

---

# 🏆 Final Result

## 🔐 Secure Amazon EC2 Instances — COMPLETED 100% 🎉

The Bastion EC2 is now secured using:

```text
Security Group
      +
IAM Role
      +
IMDSv2
      +
EBS Encryption
```

The four security layers provide:

```text
Security Group
      |
      v
Network Protection

IAM Role
      |
      v
AWS Permission Protection

IMDSv2
      |
      v
Metadata Protection

EBS Encryption
      |
      v
Data-at-Rest Protection
```

---

# 🎯 Conclusion

Successfully completed the **Secure Amazon EC2 Instances** hands-on task.

Through this task, I gained practical experience with:

- Amazon EC2
- Security Groups
- IAM Roles
- IAM Policies
- AmazonSSMManagedInstanceCore
- IMDSv2
- EBS Volumes
- EBS Snapshots
- EBS Encryption
- AWS KMS
- Availability Zones
- Root EBS Volume Replacement
- EC2 Security Hardening
- AWS Console
- AWS Troubleshooting

The final result was a Bastion EC2 with:

```text
✅ Restricted SSH Access
✅ IAM Role
✅ AmazonSSMManagedInstanceCore
✅ IMDSv2 Required
✅ Encrypted Root EBS
✅ KMS Encryption
✅ Correct Availability Zone
```

---

# 🚀 TASK COMPLETED

## 🔐 Secure Amazon EC2 Instances — 100% COMPLETE
