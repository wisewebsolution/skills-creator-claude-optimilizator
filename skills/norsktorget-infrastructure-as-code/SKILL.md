---
name: norsktorget-infrastructure-as-code
description: Use when creating or modifying infrastructure for the Norsktorget project using Terraform. This skill provides the official guidelines for project structure, resource naming conventions, security policies, and reusable boilerplate for AWS, ensuring a consistent, secure, and scalable infrastructure.
---

# Norsktorget Infrastructure as Code (IaC) Guide

## Overview

This skill is the single source of truth for provisioning and managing all cloud infrastructure for the Norsktorget project using Terraform. It ensures our infrastructure is modular, secure, scalable, and consistently managed.

**Core Principle:** All infrastructure changes MUST be implemented as code via Terraform and follow the patterns defined in this guide. Manual changes in the AWS Console are forbidden for managed resources.

## When to Use

- When setting up a new environment (staging, production).
- When adding or modifying any AWS resource (ECS, RDS, S3, etc.).
- When defining security group rules or IAM policies.
- When reviewing a pull request that includes Terraform changes.

## 1. Terraform Project Structure (CRITICAL PATTERN)

Terraform code **must** be organized into a modular structure. A single `main.tf` file for an entire environment is not acceptable.

### Required Directory Structure

```
terraform/
├── environments/
│   ├── staging/
│   │   ├── main.tf           # Main entrypoint for staging
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── backend.tf         # S3 backend for state file
│   └── production/
│       ├── ...
├── modules/
│   ├── network/               # VPC, subnets, security groups
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── database/              # RDS instance
│   │   ├── ...
│   └── application/           # ECS Fargate service
│       ├── ...
└── templates/
    └── ...
```

**Workflow:** To apply changes, navigate to the specific environment directory (e.g., `cd terraform/environments/staging`) and run `terraform apply`.

## 2. Resource Naming and Tagging Convention

All resources **must** follow a strict naming and tagging convention for consistency and cost management.

### Naming Convention

**Format:** `[project]-[environment]-[resource_type]-[name]`

- `project`: `norsktorget`
- `environment`: `staging` lub `prod`
- `resource_type`: `vpc`, `sg`, `db`, `ecs`, `s3`
- `name`: A descriptive name (e.g., `main`, `public`, `private`)

**Examples:**
- VPC: `norsktorget-staging-vpc-main`
- Security Group: `norsktorget-staging-sg-rds-access`
- RDS Instance: `norsktorget-prod-db-main`
- S3 Bucket: `norsktorget-prod-s3-images`

### Required Tags

All resources **must** have the following tags:

| Tag Key | Example Value | Description |
| :--- | :--- | :--- |
| `Project` | `norsktorget` | The name of the project. |
| `Environment`| `staging` | The deployment environment. |
| `ManagedBy` | `Terraform` | Indicates the resource is managed by IaC. |
| `Owner` | `dev-team` | The team responsible for the resource. |

## 3. Security Best Practices

### Security Groups

- **Principle of Least Privilege:** Security groups **must** be as restrictive as possible.
- **❌ Anti-Pattern:** Never use `0.0.0.0/0` for ingress rules on sensitive resources like databases.
- **✅ Required Pattern:** Allow traffic only from specific security groups.

```terraform
# ❌ BAD: Database open to the world
resource "aws_security_group_rule" "db_ingress_bad" {
  type        = "ingress"
  from_port   = 5432
  to_port     = 5432
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"] // DANGEROUS!
}

# ✅ GOOD: Database only accessible from the application's security group
resource "aws_security_group_rule" "db_ingress_good" {
  type                     = "ingress"
  from_port                = 5432
  to_port                  = 5432
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.application.id // CORRECT
}
```

### IAM (Identity and Access Management)

- Use IAM roles for AWS services (e.g., ECS Task Roles) instead of hardcoding credentials.
- Grant the minimum required permissions for each role.

### Secrets Management

Database passwords, API keys, and other secrets must not be stored in Terraform state or Git.

**Required Tool:** Use AWS Secrets Manager or HashiCorp Vault.

```terraform
# ✅ GOOD: Fetching a secret from AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "norsktorget/staging/db_password"
}

resource "aws_db_instance" "main" {
  // ...
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

## 4. Terraform Workflow

1. **Initialize:** `terraform init` (run once per environment)
2. **Plan:** `terraform plan` (review the proposed changes)
3. **Apply:** `terraform apply` (execute the changes)
4. **Commit:** Commit your `.tf` files to Git. **Never commit the `.tfstate` file.**

### State Management

Terraform state must be stored remotely in an S3 bucket with versioning and locking enabled.

Configure this in `backend.tf`.

```terraform
# terraform/environments/staging/backend.tf
terraform {
  backend "s3" {
    bucket         = "norsktorget-terraform-state"
    key            = "staging/terraform.tfstate"
    region         = "eu-north-1"
    encrypt        = true
    dynamodb_table = "norsktorget-terraform-locks"
  }
}
```
