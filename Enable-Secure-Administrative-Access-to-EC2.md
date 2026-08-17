# 🔐 Enable Secure Administrative Access to Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Systems%20Manager](https://img.shields.io/badge/AWS-Systems%20Manager-blue)
![Session%20Manager](https://img.shields.io/badge/AWS-Session%20Manager-green)
![IAM](https://img.shields.io/badge/AWS-IAM-purple)
![MFA](https://img.shields.io/badge/Security-MFA-red)

---

# 📖 Project Overview

In this hands-on task, I enabled and verified secure administrative access to my existing **Bastion EC2 instance**.

The task had three main areas:

1. Configure AWS Systems Manager Session Manager
2. Configure IAM Roles / Policies required for Session Manager
3. Configure and verify MFA controls

The main goal was to understand how an EC2 instance can be administered without depending only on SSH.

---

# 🎯 Objective

The traditional administrative access method was:

```text
My Laptop
    |
    | SSH :22
    v
Bastion EC2
```

The secure administrative access method we tested was:

```text
My Laptop
    |
    | AWS Console
    v
AWS Systems Manager
    |
    v
Session Manager
    |
    v
Bastion EC2
```

This allows me to open a shell session on the Bastion through AWS Systems Manager.

---

# 🖥️ Existing Environment

| Resource | Configuration |
|---|---|
| EC2 Instance | `Bastion-Server` |
| Instance ID | `i-0566105c1b3cd0730` |
| IAM Role | `Bastion-role` |
| SSM Policy | `AmazonSSMManagedInstanceCore` |
| SSM Agent | Active / Running |
| Session User | `ssm-user` |
| IAM Users | `0` |
| Root MFA | Virtual MFA configured |
| MFA Device | `Rahul-device` |

---

# 1️⃣ Configure AWS Systems Manager Session Manager

## 🎯 Problem We Are Solving

Normally, administrative access to an EC2 instance can be performed using SSH:

```text
Laptop
   |
   | SSH :22
   v
EC2
```

With Session Manager, we can access the EC2 through AWS Systems Manager:

```text
Laptop
   |
   v
AWS Systems Manager
   |
   v
Session Manager
   |
   v
SSM Agent
   |
   v
EC2
```

The purpose is to provide administrative shell access without using SSH for the session.

---

# 💡 Practical Understanding

Session Manager works through the **SSM Agent** running on the EC2 instance.

The basic flow is:

```text
Bastion EC2
      |
      v
SSM Agent
      |
      v
IAM Role
      |
      v
AmazonSSMManagedInstanceCore
      |
      v
AWS Systems Manager
      |
      v
Session Manager
```

---

# 🛠️ Step 1 – Open Session Manager

I opened:

```text
AWS Console
    ↓
Systems Manager
    ↓
Session Manager
    ↓
Start session
```

The existing Bastion appeared as a target.

The instance was:

```text
Bastion-Server
```

Instance ID:

```text
i-0566105c1b3cd0730
```

---

# 🛠️ Step 2 – Start a Session

I selected:

```text
Bastion-Server
```

and started a Session Manager session.

The session successfully opened a shell:

```text
sh-5.2$
```

This confirmed that Session Manager was successfully connected to the EC2.

---

# 📸 Session Manager Screenshot

![Session Manager Connected](images/session-manager.png)

---

# 🧪 Step 3 – Verify the Session User

Inside the Session Manager terminal, I ran:

```bash
whoami
```

The output was:

```text
ssm-user
```

This confirmed that the Session Manager session was running under the `ssm-user` account.

The flow was:

```text
Session Manager
      |
      v
SSM Agent
      |
      v
ssm-user
      |
      v
Bastion EC2
```

---

# 🧪 Step 4 – Verify the EC2 Host

I ran:

```bash
hostname
```

The output was:

```text
ip-10-0-0-254.ec2.internal
```

This confirmed that I was actually connected to my Bastion EC2 instance.

---

# 🧪 Step 5 – Verify SSM Agent

I checked the SSM Agent status using:

```bash
sudo systemctl status amazon-ssm-agent
```

The important result was:

```text
Active: active (running)
```

The output also showed that the SSM Agent successfully connected using the EC2 instance profile role credentials.

---

# 📸 SSM Agent Screenshot

![SSM Agent Running](images/ssm-agent.png)

---

# 🧠 What I Learned

The SSM Agent is the component running on the EC2 that communicates with AWS Systems Manager.

The architecture is:

```text
AWS Systems Manager
        |
        v
    SSM Agent
        |
        v
   Bastion EC2
```

If the SSM Agent is not running or cannot communicate with Systems Manager, Session Manager will not work.

---

# ✅ Session Manager Result

Session Manager was successfully configured and tested.

I successfully:

- Connected to `Bastion-Server`
- Received a `sh-5.2$` shell
- Verified `ssm-user`
- Verified the EC2 hostname
- Verified that the SSM Agent was running

---

# 2️⃣ Configure IAM Role / Policies

## 🎯 Problem We Are Solving

The SSM Agent running on the EC2 needs permissions to communicate with AWS Systems Manager.

The EC2 should not store long-term AWS access keys.

Instead, the EC2 uses an IAM Role.

---

# 💡 Practical Understanding

The EC2 uses:

```text
Bastion-Server
      |
      v
Bastion-role
      |
      v
AmazonSSMManagedInstanceCore
```

The IAM Role provides the AWS permissions required by the EC2.

---

# 🛠️ Step 1 – Check the Existing IAM Role

I checked the IAM Role attached to the Bastion.

The role was:

```text
Bastion-role
```

The role already had:

```text
AmazonSSMManagedInstanceCore
```

attached.

---

# 📸 IAM Role Screenshot

![Bastion IAM Role](images/iam-role.png)

---

# 🧠 What Does AmazonSSMManagedInstanceCore Do?

The `AmazonSSMManagedInstanceCore` policy provides the EC2/SSM Agent with the permissions required to communicate with Systems Manager.

The basic flow is:

```text
EC2
 |
 v
SSM Agent
 |
 v
Bastion-role
 |
 v
AmazonSSMManagedInstanceCore
 |
 v
Systems Manager
```

---

# 🧪 Hands-On Verification

The IAM configuration was verified through the successful Session Manager connection.

Inside the SSM Agent status output, the following was observed:

```text
EC2RoleProvider Successfully connected with instance profile role credentials
```

This confirmed that:

- The EC2 had an instance profile role
- The SSM Agent could obtain role credentials
- The role was working
- Systems Manager communication was successful

---

# 🧠 Important IAM Concept

There are two different sides to understand.

## EC2 Side

The EC2 needs an IAM Role:

```text
Bastion-Server
      |
      v
Bastion-role
      |
      v
AmazonSSMManagedInstanceCore
```

## Administrator Side

The person starting a Session Manager session also needs appropriate AWS permissions.

Conceptually:

```text
Administrator
      |
      | IAM permissions
      v
AWS Systems Manager
      |
      v
Session Manager
      |
      v
EC2
```

In this lab, the EC2-side configuration was already present and successfully verified.

I did not create another IAM role because the existing configuration was already working.

---

# ✅ IAM Result

The existing IAM configuration required for the EC2/SSM side was successfully verified.

---

# 3️⃣ Configure MFA Controls

## 🎯 Problem We Are Solving

MFA provides an additional authentication factor for AWS administrative access.

Without MFA:

```text
Password
   |
   v
AWS Account
```

With MFA:

```text
Password
   +
MFA
   |
   v
AWS Account
```

---

# 💡 Practical Understanding

MFA is a security credential used to provide an additional authentication factor.

It is different from an IAM policy.

```text
Security Credentials
        |
        +---- Password
        |
        +---- MFA


IAM Policy
        |
        +---- What actions are allowed
```

A simple way to remember:

```text
Credentials = Who are you?

MFA = Extra proof of identity

Policy = What are you allowed to do?
```

---

# 🛠️ Step 1 – Check IAM Users

I checked:

```text
AWS Console
    ↓
IAM
    ↓
Users
```

The account showed:

```text
IAM Users: 0
```

Therefore, there were no IAM users available for configuring an IAM-user-specific MFA policy.

---

# 📸 IAM Users Screenshot

![IAM Users](images/iam-users.png)

---

# 🛠️ Step 2 – Check Root User MFA

I checked the AWS security credentials page.

The page showed:

```text
My security credentials
Root user
```

The MFA section showed:

```text
Multi-factor authentication (MFA)

MFA devices: 1

Type: Virtual

Identifier:
Rahul-device
```

This confirmed that the root user already had a Virtual MFA device configured.

---

# 📸 Root MFA Screenshot

![Root MFA](images/root-mfa.png)

---

# 🧠 What I Learned About MFA

MFA is attached to the identity used for authentication.

It is not attached to:

```text
❌ EC2
❌ Security Group
❌ EBS Volume
❌ Bastion-role
```

Instead:

```text
Root User
    |
    +---- Password
    |
    +---- Virtual MFA
```

---

# ⚠️ Important Observation

The account currently has:

```text
IAM Users = 0
```

Therefore, I did not create an IAM user just to complete the MFA portion.

The existing root account already had Virtual MFA configured.

Creating an unnecessary IAM user would change the environment without providing value for this task.

---

# 🔄 Complete Administrative Access Flow

The complete flow we configured and verified is:

```text
                         AWS ACCOUNT
                              |
                              v
                     Root User / Admin
                              |
                       Password + MFA
                              |
                              v
                       AWS Management
                              |
                              v
                    Systems Manager
                              |
                              v
                     Session Manager
                              |
                              v
                     Bastion-Server
                              |
                              v
                         SSM Agent
                              |
                              v
                       Bastion-role
                              |
                              v
                AmazonSSMManagedInstanceCore
```

---

# 🧪 Hands-On Verification Summary

## Session Manager

Session result:

```text
sh-5.2$
```

Status:

```text
✅ Connected
```

## Session User

Command:

```bash
whoami
```

Result:

```text
ssm-user
```

Status:

```text
✅ Verified
```

## Hostname

Command:

```bash
hostname
```

Result:

```text
ip-10-0-0-254.ec2.internal
```

Status:

```text
✅ Bastion verified
```

## SSM Agent

Command:

```bash
sudo systemctl status amazon-ssm-agent
```

Result:

```text
Active: active (running)
```

Status:

```text
✅ Running
```

## IAM Role

```text
Bastion-role
```

Status:

```text
✅ Verified
```

## IAM Policy

```text
AmazonSSMManagedInstanceCore
```

Status:

```text
✅ Verified
```

## Root MFA

```text
Virtual MFA
Rahul-device
```

Status:

```text
✅ Already configured
```

---

# 📊 Final Task Verification

| Requirement | What We Did | Status |
|---|---|---|
| Session Manager | Started session to Bastion | ✅ |
| Session shell | Verified `sh-5.2$` | ✅ |
| Session user | Verified `ssm-user` | ✅ |
| Hostname | Verified Bastion hostname | ✅ |
| SSM Agent | `active (running)` | ✅ |
| EC2 IAM Role | `Bastion-role` | ✅ |
| SSM Policy | `AmazonSSMManagedInstanceCore` | ✅ |
| IAM Users | Verified `0` | ✅ |
| Root MFA | Virtual MFA configured | ✅ |
| Administrative access | Successfully tested | ✅ |

---

# 🧠 What I Learned

## 1. Session Manager

Session Manager allows me to access an EC2 instance through AWS Systems Manager.

```text
Laptop
   |
   v
Systems Manager
   |
   v
Session Manager
   |
   v
Bastion EC2
```

## 2. SSM Agent

The SSM Agent runs on the EC2 and communicates with Systems Manager.

```text
Systems Manager
      |
      v
SSM Agent
      |
      v
EC2
```

## 3. IAM Role

The EC2 uses an IAM Role rather than storing long-term AWS credentials.

```text
EC2
 |
 v
Bastion-role
 |
 v
AmazonSSMManagedInstanceCore
```

## 4. IAM Policy

An IAM Policy controls what actions an identity is allowed or denied to perform.

```text
IAM Policy
    |
    +---- Allow
    |
    +---- Deny
```

## 5. Security Credentials

Security credentials are used to authenticate an identity.

Examples include:

```text
Password
MFA
Access Keys
```

## 6. MFA

MFA provides an additional authentication factor.

Our root account already had:

```text
Virtual MFA
```

configured.

---

# ⭐ Interview Explanation

> I enabled secure administrative access to my existing Bastion EC2 using AWS Systems Manager Session Manager. The Bastion already had the `Bastion-role` IAM role with `AmazonSSMManagedInstanceCore`, so the SSM Agent could communicate with Systems Manager. I successfully started a Session Manager shell, verified the `ssm-user`, hostname, and SSM Agent status. I also checked the account's IAM users and found there were no IAM users, while the root account already had a Virtual MFA device configured.

---

# 📝 Quick Revision

```text
1. Session Manager
   |
   +---- Connect to EC2 without SSH
   |
   +---- Existing Bastion
   |
   +---- Session successful
   |
   +---- ssm-user


2. IAM
   |
   +---- Bastion-role
   |
   +---- AmazonSSMManagedInstanceCore
   |
   +---- SSM Agent uses role credentials


3. MFA
   |
   +---- Root User
   |
   +---- Virtual MFA
   |
   +---- Already configured
   |
   +---- IAM Users = 0
```

---

# 🔐 Final Security Architecture

```text
                    AWS ACCOUNT
                         |
                         v
                  Root User / Admin
                         |
                    Password + MFA
                         |
                         v
                  AWS Management
                         |
                         v
                Systems Manager
                         |
                         v
                 Session Manager
                         |
                         v
                 Bastion-Server
                         |
                         v
                    SSM Agent
                         |
                         v
                  Bastion-role
                         |
                         v
             AmazonSSMManagedInstanceCore
```

---

# 🏆 Final Result

## 🔐 Enable Secure Administrative Access to Amazon EC2 — COMPLETED 100% ✅

The task was successfully completed for the existing environment.

### Completed:

```text
✅ Session Manager connection
✅ SSM Agent verification
✅ ssm-user verification
✅ Bastion hostname verification
✅ Bastion IAM Role verification
✅ AmazonSSMManagedInstanceCore verification
✅ IAM Users verification
✅ Root Virtual MFA verification
```

The most important practical result was:

```text
AWS Console
     |
     v
Systems Manager
     |
     v
Session Manager
     |
     v
Bastion EC2
     |
     v
ssm-user
     |
     v
Shell Access
```

---

# 🚀 TASK COMPLETED

## 🔐 Enable Secure Administrative Access to Amazon EC2 — 100% COMPLETE
