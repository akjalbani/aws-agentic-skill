---
name: aws-s3
description: >
  Use this skill for Amazon S3 — creating buckets, uploading/downloading objects,
  configuring permissions, static website hosting, lifecycle policies, versioning,
  presigned URLs, and S3 event notifications. Covers common pitfalls like public
  access blocks and ACL deprecation. Safe to use in AWS Learner Lab.
---

# Amazon S3 (Simple Storage Service)

## What is S3?

S3 is AWS's object storage service. Store any file (object) in a container (bucket).
Infinitely scalable, highly durable (11 nines). Free tier: 5GB storage, 20K GET, 2K PUT.

## Core Concepts

- **Bucket**: Top-level container (globally unique name)
- **Object**: A file + metadata, identified by a key (path-like string)
- **Key**: Full "path" of the object, e.g. `images/2024/photo.jpg`
- **Region**: Buckets are regional — always specify `us-east-1` in Learner Lab

---

## Creating a Bucket

```bash
# Create bucket (us-east-1 is the default; no --create-bucket-configuration needed)
aws s3 mb s3://my-unique-bucket-name --region us-east-1

# List all your buckets
aws s3 ls
```

> ⚠️ Bucket names are **globally unique across all AWS accounts**. Use a prefix like your initials or account ID.

## Uploading & Downloading

```bash
# Upload a single file
aws s3 cp myfile.txt s3://my-bucket/

# Upload entire folder (recursive)
aws s3 cp ./local-folder/ s3://my-bucket/folder/ --recursive

# Download a file
aws s3 cp s3://my-bucket/myfile.txt ./

# Sync local folder to S3 (only changed files)
aws s3 sync ./dist/ s3://my-bucket/ --delete
```

## Listing & Deleting

```bash
# List objects in a bucket
aws s3 ls s3://my-bucket/
aws s3 ls s3://my-bucket/folder/ --recursive

# Delete a single object
aws s3 rm s3://my-bucket/myfile.txt

# Delete all objects (empty the bucket)
aws s3 rm s3://my-bucket/ --recursive

# Delete the bucket (must be empty first)
aws s3 rb s3://my-bucket
```

## Permissions & Public Access

**By default, all buckets block public access.** To host a public website or share files:

```bash
# Step 1: Disable the public access block
aws s3api put-public-access-block \
  --bucket my-bucket \
  --public-access-block-configuration \
    "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"

# Step 2: Apply a bucket policy to allow public reads
aws s3api put-bucket-policy --bucket my-bucket --policy '{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}'
```

## Static Website Hosting

```bash
# Enable website hosting
aws s3 website s3://my-bucket/ \
  --index-document index.html \
  --error-document error.html

# URL format: http://my-bucket.s3-website-us-east-1.amazonaws.com
```

After enabling, also apply the public read bucket policy above.

## Presigned URLs (Temporary Access)

```bash
# Generate a URL that expires in 1 hour (3600 seconds)
aws s3 presign s3://my-bucket/private-file.pdf --expires-in 3600
```

## Versioning

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# List versions of an object
aws s3api list-object-versions --bucket my-bucket --prefix myfile.txt
```

## S3 Event Notifications → Lambda

In the console: Bucket → Properties → Event notifications → Create notification
- Event type: `PUT` (uploads)
- Destination: Lambda function

Or via CLI:
```bash
aws s3api put-bucket-notification-configuration \
  --bucket my-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:ACCOUNT_ID:function:my-function",
      "Events": ["s3:ObjectCreated:*"]
    }]
  }'
```

## Using S3 from Python (boto3)

```python
import boto3

s3 = boto3.client('s3', region_name='us-east-1')

# Upload
s3.upload_file('local.txt', 'my-bucket', 'remote.txt')

# Download
s3.download_file('my-bucket', 'remote.txt', 'local-copy.txt')

# List objects
response = s3.list_objects_v2(Bucket='my-bucket')
for obj in response.get('Contents', []):
    print(obj['Key'])

# Generate presigned URL
url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'file.pdf'},
    ExpiresIn=3600
)
```

## ⚠️ Learner Lab Notes

- S3 is safe to use — very cheap and free-tier friendly
- Empty buckets before ending your lab (storage persists across sessions)
- ACLs are deprecated — use bucket policies for permissions instead
- Cross-region replication is not needed in Learner Lab

---

## Documentation Links

- [S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [S3 CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/s3/)
- [Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
- [Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [S3 Pricing](https://aws.amazon.com/s3/pricing/)
