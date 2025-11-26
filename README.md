# Multicloud Landing Zone – Layer 2 (Terraform)

This repository provides a multi-cloud **Layer-2 Landing Zone reference template** using Terraform for AWS, Azure, Google Cloud, and Oracle Cloud Infrastructure (OCI). This layer focuses on the secure account/subscription/project baseline—not organizational guardrails. 

Layer-2 means:
- Foundation applied inside an account, subscription, project, or compartment
- Baseline networking, logging, monitoring, and IAM/RBAC primitives
- Deployment-level security controls
- Consistency across clouds

Layer-1 Organizational controls (AWS Organizations, Azure Management Groups, GCP Organization policies, SCPs, etc.) intentionally live above this layer.

---

## 🧩 What Layer-2 Deploys Across Clouds

Each cloud implementation includes the same architectural patterns:

### Networking
- AWS VPC, Azure VNet, GCP VPC, OCI VCN
- Public and private subnets
- Route tables and default segmentation

### Logging & Monitoring
- Logging and telemetry enabled by default:
  - AWS: CloudWatch flow logs
  - Azure: Diagnostic Settings → Log Analytics
  - GCP: VPC Flow Logs → Cloud Logging
  - OCI: Logging → Log Group

### Identity & Access (Baseline)
- Example IAM or RBAC assignments
- Minimal-privilege patterns
- No hardcoded credentials

This layer provides the **application-ready cloud foundation**.

---

## ✔ Why This Repository Exists

Cloud Landing Zones are multi-layered by design. This repo focuses deliberately on:

### Layer-2:
Secure account/subscription/project setup that is ready for workloads.

This repository demonstrates:
- Architecture and design patterns across clouds
- Secure defaults and guardrails at the resource level
- Consistency in multi-cloud infrastructure structure
- Landing zone fundamentals applied in Terraform

It is not a one-click production deployment.  
It is a reusable starting point for secure cloud environments.

---

## 🌐 Supported Clouds

The repository includes one folder per cloud:

```
/aws
/azure
/gcp
/oci
```

Each folder contains:
- `main.tf`
- `variables.tf`
- `outputs.tf` (where supported)
- `terraform.tfvars.example`

All examples are fictionalized and safe for public use.

---

## 🧠 Architecture Philosophy

This Landing Zone reflects architecture-first thinking:

- Security is applied by default
- Logging and visibility are mandatory
- Identity uses native workload identity mechanisms
- Infrastructure layout is standardized across clouds
- Each cloud implementation expresses the same pattern

The design rationale is documented in `architecturephilosophy.md`.

---

## 🛠 How to Use

Select a cloud folder:

```
cd aws
cd azure
cd gcp
cd oci
```

Copy and customize the example variables file:

```
cp terraform.tfvars.example terraform.tfvars
```

Deploy:

```
terraform init
terraform plan
terraform apply
```

---

## 📐 Layer-1 vs Layer-2

This repository is **Layer-2**.

| Layer | Description | Examples |
|-------|-------------|----------|
| Layer-1 | Organizational Guardrails | AWS Organizations, SCPs, Azure Mgmt Groups |
| Layer-2 | Cloud Account/Subscription Baseline | This repo |
| Layer-3 | Workloads / Pipelines / Apps | EKS, app tiers, CI/CD, etc. |

This separation mirrors industry frameworks (AWS, Azure CAF, GCP best practices).

---

## 🧱 Recommended Extensions

Future layers may include:

- Layer-1 organizational guardrails (e.g., SCPs, policies)
- Layer-3 workloads and platform modules
- Budget guardrails and policy-as-code
- Secure Kubernetes baseline
- Multi-account/multi-subscription topology

This repo is the foundation those are built on.

---

## Related Hands-On Projects

This Landing Zone complements other real cloud security projects:
- Secure AWS CI/CD Pipeline
- Azure Log Analytics and alerting
- GCP IAM Conditions and Cloud Logging
- OCI VCN + IAM policy structure
- Kubernetes secure architecture
- Multi-cloud security automation

These demonstrate applied cloud architecture beyond Terraform.

---

## Disclaimer

This repository is for reference and educational use.  
Replace placeholder identities and fictionalized values for real deployments.

---

## Author

Multicloud Landing Zone – Layer 2  
Terraform Architecture Reference Template
