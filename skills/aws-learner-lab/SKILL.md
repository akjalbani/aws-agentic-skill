---
name: aws-learner-lab
description: >
  Use this skill whenever working inside an AWS Academy Learner Lab environment.
  Provides critical constraints, workarounds, and best practices specific to
  Learner Lab sessions. Always load this skill before any other AWS skill when
  the user mentions "Learner Lab", "Academy", "vockey", or session-based credentials.
---

# AWS Academy Learner Lab

## What is AWS Learner Lab?

AWS Academy Learner Lab is a sandbox AWS environment provided to students through
AWS Academy institutions. It gives you a real AWS account with a pre-configured
IAM role (`LabRole` / `vockey`), a budget limit, and a time-limited session.

## ⚠️ Critical Constraints — Read Before Building Anything

### IAM Restrictions
- **You CANNOT create IAM users** — the lab blocks this
- **You CANNOT create new IAM roles** — use the pre-existing `LabRole`
- **You CANNOT attach arbitrary policies** — only use what's pre-attached to LabRole
- When a service asks "Choose or create a role", always select the existing **LabRole**
- ARN format for LabRole: `arn:aws:iam::<your-account-id>:role/LabRole`

### Credentials & Sessions
- Credentials expire when your lab session ends (typically after 4 hours)
- To get credentials: click **"AWS Details"** in the Learner Lab panel → **"Show"** next to AWS CLI
- Copy the three values into `~/.aws/credentials`:
  ```ini
  [default]
  aws_access_key_id = ASIA...
  aws_secret_access_key = ...
  aws_session_token = ...
  ```
- **Always set a region** — the lab defaults to `us-east-1`:
  ```bash
  aws configure set region us-east-1
  ```
- Re-run `aws sts get-caller-identity` to confirm credentials are valid

### Budget & Service Limits
- You have a **$50–$100 USD credit budget** — do not exceed it
- **Stop all resources** at the end of each session (EC2, RDS, SageMaker especially)
- Services with ongoing costs: EC2 instances, RDS, NAT Gateways, Elastic IPs
- Free-tier-safe services: S3 (small data), Lambda (first 1M requests), DynamoDB (25GB)

### Unsupported / Restricted Services
- ❌ Route 53 (DNS) — not available
- ❌ ACM Certificate Manager — limited
- ❌ Organizations / Control Tower — not available
- ❌ Marketplace AMIs (paid) — use free AMIs only
- ❌ Multi-region setups — stick to `us-east-1` unless instructed otherwise
- ⚠️ SageMaker — available but expensive; stop notebooks when not in use

### Region
- **Always use `us-east-1`** (N. Virginia) unless your lab explicitly says otherwise
- Some services (e.g. CloudFront, IAM) are global, not region-specific

---

## ✅ Best Practices for Learner Lab

### Start of Session
1. Start your lab session from the Academy portal
2. Wait for the status dot to turn **green**
3. Click **AWS** (the console link) or get CLI credentials from **AWS Details**
4. Verify: `aws sts get-caller-identity`

### End of Session
1. Terminate any EC2 instances you don't need
2. Delete RDS instances (they bill by the hour)
3. Remove NAT Gateways (expensive even when idle)
4. Release Elastic IPs (billed when unattached)
5. Empty and consider deleting S3 buckets with large data
6. Click **"End Lab"** in the portal

### CLI Setup Template
```bash
# After copying credentials from AWS Details panel:
export AWS_DEFAULT_REGION=us-east-1

# Verify it works:
aws sts get-caller-identity
aws s3 ls
```

### CloudFormation in Learner Lab
- Use CloudFormation to provision resources — it's the cleanest approach
- Always include a **DeletionPolicy** and clean up stacks after the lab
- Pass `LabRole` as the execution role for any stack that needs IAM permissions:
  ```yaml
  # In your stack or when deploying Lambda, pass:
  Role: !Sub "arn:aws:iam::${AWS::AccountId}:role/LabRole"
  ```

---

## Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `ExpiredTokenException` | Lab session ended | Restart lab, refresh credentials |
| `AccessDenied` creating IAM role | Lab blocks IAM role creation | Use existing `LabRole` |
| `UnauthorizedOperation` | LabRole lacks permission | Check if the service is supported in lab |
| `InvalidClientTokenId` | Wrong credentials format | Re-copy all 3 values from AWS Details |
| `OptInRequired` | Service not enabled | Some services need manual opt-in in console |

---

## Documentation Links

- [AWS Academy Learner Lab Guide](https://awsacademy.instructure.com/)
- [AWS CLI Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [IAM Roles (concepts)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [AWS Free Tier](https://aws.amazon.com/free/)
