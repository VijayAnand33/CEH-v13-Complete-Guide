# AWS CLI

## Overview

AWS Command Line Interface (AWS CLI) is Amazon Web Services' official command-line tool for managing AWS resources. It enables administrators and security professionals to interact with AWS services, automate administrative tasks, perform cloud enumeration, and assess cloud security through terminal-based commands.

---

## Purpose

AWS CLI is used to:

- Manage AWS resources.
- Enumerate cloud services.
- Interact with Amazon S3.
- Manage IAM users and policies.
- Automate AWS administration.
- Perform cloud security assessments.

---

## Key Features

- Command-line access to AWS services.
- Amazon S3 management.
- IAM administration.
- Resource enumeration.
- Automation through scripting.
- Cross-platform support.

---

## Installation

Install AWS CLI:

```bash
sudo apt install awscli
```

Verify installation:

```bash
aws --version
```

---

## Typical Workflow

```mermaid
flowchart LR
    A[Configure AWS CLI] --> B[Authenticate]
    B --> C[Enumerate AWS Resources]
    C --> D[Manage Cloud Services]
    D --> E[Assess Security]

    style A fill:#1f2937,color:#fff,stroke:#2563eb
    style B fill:#374151,color:#fff,stroke:#2563eb
    style C fill:#4b5563,color:#fff,stroke:#2563eb
    style D fill:#7c3aed,color:#fff,stroke:#8b5cf6
    style E fill:#059669,color:#fff,stroke:#10b981
```

---

## CEH Practical Example

In **Module 19 – Cloud Computing**, AWS CLI was used to enumerate Amazon S3 buckets, upload and delete objects, create and attach IAM policies, perform privilege escalation demonstrations, and enumerate AWS resources.

---

## Advantages

- Official AWS management tool.
- Supports all AWS services.
- Easy to automate.
- Cross-platform compatibility.
- Widely used in cloud administration.

---

## Limitations

- Requires valid AWS credentials.
- Permissions are controlled through IAM.
- Incorrect configurations may expose cloud resources.

---

## Best Practices

- Follow the principle of least privilege.
- Rotate access keys regularly.
- Secure AWS credentials.
- Enable logging through AWS CloudTrail.

---

## Used In

- Module 19 – Cloud Computing

---

## References

- https://aws.amazon.com/cli/