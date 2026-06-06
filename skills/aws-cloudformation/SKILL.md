---
name: aws-cloudformation
description: >
  Use this skill for AWS CloudFormation — writing templates (YAML/JSON),
  deploying stacks, updating and deleting stacks, using parameters and outputs,
  and common resource types. Ideal for Learner Lab as it automates provisioning
  and makes cleanup easy. Always use LabRole as the stack execution role.
---

# AWS CloudFormation

## What is CloudFormation?

CloudFormation is AWS's Infrastructure as Code (IaC) service. You describe your
infrastructure in a YAML or JSON template, and CloudFormation creates/updates/deletes
all the resources for you as a single unit called a **stack**.

## Why Use CloudFormation in Learner Lab?

- Provision everything consistently with one command
- **Delete the entire stack** to clean up all resources at once
- Reproducible — re-run the template to rebuild everything
- Version control your infrastructure

---

## Template Structure

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: My stack description

Parameters:
  BucketName:
    Type: String
    Default: my-app-bucket

Resources:              # Required — your AWS resources
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      
  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: my-function
      Runtime: python3.12
      Role: !Sub "arn:aws:iam::${AWS::AccountId}:role/LabRole"
      Handler: index.handler
      Code:
        ZipFile: |
          def handler(event, context):
              return {"statusCode": 200, "body": "Hello!"}

Outputs:
  BucketArn:
    Value: !GetAtt MyBucket.Arn
    Export:
      Name: MyBucketArn
```

## Deploying a Stack

```bash
# Deploy (create or update)
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name my-stack \
  --region us-east-1 \
  --capabilities CAPABILITY_NAMED_IAM

# With parameter overrides
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name my-stack \
  --parameter-overrides BucketName=my-special-bucket \
  --capabilities CAPABILITY_NAMED_IAM
```

> ⚠️ `--capabilities CAPABILITY_NAMED_IAM` is required if your template references IAM resources (even just passing LabRole).

## Checking Stack Status

```bash
# List all stacks
aws cloudformation list-stacks \
  --query "StackSummaries[?StackStatus!='DELETE_COMPLETE'].[StackName,StackStatus]" \
  --output table

# Describe a stack (see outputs, parameters, status)
aws cloudformation describe-stacks --stack-name my-stack

# Watch events as a stack deploys
aws cloudformation describe-stack-events \
  --stack-name my-stack \
  --query "StackEvents[*].[Timestamp,ResourceType,ResourceStatus,ResourceStatusReason]" \
  --output table
```

## Deleting a Stack (Cleanup)

```bash
# Delete stack and ALL resources it created
aws cloudformation delete-stack --stack-name my-stack

# Wait for deletion to complete
aws cloudformation wait stack-delete-complete --stack-name my-stack
echo "Stack deleted!"
```

## Useful Pseudo Parameters

These are built-in variables you can use anywhere in a template:

| Pseudo Parameter | Value |
|-----------------|-------|
| `AWS::AccountId` | Your 12-digit account ID |
| `AWS::Region` | Current region (e.g. `us-east-1`) |
| `AWS::StackName` | Name of the current stack |

```yaml
Role: !Sub "arn:aws:iam::${AWS::AccountId}:role/LabRole"
```

## Intrinsic Functions Reference

| Function | Purpose | Example |
|----------|---------|---------|
| `!Ref` | Reference a parameter or resource | `!Ref MyBucket` |
| `!Sub` | String substitution | `!Sub "bucket-${AWS::AccountId}"` |
| `!GetAtt` | Get resource attribute | `!GetAtt MyBucket.Arn` |
| `!Join` | Join strings | `!Join ["-", ["prefix", !Ref Env]]` |
| `!If` | Conditional value | `!If [IsProd, value1, value2]` |

## Common Resource Types

| Resource | Type |
|----------|------|
| S3 Bucket | `AWS::S3::Bucket` |
| Lambda Function | `AWS::Lambda::Function` |
| DynamoDB Table | `AWS::DynamoDB::Table` |
| EC2 Instance | `AWS::EC2::Instance` |
| Security Group | `AWS::EC2::SecurityGroup` |
| API Gateway | `AWS::ApiGateway::RestApi` |
| SNS Topic | `AWS::SNS::Topic` |
| SQS Queue | `AWS::SQS::Queue` |

## Complete Working Example — S3 + Lambda

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: S3 bucket that triggers a Lambda function on upload

Resources:

  UploadBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "uploads-${AWS::AccountId}-${AWS::Region}"
      NotificationConfiguration:
        LambdaConfigurations:
          - Event: s3:ObjectCreated:*
            Function: !GetAtt ProcessorFunction.Arn

  ProcessorFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: s3-processor
      Runtime: python3.12
      Role: !Sub "arn:aws:iam::${AWS::AccountId}:role/LabRole"
      Handler: index.handler
      Timeout: 30
      Code:
        ZipFile: |
          import json
          def handler(event, context):
              for record in event['Records']:
                  bucket = record['s3']['bucket']['name']
                  key = record['s3']['object']['key']
                  print(f"New file: s3://{bucket}/{key}")
              return {"status": "ok"}

  LambdaPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref ProcessorFunction
      Action: lambda:InvokeFunction
      Principal: s3.amazonaws.com
      SourceArn: !GetAtt UploadBucket.Arn

Outputs:
  BucketName:
    Value: !Ref UploadBucket
  FunctionName:
    Value: !Ref ProcessorFunction
```

## ⚠️ Learner Lab Notes

- Always use `!Sub "arn:aws:iam::${AWS::AccountId}:role/LabRole"` for role ARNs
- Delete stacks before ending your lab session to avoid lingering costs
- If a stack gets stuck in `ROLLBACK_COMPLETE`, delete it and redeploy
- Use `--capabilities CAPABILITY_NAMED_IAM` every time to avoid deployment errors

---

## Documentation Links

- [CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [Resource Type Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
- [CloudFormation Pricing](https://aws.amazon.com/cloudformation/pricing/) (free!)
