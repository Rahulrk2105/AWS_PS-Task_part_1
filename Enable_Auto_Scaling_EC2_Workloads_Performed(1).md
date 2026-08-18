# ⚙️ Enable Auto Scaling for Amazon EC2 Workloads

## 📖 Project Overview

In this hands-on task, I configured and verified **Amazon EC2 Auto Scaling**.

The task included:

1. Creating a dedicated VPC and public subnets
2. Configuring Internet connectivity
3. Creating a Security Group
4. Creating an EC2 Launch Template
5. Creating an Auto Scaling Group
6. Configuring EC2 health checks
7. Configuring a Target Tracking scaling policy
8. Verifying the application
9. Generating CPU load
10. Verifying automatic scale-out

---

# 🎯 Objective

The objective was to configure an Auto Scaling Group that automatically increases EC2 capacity when CPU utilization becomes high.

The architecture used for the lab was:

```text
                         Internet
                            |
                            v
                   Internet Gateway
                            |
              +-------------+-------------+
              |                           |
              v                           v
       Public Subnet 1              Public Subnet 2
        us-east-1a                   us-east-1b
              |                           |
              +-------------+-------------+
                            |
                            v
                    Auto Scaling Group
                            |
                     Launch Template
                            |
                 +----------+----------+
                 |                     |
                EC2-1                 EC2-2
```

The initial configuration was:

```text
Minimum Capacity: 1
Desired Capacity: 1
Maximum Capacity: 2
```

The scaling policy was:

```text
Metric: Average CPU Utilization
Target: 50%
```

---

# 🖥️ Environment Configuration

| Resource | Configuration |
|---|---|
| VPC | New dedicated VPC |
| VPC CIDR | `10.20.0.0/16` |
| Subnet 1 | `ASG-public-subnet-1` |
| Subnet 1 AZ | `us-east-1a` |
| Subnet 1 CIDR | `10.20.1.0/24` |
| Subnet 2 | `ASG-public-subnet-2` |
| Subnet 2 AZ | `us-east-1b` |
| Subnet 2 CIDR | `10.20.2.0/24` |
| Internet Gateway | Attached to VPC |
| Security Group | `autoscaling-lab-sg` |
| Launch Template | `autoscaling-lab-template` |
| Auto Scaling Group | `ASG-1` |
| Instance Type | `t3.micro` |
| AMI | Amazon Linux 2023 |

---

# 1️⃣ Create VPC

I created a new VPC for the Auto Scaling lab.

```text
VPC CIDR: 10.20.0.0/16
```

The VPC was used as the network for the Auto Scaling environment.

### 📸 VPC Screenshot

_Add screenshot here._

---

# 2️⃣ Create Public Subnets

I created two public subnets in different Availability Zones.

```text
ASG-public-subnet-1
AZ: us-east-1a
CIDR: 10.20.1.0/24
```

```text
ASG-public-subnet-2
AZ: us-east-1b
CIDR: 10.20.2.0/24
```

I enabled automatic assignment of public IPv4 addresses for the public subnets.

### 📸 Subnet Screenshot

_Add screenshot here._

---

# 3️⃣ Configure Internet Gateway

I created an Internet Gateway and attached it to the new VPC.

The Internet Gateway provided internet connectivity for the public subnets.

### 📸 Internet Gateway Screenshot

_Add screenshot here._

---

# 4️⃣ Configure Public Route Table

I created a route table for the public subnets.

The route was configured as:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Both public subnets were associated with this route table.

### 📸 Route Table Screenshot

_Add screenshot here._

---

# 5️⃣ Create Security Group

I created the Security Group:

```text
autoscaling-lab-sg
```

## Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | 22 | My IP |
| HTTP | TCP | 80 | 0.0.0.0/0 |

The default outbound rule was retained.

### 📸 Security Group Screenshot

_Add screenshot here._

---

# 6️⃣ Create Launch Template

I created the Launch Template:

```text
autoscaling-lab-template
```

## Launch Template Configuration

```text
AMI: Amazon Linux 2023
Instance Type: t3.micro
Key Pair: asg-launch-key
Security Group: autoscaling-lab-sg
Root Volume: 8 GiB gp3
Encryption: Enabled
Delete on Termination: Enabled
```

I did not select a subnet directly in the Launch Template because the Auto Scaling Group selects the configured subnets.

## User Data

I configured Apache using User Data:

```bash
#!/bin/bash

dnf install -y httpd

systemctl enable httpd
systemctl start httpd

TOKEN=$(curl -sX PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

INSTANCE_ID=$(curl -sH "http://169.254.169.254/latest/meta-data/instance-id" -H "X-aws-ec2-metadata-token: $TOKEN")

cat > /var/www/html/index.html <<EOF
<h1>Auto Scaling Lab</h1>
<h2>Instance ID: $INSTANCE_ID</h2>
<p>This EC2 was launched by Auto Scaling.</p>
EOF
```

### 📸 Launch Template Screenshot

_Add screenshot here._

---

# 7️⃣ Create Auto Scaling Group

I created the Auto Scaling Group:

```text
ASG-1
```

I selected:

```text
Launch Template: autoscaling-lab-template
```

## Network Configuration

I selected the new VPC and both public subnets:

```text
us-east-1a → ASG-public-subnet-1
us-east-1b → ASG-public-subnet-2
```

## Group Size

