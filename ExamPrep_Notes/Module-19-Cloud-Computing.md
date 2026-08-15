# Module 19: Cloud Computing

## Tier 1: Core Concepts & Principles

### Cloud Delivery & Deployment Architecture
* **NIST Cloud Service Models:**
  * **Infrastructure as a Service (IaaS):** Cloud Service Provider (CSP) manages physical facilities, networking hardware, hypervisors, and host virtualization. The customer retains full administrative ownership and responsibility for the Guest OS, network firewall configurations (Security Groups/NSGs), middleware, runtime environments, application binaries, and data layers (e.g., AWS EC2, Azure VMs, Google Compute Engine).
  * **Platform as a Service (PaaS):** CSP manages the hardware, virtualization, OS, and runtime framework. The customer manages only application logic, database schemas, and data access controls (e.g., AWS Elastic Beanstalk, Azure App Service, Google App Engine).
  * **Software as a Service (SaaS):** CSP owns and manages the complete underlying infrastructure, code, and operational pipeline. The customer is responsible solely for user identity, data access policies, and tenant-level configurations (e.g., Microsoft 365, Salesforce, Google Workspace).
* **NIST Deployment Models:** Public Cloud (multi-tenant infrastructure over Internet), Private Cloud (dedicated single-organization infrastructure), Community Cloud (shared multi-organization infrastructure with shared compliance goals), and Hybrid Cloud (composition of on-premises and public cloud connected via IPsec VPN, AWS DirectConnect, or Azure ExpressRoute).
* **The Shared Responsibility Model:**
  * **Security OF the Cloud:** Physical perimeter security, hardware maintenance, facilities, power, host virtualization isolation $\rightarrow$ Exclusively the **CSP's Responsibility**.
  * **Security IN the Cloud:** IAM permissions, guest OS security patches, S3 bucket ACLs, ingress/egress firewall rules, database encryption, customer data $\rightarrow$ Exclusively the **Customer's Responsibility**.

---

## Tier 2: Technical Analysis & Mechanics

### Cloud Attack Vectors & Exploitation Mechanics

#### 1. Azure Active Directory (Azure AD / Entra ID) Reconnaissance
* **Mechanism:** Azure AD maintains public discovery endpoints (`login.microsoftonline.com`, OpenID configuration JSONs, Autodiscover XMLs) to support cross-tenant authentication and federated identity services.
* **Exploitation Pipeline:**
  * Attackers submit a target organization domain (e.g., `targetcompany.com`) to public endpoints via specialized PowerShell modules like `AADInternals`.
  * Public OpenID metadata reveals the unique **Tenant ID** (GUID) and verified alternate domain names.
  * Timing attacks and HTTP response status codes on login endpoints disclose whether specific user email accounts exist (`User Realm Discovery`).
  * Identifies identity configuration: **Managed** (cloud-only / password hash sync) vs. **Federated** (Active Directory Federation Services - ADFS).

  An attacker can use publicly exposed Entra ID authentication/discovery infrastructure to identify an organization's tenant, domains, authentication configuration, and potentially information about user accounts—all without initially accessing the organization's internal network.

