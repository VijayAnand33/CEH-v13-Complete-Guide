# Trivy

## Overview

Trivy is an open-source vulnerability scanner developed by Aqua Security for identifying vulnerabilities, misconfigurations, and security issues in container images, file systems, Git repositories, Kubernetes environments, and cloud infrastructure. It enables security professionals to identify known vulnerabilities before deployment.

---

## Purpose

Trivy is used to:

- Scan Docker container images.
- Detect known vulnerabilities.
- Identify affected software packages.
- Display CVE information.
- Classify vulnerabilities by severity.
- Support secure container deployments.

---

## Key Features

- Container image scanning.
- CVE detection.
- Severity classification.
- Misconfiguration detection.
- Fast vulnerability analysis.
- Lightweight operation.

---

## Installation

Install Trivy:

```bash
sudo apt install trivy
```

Verify installation:

```bash
trivy --version
```

---

## Typical Workflow

```mermaid
flowchart LR
    A[Select Container Image] --> B[Scan with Trivy]
    B --> C[Identify Vulnerabilities]
    C --> D[Review Severity]
    D --> E[Plan Remediation]

    style A fill:#1f2937,color:#fff,stroke:#2563eb
    style B fill:#374151,color:#fff,stroke:#2563eb
    style C fill:#4b5563,color:#fff,stroke:#2563eb
    style D fill:#7c3aed,color:#fff,stroke:#8b5cf6
    style E fill:#059669,color:#fff,stroke:#10b981
```

---

## CEH Practical Example

In **Module 19 – Cloud Computing**, Trivy was used to assess Docker container images by scanning Ubuntu and Nginx images, identifying known vulnerabilities, reviewing severity levels, and analyzing associated CVE information.

---

## Advantages

- Fast and lightweight.
- Open-source.
- Accurate vulnerability detection.
- Supports multiple scanning targets.
- Easy CI/CD integration.

---

## Limitations

- Detects only known vulnerabilities.
- Requires updated vulnerability databases.
- Cannot identify custom application logic flaws.

---

## Best Practices

- Scan images before deployment.
- Update vulnerability databases regularly.
- Remove vulnerable packages when possible.
- Integrate scanning into CI/CD pipelines.

---

## Used In

- Module 19 – Cloud Computing

---

## References

- https://trivy.dev/