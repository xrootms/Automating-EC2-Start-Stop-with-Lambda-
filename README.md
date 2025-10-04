# AWS Lambda EC2 Scheduler

Automate starting and stopping Amazon EC2 instances using AWS Lambda and Amazon EventBridge.  
This project helps optimize cloud costs by shutting down instances during non-working hours and restarting them when needed.

---

## 🧠 Project Overview

This automation solution is built using:
- **AWS Lambda** – Executes the start/stop logic.
- **Amazon EventBridge** – Triggers the Lambda function based on a schedule.
- **AWS IAM Role** – Grants Lambda permissions to manage EC2 and write logs to CloudWatch.

The system ensures that EC2 instances are stopped automatically during idle periods (like nights or weekends) and started again during active hours.

---

## 🏗️ Architecture Diagram

**EventBridge → Lambda → EC2 + CloudWatch**

```
+-------------------+
| Amazon EventBridge |
+---------+----------+
          |
          v
+-------------------+
| AWS Lambda        |
| - Start EC2       |
| - Stop EC2        |
+---------+----------+
          |
          v
+-------------------+
| Amazon EC2        |
+-------------------+
```

---

## ⚙️ Steps to Set Up

### 1. Create an EC2 Instance
Launch an EC2 instance you want to manage automatically.  
Tag it appropriately (e.g., `AutoStop = true`) so the Lambda can identify it.

### 2. Create an IAM Role for Lambda
Attach the following policy to your Lambda role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Start*",
        "ec2:Stop*",
        "ec2:DescribeInstanceStatus"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### 3. Configure the AWS Lambda Function

Example Python code:

```python
import boto3

region = 'us-east-1'
instances = ['i-0abcdef1234567890']  # Example instance ID
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    action = event.get('action')
    if action == 'stop':
        ec2.stop_instances(InstanceIds=instances)
        print("EC2 instances stopped")
    elif action == 'start':
        ec2.start_instances(InstanceIds=instances)
        print("EC2 instances started")
    else:
        print("No valid action specified")
```

---

### 4. Create EventBridge Rules

Use EventBridge (CloudWatch Events) to schedule Lambda invocations.

#### Example cron expressions:

| Purpose | Cron (UTC) | Description |
|----------|------------|-------------|
| Stop instances | `cron(30 16 * * ? *)` | Runs every day at **10:00 PM IST** |
| Start instances | `cron(30 3 * * ? *)` | Runs every day at **9:00 AM IST** |

*(Note: Adjust the UTC time based on your local time zone.)*

---

## 💰 Cost Optimization

By automating instance management:
- Reduce EC2 running hours by up to **70–80%** during inactive times.
- Minimize manual management effort.
- Maintain predictable uptime schedules.

---

## 🧾 Example Use Cases
- Development or testing environments that don’t need 24/7 uptime.
- Cost control for student or demo AWS accounts.
- Auto start/stop Jenkins or test servers on schedule.

---

## 🧩 Future Enhancements
- Add SNS notifications for status updates.
- Include DynamoDB or SSM Parameter Store for dynamic instance lists.
- Integrate with Terraform for infrastructure automation.

---

## ⚠️ Disclaimer

This project uses **example values only**.  
It does **not** include any AWS credentials, secrets, or private information.  
Always store your keys securely in **AWS Secrets Manager** or **AWS Systems Manager Parameter Store**.

---

## 👨‍💻 Author
**Saif Uddin**  
_AWS & DevOps Enthusiast_  
[GitHub Profile](https://github.com/) _(add your link here)_

---
