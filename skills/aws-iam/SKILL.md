---
name: aws-iam
description: >
  Use this skill for AWS IAM — understanding roles, policies, and permissions.
  Critical for Learner Lab: you cannot create IAM users or roles, only use the
  pre-existing LabRole. Covers reading and understanding policies, passing roles
  to services, and common IAM errors. Also covers boto3 with instance profiles.
---

# AWS IAM (Identity and Access Management)

## What is IAM?

IAM controls **who** can do **what** in your AWS account.
- **Users**: Human identities (cannot create in Learner Lab)
- **Roles**: Machine identities — assumed by services, EC2, Lambda, etc.
- **Policies**: JSON documents defining allowed/denied actions
- **Groups**: Collections of users (not usable in Learner Lab)

---

## ⚠️ Learner Lab IAM Constraints

| Action | Allowed? | Alternative |
|--------|----------|-------------|
| Create IAM user | ❌ No | Use LabRole |
| Create IAM role | ❌ No | Use LabRole |
| Attach policies to roles | ❌ No | LabRole already has permissions |
| Read/view existing policies | ✅ Yes | — |
| Pass LabRole to services | ✅ Yes | Standard approach |
| Create instance profiles | ❌ No | LabRole profile exists |

**The pre-existing role you always use is: `LabRole`**

```bash
# Get your account ID and form the LabRole ARN:
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo "LabRole ARN: arn:aws:iam::${ACCOUNT_ID}:role/LabRole"

# Verify what LabRole can do:
aws iam list-attached-role-policies --role-name LabRole
aws iam list-role-policies --role-name LabRole
```

---

## Understanding Policies

Policies are JSON documents. Every policy has:
- **Effect**: `Allow` or `Deny`
- **Action**: What API calls are allowed (e.g. `s3:GetObject`, `ec2:*`)
- **Resource**: Which resources the action applies to

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-bucket"
    }
  ]
}
```

## Common AWS Managed Policies (Pre-built)

| Policy Name | What it allows |
|-------------|----------------|
| `AmazonS3FullAccess` | All S3 operations |
| `AmazonDynamoDBFullAccess` | All DynamoDB operations |
| `AWSLambdaBasicExecutionRole` | Lambda → CloudWatch Logs |
| `AmazonEC2FullAccess` | All EC2 operations |
| `AdministratorAccess` | Everything (LabRole has this) |

## Passing a Role to a Service

When creating Lambda, EC2, ECS tasks, etc., you "pass" LabRole to the service.
This is done via `--role` (Lambda) or `--iam-instance-profile` (EC2).

```bash
# Lambda: pass LabRole as execution role
aws lambda create-function \
  --function-name my-fn \
  --role arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/LabRole \
  ...

# EC2: attach LabRole as instance profile
aws ec2 run-instances \
  --iam-instance-profile Name=LabRole \
  ...
```

## Using IAM Roles in Code (boto3)

When your code runs **on EC2 or Lambda with LabRole attached**, boto3 picks up
credentials automatically — no hardcoding needed:

```python
import boto3

# boto3 automatically uses the instance/function's role
s3 = boto3.client('s3')  # No credentials needed!
dynamodb = boto3.resource('dynamodb')

# To explicitly specify a region:
s3 = boto3.client('s3', region_name='us-east-1')
```

When running **locally** (not on EC2/Lambda), use the credentials from AWS Details:

```python
import boto3

session = boto3.Session(
    aws_access_key_id='ASIA...',
    aws_secret_access_key='...',
    aws_session_token='...',  # Required for Learner Lab
    region_name='us-east-1'
)
s3 = session.client('s3')
```

> 💡 **Best practice**: Don't hardcode credentials. Use environment variables or `~/.aws/credentials`.

## Common IAM Errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `AccessDenied` | Your role lacks the permission | Check if the service/action is supported with LabRole |
| `is not authorized to perform: iam:PassRole` | Can't pass a role | Use LabRole ARN, not a custom role |
| `NoCredentialProviders` | No valid credentials | Re-copy credentials from AWS Details panel |
| `ExpiredTokenException` | Session token expired | Restart lab session and refresh credentials |
| `InvalidClientTokenId` | Wrong access key | Re-copy all 3 credential values |

## Checking Your Current Identity

```bash
# Always do this to confirm who you're acting as
aws sts get-caller-identity
```

Expected output in Learner Lab:
```json
{
    "UserId": "AROA...:user...",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/LabRole/..."
}
```

---

## Documentation Links

- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [boto3 Credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html)
