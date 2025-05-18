# AWS EC2 Auto Scheduler

A cost-saving solution that automatically shuts down test and development EC2 instances during non-working hours and restarts them when needed.

## Overview

This repository contains a complete AWS solution for automatically managing EC2 instances based on tags. The scheduler:

- Stops tagged instances at 8:00 PM every day
- Starts tagged instances at 7:00 AM every day
- Sends detailed email notifications about affected instances
- Uses CloudFormation for easy deployment

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Deployment Guide](#deployment-guide)
- [Testing](#testing)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [License](#license)

## Features

- **Tag-Based Instance Selection**: Only manages instances with specific tags
- **Scheduled Operations**: Configured for business hours, but fully customizable
- **Detailed Email Notifications**: Comprehensive reports on affected instances
- **Error Handling**: Robust error management and reporting
- **CloudFormation Deployment**: Infrastructure as code for consistent deployments

## Architecture

![image](https://github.com/user-attachments/assets/bb84444c-f000-4721-9f7e-7bc35bc03343)

The solution consists of:

1. **Lambda Function**: Core logic that identifies and manages tagged EC2 instances
2. **SNS Topic**: Communication channel for email notifications
3. **EventBridge Rules**: Schedule triggers for automated execution
4. **IAM Role**: Securely scoped permissions for Lambda execution
5. **CloudWatch**: All function activity is logged

## Prerequisites

- AWS Account with administrative access
- Basic knowledge of AWS services (Lambda, EC2, CloudFormation)
- Git (for cloning this repository)
- AWS CLI (optional, for command-line deployment)

## Deployment Guide

### Method 1: Using the AWS Console

1. **Clone the Repository**
   ```bash
   git clone https://github.com/Curious4Tech/AWS_INFRA_ESSENTIALS.git
   cd AWS_INFRA_ESSENTIALS/serverless-ec2-manager
   ```

2. **Upload CloudFormation Template**
   - Open the [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation)
   - Click "Create stack" > "With new resources (standard)"

![image](https://github.com/user-attachments/assets/ae6fc008-d16d-4a0c-9bae-a9791353275c)


   - Select "Upload a template file"
   - Click "Choose file" and select `serverless-ec2-manager/ec2-scheduler.yaml` from the cloned repository
   - Click "Next"

![image](https://github.com/user-attachments/assets/8043cfa0-585e-44a2-81ec-81cd13bc69f9)

3. **Configure Stack Parameters**
   - Stack name: `EC2-Auto-Scheduler` (or your preferred name)
   - EmailAddress: Enter your email to receive notifications
   - TargetEnvironments: Leave as default (`test,dev`) or customize
   - Click "Next"

![image](https://github.com/user-attachments/assets/cd15dc44-303c-4ff5-992a-6c86e50f1807)

4. **Configure Stack Options**
   - Add any tags if desired (optional)
   - Check the box acknowledging IAM resource creation
   - Click "Next"

![image](https://github.com/user-attachments/assets/5da63d37-cf41-464d-b5a9-784f2b64f798)

5. **Review and Create**
   - Review all settings
   - Click "Submit"

![image](https://github.com/user-attachments/assets/3ff5e7ca-4973-498f-89c0-b63aeb0d5646)

6. **Confirm Subscription**
   - Check your email for an AWS SNS subscription confirmation
   - Click the "Confirm subscription" link in the email

![image](https://github.com/user-attachments/assets/810a0625-b85a-474a-83b3-284624dd4ea3)

### Method 2: Using AWS CLI

1. **Clone the Repository**
   ```bash
   git clone https://github.com/Curious4Tech/AWS_INFRA_ESSENTIALS.git
   cd AWS_INFRA_ESSENTIALS/serverless-ec2-manager
   ```

2. **Deploy Using AWS CLI**
   ```bash
   aws cloudformation create-stack \
     --stack-name EC2-Auto-Scheduler \
     --template-body file://serverless-ec2-manager/ec2-scheduler.yaml \
     --parameters ParameterKey=EmailAddress,ParameterValue=your-email@example.com \
     --capabilities CAPABILITY_IAM
   ```

3. **Confirm Subscription**
   - Check your email for an AWS SNS subscription confirmation
   - Click the "Confirm subscription" link in the email

![image](https://github.com/user-attachments/assets/d8b8cad1-6b76-4d0b-9c93-a5fa553167d0)

## Testing

### Method 1: Using the AWS Console

1. **Access the Lambda Function**
   - Open the [AWS Lambda Console](https://console.aws.amazon.com/lambda)
   - Find and select the function named `EC2-Auto-Scheduler`

![image](https://github.com/user-attachments/assets/47e5b07c-3f19-4cf8-8095-670f8884a519)

   - Go to the "Test" tab

2. **Create Test Events**
   
   Create a "Start" test event:
   - Click "Create new event"
   - Event name: `StartTest`
   - Enter the following JSON:
     ```json
     {
       "action": "start"
     }
     ```
   - Click "Save"

![image](https://github.com/user-attachments/assets/41be8e02-19cf-4aae-88b4-6ead106aad6e)

   Create a "Stop" test event:
   - Click "Create new event"
   - Event name: `StopTest`
   - Enter the following JSON:
     ```json
     {
       "action": "stop"
     }
     ```
   - Click "Save"

![image](https://github.com/user-attachments/assets/46b35723-6e7f-4e6c-99ab-b801ea0c48a8)

2. **Execute Tests**
   - Select the test event you want to run
   - Click "Test" button

![image](https://github.com/user-attachments/assets/7fe08a26-280d-4015-baf4-747bc7d91035)

   - View the execution results and check your email for notifications

![image](https://github.com/user-attachments/assets/47b3baf6-15e5-4af4-9b5d-9745f7deb148)

### Method 2: Using AWS CLI

1. **Invoke Lambda for Start Test**
   ```bash
   aws lambda invoke \
     --function-name EC2-Auto-Scheduler \
     --payload '{"action":"start"}' \
     output.txt
   ```

2. **Invoke Lambda for Stop Test**
   ```bash
   aws lambda invoke \
     --function-name EC2-Auto-Scheduler \
     --payload '{"action":"stop"}' \
     output.txt
   ```

## Customization

### Modify Schedule

Edit the CloudFormation template to change the schedule expressions:

```yaml
EC2ShutdownScheduleRule:
  Properties:
    ScheduleExpression: cron(0 20 * * ? *)  # 8 PM every day

EC2StartupScheduleRule:
  Properties:
    ScheduleExpression: cron(0 7 * * ? *)  # 7 AM every day
```

Common customizations:
- Weekdays only: `cron(0 7 ? * MON-FRI *)`
- Different hours: Change `7` or `20` to your preferred hour (in UTC)

### Change Target Tags

Modify the Lambda function code to update target tags:

```python
# Tags configuration - instances with these tags will be managed
TARGET_TAG_KEY = 'Environment'
TARGET_TAG_VALUES = ['test', 'dev']
```

After changing, update the Lambda function:
1. Go to the Lambda Console
2. Select your function
3. Edit the code
4. Save and test your changes

## Troubleshooting

### Common Issues

1. **No instances are being stopped/started**
   - Verify EC2 instances have the correct tags (`Environment: test` or `Environment: dev`)
   - Check CloudWatch Logs for the Lambda function
   - Ensure the Lambda IAM role has proper permissions

2. **Not receiving email notifications**
   - Confirm you've clicked the subscription confirmation link
   - Check spam/junk folder
   - Verify the Lambda function has permission to publish to SNS

3. **Lambda execution errors**
   - Check CloudWatch Logs for detailed error messages
   - Verify IAM permissions for EC2 and SNS actions

### Viewing Logs

1. Open the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch)
2. Go to "Logs" > "Log groups"
3. Find and select `/aws/lambda/EC2-Auto-Scheduler`

![image](https://github.com/user-attachments/assets/43c7d77a-0a5a-4d3e-bfd0-ffb808995c43)

4. View the latest log stream for execution details

![image](https://github.com/user-attachments/assets/bab82765-3fc8-4e78-aa4d-a150b614f597)

## Security Considerations

- The solution uses IAM roles with least privilege permissions
- Only instances explicitly tagged will be affected
- All actions are logged in CloudWatch Logs
- Consider using AWS KMS for encrypting SNS messages if sending sensitive information

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