#### 2. Amazon S3 Bucket Exploitation
* **Mechanics:** S3 buckets use standard REST endpoint URL structures:
  $$\text{https://} \langle \text{bucket-name} \rangle \text{.s3.amazonaws.com} \quad \text{or} \quad \text{https://s3.amazonaws.com/} \langle \text{bucket-name} \rangle$$
* **Misconfiguration Flaws:**
  * **`AllUsers` Group:** Grants unauthenticated access to anyone on the public Internet.
  * **`AuthenticatedUsers` Group:** Grants access to *any* individual possessing a valid AWS account worldwide, failing to enforce organizational boundaries.
* **Exploitation Impact:** Attackers perform bucket enumeration (`s3:ListBucket`), download sensitive objects via unauthenticated CLI requests (`--no-sign-request`), or execute arbitrary file uploads/overwrites (`s3:PutObject`).

An attacker can exploit publicly accessible or misconfigured S3 buckets to enumerate stored objects, access sensitive data, or upload/overwrite files without proper authorization.

#### 3. AWS IAM Privilege Escalation
* **Mechanics:** Insecure IAM permission policies allow low-privileged users to attach policies or modify policy versions.
* **Exploitation Pattern (`iam:AttachUserPolicy`):**
  $$\text{Vulnerable Permission: } \texttt{"Action": "iam:AttachUserPolicy", "Resource": "*"} \longrightarrow \text{Attach "AdministratorAccess" policy to self}$$
* **Escalation Path:** An attacker with compromised low-privilege credentials executes `iam:AttachUserPolicy` to attach the AWS-managed policy `arn:aws:iam::aws:policy/AdministratorAccess` directly to their own IAM user identity, gaining full administrative control across the cloud infrastructure.

An attacker with low-privileged AWS credentials can exploit overly permissive IAM policies to modify their own permissions, escalate to administrator privileges, and gain broad control over cloud resources.


#### 4. Docker & Container Security Lifecycles
* **Layer Vulnerabilities:** Docker images inherit vulnerabilities from parent base images (e.g., outdated Linux kernel packages, libraries).
* **Container Misconfigurations:**
  * **Privileged Mode (`--privileged`):** Disables container isolation, giving container processes direct access to host kernel capabilities and host devices.
  * **Exposed Docker Socket (`/var/run/docker.sock`):** Mounting the Docker daemon socket inside a container allows an attacker to spawn sibling containers with root access directly on the host OS.

  An attacker who gains access to a misconfigured container can exploit weak isolation, privileged mode, or an exposed Docker socket to interact with the host system and potentially gain host-level control.

---

## Tier 3: CLI, Tools & Framework Syntax

### Core Cloud Reconnaissance & Exploitation Utilities
* **AADInternals:** PowerShell framework for performing reconnaissance, user enumeration, and post-exploitation against Azure Active Directory / Microsoft 365.
* **AWS CLI (`aws`):** Unified command-line interface for interacting with AWS services, S3 object stores, and IAM policies.
* **Trivy:** Open-source static analysis vulnerability scanner that inspects container images, file systems, and Git repositories for CVEs, exposed secrets, and misconfigurations.

### Key Commands & Execution Syntax

```powershell
# AADInternals: Azure AD Reconnaissance (PowerShell)
Install-Module -Name AADInternals -Force
Import-Module AADInternals

# Gather public tenant metadata, tenant ID, and domains from domain name
Invoke-AADIntReconAsOutsider -DomainName targetorganization.com

# Enumerate valid Azure AD / Microsoft 365 usernames
Invoke-AADIntUserEnumeration -UserName admin@targetorganization.com
```

```bash
# AWS CLI: S3 Bucket Enumeration & Object Manipulation
# Configure AWS CLI authentication credentials
aws configure

# List objects in a public S3 bucket without requiring AWS credentials
aws s3 ls s3://target-company-backups/ --no-sign-request

# Recursively download entire target S3 bucket contents locally
aws s3 sync s3://target-company-backups/ /tmp/looted_s3/ --no-sign-request

# Upload file to public S3 bucket (verifying write permissions)
aws s3 cp /tmp/webshell.php s3://target-company-backups/webshell.php

# Delete an object from an S3 bucket
aws s3 rm s3://target-company-backups/sensitive_backup.tar.gz

# AWS CLI: IAM Privilege Escalation
# List policies attached to the current low-privileged user
aws iam list-attached-user-policies --user-name lowpriv-user

# Attach AdministratorAccess managed policy to the target low-privileged user
aws iam attach-user-policy --user-name lowpriv-user --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Enumerate all account users following successful escalation
aws iam list-users

# Trivy: Container Image Vulnerability Scanning
# Scan a local/remote Docker container image for known CVEs
trivy image nginx:latest

# Scan Docker image filtering output strictly for High and Critical vulnerabilities
trivy image --severity HIGH,CRITICAL ubuntu:20.04

# Scan image and exit with non-zero code on Critical findings (CI/CD integration)
trivy image --exit-code 1 --severity CRITICAL vulnerable-app:v1.0
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Defensive Architecture & Hardening Controls
* **S3 Storage Security Controls:**
  * **S3 Block Public Access:** Enforce "Block Public Access" at the AWS Account and individual bucket levels to centrally override any permissive ACLs or Bucket Policies.
  * **Server-Side Encryption:** Enforce `SSE-KMS` using Customer Master Keys (CMKs) to ensure data at rest is cryptographically protected.
* **IAM Least Privilege & Permission Boundaries:**
  * Eliminate wildcard (`"*"`) resource and action declarations from custom IAM policies.
  * Strictly restrict high-risk IAM actions (`iam:AttachUserPolicy`, `iam:PutUserPolicy`, `iam:CreatePolicyVersion`, `iam:SetDefaultPolicyVersion`).
  * Enforce **IAM Permissions Boundaries** to set the absolute maximum authorization ceiling for users and roles, neutralizing privilege escalation attempts even if broad policies are attached.
* **Container Security Controls:**
  * Integrate vulnerability scanners (`Trivy`, `Clair`, `Amazon ECR basic scanning`) into CI/CD pipelines to block deployments failing CVE thresholds.
  * Enforce non-root execution inside Dockerfiles (`USER nonroot`).
  * Never mount `/var/run/docker.sock` inside untrusted containers and permanently disable `--privileged` flags across container runtime environments.
* **Cloud Security Posture Management (CSPM):** Deploy CSPM platforms (e.g., Microsoft Defender for Cloud, AWS Security Hub, Prisma Cloud) to continuously audit multi-cloud assets against CIS Cloud Benchmarks.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **IaaS Model** | Customer manages OS, runtime, applications, and network firewalls | CSP manages physical data centers, servers, and hypervisors |
| **PaaS Model** | Customer manages only application code and data schemas | CSP manages underlying OS, runtime, and server maintenance |
| **SaaS Model** | CSP manages the entire technology stack | Customer manages only user identities and access policies |
| **Shared Responsibility Model** | Security **OF** the cloud vs Security **IN** the cloud | CSP = Infrastructure security; Customer = Data, IAM, and configuration security |
| **S3 Public Access Risk** | Access granted to `AllUsers` or `AuthenticatedUsers` | Enables unauthenticated data exfiltration via `--no-sign-request` |
| **`iam:AttachUserPolicy`** | Key IAM privilege escalation vulnerability | Attaching `AdministratorAccess` policy to self yields full account takeover |
| **AADInternals** | PowerShell tool for Azure Active Directory / M365 reconnaissance | Enumerates tenant IDs, registered domains, and user accounts via public APIs |
| **Trivy** | CLI scanner for Docker container images and filesystems | Identifies known CVEs, OS package flaws, and embedded secrets |
| **Docker `--privileged`** | Major container misconfiguration risk | Disables container isolation, granting root access directly to host kernel |
| **IAM Permissions Boundary** | Advanced AWS policy setting the maximum ceiling of permissions | Prevents users from escalating privileges beyond a designated ceiling |
| **AWS S3 URL Format** | `https://<bucket-name>.s3.amazonaws.com` | Standard REST format used to query and identify cloud storage endpoints |
| **CSPM Tooling** | Automated compliance and misconfiguration discovery | Focuses on cloud control plane configurations, not endpoint agent logs |