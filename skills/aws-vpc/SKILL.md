---
name: aws-vpc
description: >
  Use this skill for AWS VPC and networking — understanding default VPCs, subnets,
  security groups, internet gateways, and basic network architecture. Covers Learner
  Lab networking constraints: use default VPC, avoid NAT Gateways (expensive), and
  work within the pre-existing network setup. Good for EC2, Lambda, and RDS networking.
---

# AWS VPC (Virtual Private Cloud)

## What is a VPC?

A VPC is your own private network in AWS. All resources (EC2, RDS, Lambda in VPC, etc.)
live inside a VPC. In Learner Lab, a **default VPC** is pre-created for you — use it.

## Core Concepts

| Concept | Description |
|---------|-------------|
| **VPC** | Your isolated network (CIDR: e.g. `172.31.0.0/16`) |
| **Subnet** | Sub-division of VPC, tied to one Availability Zone |
| **Public Subnet** | Has a route to the Internet Gateway — EC2 here gets public IPs |
| **Private Subnet** | No direct internet access — needs NAT Gateway (expensive!) |
| **Internet Gateway** | Connects your VPC to the internet |
| **Security Group** | Stateful firewall at the instance level |
| **NACL** | Stateless firewall at the subnet level (advanced) |
| **Route Table** | Rules for where network traffic goes |

---

## Using the Default VPC (Recommended for Learner Lab)

```bash
# Get the default VPC ID
aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text

# Get all subnets in the default VPC
aws ec2 describe-subnets \
  --filters "Name=default-for-az,Values=true" \
  --query "Subnets[*].[SubnetId,AvailabilityZone,CidrBlock]" \
  --output table

# Get the Internet Gateway attached to the default VPC
aws ec2 describe-internet-gateways \
  --filters "Name=attachment.vpc-id,Values=<VPC_ID>" \
  --query "InternetGateways[0].InternetGatewayId" \
  --output text
```

## Security Groups

Security groups are **stateful** firewalls. If you allow inbound traffic on port 80,
the response traffic is automatically allowed out.

```bash
# Create a security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id <VPC_ID>

# Allow inbound HTTP (port 80)
aws ec2 authorize-security-group-ingress \
  --group-id <SG_ID> \
  --protocol tcp --port 80 --cidr 0.0.0.0/0

# Allow inbound HTTPS (port 443)
aws ec2 authorize-security-group-ingress \
  --group-id <SG_ID> \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow SSH only from your IP
aws ec2 authorize-security-group-ingress \
  --group-id <SG_ID> \
  --protocol tcp --port 22 --cidr $(curl -s ifconfig.me)/32

# List security group rules
aws ec2 describe-security-groups --group-ids <SG_ID>
```

## Common Security Group Patterns

```yaml
# Web Server
Inbound: 80 (HTTP) from 0.0.0.0/0
Inbound: 443 (HTTPS) from 0.0.0.0/0
Inbound: 22 (SSH) from your-ip/32
Outbound: All traffic

# Database (only accessible from web server SG)
Inbound: 3306 (MySQL) from web-server-sg
Inbound: 5432 (PostgreSQL) from web-server-sg
Outbound: All traffic

# Lambda (if in VPC)
Outbound: All traffic (to reach other services)
```

## Checking Network Connectivity

```bash
# From inside an EC2 instance, test connectivity
curl -I http://example.com           # Test internet access
nc -zv <rds-endpoint> 3306           # Test DB connectivity
telnet <host> <port>                 # Another connectivity test
```

## ⚠️ Learner Lab Networking Notes

- **Use the default VPC** — don't create a new VPC (complex, error-prone for beginners)
- **Avoid NAT Gateways** — they cost ~$0.045/hr even when idle, quickly draining budget
- If you need a private subnet with internet access, put your resources in the **public subnet** instead (simpler for lab work)
- **Elastic IPs cost money** when not attached — release them after use:
  ```bash
  aws ec2 release-address --allocation-id eipalloc-XXXXXXXXXXXXXXXXX
  ```
- Security groups persist after instance termination — clean them up:
  ```bash
  aws ec2 delete-security-group --group-id sg-XXXXXXXXXXXXXXXXX
  ```

## VPC Endpoints (Free Alternative to NAT Gateway)

If you have a Lambda or EC2 in a private subnet that needs S3/DynamoDB access,
use a **VPC Gateway Endpoint** (free!) instead of a NAT Gateway:

```bash
# Create a free S3 VPC Endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id <VPC_ID> \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids <ROUTE_TABLE_ID>
```

---

## Documentation Links

- [VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Default VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
