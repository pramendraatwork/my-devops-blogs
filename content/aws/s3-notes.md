---
title: "AWS S3 — Everything I Keep Looking Up"
date: 2024-01-15
draft: false
description: "Practical S3 notes: bucket policies, lifecycle rules, versioning, and CLI cheatsheet."
categories: ["aws"]
tags: ["s3", "storage", "cli", "iam"]
showToc: true
---

## What is S3?

S3 is AWS object storage — infinitely scalable key-value store. Objects live in **buckets**, each with a **key**.

> S3 is NOT a filesystem. The `/` in a key is just a character.

## CLI cheatsheet

```bash
aws s3 ls
aws s3 ls s3://my-bucket/
aws s3 cp file.txt s3://my-bucket/file.txt
aws s3 cp ./dist/ s3://my-bucket/ --recursive
aws s3 sync ./dist/ s3://my-bucket/ --delete
aws s3 cp s3://my-bucket/file.txt ./file.txt
aws s3 rm s3://my-bucket/old-file.txt
```

## Storage classes

| Class | Use case | Retrieval |
|---|---|---|
| Standard | Frequently accessed | Instant |
| Standard-IA | Infrequent access | Instant |
| Glacier Instant | Archives quarterly | Instant |
| Glacier Deep Archive | Compliance | 12-48 hours |
| Intelligent-Tiering | Unknown pattern | Instant |

## Versioning

```bash
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled
```

## Things that bit me

- Block Public Access is ON by default — disable for static sites
- Presigned URLs give temporary access to private objects
- Once versioning is enabled it can only be suspended, not disabled
