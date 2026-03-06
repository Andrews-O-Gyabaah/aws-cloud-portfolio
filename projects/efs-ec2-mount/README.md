PROJECT 2
Amazon EFS: Shared File System with EC2 & Performance Monitoring

## Overview
Created and configured an Amazon Elastic File System (EFS), mounted it to an EC2 instance, and monitored its performance using CloudWatch. This project demonstrates how EFS provides scalable, shared network storage for cloud workloads.

This project was built as part of my AWS Solutions Architect Associate (SAA) studies.

---

## Architecture

```
VPC (Lab VPC)
├── Security Group (EFS Mount Target)
│   └── Inbound: TCP port 2049 (NFS) from EFSClient security group
│
├── EC2 Instance (Amazon Linux)
│   └── Mounted EFS at /home/ec2-user/efs
│
└── EFS File System (My First EFS File System)
    ├── Mount Target — Availability Zone 1
    └── Mount Target — Availability Zone 2
```

---

## What I Built

### 1. Security Group for EFS Access
Created a dedicated security group (`EFS Mount Target`) allowing inbound NFS traffic on TCP port 2049 from the EFSClient security group. This restricts file system access to authorized EC2 instances only.

### 2. EFS File System
Created an EFS file system in the Lab VPC with:
- Automatic backups disabled (for lab purposes)
- Custom mount targets per Availability Zone
- EFS Mount Target security group attached to each mount target

### 3. Mounting EFS to EC2
Connected to the EC2 instance via AWS Systems Manager Session Manager (no SSH key required). Installed `amazon-efs-utils` and mounted the EFS file system using NFSv4.1:

```bash
sudo yum install -y amazon-efs-utils
sudo mkdir efs
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport <fs-id>.efs.<region>.amazonaws.com:/ efs
```

Verified the mount with:
```bash
sudo df -hT
```

### 4. Performance Benchmarking with fio
Ran a synthetic I/O benchmark using Flexible IO (fio) to test write performance:

```bash
sudo fio --name=fio-efs --filesize=10G --filename=./efs/fio-efs-test.img --bs=1M --nrfiles=1 --direct=1 --sync=0 --rw=write --iodepth=200 --ioengine=libaio
```

### 5. Performance Monitoring with CloudWatch
Monitored EFS performance using two CloudWatch metrics:
- **PermittedThroughput** — peaked at ~5.3 GB/s
- **DataWriteIOBytes** — peaked at ~7G over a 1-minute period, confirming strong write throughput

---

## Key Findings

- EFS file system size shows as **8.0 exabytes** — this reflects EFS's elastic nature; it scales automatically as data is added
- Write throughput peaked at approximately **7G/minute (~116 MB/s)** during the fio benchmark
- EFS bursts to high throughput for short periods, then returns to baseline — ideal for spiky file-based workloads

---

## Lessons Learned

- **Session Manager vs SSH** — Connected to EC2 without an SSH key pair using AWS Systems Manager Session Manager. Simpler and more secure for managed instances.
- **NFS port 2049** — EFS uses NFS protocol, which requires TCP port 2049 open in the security group. Easy to misconfigure if you forget this.
- **Mount targets per AZ** — EFS requires a mount target in each Availability Zone where EC2 instances will access it. Important for multi-AZ architectures.
- **EFS throughput scales with storage** — Baseline is 50 MiB/s per TiB. All file systems can burst to at least 100 MiB/s regardless of size.

---

## AWS Services Used
- Amazon EFS
- Amazon EC2
- Amazon CloudWatch
- AWS Systems Manager (Session Manager)
- Amazon VPC / Security Groups

## SAA Domains Covered
- Design Resilient Architectures (multi-AZ mount targets)
- Design High-Performing Architectures (EFS throughput & bursting)
- Design Secure Architectures (security groups, Session Manager)

---

## Screenshots
See the `/screenshots` folder for evidence of each configuration step.

---

## Resources
- [Amazon EFS Documentation](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
- [EFS Performance Docs](https://docs.aws.amazon.com/efs/latest/ug/performance.html)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
