# Enable KMS Encryption for CloudWatch Log Groups

## Overview
This AWS Lambda function enables or disables KMS encryption for Amazon CloudWatch log groups in a given AWS region. Users can specify a KMS key, enable or disable encryption, and optionally apply the change to a specific log group.

## Features
- Enables KMS encryption for all or a specific CloudWatch log group.
- Allows disabling KMS encryption.
- Uses AWS Lambda and Boto3 for automation.
- Supports dynamic KMS key definition via payload.

## Prerequisites
- AWS account with appropriate IAM permissions.
- An existing KMS Key (for enabling encryption).
- AWS Lambda execution role with required permissions.

## IAM Policy for Lambda
Attach the following IAM policy to your Lambda execution role:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:DescribeLogGroups",
                "logs:AssociateKmsKey",
                "logs:DisassociateKmsKey"
            ],
            "Resource": "arn:aws:logs:*:*:log-group:*"
        },
        {
            "Effect": "Allow",
            "Action": "kms:DescribeKey",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "kms:ListAliases",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "kms:CreateGrant",
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": "true"
                }
            }
        }
    ]
}
```

## Deployment Steps
### 1. Create a GitHub Repository
1. Log in to GitHub and create a new repository.
2. Name the repository (e.g., `cloudwatch-kms-encryption`).
3. Initialize it with a `README.md` (or add later).

### 2. Clone and Upload Code
```sh
git clone https://github.com/YOUR_USERNAME/cloudwatch-kms-encryption.git
cd cloudwatch-kms-encryption
git add .
git commit -m "Initial commit"
git push origin main
```

### 3. Deploy Lambda Function
1. Create a new AWS Lambda function.
2. Upload the `lambda_function.py` script.
3. Assign the IAM role with the necessary permissions.
4. Set the Lambda timeout to at least **30 seconds**.
5. Deploy and test using the sample event payloads below.

## Usage
### Enable KMS for all log groups
```json
{
    "action": "enable",
    "kms_key_arn": "arn:aws:kms:region:account-id:key/key-id"
}
```

### Enable KMS for a specific log group
```json
{
    "action": "enable",
    "kms_key_arn": "arn:aws:kms:region:account-id:key/key-id",
    "log_group_name": "/aws/lambda/my-log-group"
}
```

### Disable KMS for all log groups
```json
{
    "action": "disable"
}
```

### Disable KMS for a specific log group
```json
{
    "action": "disable",
    "log_group_name": "/aws/lambda/my-log-group"
}
```

## License
This project is licensed under the MIT License.

