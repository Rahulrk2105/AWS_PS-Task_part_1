# 🚀 Lab 06 – Create a Bastion Host on Amazon EC2


![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![EC2](https://img.shields.io/badge/EC2-Bastion%20Host-blue)
![SSM](https://img.shields.io/badge/AWS-Session%20Manager-purple)
![SSH](https://img.shields.io/badge/Access-SSH-green)
![Security](https://img.shields.io/badge/Security-Security%20Groups-red)
![Linux](https://img.shields.io/badge/Linux-Amazon%20Linux%202023-black)

---

# 📖 Project Overview

In this hands-on lab, I configured a **Bastion Host architecture** to securely access an EC2 instance running inside a private subnet.

I already had an existing VPC named `App-VPC` with a public subnet and a private subnet.

The Bastion Host was launched in the **public subnet** with a public IP address.

The private EC2 instance was launched in the **private subnet** without a public IP address.

I configured:

- Public and private subnet connectivity
- Internet Gateway routing
- Bastion Security Group
- Private Security Group
- IAM Role
- AWS Systems Manager Session Manager
- SSH access to Bastion
- SSH access from Bastion to Private EC2
- SSH key authentication
- Linux verification
- Troubleshooting of SSH connectivity and authentication

---

# 🎯 Objective

The main objective was to create a secure access path to a private EC2 instance through a Bastion Host.

The final access path was:

    Windows PC
        |
        | SSH :22
        | bastion-key.pem
        v
    Bastion Host
        |
        | SSH :22
        | private-key.pem
        v
    Private EC2

The private EC2 remained without a public IP.

---

# 🏗️ Architecture

    AWS Cloud
        |
        +---------------- App-VPC ----------------+
        |                                         |
        |                                         |
        v                                         v
    Public Subnet                           Private Subnet
        |                                         |
        v                                         v
    Bastion EC2                             Private EC2
    Public IP                               No Public IP
    34.229.239.61                           10.0.1.187
    10.0.0.254
        |                                         |
    Bastion-SG                              Private-SG
        |                                         |
        +------------- SSH :22 ------------------+

### Security Group Flow

    My Public IP
          |
          | SSH :22
          v
    Bastion-SG
          |
          | SSH :22
          v
    Private-SG

---

# 🧰 Prerequisites

Before starting the lab, I had:

- AWS Account
- Existing `App-VPC`
- Public subnet
- Private subnet
- Internet Gateway
- EC2 access
- SSH key pairs
- IAM permissions
- Amazon Linux 2023
- Windows PowerShell
- Basic Linux commands

---

# 🖥️ Environment Details

| Resource | Configuration |
|---|---|
| VPC | `App-VPC` |
| Public Subnet | Existing Public Subnet |
| Private Subnet | Existing Private Subnet |
| Bastion Server | `Bastion-Server` |
| Bastion Public IP | `34.229.239.61` |
| Bastion Private IP | `10.0.0.254` |
| Private Server | `private-server` |
| Private IP | `10.0.1.187` |
| Bastion Security Group | `Bastion-SG` |
| Private Security Group | `Private-SG` |
| Bastion Key | `bastion-key.pem` |
| Private EC2 Key | `private-key.pem` |
| IAM Role | `Bastion-role` |
| SSM Policy | `AmazonSSMManagedInstanceCore` |
| My Public IP | `49.205.145.213` |

---

# 🔹 Step 1 – Verify Existing VPC

I first checked the existing AWS environment.

The VPC already contained:

    App-VPC
    ├── Public Subnet
    └── Private Subnet

I decided to use the existing VPC instead of creating a new VPC.

The Bastion Host would be placed in the public subnet.

The private EC2 would be placed in the private subnet.

### 📸 Screenshot

<img width="1608" height="864" alt="image" src="https://github.com/user-attachments/assets/40c61139-f83e-49bc-ac61-98aa6abb7816" />


---

# 🔹 Step 2 – Verify Public Subnet Route

I checked the route table associated with the public subnet.

The public subnet needed a default route to the Internet Gateway.

The route was:

    Destination: 0.0.0.0/0
    Target: Internet Gateway

This was required because the Bastion Host needed internet connectivity.

The traffic flow was:

    Bastion
       |
       v
    Public Subnet
       |
       v
    Route Table
       |
       v
    Internet Gateway
       |
       v
    Internet

### 📸 Screenshot
<img width="1611" height="512" alt="image" src="https://github.com/user-attachments/assets/03617c27-4f33-4c3e-ba15-19decea32d3c" />

---

# 🔹 Step 3 – Configure Bastion Security Group

I created/configured a Security Group for the Bastion.

Security Group:

    Bastion-SG

I configured an inbound SSH rule.

    Type: SSH
    Protocol: TCP
    Port: 22
    Source: My IP

My public IP during the lab was:

    49.205.145.213/32

This means SSH access to the Bastion was restricted to my public IP.

I did not use:

    0.0.0.0/0

for SSH access.

### Security Flow

    My Public IP
          |
          | SSH :22
          v
    Bastion-SG
          |
          v
    Bastion EC2

### 📸 Screenshot

<img width="1616" height="721" alt="image" src="https://github.com/user-attachments/assets/40729120-bc71-4556-a242-03292f75a285" />


---

# 🔹 Step 4 – Configure Private Security Group

I created/configured another Security Group for the private EC2.

Security Group:

    Private-SG

Inbound rule:

    Type: SSH
    Protocol: TCP
    Port: 22
    Source: Bastion-SG

Instead of allowing SSH from the internet, I used the Bastion Security Group as the source.

The traffic flow was:

    Bastion-SG
          |
          | SSH :22
          v
    Private-SG
          |
          v
    Private EC2

This prevented direct SSH access to the private EC2 from the internet.

### 📸 Screenshot

<img width="1627" height="750" alt="image" src="https://github.com/user-attachments/assets/597cefbe-1c44-47d0-9424-336d88902b59" />


---

# 🔹 Step 5 – Launch Bastion EC2

I launched a new EC2 instance to act as the Bastion Host.

Configuration:

    Name: Bastion-Server
    VPC: App-VPC
    Subnet: Public Subnet
    Auto-assign Public IP: Enabled
    Security Group: Bastion-SG
    Key Pair: bastion-key

The Bastion received:

    Public IP: 34.229.239.61
    Private IP: 10.0.0.254

The Bastion was now reachable from the internet through its public IP, subject to the Security Group rule.

### 📸 Screenshot

<img width="1602" height="895" alt="image" src="https://github.com/user-attachments/assets/86cfb4a4-a704-498d-b0f2-4b8b9768fa01" />

---

# 🔹 Step 6 – Configure IAM Role for Session Manager

I created an IAM role for the Bastion.

Role name:

    Bastion-role

I attached the AWS managed policy:

    AmazonSSMManagedInstanceCore

This policy provides the permissions required for Systems Manager Session Manager.

I then attached the IAM role to the Bastion EC2.

### IAM Flow

    IAM Role
       |
       | AmazonSSMManagedInstanceCore
       v
    Bastion EC2
       |
       v
    AWS Systems Manager


---

# 🔹 Step 7 – Connect to Bastion Using Session Manager

After configuring the IAM role, I opened the EC2 console.

I selected:

    Bastion-Server

Then:

    Connect
       ↓
    Session Manager
       ↓
    Connect

A Linux shell opened successfully.

The shell showed:

    sh-5.2$

This confirmed that Session Manager was working.

### 📸 Screenshot

<img width="800" height="350" alt="image" src="https://github.com/user-attachments/assets/5eaf99b8-3ccd-4797-8d13-2bebe7273209" />


---

# 🔹 Step 8 – Launch Private EC2

Next, I launched the private EC2 instance.

Configuration:

    Name: private-server
    VPC: App-VPC
    Subnet: Private Subnet
    Public IP: None
    Private IP: 10.0.1.187
    Security Group: Private-SG
    Key Pair: private-key

The important configuration was:

    Public IP: None

The server was therefore not directly reachable from the internet.

The private EC2 would instead be accessed through the Bastion.

### 📸 Screenshot

<img width="1588" height="886" alt="image" src="https://github.com/user-attachments/assets/21e79480-fe8e-427d-8c8d-c8ba433516a9" />


---

# 🔹 Step 9 – Test Network Access from Bastion

I connected to the Bastion and tested SSH to the private EC2.

Command used:

    ssh ec2-user@10.0.1.187

The private EC2 responded, but SSH authentication failed.

The error was:

    Permission denied (publickey,gssapi-keyex,gssapi-with-mic)

This was an important troubleshooting point.

It showed that the Bastion could reach the private EC2, but authentication was not successful.

The situation was:

    Bastion → Private EC2 :22

    Network Connectivity    ✅
    SSH Authentication      ❌

The problem was not the Security Group.

The problem was the SSH authentication key.

---

# 🔹 Step 10 – Troubleshoot SSH Access to Bastion

I then tested SSH access from my Windows machine to the Bastion.

Command:

    ssh -i .\bastion-key.pem ec2-user@34.229.239.61

Initially, I received:

    Connection timed out

This indicated that the Windows machine could not establish the SSH connection to the Bastion.

I checked my current public IP.

Current public IP:

    49.205.145.213

I discovered that the Bastion Security Group had an older public IP configured.

I updated the Bastion Security Group SSH rule.

New rule:

    Type: SSH
    Protocol: TCP
    Port: 22
    Source: 49.205.145.213/32

After updating the Security Group, I tested SSH again.

The SSH connection worked successfully.

---

# 🔹 Step 11 – Connect to Bastion from Windows

From Windows PowerShell:

    ssh -i .\bastion-key.pem ec2-user@34.229.239.61

The connection was successful.

I reached the Bastion Linux shell.

The access path was now:

    Windows PC
         |
         | bastion-key.pem
         | SSH :22
         v
    Bastion Host


---

# 🔹 Step 12 – Understand the SSH Key Requirement

At this stage, I had two separate EC2 instances and two separate SSH keys.

The Bastion used:

    bastion-key.pem

The Private EC2 used:

    private-key.pem

Therefore:

    Windows PC
         |
         | bastion-key.pem
         v
    Bastion
         |
         | private-key.pem
         v
    Private EC2

The Bastion did not initially have the `private-key.pem`.

This was why the earlier SSH attempt resulted in:

    Permission denied (publickey)

---

# 🔹 Step 13 – Copy Private Key to Bastion

For this controlled learning exercise, I copied the Private EC2 key from my Windows machine to the Bastion.

From Windows PowerShell:

    scp -o "HostKeyAlias=ec2-34-229-239-61.compute-1.amazonaws.com" -i .\bastion-key.pem .\private-key.pem ec2-user@34.229.239.61:/home/ec2-user/

The file was successfully copied.

The transfer showed:

    private-key.pem  100%

### Why was HostKeyAlias used?

During the `scp` operation, SSH host-key verification caused a problem because the Bastion was already known under another hostname.

I used:

    HostKeyAlias=ec2-34-229-239-61.compute-1.amazonaws.com

This allowed the existing known host entry to be used.


---

# 🔹 Step 14 – Verify Private Key on Bastion

After logging into the Bastion, I checked whether the private key was present.

Command:

    ls -l ~/private-key.pem

The key was present:

    /home/ec2-user/private-key.pem

This confirmed that the file had been copied successfully.

---

# 🔹 Step 15 – Secure the Private Key

SSH requires private keys to have restrictive permissions.

I changed the permissions using:

    chmod 400 ~/private-key.pem

I then verified the key permissions.

The private key was now ready to be used for SSH authentication.

### ⚠️ Security Note

Copying a private SSH key to a Bastion was done only for this controlled learning exercise.

In a production environment, private keys should generally not be stored on Bastion hosts.

A better production design would use alternatives such as:

- AWS Systems Manager Session Manager
- SSH agent forwarding where appropriate
- Just-in-time access
- Centralized identity/access management

---

# 🔹 Step 16 – Connect from Bastion to Private EC2

From the Bastion, I ran:

    ssh -i ~/private-key.pem ec2-user@10.0.1.187

This time, the SSH authentication succeeded.

The final shell prompt was:

    [ec2-user@ip-10-0-1-187 ~]$

This confirmed that I was now connected to the Private EC2.

### 📸 Screenshot
<img width="1172" height="796" alt="image" src="https://github.com/user-attachments/assets/821cbdd7-b15d-414c-a150-836a1775161a" />


---

# 🔹 Step 17 – Verify the Private EC2

Once connected to the private EC2, I performed Linux verification.

I ran:

    hostname

Then:

    hostname -I

Then:

    whoami

The results confirmed:

    Private IP: 10.0.1.187
    User: ec2-user

This confirmed that I was inside the Private EC2 and not the Bastion.


---

# 🔹 Step 18 – Verify the Complete Access Path

The complete access path was successfully established.

    Windows PC
         |
         | SSH :22
         | bastion-key.pem
         v
    Bastion Host
    34.229.239.61
         |
         | SSH :22
         | private-key.pem
         v
    Private EC2
    10.0.1.187

The Private EC2 did not require a public IP.

The Bastion acted as the controlled entry point.

---

# 🔍 Troubleshooting Performed

## Issue 1 – Session Manager Connection

### Problem

Initially, Session Manager was not connecting.

### Investigation

I checked the IAM role attached to the Bastion.

### Solution

I configured the IAM role:

    Bastion-role

and attached:

    AmazonSSMManagedInstanceCore

After correcting the configuration and connectivity, Session Manager successfully opened a Linux shell.

---

## Issue 2 – SSH Connection Timeout to Bastion

### Problem

From Windows:

    ssh -i .\bastion-key.pem ec2-user@34.229.239.61

returned:

    Connection timed out

### Cause

The Bastion Security Group had an old public IP address.

### Solution

I checked my current public IP:

    49.205.145.213

Then updated Bastion-SG:

    SSH
    TCP
    22
    49.205.145.213/32

After the update, SSH access worked.

---

## Issue 3 – Permission Denied to Private EC2

### Problem

From the Bastion:

    ssh ec2-user@10.0.1.187

returned:

    Permission denied (publickey,gssapi-keyex,gssapi-with-mic)

### Cause

The Bastion did not have the SSH private key required by the Private EC2.

### Solution

I copied:

    private-key.pem

to the Bastion.

Then:

    chmod 400 ~/private-key.pem

Finally:

    ssh -i ~/private-key.pem ec2-user@10.0.1.187

The connection succeeded.

---

## Issue 4 – SCP Host Key Verification

### Problem

While using `scp`, SSH displayed a host-key verification message and I initially could not type into the prompt.

### Solution

I used:

    -o "HostKeyAlias=ec2-34-229-239-61.compute-1.amazonaws.com"

The command then successfully copied the private key.

---

# 🧪 Important Commands Used

## Check Current Public IP

    (Invoke-RestMethod https://checkip.amazonaws.com).Trim()

## SSH to Bastion

    ssh -i .\bastion-key.pem ec2-user@34.229.239.61

## Copy Private Key to Bastion

    scp -o "HostKeyAlias=ec2-34-229-239-61.compute-1.amazonaws.com" -i .\bastion-key.pem .\private-key.pem ec2-user@34.229.239.61:/home/ec2-user/

## Check Private Key

    ls -l ~/private-key.pem

## Change Private Key Permissions

    chmod 400 ~/private-key.pem

## SSH to Private EC2

    ssh -i ~/private-key.pem ec2-user@10.0.1.187

## Verify Hostname

    hostname

## Verify IP

    hostname -I

## Verify Current User

    whoami

---

# 📊 Final Verification

| Component | Status |
|---|---|
| Existing App-VPC | ✅ |
| Public Subnet | ✅ |
| Private Subnet | ✅ |
| Internet Gateway Route | ✅ |
| Bastion EC2 | ✅ |
| Bastion Public IP | ✅ |
| Bastion Security Group | ✅ |
| Private EC2 | ✅ |
| Private EC2 without Public IP | ✅ |
| Private Security Group | ✅ |
| IAM Role | ✅ |
| SSM Policy | ✅ |
| Session Manager | ✅ |
| SSH to Bastion | ✅ |
| SSH from Bastion to Private EC2 | ✅ |
| Linux Verification | ✅ |

---

# 🧠 What I Learned

## 1. Bastion Host

A Bastion Host provides a controlled entry point to resources that are located in a private subnet.

## 2. Public Subnet

The Bastion was placed in the public subnet because it needed controlled access from my system.

## 3. Private Subnet

The Private EC2 was placed in the private subnet and did not have a public IP.

## 4. Security Groups

The Bastion Security Group allowed SSH only from my public IP.

The Private Security Group allowed SSH only from the Bastion Security Group.

This created the following security flow:

    My Public IP
          |
          | SSH
          v
    Bastion-SG
          |
          | SSH
          v
    Private-SG

## 5. Session Manager

AWS Systems Manager Session Manager allowed me to connect to the Bastion through the AWS Console.

## 6. SSH Authentication

The two servers used different key pairs.

    bastion-key.pem
    Windows → Bastion

    private-key.pem
    Bastion → Private EC2

## 7. Troubleshooting

I learned that:

    Connection timed out

usually indicates a network, routing or Security Group problem.

While:

    Permission denied (publickey)

indicates that network connectivity is working but SSH authentication has failed.

---

# 🔐 Security Considerations

The Bastion SSH rule should not normally allow:

    0.0.0.0/0

SSH should be restricted to a trusted IP address.

The Private EC2 should not have a public IP when the objective is to keep it private.

The Private Security Group should allow SSH from:

    Bastion-SG

rather than:

    0.0.0.0/0

The temporary private key copied to the Bastion should be removed after the lab:

    rm -f ~/private-key.pem


---

# 🎯 Conclusion

Successfully completed the **Launch EC2 Instance, Configure Session Manager, Configure SSH Access and Configure Security Groups** hands-on lab.

Through this lab, I gained practical experience with:

- Amazon EC2
- VPC
- Public Subnets
- Private Subnets
- Internet Gateway
- Security Groups
- IAM Roles
- AWS Systems Manager Session Manager
- SSH
- SSH Key Authentication
- Bastion Host Architecture
- Linux Verification
- AWS Troubleshooting

The final result demonstrated how a private EC2 instance can remain without a public IP while still being securely accessed through a Bastion Host.
