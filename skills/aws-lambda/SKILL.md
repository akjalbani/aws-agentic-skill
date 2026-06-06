---
name: aws-lambda
description: >
  Use this skill for AWS Lambda — creating functions, configuring triggers,
  managing permissions, setting environment variables, packaging dependencies,
  and deploying. Covers Python, Node.js, and container-based Lambda. Also covers
  Lambda in AWS Learner Lab (always use LabRole, not new IAM roles).
---

# AWS Lambda

## What is Lambda?

AWS Lambda is a serverless compute service — you write a function, upload it,
and AWS runs it on-demand. You pay only for execution time (first 1M requests/month free).

## Core Concepts

- **Function**: Your code + runtime + config
- **Handler**: Entry point (e.g. `lambda_function.lambda_handler`)
- **Trigger**: What invokes the function (API Gateway, S3, EventBridge, etc.)
- **Execution Role**: IAM role the function assumes — in Learner Lab, always use **LabRole**
- **Layers**: Shared libraries packaged separately

---

## Creating a Lambda Function (Console)

1. Go to **Lambda → Create function**
2. Choose **Author from scratch**
3. Set Runtime (Python 3.12, Node.js 20.x, etc.)
4. **Execution role**: Choose **"Use an existing role"** → select `LabRole`
5. Click **Create function**
6. Write/paste your code in the inline editor or upload a `.zip`

## Creating a Lambda Function (CLI)

```bash
# Package your code
zip function.zip lambda_function.py

# Deploy (use LabRole ARN)
aws lambda create-function \
  --function-name my-function \
  --runtime python3.12 \
  --role arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/LabRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --region us-east-1

# Update code after changes:
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip
```

## Basic Python Handler Template

```python
import json

def lambda_handler(event, context):
    print("Event:", json.dumps(event))
    
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Hello from Lambda!'})
    }
```

## Environment Variables

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --environment "Variables={TABLE_NAME=my-table,ENV=prod}"
```

## Packaging Dependencies (Python)

```bash
# Install dependencies into a package/ folder
pip install requests -t package/
cp lambda_function.py package/
cd package && zip -r ../function.zip . && cd ..

# Deploy the zip
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip
```

## Common Triggers

| Trigger | Use case |
|---------|----------|
| API Gateway | HTTP REST/WebSocket endpoint |
| S3 | React to file uploads |
| DynamoDB Streams | React to DB changes |
| EventBridge (CloudWatch Events) | Scheduled jobs (cron) |
| SQS | Queue processing |

## Testing

```bash
# Invoke synchronously
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  response.json

cat response.json
```

## Viewing Logs

```bash
# Logs go to CloudWatch automatically
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/my-function

# Tail recent logs
aws logs tail /aws/lambda/my-function --follow
```

## ⚠️ Learner Lab Notes

- Always use existing `LabRole` — never create a new execution role
- Default timeout is 3 seconds — increase for longer tasks (max 15 min)
- Memory default is 128MB — increase for CPU-heavy tasks
- Lambda is free-tier friendly (1M free invocations/month)

---

## Documentation Links

- [Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Python Handler](https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html)
- [Lambda with S3](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)
- [Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
