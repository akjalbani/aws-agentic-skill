---
name: aws-ec2
description: >
  Use this skill for Amazon EC2 — launching instances, choosing instance types,
  configuring security groups, connecting via SSH, managing key pairs, and
  stopping/terminating instances. Includes Learner Lab constraints: always use
  LabRole instance profile, free-tier instance types, and stop instances to save budget.
---

# Amazon EC2 (Elastic Compute Cloud)

## What is EC2?

EC2 gives you virtual machines (instances) in the cloud. You pick the OS, CPU, memory,
and storage. Free tier: 750 hours/month of `t2.micro` or `t3.micro`.

## Core Concepts

- **Instance**: A virtual machine
- **AMI**: Amazon Machine Image — the OS template (Amazon Linux 2023, Ubuntu, etc.)
- **Instance Type**: CPU + memory size (t3.micro, t3.small, m5.large, etc.)
- **Key Pair**: SSH credentials to connect to your instance
- **Security Group**: Firewall rules (inbound/outbound)
- **Instance Profile**: IAM role attached to the instance — use **LabRole** in Learner Lab

---

## Launching an Instance (Console)

1. EC2 → **Launch Instance**
2. Name your instance
3. AMI: Choose **Amazon Linux 2023** (free, fast, AWS-native)
4. Instance type: **t3.micro** (free tier eligible)
5. Key pair: Create new or use existing (save the `.pem` file!)
6. Security group: Allow SSH (port 22) from your IP
7. Advanced → IAM instance profile → select **LabRole**
8. Click **Launch**

## Launching an Instance (CLI)

```bash
# First, find a free Amazon Linux 2023 AMI in us-east-1
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-*-x86_64" \
            "Name=state,Values=available" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text

# Create a key pair (saves private key to file)
aws ec2 create-key-pair \
  --key-name my-key \
  --query 'KeyMaterial' \
  --output text > my-key.pem
chmod 400 my-key.pem

# Get your default VPC and a subnet
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" --output text)
SUBNET_ID=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
  --query "Subnets[0].SubnetId" --output text)

# Create a security group
SG_ID=$(aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group" \
  --vpc-id $VPC_ID \
  --query 'GroupId' --output text)

# Allow SSH from anywhere (restrict to your IP in production)
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp --port 22 --cidr 0.0.0.0/0

# Launch instance
aws ec2 run-instances \
  --image-id ami-XXXXXXXXXXXXXXXXX \  # Replace with AMI ID from above
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids $SG_ID \
  --subnet-id $SUBNET_ID \
  --iam-instance-profile Name=LabRole \
  --associate-public-ip-address \
  --count 1
```

## Connecting via SSH

```bash
# Get the public IP/DNS of your instance
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress,State.Name]" \
  --output table

# Connect (Amazon Linux 2023)
ssh -i my-key.pem ec2-user@<PUBLIC_IP>

# Connect (Ubuntu)
ssh -i my-key.pem ubuntu@<PUBLIC_IP>
```

## Common Instance Management

```bash
# Stop an instance (pauses billing for compute; storage still billed)
aws ec2 stop-instances --instance-ids i-XXXXXXXXXXXXXXXXX

# Start a stopped instance
aws ec2 start-instances --instance-ids i-XXXXXXXXXXXXXXXXX

# Terminate (permanently delete) an instance
aws ec2 terminate-instances --instance-ids i-XXXXXXXXXXXXXXXXX

# List all running instances
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key=='Name'].Value|[0]]" \
  --output table
```

## User Data (Bootstrap Script)

Run commands automatically at launch:

```bash
aws ec2 run-instances \
  --image-id ami-XXXXXXXXXXXXXXXXX \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids $SG_ID \
  --user-data '#!/bin/bash
yum update -y
yum install -y python3 git
pip3 install boto3
echo "Setup complete" > /tmp/setup.log'
```

## Security Group Rules Reference

| Use case | Protocol | Port |
|----------|----------|------|
| SSH | TCP | 22 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| Custom app | TCP | your port |
| All traffic | All | All |

## ⚠️ Learner Lab Notes

- **Stop instances when not in use** — they bill by the hour (~$0.0104/hr for t3.micro)
- Use **t3.micro** or **t2.micro** only — larger types drain your budget fast
- Always attach **LabRole** as the instance profile (needed to call other AWS services)
- Key pair `.pem` files are shown **only once** — save them immediately
- Elastic IPs cost money when not attached — release them when done
- Instance stops when the lab session ends; data on EBS root volume persists

---

## Documentation Links

- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Amazon Linux 2023](https://docs.aws.amazon.com/linux/al2023/ug/what-is-amazon-linux.html)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/on-demand/)
- [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
