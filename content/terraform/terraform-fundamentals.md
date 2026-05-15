---
title: "Terraform — Infrastructure as Code Fundamentals"
date: 2024-02-01
draft: false
description: "Terraform core concepts: providers, resources, state, modules, and real AWS examples."
categories: ["terraform"]
tags: ["terraform", "iac", "aws", "hcl"]
showToc: true
---

## Core workflow

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

Always `plan` before `apply`. Read it carefully.

## HCL basics

```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" { region = var.aws_region }

variable "aws_region" {
  type    = string
  default = "us-east-1"
}

resource "aws_s3_bucket" "assets" {
  bucket = "my-app-assets"
  tags   = { ManagedBy = "terraform" }
}

output "bucket_name" {
  value = aws_s3_bucket.assets.bucket
}
```

## EC2 + VPC example

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags       = { Name = "main-vpc" }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id
  tags          = { Name = "web-server" }
}
```

## State commands

```bash
terraform state list
terraform state show aws_instance.web
terraform state rm aws_s3_bucket.old
terraform import aws_s3_bucket.existing my-bucket
```

## Things that got me

- Never edit state manually — use terraform state commands
- Remote state in S3 is essential for teams
- terraform destroy in CI needs a manual approval gate
