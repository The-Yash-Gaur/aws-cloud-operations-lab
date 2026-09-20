# AWS Cloud Operations Lab

A hands-on AWS infrastructure and cloud operations project built to demonstrate practical experience with **AWS, Terraform, Linux, networking, IAM, Systems Manager, and NGINX**.

The project provisions and operates a small internet-facing workload on AWS while following infrastructure-as-code and cloud security practices.

> **Live Demo:** http://13.205.180.156/

---

## Architecture

```text
                         Internet
                            |
                            | HTTP :80
                            v
                    Internet Gateway
                            |
                            v
                     Public Route Table
                            |
                            v
                       Public Subnet
                            |
                            v
                     Security Group
                  HTTP :80 allowed
                  SSH  :22 blocked
                            |
                            v
                      EC2 (t3.micro)
                       Amazon Linux
                            |
              +-------------+-------------+
              |                           |
              v                           v
          NGINX :80                 AWS SSM Agent
              |                           |
              v                           v
       Static Web Page          Systems Manager
                                Administration
                                without public SSH
```

## What I Built

### Infrastructure as Code

The AWS infrastructure is provisioned using **Terraform** rather than being created manually through the AWS Console.

Terraform currently manages:

* VPC
* Public subnet
* Internet Gateway
* Public route table
* Route table association
* Security Group
* EC2 instance
* Encrypted EBS root volume
* IAM role
* IAM instance profile
* AWS Systems Manager permissions

The infrastructure follows the standard Terraform workflow:

```text
Write → Format → Validate → Plan → Review → Apply → Verify
```

---

## Networking

The environment uses a custom VPC:

```text
VPC
10.0.0.0/16

└── Public Subnet
    10.0.1.0/24
```

The public subnet is associated with a route table containing:

```text
10.0.0.0/16 → local
0.0.0.0/0   → Internet Gateway
```

This allows internet-bound traffic from resources in the public subnet to reach the Internet Gateway.

---

## Compute

The workload runs on an **Amazon EC2 t3.micro** instance using Amazon Linux 2023.

The instance uses an encrypted **gp3 EBS root volume**.

NGINX runs as the internet-facing web server and serves the project website over HTTP port 80.

---

## Security

The project intentionally avoids exposing SSH to the internet.

### Inbound access

```text
TCP 80   → Allowed
TCP 22   → Not exposed
TCP 8000 → Not exposed
```

Instead of SSH, instance administration is performed using **AWS Systems Manager Session Manager**.

This allows the EC2 instance to be managed through AWS without maintaining an internet-facing SSH service or distributing SSH private keys.

---

## IAM

The EC2 instance uses an IAM role through an instance profile.

The trust policy allows the EC2 service to assume the role, while the attached Systems Manager policy provides the permissions required for the instance to register with AWS Systems Manager.

This avoids storing long-lived AWS credentials on the server.

---

## Linux Operations

The project also demonstrates basic Linux service administration.

A Python application was initially deployed on port `8000` and configured as a `systemd` service to understand application process management and boot persistence.

NGINX is configured as the public-facing web server on port `80`.

Service persistence was tested by rebooting the EC2 instance and verifying that the deployed website became available again without manual intervention.

---

## Web Server

The public project page is served using **NGINX**.

```text
Internet
    |
    | TCP 80
    v
Security Group
    |
    v
NGINX
    |
    v
Static Project Website
```

The website source is maintained under:

```text
website/
└── index.html
```

---

## Repository Structure

```text
aws-cloud-operations-lab/
│
├── terraform/
│   ├── main.tf
│   └── .terraform.lock.hcl
│
├── website/
│   └── index.html
│
├── .gitignore
│
└── README.md
```

Terraform state and local Terraform working files are excluded from Git.

---

## Validation & Testing

The infrastructure and application have been manually validated through several tests.

### Infrastructure

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

### AWS

AWS CLI was used to verify deployed resources and instance state.

### Web Server

```bash
curl http://localhost
```

was used from the EC2 instance to verify the local NGINX service.

The public endpoint was then tested from an external browser.

### Service Resilience

The EC2 instance was rebooted to verify that the required services automatically returned without manually starting the application or web server.

### Remote Administration

AWS Systems Manager Session Manager and SSM Run Command were tested successfully without exposing TCP port 22.

---

## Security Decisions

Several design decisions were made deliberately:

| Decision             | Reason                                                                |
| -------------------- | --------------------------------------------------------------------- |
| No public SSH        | Reduce unnecessary externally reachable services                      |
| AWS Systems Manager  | Secure instance administration without SSH keys                       |
| IAM instance role    | Avoid long-lived AWS credentials                                      |
| Encrypted EBS        | Protect data stored on the instance volume                            |
| Terraform            | Reproducible infrastructure configuration                             |
| Port 8000 not public | Backend/application ports should not be unnecessarily internet-facing |
| NGINX on port 80     | Dedicated internet-facing web-server layer                            |

---

## Planned Improvements

This project is being developed incrementally.

Future improvements include:

* GitHub Actions CI/CD
* Automated deployments using AWS Systems Manager
* Docker containerization
* CloudWatch metrics, logs, and alarms
* CloudTrail auditing
* Automated cloud security checks using Python/Boto3
* AWS resource cost optimization checks
* HTTPS/TLS
* Custom domain
* Additional infrastructure hardening

---

## Key Technologies

**Cloud:** AWS EC2, VPC, IAM, Systems Manager, EBS

**Infrastructure as Code:** Terraform

**Operating System:** Amazon Linux 2023

**Web Server:** NGINX

**Programming:** Python

**Tools:** AWS CLI, Git, GitHub

---

## Project Goal

The goal of this project is not simply to deploy a webpage.

It is to understand and demonstrate the complete lifecycle of operating a small cloud workload:

```text
Architect
   ↓
Provision
   ↓
Deploy
   ↓
Secure
   ↓
Operate
   ↓
Monitor
   ↓
Troubleshoot
   ↓
Optimize
```

The project will continue to evolve as additional cloud operations, automation, observability, security, and deployment capabilities are implemented.

---

## Author

**Yash Gaur**

GitHub: The-Yash-Gaur
