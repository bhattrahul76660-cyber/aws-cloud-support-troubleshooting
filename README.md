# AWS Cloud Support Troubleshooting Project

Hands-on AWS lab built to practice **Cloud Support** skills: networking, load balancing, high availability, IAM-based access, monitoring and alerting, plus simulated incidents with root-cause analysis.

**Services used:** VPC • EC2 • Application Load Balancer • Auto Scaling • S3 • IAM • CloudWatch • SNS
**Region:** ap-south-1 (Mumbai)

> This is a learning / portfolio lab, not a production deployment.

---

## Architecture

```
User → ALB → Target Group → EC2 (Auto Scaling Group)
                                 │
                                 ├── S3 (via IAM role, no access keys)
                                 └── CloudWatch (CPU alarm) → SNS → Email
```

| Component | Details |
| --- | --- |
| VPC | `cloud-support-vpc`, 10.0.0.0/16, 2 public + 1 private subnet |
| Internet Gateway | `cloud-support-igw`, public route table with 0.0.0.0/0 route |
| Load Balancer | Internet-facing ALB, HTTP:80 listener, target group `web-tg` |
| Auto Scaling | Desired / Min / Max = 2 / 2 / 2, `t3.micro`, attached to ALB target group |
| S3 + IAM | EC2 role `ec2-s3-read-role` with read-only S3 access |
| Monitoring | CloudWatch CPU alarm (60% threshold) → SNS email notification |

---

## What I did

- Built a custom VPC with public and private subnets across two Availability Zones
- Configured an ALB with a target group and verified target health
- Set up an Auto Scaling Group and tested self-healing by terminating an instance
- Accessed S3 from EC2 using an IAM role and verified it with the AWS CLI
- Created a CloudWatch alarm and SNS topic, generated CPU load, and confirmed the email alert
- Cleaned up all resources after the lab to avoid charges

---

## Troubleshooting cases

| # | Scenario | Root cause | Fix / Verification |
| --- | --- | --- | --- |
| 1 | EC2 Instance Connect failed | Public subnet was not associated with the public route table | Associated the subnet with the public route table and reconnected |
| 2 | ALB target health needed verification | Target registration / health state had to be checked first | Verified target group health before testing the ALB |
| 3 | ASG instance terminated (simulated) | Simulated instance failure | ASG launched a replacement automatically; capacity back to 2 |
| 4 | EC2 to S3 access without stored keys | Access depended on the attached IAM role | Verified with `aws sts get-caller-identity` and `aws s3 ls` |
| 5 | High CPU alert (simulated) | Intentional CPU stress test | Alarm moved to In Alarm and SNS email was received |

**Troubleshooting approach:** symptom → evidence → root cause → fix → verification.

---

## Documentation

- [Project Documentation (PDF)](Docs/AWS_Cloud_Support_Project_Documentation.pdf): full write-up with architecture, setup steps, troubleshooting cases and screenshots
- [Project Presentation (PDF)](Docs/AWS_Cloud_Support_Project_Presentation.pdf): 13-slide summary

Account IDs, resource IDs and other private details have been masked in all screenshots.

---

## Scope and limitations

Not part of this lab: **RDS**, **NAT Gateway**, **Route 53 domain mapping**.

---

## Key learnings

- Isolate AWS network issues in order: subnet → route table → Internet Gateway → security group → target health
- ALB health checks should be verified before diagnosing application availability
- IAM roles are safer than hard-coded access keys for EC2-to-service access
- CloudWatch + SNS gives a practical monitoring-to-notification workflow

---

## Repository structure

```
aws-cloud-support-troubleshooting/
├── README.md
└── Docs/
    ├── AWS_Cloud_Support_Project_Documentation.pdf
    └── AWS_Cloud_Support_Project_Presentation.pdf
```
