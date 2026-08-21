# 🔐 Enable Private Connectivity to Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![VPC](https://img.shields.io/badge/AWS-VPC-purple)
![VPC%20Endpoints](https://img.shields.io/badge/AWS-VPC%20Endpoints-blue)
![Systems%20Manager](https://img.shields.io/badge/AWS-Systems%20Manager-green)
![S3](https://img.shields.io/badge/AWS-S3-yellow)
![IAM](https://img.shields.io/badge/AWS-IAM-red)

---

## 📖 Project Overview

In this hands-on task, I configured private connectivity for an Amazon EC2 instance.

The private EC2 instance was configured without a public IP address and without a route to the Internet Gateway. I configured VPC Endpoints so that the private EC2 could communicate with AWS services without requiring Internet access.

The task included:

1. Creating a VPC with public and private subnets
2. Configuring a public route table and Internet Gateway
3. Configuring a dedicated private route table
4. Configuring Security Groups
5. Creating an S3 Gateway VPC Endpoint
6. Creating Systems Manager Interface VPC Endpoints
7. Attaching an IAM role with `AmazonSSMManagedInstanceCore`
8. Launching a private EC2 instance
9. Connecting to the private EC2 using Session Manager
10. Verifying that Internet access was unavailable
11. Verifying private connectivity to Amazon S3

---

# 🎯 Objective

The objective was to configure an EC2 instance in a **private subnet** that:

```text
❌ Cannot access the Internet directly
✅ Can communicate with AWS services privately
```

The final architecture was:

```text
                         VPC
                    10.0.0.0/16
                         |
              +----------+----------+
              |                     |
              v                     v
        Public Subnet          Private Subnet
        10.0.1.0/24            10.0.0.0/24
              |                     |
             IGW                Private EC2
                                    |
                       +------------+------------+
                       |                         |
                       v                         v
                SSM Interface              S3 Gateway
                   Endpoints                Endpoint
                       |                         |
                       v                         v
                AWS Systems Manager            S3
```

---

# 1️⃣ VPC Configuration

I used the VPC:

```text
VPC CIDR: 10.0.0.0/16
```

The VPC contained:

```text
Public Subnet
10.0.1.0/24

Private Subnet
10.0.0.0/24
```

### 📸 VPC Screenshot

_Add screenshot here._

---

# 2️⃣ Subnet Configuration

I configured two subnets in the VPC.

### Public Subnet

```text
CIDR: 10.0.1.0/24
```

The public subnet was configured for Internet connectivity through the Internet Gateway.

### Private Subnet

```text
CIDR: 10.0.0.0/24
```

The private subnet was used for the private EC2 instance.

The private subnet did not have a default route to the Internet Gateway.

### 📸 Subnet Screenshot

_Add screenshot here._

---

# 3️⃣ Internet Gateway and Public Route Table

I configured an Internet Gateway for the VPC.

The public route table contained:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

The public subnet was associated with this route table.

### 📸 Internet Gateway Screenshot

_Add screenshot here._

### 📸 Public Route Table Screenshot

_Add screenshot here._

---

# 4️⃣ Private Route Table

I created a dedicated route table:

```text
private-rt
```

The private subnet was explicitly associated with this route table.

The local VPC route was:

```text
10.0.0.0/16 → local
```

There was no:

```text
0.0.0.0/0 → Internet Gateway
```

route in the private route table.

### 📸 Private Route Table Screenshot

_Add screenshot here._

---

# 5️⃣ Security Groups

I used the existing Security Group:

```text
private-ec2-SG
```

for the private EC2 instance.

I also created:

```text
SSM-Endpoint-SG
```

for the Systems Manager Interface VPC Endpoints.

The endpoint Security Group allowed:

| Type | Protocol | Port | Source |
|---|---|---:|---|
| HTTPS | TCP | 443 | `private-ec2-SG` |

The endpoint Security Group was reused for all three Systems Manager Interface Endpoints.

### 📸 Security Group Screenshots

_Add screenshot here._

---

# 6️⃣ Create S3 Gateway VPC Endpoint

I created an S3 Gateway VPC Endpoint.

### Configuration

```text
Service: Amazon S3
Endpoint Type: Gateway
VPC: 10.0.0.0/16
Route Table: private-rt
```

The endpoint was associated with the private route table.

The traffic path was:

```text
Private EC2
     |
     v
Private Route Table
     |
     v
S3 Gateway VPC Endpoint
     |
     v
Amazon S3
```

### 📸 S3 Endpoint Screenshot

_Add screenshot here._

---

# 7️⃣ Configure IAM Role for Session Manager

I reused the existing IAM role:

```text
Bastion-role
```

The role contained:

```text
AmazonSSMManagedInstanceCore
```

I attached this IAM role to the private EC2 instance.

### 📸 IAM Role Screenshot

_Add screenshot here._

---

# 8️⃣ Create Systems Manager Interface VPC Endpoints

Because the EC2 instance did not have Internet access, I created private Interface VPC Endpoints for Systems Manager.

The following endpoints were created:

```text
com.amazonaws.us-east-1.ssm
com.amazonaws.us-east-1.ssmmessages
com.amazonaws.us-east-1.ec2messages
```

### Endpoint Type

```text
Interface
```

The endpoints were placed in the private subnet and associated with:

```text
SSM-Endpoint-SG
```

Private DNS was enabled.

### 📸 SSM Endpoint Screenshots

_Add screenshots here._

---

# 9️⃣ Launch Private EC2

I launched the EC2 instance in the private subnet.

### Configuration

```text
Name: Private-EC2
AMI: Amazon Linux 2023
Instance Type: t3.micro
Subnet: Private Subnet
Security Group: private-ec2-SG
Public IP: Disabled
IAM Role: Bastion-role
```

The instance was successfully launched.

### Instance Details

```text
State: Running
Private IPv4: 10.0.0.58
Public IPv4: -
Public DNS: -
```

The instance passed its status checks.

### 📸 Private EC2 Screenshot

_Add screenshot here._

---

# 🔟 Connect Using Session Manager

Because the EC2 instance did not have a public IP, I connected to it using:

```text
EC2 → Instance → Connect → Session Manager
```

The Session Manager terminal opened successfully.

I verified the logged-in user using:

```bash
whoami
```

The result was:

```text
ssm-user
```

### 📸 Session Manager Screenshot

_Add screenshot here._

---

# 1️⃣1️⃣ Verify Internet Access Is Blocked

From the private EC2 Session Manager terminal, I tested Internet connectivity:

```bash
curl https://checkip.amazonaws.com
```

The request failed with:

```text
curl: (28) Failed to connect to checkip.amazonaws.com port 443
```

This confirmed that the private EC2 did not have direct Internet connectivity.

### 📸 Internet Connectivity Test Screenshot

_Add screenshot here._

---

# 1️⃣2️⃣ Configure S3 Permissions

The initial S3 upload test returned an IAM permission error because the role did not have permission to perform `s3:PutObject`.

I added the required S3 permissions to:

```text
Bastion-role
```

After the permission was added, the private EC2 was able to upload an object to the S3 bucket.

### 📸 IAM S3 Permission Screenshot

_Add screenshot here._

---

# 1️⃣3️⃣ Create Test Object

I created a test file on the private EC2 using `/tmp`:

```bash
printf '%s\n' 'Private EC2 to S3 through VPC Endpoint' > /tmp/test.txt
```

I verified the file:

```bash
cat /tmp/test.txt
```

Output:

```text
Private EC2 to S3 through VPC Endpoint
```

### 📸 Test File Screenshot

_Add screenshot here._

---

# 1️⃣4️⃣ Upload Object to S3

I uploaded the test file from the private EC2 to the S3 bucket:

```bash
aws s3 cp /tmp/test.txt s3://pvt-connectivity-bkt/
```

The upload completed successfully:

```text
upload: ../../tmp/test.txt to s3://pvt-connectivity-bkt/test.txt
```

### 📸 S3 Upload Screenshot

_Add screenshot here._

---

# 1️⃣5️⃣ Verify S3 Object

I verified the uploaded object using:

```bash
aws s3 ls s3://pvt-connectivity-bkt/
```

The result showed:

```text
2026-08-21 05:25:55 39 test.txt
```

### 📸 S3 Verification Screenshot

_Add screenshot here._

---

# 🔄 Private Connectivity Flow

The final connectivity flow was:

```text
                         Private EC2
                         10.0.0.58
                              |
                 +------------+------------+
                 |                         |
                 v                         v
        SSM Interface Endpoints       S3 Gateway Endpoint
                 |                         |
                 v                         v
        AWS Systems Manager               S3
```

Internet connectivity was not available:

```text
Private EC2
     |
     v
Internet
     ❌
```

S3 connectivity was available:

```text
Private EC2
     |
     v
S3 Gateway Endpoint
     |
     v
Amazon S3
     ✅
```

---

# 📊 Final Configuration

| Component | Configuration |
|---|---|
| VPC | `10.0.0.0/16` |
| Public Subnet | `10.0.1.0/24` |
| Private Subnet | `10.0.0.0/24` |
| Internet Gateway | Attached to VPC |
| Public Route | `0.0.0.0/0 → IGW` |
| Private Route Table | `private-rt` |
| Private Internet Route | Not configured |
| Private EC2 | `10.0.0.58` |
| Public IP | Disabled |
| EC2 Security Group | `private-ec2-SG` |
| Endpoint Security Group | `SSM-Endpoint-SG` |
| S3 Endpoint | Gateway |
| SSM Endpoint | Interface |
| SSM Messages Endpoint | Interface |
| EC2 Messages Endpoint | Interface |
| IAM Role | `Bastion-role` |
| SSM Policy | `AmazonSSMManagedInstanceCore` |
| S3 Test Bucket | `pvt-connectivity-bkt` |
| Test Object | `test.txt` |

---

# ✅ Task Result

The private connectivity environment was successfully configured.

The EC2 instance was placed in a private subnet without a public IP address and without a route to the Internet Gateway.

Systems Manager Interface VPC Endpoints were configured to allow Session Manager access to the private EC2 without requiring a bastion host or public Internet connectivity.

An S3 Gateway VPC Endpoint was configured for the private subnet.

The final tests confirmed:

```text
Private EC2 → Internet
❌ Failed as expected

Private EC2 → Session Manager
✅ Successful

Private EC2 → S3 through VPC Endpoint
✅ Successful

S3 object upload
✅ Successful
```

The test object `test.txt` was successfully uploaded from the private EC2 instance to the S3 bucket through the configured private connectivity.
