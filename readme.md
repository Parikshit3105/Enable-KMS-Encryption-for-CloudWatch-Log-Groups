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

### Deploy Lambda Function
1. Create a new AWS Lambda function.
2. Upload the `lambda_function.py` script.
   ```sh
import json
import boto3

def lambda_handler(event, context):
    client = boto3.client('logs')
    
    # Extract parameters from the event payload
    action = event.get('action', 'enable')  # 'enable' or 'disable'
    kms_key_arn = event.get('kms_key_arn')  # KMS Key ARN (required for enabling)
    log_group_name = event.get('log_group_name')  # Optional: Specific log group
    
    try:
        if log_group_name:
            # Apply to a specific log group
            update_log_group_encryption(client, log_group_name, action, kms_key_arn)
        else:
            # Apply to all log groups
            paginator = client.get_paginator('describe_log_groups')
            for page in paginator.paginate():
                for log_group in page.get('logGroups', []):
                    update_log_group_encryption(client, log_group['logGroupName'], action, kms_key_arn)
                    
        return {
            'statusCode': 200,
            'body': json.dumps(f"KMS encryption {action}d successfully.")
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(str(e))
        }

def update_log_group_encryption(client, log_group_name, action, kms_key_arn):
    if action == 'enable' and kms_key_arn:
        client.associate_kms_key(logGroupName=log_group_name, kmsKeyId=kms_key_arn)
        print(f"Enabled KMS encryption for {log_group_name} with key {kms_key_arn}")
    elif action == 'disable':
        client.disassociate_kms_key(logGroupName=log_group_name)
        print(f"Disabled KMS encryption for {log_group_name}")
    else:
        raise ValueError("Invalid action or missing KMS Key for enabling encryption.")

```
4. Assign the IAM role with the necessary permissions.
5. Set the Lambda timeout to at least **30 seconds**.
6. Deploy and test using the sample event payloads below.

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

