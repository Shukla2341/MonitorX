# MonitorX – AWS Infrastructure Monitoring, Alerting & Audit Logging Platform

## 📌 Project Overview

**MonitorX** is a production-style AWS cloud monitoring project that monitors the health of an Amazon EC2 instance by collecting CPU, memory, and disk metrics. It automatically generates alerts when resource utilization exceeds predefined thresholds, stores alert events in Amazon S3, and records AWS account activity using CloudTrail for auditing.

This project demonstrates how multiple AWS services work together to build an automated monitoring and alerting solution similar to those used in real production environments.

---

## 🚀 Features

- Monitor EC2 CPU, Memory, and Disk utilization
- Collect custom metrics using CloudWatch Agent
- Create CloudWatch Alarms for high resource usage
- Trigger AWS Lambda automatically from alarms
- Store alert events in Amazon S3
- Configure SNS email notifications
- Track AWS account activities using CloudTrail
- Audit logs stored securely in Amazon S3

---

## 🏗️ Architecture

```text
                    +----------------+
                    |   Amazon EC2   |
                    +----------------+
                             |
                             |
                  CloudWatch Agent
                             |
                             ▼
                 Amazon CloudWatch
                      (Metrics)
                             |
                             ▼
                 CloudWatch Alarms
                             |
                             ▼
                    AWS Lambda
                             |
                 +-----------+-----------+
                 |                       |
                 ▼                       ▼
           Amazon SNS              Amazon S3
        (Email Notification)    (Alert Storage)

----------------------------------------------------

 AWS Account Activity
          |
          ▼
     AWS CloudTrail
          |
          ▼
      Amazon S3
    (Audit Log Storage)
```

---

## 🛠️ AWS Services Used

- Amazon EC2
- Amazon CloudWatch
- Amazon CloudWatch Agent
- AWS Lambda
- Amazon SNS
- Amazon S3
- AWS CloudTrail
- AWS IAM

---

## 📂 Project Workflow

1. Launch an Ubuntu EC2 instance.
2. Install and configure the CloudWatch Agent.
3. Collect CPU, memory, and disk metrics.
4. Send metrics to CloudWatch.
5. Create CloudWatch Alarms.
6. Invoke a Lambda function when alarms are triggered.
7. Store alert details in Amazon S3.
8. Send email notifications using Amazon SNS.
9. Record AWS account activity using CloudTrail.
10. Store audit logs in Amazon S3.

---

## 📊 Monitoring Metrics

- CPU Utilization
- Memory Usage
- Disk Usage

---

## 🔔 Alarm Configuration

| Metric | Threshold |
|---------|-----------|
| CPU Utilization | > 80% |
| Memory Usage | > 80% |
| Disk Usage | > 85% |

---

## 📁 Project Structure

```text
MonitorX/
│
├── README.md
├── instruction.md
├── lambda/
│   └── lambda_function.py
├── cloudwatch/
│   └── amazon-cloudwatch-agent.json
├── screenshots/
└── architecture.png
```

---

## ⚙️ Technologies Used

- AWS EC2
- CloudWatch
- CloudWatch Agent
- AWS Lambda
- Amazon SNS
- Amazon S3
- AWS CloudTrail
- IAM
- Ubuntu 24.04 LTS
- Python (Boto3)

---

## 🧪 Testing

### CPU Test

```bash
sudo apt install stress -y
stress --cpu 2 --timeout 300
```

### Memory Test

```bash
sudo apt install stress-ng -y
stress-ng --vm 1 --vm-bytes 80% --timeout 300s
```

### Disk Test

```bash
fallocate -l 5G testfile
```

or

```bash
dd if=/dev/zero of=testfile bs=1M count=5000
```

---

## 📸 Suggested Screenshots

Include screenshots of:

- EC2 Instance
- IAM Roles
- CloudWatch Metrics
- CloudWatch Dashboard
- CloudWatch Alarms
- SNS Topic
- Email Notification
- Lambda Function
- S3 Alert Bucket
- CloudTrail Event History
- CloudTrail S3 Logs

---

## 🎯 Learning Outcomes

By completing this project, I gained hands-on experience with:

- AWS IAM Roles and Permissions
- Amazon EC2 Management
- CloudWatch Metrics and Dashboards
- CloudWatch Agent Configuration
- CloudWatch Alarms
- Event-driven automation using AWS Lambda
- Amazon SNS Notifications
- Amazon S3 Object Storage
- AWS CloudTrail Audit Logging
- Production-style cloud monitoring architecture

---

## 💡 Future Improvements

- CloudWatch Dashboard with custom widgets
- Slack or Microsoft Teams notifications
- Auto-remediation using AWS Systems Manager
- Terraform Infrastructure as Code
- AWS CDK deployment
- Multi-instance monitoring
- Auto Scaling integration

---

## 👨‍💻 Author

**Ashutosh Shukla**

AWS | Cloud Computing | DevOps Enthusiast

---

## ⭐ Project Summary

MonitorX is a production-style AWS monitoring platform that combines Amazon EC2, CloudWatch, CloudWatch Agent, Lambda, SNS, S3, IAM, and CloudTrail to monitor infrastructure health, automate alerts, store alert data, and maintain audit logs. This project demonstrates practical cloud monitoring, automation, and security concepts commonly used in modern AWS environments.