```text
Minimum Capacity: 1
Desired Capacity: 1
Maximum Capacity: 2
```

### 📸 Auto Scaling Group Screenshot

_Add screenshot here._

---

# 8️⃣ Configure Health Checks

I enabled the EC2 health check.

```text
Health Check Type: EC2
Health Check Grace Period: 300 seconds
```

The Auto Scaling Group used the EC2 health status to maintain healthy instances.

### 📸 Health Check Screenshot

_Add screenshot here._

---

# 9️⃣ Configure Target Tracking Scaling Policy

I created a dynamic scaling policy using Target Tracking.

```text
Policy Type: Target Tracking Scaling
Metric: Average CPU Utilization
Target Value: 50%
Instance Warmup: 300 seconds
Scale In: Enabled
```

The policy was enabled successfully.

### 📸 Target Tracking Screenshot

_Add screenshot here._

---

# 🔟 Verify Initial EC2 Instance

The Auto Scaling Group launched the initial EC2 instance.

The instance was:

```text
State: Running
Lifecycle: InService
Health: Healthy
```

The instance was launched in one of the configured public subnets and received a public IPv4 address.

### 📸 Initial Instance Screenshot

_Add screenshot here._

---

# 1️⃣1️⃣ Verify Application

I opened the public IPv4 address of the EC2 instance in a browser.

The Apache page displayed:

```text
Auto Scaling Lab
Instance ID: i-xxxxxxxxxxxxxxxxx
This EC2 was launched by Auto Scaling.
```

This verified that the web server was running successfully.

### 📸 Application Screenshot

_Add screenshot here._

---

# 1️⃣2️⃣ Connect to EC2 Using SSH

I connected to the running Auto Scaling instance from PowerShell using the configured key pair.

```bash
ssh -i asg-launch-key.pem ec2-user@<PUBLIC-IP>
```

The SSH connection was successful.

### 📸 SSH Screenshot

_Add screenshot here._

---

# 1️⃣3️⃣ Generate CPU Load

Inside the EC2 instance, I generated CPU load using:

```bash
yes > /dev/null &
```

Additional CPU workers were started:

```bash
yes > /dev/null &
yes > /dev/null &
yes > /dev/null &
```

I verified the CPU utilization using:

```bash
top
```

The CPU utilization increased above the configured 50% target.

### 📸 CPU Load Screenshot

_Add screenshot here._

---

# 1️⃣4️⃣ Automatic Scale-Out

Before starting the automatic scaling test, the Auto Scaling Group was configured with:

```text
Minimum Capacity: 1
Desired Capacity: 1
Maximum Capacity: 2
```

I generated CPU load on the running EC2 instance.

The Target Tracking policy monitored the average CPU utilization.

The scaling process was:

```text
EC2 CPU utilization increases
        |
        v
CloudWatch CPU metric
        |
        v
Target Tracking Policy
        |
        v
CPU exceeds 50% target
        |
        v
Auto Scaling Group increases capacity
        |
        v
Launch Template
        |
        v
Second EC2 instance launched
```

The Auto Scaling Group automatically launched a second EC2 instance.

The group then showed:

```text
Instances: 2
```

Both instances were:

```text
InService
Healthy
```

The instances were distributed across the configured Availability Zones.

### 📸 Automatic Scale-Out Screenshot

_Add screenshot here._

---

# 1️⃣5️⃣ Verify Auto Scaling Activity

I opened:

```text
EC2
  ↓
Auto Scaling Groups
  ↓
ASG-1
  ↓
Activity
```

The activity history showed the launch of the additional EC2 instance.

This confirmed that the second instance was launched by the Auto Scaling Group as part of the scaling event.

### 📸 Activity History Screenshot

_Add screenshot here._

---

# 🔄 Automatic Scaling Flow

The final automatic scaling flow was:

```text
                    EC2 Instance
                         |
                         v
                  CPU Utilization
                         |
                         v
                  CloudWatch Metric
                         |
                         v
              Target Tracking Policy
                    CPU Target 50%
                         |
                         v
                Auto Scaling Group
                         |
                         v
                  Capacity Increase
                         |
                         v
                  Launch Template
                         |
                         v
                  Second EC2 Instance
```

---

# 📊 Final Configuration

| Component | Configuration |
|---|---|
| VPC | New VPC |
| Public Subnets | 2 |
| Availability Zones | us-east-1a, us-east-1b |
| Internet Gateway | Attached |
| Security Group | `autoscaling-lab-sg` |
| Launch Template | `autoscaling-lab-template` |
| Instance Type | `t3.micro` |
| Auto Scaling Group | `ASG-1` |
| Minimum Capacity | 1 |
| Desired Capacity | 1 |
| Maximum Capacity | 2 |
| Health Check | EC2 |
| Scaling Policy | Target Tracking |
| CPU Target | 50% |
| Application | Apache HTTP Server |

---

# ✅ Task Result

The Amazon EC2 Auto Scaling environment was configured successfully.

The Auto Scaling Group was configured across two Availability Zones with a minimum capacity of 1 and a maximum capacity of 2.

A Target Tracking policy was configured to monitor average CPU utilization at a 50% target.

After generating high CPU utilization on the EC2 instance, the Auto Scaling Group automatically launched a second EC2 instance.

The automatic scale-out was verified through:

- Auto Scaling Group instance management
- Two healthy InService instances
- Auto Scaling activity history
