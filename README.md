# AWS Lambda: S3SecurityAuditor

This directory contains the Python 3.12 AWS Lambda function that audits Amazon S3 buckets, logs events to Amazon DynamoDB, publishes alerts to Amazon SNS, and drives the physical LED lamp via AWS IoT Core.

## Execution Flow
1. **Trigger**: AWS EventBridge (every 5 minutes or upon S3 `PutBucketPolicy` / `PutBucketAcl` events) or direct invocation from the web dashboard.
2. **Audit**: Calls S3 API (`GetPublicAccessBlock`, `GetBucketPolicyStatus`, `GetBucketAcl`).
3. **Detection**: If any bucket allows unauthenticated public access:
   - Sets physical LED to `BLINKING` via MQTT on `security/alert/led`.
   - Records security event in DynamoDB `SecurityEvents`.
   - Fires high-priority SNS push notification.
4. **Resolution**: If all buckets are private:
   - Sets physical LED to `OFF`.
   - Logs `SECURE` status.

## Required IAM Policy for Lambda Execution Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "s3:GetBucketPolicyStatus",
        "s3:GetPublicAccessBlock",
        "s3:GetBucketAcl",
        "s3:GetBucketPolicy"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:Scan",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/SecurityEvents"
    },
    {
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iot:Publish"
      ],
      "Resource": "arn:aws:iot:*:*:topic/security/alert/led"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

## Sample Test Event (JSON for AWS Console)

```json
{
  "targetBucket": "demo-bucket"
}
```
