# 🚀 Lab 05 – Create and Configure EBS Volumes on Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![EBS](https://img.shields.io/badge/AWS-EBS-blue)
![Storage](https://img.shields.io/badge/Storage-EBS-green)
![gp3](https://img.shields.io/badge/Volume-gp3-purple)
![Encryption](https://img.shields.io/badge/Encryption-AWS%20KMS-success)
![Linux](https://img.shields.io/badge/Linux-Amazon%20Linux%202023-black)
![XFS](https://img.shields.io/badge/Filesystem-XFS-yellow)

---

## 📖 Project Overview

In this lab, I expanded the storage of my existing Amazon EC2 Database Server by provisioning and attaching an additional Amazon EBS volume.

Instead of creating a new EC2 instance, I used my existing `Database-server` instance and attached an additional **5 GiB gp3 EBS volume**.

The volume was encrypted using the AWS-managed **`aws/ebs` KMS key**, attached to the EC2 instance, detected by Amazon Linux 2023 as `nvme1n1`, and mounted at `/data`.

This lab demonstrates how additional EBS storage can be provisioned and made available to an existing EC2 instance without modifying the existing root volume.

---

# 🎯 Objectives

The objectives of this lab were:

* Create an additional Amazon EBS volume
* Configure the volume as a `gp3` volume
* Enable EBS encryption
* Attach the volume to an existing EC2 instance
* Verify the attached volume from Linux
* Configure the XFS filesystem
* Mount the volume to `/data`
* Verify the available storage
* Confirm the EBS volume is encrypted

---

# 🏗️ Architecture

```text
                         AWS Cloud
                             │
                             ▼
                    Amazon EC2 Instance
                     Database-server
                     Amazon Linux 2023
                             │
                  ┌──────────┴──────────┐
                  │                     │
                  ▼                     ▼
            Root EBS Volume        Additional EBS
                8 GiB                  5 GiB
                 gp3                    gp3
                  │                     │
              nvme0n1                nvme1n1
                  │                     │
                  /                    /data
                                        │
                                  XFS Filesystem
                                        │
                                  EBS Encryption
                                        │
                                    aws/ebs KMS
```

---

# 🛠️ Prerequisites

* AWS Account
* Amazon EC2
* Existing EC2 instance
* Amazon Linux 2023
* EC2 instance in a suitable Availability Zone
* SSH access to the EC2 instance
* Basic Linux commands

---

# 🖥️ Environment

| Resource          | Configuration     |
| ----------------- | ----------------- |
| Cloud Provider    | AWS               |
| Service           | Amazon EC2        |
| Storage Service   | Amazon EBS        |
| EC2 Instance      | Database-server   |
| Operating System  | Amazon Linux 2023 |
| Root Volume       | 8 GiB             |
| Additional Volume | 5 GiB             |
| Volume Type       | gp3               |
| Filesystem        | XFS               |
| Mount Point       | `/data`           |
| Encryption        | Enabled           |
| KMS Key           | `aws/ebs`         |

---

# 📷 Initial Storage Configuration

Before adding the new EBS volume, I verified the existing storage configuration of the EC2 instance.


The initial `lsblk` output showed the existing 8 GiB root volume:

```bash
lsblk
```

output:

```text
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
```

At this stage, the additional EBS volume was not yet attached.

---

# ⚙️ Implementation

## Step 1 – Create the EBS Volume

I opened:

**AWS Console → EC2 → Elastic Block Store → Volumes**

Then selected:

**Create volume**

The additional volume was configured with:

* Volume type: `gp3`
* Size: `5 GiB`
* Availability Zone: same Availability Zone as the `Database-server`
* Encryption: Enabled

The volume was created with the AWS-managed EBS encryption key.

---

# 📷 EBS Volume Created

<img width="1624" height="880" alt="image" src="https://github.com/user-attachments/assets/d40156f7-7151-4fae-b3d7-2b6d37d20540" />


The created volume had the following configuration:

```text
Volume ID: vol-0823995c480113965
Size: 5 GiB
Type: gp3
IOPS: 3000
Throughput: 125 MiB/s
Encryption: Enabled
KMS Key Alias: aws/ebs
```

The volume was initially in the:

```text
Available
```

state.

---

# 🔐 Step 2 – Configure EBS Encryption

EBS encryption was enabled while creating the volume.

The volume used the AWS-managed KMS key:

```text
aws/ebs
```

Encryption protects the data stored on the EBS volume.

The volume details showed:

```text
Encryption: Encrypted
KMS Key Alias: aws/ebs
```

---

# 📷 EBS Encryption Verification


The AWS Console confirmed that the volume was encrypted.

The encryption section showed:

```text
Encryption: Encrypted
KMS Key Alias: aws/ebs
```

This confirms that the additional EBS volume was created with encryption enabled.

---

# 🔗 Step 3 – Attach the EBS Volume

The newly created volume was attached to the existing:

```text
Database-server
```

instance.

In the AWS Console, the volume was attached using the EC2 **Attach volume** action.

The device name selected during attachment was:

```text
/dev/sdf
```

Because the instance uses Nitro-based NVMe storage, Linux exposed the attached EBS volume as:

```text
/dev/nvme1n1
```

This is normal behavior for Nitro-based EC2 instances.

---


```text
Attached resources:
i-00e20aa388697ceb5 (Database-server)

EBS card index:
0

Device:
 /dev/sdf
```

The volume state changed from:

```text
Available
```

to:

```text
In-use
```

---

# 🖥️ Step 4 – Verify the Attached Volume

After connecting to the `Database-server` using SSH, I checked the block devices.

Command:

```bash
lsblk
```

The output showed the newly attached 5 GiB device:

```text
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme1n1       259:0    0   5G  0 disk
nvme0n1       259:1    0   8G  0 disk
├─nvme0n1p1   259:2    0   8G  0 part /
├─nvme0n1p127 259:3    0   1M  0 part
└─nvme0n1p128 259:4    0  10M  0 part /boot/efi
```

The new volume appeared as:

```text
/dev/nvme1n1
```

with a size of:

```text
5G
```

---

# 📷 Linux Volume Detection

<img width="1202" height="595" alt="image" src="https://github.com/user-attachments/assets/9bb6393e-b2b6-4818-8c2f-ad0b2a4a0bbb" />

This verified that Amazon Linux successfully detected the newly attached EBS volume.

---

# 🗂️ Step 5 – Check the Existing Filesystem

Before formatting the device, I checked whether the volume already contained a filesystem.

Command:

```bash
sudo blkid /dev/nvme1n1
```

Output:

```text
/dev/nvme1n1:
UUID="3edfc932-89f0-4e6d-a3a8-459d97db74a0"
BLOCK_SIZE="512"
TYPE="xfs"
```

The output showed that the volume already contained an **XFS filesystem**.

Because an existing filesystem was detected, I did not force-format the volume.

---

# ⚠️ Filesystem Formatting Note

I initially entered an incorrect filesystem type:

```bash
sudo mkfs -t xfx /dev/nvme1n1
```

This returned:

```text
mkfs: failed to execute mkfs.xfx:
No such file or directory
```

The correct filesystem type is:

```text
xfs
```

I then checked the correct device:

```bash
sudo mkfs -t xfs /dev/nvme1n1
```

The system reported:

```text
mkfs.xfs: /dev/nvme1n1 appears to contain an existing filesystem (xfs).
mkfs.xfs: Use the -f option to force overwrite.
```

Since the volume already contained an XFS filesystem, I did **not** use `-f`, because forcing a format would overwrite the existing filesystem.

---

# 📁 Step 6 – Create the Mount Point

I created a directory to use as the mount point:

```bash
sudo mkdir -p /data
```

The `/data` directory was selected as the location where the additional storage would be accessible.

---

# 💾 Step 7 – Mount the EBS Volume

I mounted the existing XFS filesystem:

```bash
sudo mount /dev/nvme1n1 /data
```

The command completed successfully.

---

# 📊 Step 8 – Verify the Mounted Filesystem

I verified the mounted filesystem using:

```bash
df -h
```

The output showed:

```text
Filesystem        Size  Used Avail Use% Mounted on
/dev/nvme0n1p1    8.0G  2.0G  6.0G  26% /
/dev/nvme1n1      5.0G   68M  4.9G   2% /data
```

This confirmed that the additional 5 GiB EBS volume was successfully mounted at:

```text
/data
```

---

# 📷 Mount Verification

<img width="1835" height="811" alt="image" src="https://github.com/user-attachments/assets/2140cbea-2d12-4b76-b9cb-c6ec3daadf33" />


The `df -h` output confirmed:

```text
/dev/nvme1n1      5.0G   68M   4.9G   2% /data
```

This verified that the additional storage was available to the operating system.

---

# 🔍 Step 9 – Verify Block Device and Filesystem

I used `lsblk` to verify the device and mount point:

```bash
lsblk
```

Output:

```text
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme1n1       259:0    0   5G  0 disk /data
nvme0n1       259:1    0   8G  0 disk
├─nvme0n1p1   259:2    0   8G  0 part /
├─nvme0n1p127 259:3    0   1M  0 part
└─nvme0n1p128 259:4    0  10M  0 part /boot/efi
```

I also verified the filesystem information:

```bash
lsblk -f
```

The output showed:

```text
NAME     FSTYPE UUID                                  FSAVAIL FSUSE% MOUNTPOINTS
nvme1n1  xfs    3edfc932-89f0-4e6d-a3a8-459d97db74a0  4.9G       1% /data
nvme0n1
└─nvme0n1p1 xfs 23799fd8-bbb7-4eac-805e-0e9b0b9c2389 5.9G      25% /
```

This confirmed:

* Device: `/dev/nvme1n1`
* Filesystem: `xfs`
* Mount point: `/data`
* Available space: approximately `4.9G`

---

# 📷 Final Linux Verification

<img width="1172" height="677" alt="image" src="https://github.com/user-attachments/assets/339c0594-6df1-4356-a9d9-4258a1701e47" />


The final verification confirmed that the additional EBS volume was:

```text
5 GiB
   │
   ▼
/dev/nvme1n1
   │
   ▼
XFS filesystem
   │
   ▼
/data
```

---

# ☁️ Step 10 – Final AWS Console Verification

I returned to the Amazon EC2 EBS Volume details page to confirm the final state.

The volume showed:

```text
Volume ID: vol-0823995c480113965
Size: 5 GiB
Type: gp3
Volume State: In-use
Attached Resource: Database-server
Encryption: Encrypted
KMS Key Alias: aws/ebs
Status Check: Okay
```

---

# Final EBS Volume Status

The AWS Console confirmed that the volume was:

* Attached to the `Database-server`
* In the `In-use` state
* 5 GiB gp3
* Encrypted
* Using the `aws/ebs` KMS key
* Passing its status check

---

# 🧪 Verification

The following components were successfully verified:

| Component                  | Status           |
| -------------------------- | ---------------- |
| EBS Volume Created         | ✅                |
| Volume Type                | ✅ gp3            |
| Volume Size                | ✅ 5 GiB          |
| EBS Encryption             | ✅ Enabled        |
| KMS Key                    | ✅ aws/ebs        |
| Volume Attached            | ✅                |
| Linux Device Detected      | ✅ `/dev/nvme1n1` |
| Filesystem                 | ✅ XFS            |
| Mount Point                | ✅ `/data`        |
| Storage Visible in `df -h` | ✅                |
| Storage Visible in `lsblk` | ✅                |
| Volume State               | ✅ In-use         |

---

# 📝 Commands Used

The main Linux commands used during the implementation were:

```bash
lsblk
```

```bash
sudo blkid /dev/nvme1n1
```

```bash
sudo mkdir -p /data
```

```bash
sudo mount /dev/nvme1n1 /data
```

```bash
df -h
```

```bash
lsblk -f
```

The following command was also attempted with an incorrect filesystem type:

```bash
sudo mkfs -t xfx /dev/nvme1n1
```

The correct filesystem type is:

```bash
sudo mkfs -t xfs /dev/nvme1n1
```

However, because the device already contained an XFS filesystem, I did not force a reformat.

---

# 🔍 Troubleshooting

| Issue                                                       | Resolution                                               |
| ----------------------------------------------------------- | -------------------------------------------------------- |
| Instance did not initially show the volume                  | Verified the volume and instance Availability Zone       |
| `mkfs -t xfx` failed                                        | Corrected the filesystem type from `xfx` to `xfs`        |
| `mkfs.xfs` reported an existing filesystem                  | Used `blkid` to confirm XFS and avoided forcing a format |
| EBS device appeared as `/dev/nvme1n1` instead of `/dev/sdf` | Verified Nitro/NVMe device mapping using `lsblk`         |
| `lsbk` command failed                                       | Corrected the typo and used `lsblk`                      |
| `df -h` initially did not show the new volume               | Mounted `/dev/nvme1n1` to `/data`                        |

---

# 🧠 Key Learnings

Through this lab, I learned:

* How to provision additional storage using Amazon EBS
* How to select the `gp3` EBS volume type
* How to enable EBS encryption
* How AWS KMS is used with encrypted EBS volumes
* How to attach an EBS volume to an existing EC2 instance
* How EC2 Nitro instances expose EBS volumes through NVMe device names
* How to identify filesystems using `blkid`
* How to work with the XFS filesystem
* How to create and configure a mount point
* How to verify storage using `df -h` and `lsblk`
* Why forcing `mkfs` on a volume containing an existing filesystem can destroy data

---

# ⚠️ Important Note

The additional EBS volume was mounted manually using:

```bash
sudo mount /dev/nvme1n1 /data
```

The lab verified the volume successfully at `/data`.

This implementation did not include a completed `/etc/fstab` configuration for persistent mounting across reboots.

---

---

# 🎯 Conclusion

Successfully provisioned and configured an additional **5 GiB gp3 Amazon EBS volume** for the existing `Database-server` EC2 instance.

The volume was encrypted using the AWS-managed `aws/ebs` KMS key, attached to the EC2 instance, detected by Amazon Linux 2023 as `/dev/nvme1n1`, and mounted using the existing XFS filesystem at `/data`.

This lab provided hands-on experience with **Amazon EBS provisioning, volume attachment, Linux filesystem management, NVMe device mapping, mounting, and EBS encryption**.

The additional storage was successfully made available to the existing EC2 database server without replacing or modifying its 8 GiB root volume.
