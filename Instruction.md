Phase 1 — Create IAM Roles
Step 1. Create EC2 IAM Role
Open
AWS Console → IAM → Roles → Create Role
Choose
AWS Service
Select
EC2
Attach policies
CloudWatchAgentServerPolicy
AmazonSSMManagedInstanceCore
Role Name
MonitorX-EC2-Role
Create Role.

Step 2. Create Lambda Role
IAM
Create Role
Choose
Lambda
Attach
AWSLambdaBasicExecutionRole
AmazonSNSFullAccess
Name
MonitorX-Lambda-Role

Phase 2 — Launch Ubuntu EC2
Launch
Ubuntu 24.04 LTS
Instance type
t2.micro
Attach IAM Role
MonitorX-EC2-Role
Allow
SSH (22)

HTTP (optional)
Launch instance.
SSH into server
ssh -i key.pem ubuntu@Public-IP

Phase 3 — Install CloudWatch Agent
Update packages
sudo apt update

sudo apt upgrade -y
Install
wget https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

sudo dpkg -i amazon-cloudwatch-agent.deb
Verify
amazon-cloudwatch-agent-ctl -a status

Phase 4 — Install CollectD
Memory and disk metrics require CollectD on Ubuntu.
sudo apt install collectd -y

sudo systemctl enable collectd

sudo systemctl start collectd
Verify
systemctl status collectd

Phase 5 — Create CloudWatch Agent Configuration
sudo mkdir -p /opt/aws/amazon-cloudwatch-agent/etc
Create file
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
Paste
{
 "agent": {
   "metrics_collection_interval": 60,
   "run_as_user": "root"
 },
 "metrics": {
   "namespace": "MonitorX",
   "append_dimensions": {
     "InstanceId": "${aws:InstanceId}"
   },
   "metrics_collected": {
     "cpu": {
       "measurement": [
         "cpu_usage_idle",
         "cpu_usage_user",
         "cpu_usage_system"
       ],
       "metrics_collection_interval": 60,
       "resources": [
         "*"
       ],
       "totalcpu": true
     },
     "mem": {
       "measurement": [
         "mem_used_percent"
       ],
       "metrics_collection_interval": 60
     },
     "disk": {
       "measurement": [
         "used_percent"
       ],
       "metrics_collection_interval": 60,
       "resources": [
         "*"
       ]
     }
   }
 }
}
Save.

Phase 6 — Start CloudWatch Agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
-s
Check
sudo systemctl status amazon-cloudwatch-agent

Phase 7 — Verify Metrics
AWS Console
CloudWatch
↓
Metrics
↓
Custom Namespace
MonitorX
Verify
CPU

Memory

Disk

Phase 8 — Create SNS Topic
CloudWatch will invoke Lambda, which publishes to SNS.
Open
Amazon SNS
Create Topic
Standard
Name
MonitorX-Alerts
Create.
Create Subscription
Protocol
Email
Enter
your-email@gmail.com
Confirm the email.

Step 1: Create an S3 Bucket
Open S3 Console.
Click Create bucket.
Bucket name:

 monitorx-alert-storage


Region:

 ap-south-1


Leave the remaining settings as default.
Click Create bucket.

Step 2: Update the Lambda IAM Role
Go to:
IAM
→ Roles
→ Monitorx_lambda_role
Click
Add permissions
→ Create inline policy
Select JSON and paste:
{
 "Version": "2012-10-17",
 "Statement": [
   {
     "Sid": "StoreAlertInS3",
     "Effect": "Allow",
     "Action": [
       "s3:PutObject"
     ],
     "Resource": "arn:aws:s3:::monitorx-alert-storage/*"
   }
 ]
}
Click
Next
Policy name:
Monitorx-S3-Policy
Click
Create policy

Step 3: Update Lambda Code
Go to
Lambda
→ monitor-function
→ Code
Replace everything with:
import json
import boto3
from datetime import datetime

s3 = boto3.client('s3')

BUCKET_NAME = "monitorx-alert-storage"

def lambda_handler(event, context):

   timestamp = datetime.utcnow().strftime("%Y-%m-%d_%H-%M-%S")

   filename = f"alerts/{timestamp}.json"

   s3.put_object(
       Bucket=BUCKET_NAME,
       Key=filename,
       Body=json.dumps(event, indent=4),
       ContentType="application/json"
   )

   return {
       "statusCode": 200,
       "body": "Alert Stored Successfully"
   }
Click
Deploy

Step 4: Add SNS Trigger to Lambda
Go to
Lambda
→ monitor-function
Click
Add Trigger
Choose
SNS
Select
Monitorx-alert
Check
Enable Trigger
Click
Add
You should now see
SNS
Monitorx-alert
under Triggers.

Phase 10 — Create CloudWatch Alarms
CPU Alarm
CloudWatch
↓
Alarms
↓
Create Alarm
Metric
CPUUtilization
Threshold
Greater than

80%
Evaluation
2 Datapoints

2 Minutes
Action
Instead of sending SNS directly,
Choose
Invoke Lambda Function
Select
MonitorXAlertHandler

Memory Alarm
Namespace
MonitorX
Metric
mem_used_percent
Threshold
80%
Invoke Lambda.

Disk Alarm
Metric
used_percent
Threshold
85%
Invoke Lambda.

Phase 11 — Create CloudTrail
CloudTrail
↓
Create Trail
Trail Name
MonitorX-Trail
Apply to
All Regions
Create new S3 Bucket
monitorx-audit-logs-xxxxx
Enable
Management Events

Read

Write
Create Trail.
Now every AWS API call is logged.

Phase 12 — Verify CloudTrail
Perform actions like:
Create IAM user
Stop EC2
Start EC2
Create S3 bucket
Delete security group
Then
CloudTrail
↓
Event History
You should see
Who

When

IP Address

Action

Console/Login

CLI/API

Phase 13 — Verify S3 Logs
Open
S3 Bucket
monitorx-audit-logs
You'll find
AWSLogs/

AccountID/

CloudTrail/

...
JSON log files are stored there.

Phase 14 — Generate CPU Load
Install stress tool
sudo apt install stress -y
Run
stress --cpu 2 --timeout 300
CPU should exceed 80%.
Flow
EC2

↓

CloudWatch Agent

↓

CloudWatch

↓

Alarm

↓

Lambda

↓

SNS

↓

Email

Phase 15 — Generate Memory Load
Install stress-ng
sudo apt install stress-ng -y
Run
stress-ng --vm 1 --vm-bytes 80% --timeout 300s

Phase 16 — Generate Disk Usage
Create a large file
fallocate -l 5G testfile
or
dd if=/dev/zero of=testfile bs=1M count=5000
Disk usage rises.
CloudWatch
↓
Alarm
↓
Lambda
↓
SNS
↓
Email
