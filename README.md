# Automating EC2 Start/Stop with AWS Lambda & EventBridge
## This project helps optimize cloud costs by shutting down instances during non-working hours and restarting them when needed.
I built an automation to optimize EC2 usage by stopping and starting instances on a schedule.
The system ensures that EC2 instances are stopped automatically during idle periods (like nights or weekends) and started again during active hours.

## Key Steps:
       1.	Provisioned an EC2 Instance – Created an EC2 instance to demonstrate the automation.
       2.	Configured IAM Role & Policies – Assigned an IAM role with least-privilege permissions to allow Lambda to start/stop EC2 securely.
       3.	Configured Lambda Function – Wrote a Python-based Lambda function to handle EC2 start/stop logic.
       4.	Integrated with EventBridge Scheduler – Set up cron-based EventBridge rules to trigger Lambda at specific times (e.g., stop at 10 PM, start at 8 AM).
       5.	Verify the Lambda Function & EventBridge Rule – Tested the workflow to confirm EC2 stops/starts as expected and verified EventBridge rule execution.
	
## Use Cases & Cost Optimization:
•	Reduce EC2 running hours by up to 70–80% during inactive times.
•	Development or testing environments that don’t need 24/7 uptime.
•	Improved operational efficiency with hands-free instance management.

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

### 1. Provisioned an EC2 Instance
Launch an EC2 instance you want to manage automatically.  
Tag it appropriately (e.g., `AutoStop = true`) so the Lambda can identify it.

### 2. Configure IAM Role & Policies. 
Create Policy with existing Policy with the below policy.
Create Role and Attach the following policy to your role:

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
Search for Lambda and Create a function.
Select the Runtime as Python 3.9. 
Use an existing role chooses the role we created.
Under Code, Replace the existing code with the below 

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


### 4. Integrated with EventBridge Scheduler

Use EventBridge (CloudWatch Events) to schedule Lambda invocations.

#### Example cron expressions:

| Purpose | Cron (UTC) | Description |
|----------|------------|-------------|
| Stop instances | `cron(30 16 * * ? *)` | Runs every day at **10:00 PM IST** |

*(Note: Adjust the UTC time based on your local time zone.)*

### 5. Verify the Lambda Function & EventBridge Rule 
refresh the EC2 Console and you can EC2 Instance has Stopped Automatically.

--
##  Author
**Saif Uddin**  


---
