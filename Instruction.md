# MonitorX – AWS Infrastructure Monitoring, Alerting & Audit Logging Platform

## Overview
This project demonstrates a production-style AWS monitoring solution using Amazon EC2, CloudWatch, CloudWatch Agent, SNS, Lambda, S3, and CloudTrail. It monitors CPU, memory, and disk utilization, triggers automated alerts, stores alert events in Amazon S3, and records AWS account activity for auditing.

---

# Architecture

EC2
↓
CloudWatch Agent
↓
CloudWatch Metrics
↓
CloudWatch Alarms
↓
Lambda
↓
SNS
↓
Email Notification

CloudTrail
↓
S3 Audit Logs

---

# AWS Services Used

- Amazon EC2
- Amazon CloudWatch
- CloudWatch Agent
- Amazon SNS
- AWS Lambda
- Amazon S3
- AWS CloudTrail
- AWS IAM

---

# Prerequisites

- AWS Account
- Ubuntu 24.04 EC2 Instance
- AWS CLI (optional)
- SSH Key Pair

---

# Phase 1 – Create IAM Roles

## EC2 Role

Create an IAM Role for EC2.

Attach Policies:

- CloudWatchAgentServerPolicy
- AmazonSSMManagedInstanceCore

Role Name:

```
MonitorX-EC2-Role
```

---

## Lambda Role

Create an IAM Role for Lambda.

Attach Policies:

- AWSLambdaBasicExecutionRole
- AmazonSNSFullAccess

Role Name:

```
MonitorX-Lambda-Role
```

---

# Phase 2 – Launch EC2

Launch:

- Ubuntu 24.04 LTS
- t2.micro
- Attach IAM Role:
  - MonitorX-EC2-Role

Security Group:

- SSH (22)
- HTTP (Optional)

Connect:

```bash
ssh -i key.pem ubuntu@<PUBLIC-IP>
```

---

# Phase 3 – Install CloudWatch Agent

Update packages

```bash
sudo apt update
sudo apt upgrade -y
```

Download CloudWatch Agent

```bash
wget https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
```

Install

```bash
sudo dpkg -i amazon-cloudwatch-agent.deb
```

Verify

```bash
amazon-cloudwatch-agent-ctl -a status
```

---

# Phase 4 – Install CollectD

```bash
sudo apt install collectd -y
```

Enable

```bash
sudo systemctl enable collectd
```

Start

```bash
sudo systemctl start collectd
```

Verify

```bash
systemctl status collectd
```

---

# Phase 5 – Configure CloudWatch Agent

Create directory

```bash
sudo mkdir -p /opt/aws/amazon-cloudwatch-agent/etc
```

Create configuration

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

Paste

```json
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
```

---

# Phase 6 – Start CloudWatch Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
-s
```

Verify

```bash
sudo systemctl status amazon-cloudwatch-agent
```

---

# Phase 7 – Verify Metrics

Navigate to

CloudWatch

→ Metrics

→ Custom Namespace

→ MonitorX

Verify

- CPU
- Memory
- Disk

---

# Phase 8 – Create SNS Topic

Create Standard Topic

```
MonitorX-Alerts
```

Create Email Subscription

```
your-email@gmail.com
```

Confirm the subscription from your inbox.

---

# Phase 9 – Create Lambda Alert Storage

## Create S3 Bucket

Bucket Name

```
monitorx-alert-storage
```

Region

```
ap-south-1
```

---

## Add S3 Permission to Lambda Role

Inline Policy

```json
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
```

Policy Name

```
Monitorx-S3-Policy
```

---

## Lambda Code

```python
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
```

Deploy the function.

---

## Add SNS Trigger

Trigger Type

```
SNS
```

Topic

```
MonitorX-Alerts
```

---

# Phase 10 – Create CloudWatch Alarms

## CPU Alarm

Metric

```
CPUUtilization
```

Threshold

```
> 80%
```

Evaluation

```
2 Datapoints
2 Minutes
```

Action

```
Invoke Lambda
```

---

## Memory Alarm

Namespace

```
MonitorX
```

Metric

```
mem_used_percent
```

Threshold

```
80%
```

Action

```
Invoke Lambda
```

---

## Disk Alarm

Metric

```
used_percent
```

Threshold

```
85%
```

Action

```
Invoke Lambda
```

---

# Phase 11 – Configure CloudTrail

Create Trail

```
MonitorX-Trail
```

Scope

```
All Regions
```

Storage

Create New S3 Bucket

Enable

- Read Events
- Write Events

---

# Phase 12 – Verify CloudTrail

Perform activities

- Create IAM User
- Stop EC2
- Start EC2
- Create S3 Bucket
- Delete Security Group

Open

CloudTrail

→ Event History

Verify

- Username
- Event Name
- Source IP
- Event Time

---

# Phase 13 – Verify Audit Logs

Open CloudTrail S3 Bucket

Verify

```
AWSLogs/
AccountID/
CloudTrail/
```

JSON log files should be present.

---

# Phase 14 – Test CPU Alarm

Install stress

```bash
sudo apt install stress -y
```

Generate load

```bash
stress --cpu 2 --timeout 300
```

Expected Flow

```
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
```

---

# Phase 15 – Test Memory Alarm

Install

```bash
sudo apt install stress-ng -y
```

Generate load

```bash
stress-ng --vm 1 --vm-bytes 80% --timeout 300s
```

---

# Phase 16 – Test Disk Alarm

Create large file

```bash
fallocate -l 5G testfile
```

OR

```bash
dd if=/dev/zero of=testfile bs=1M count=5000
```

Expected Flow

```
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
```

---

# Project Outcome

After completing this project, you will be able to:

- Monitor EC2 CPU, memory, and disk usage.
- Configure CloudWatch Agent for custom metrics.
- Create CloudWatch dashboards and alarms.
- Automate alerts using Lambda and SNS.
- Store alert events in Amazon S3.
- Audit AWS account activity using CloudTrail.
- Build a production-style cloud monitoring solution suitable for portfolio and interview demonstrations.
