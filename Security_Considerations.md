# Security Considerations

This document outlines key security considerations for the Multicloud Landing Zone – Layer 2 Terraform templates. The goal is to make the architectural intent explicit, not to claim a complete or audited security posture.

The templates focus on **account/subscription/project-level baselines** for AWS, Azure, GCP, and OCI.

---

## 1. Scope and Assumptions

**In scope (Layer 2):**

- Per-account / subscription / project / compartment foundations
- Baseline networking (VPC/VNet/VCN and subnets)
- Logging and visibility (flow logs, diagnostic settings, logging groups)
- Example IAM/RBAC constructs for least-privilege patterns

**Out of scope (Layer 1 and Layer 3):**

- Enterprise identity (SSO, IdPs, federation, multi-factor enforcement)
- Organization-level guardrails (AWS Organizations/SCPs, Azure Management Groups/Policies, GCP Org Policies, OCI tenancy-wide controls)
- Application workloads (EC2/VMs, containers, databases, serverless functions)
- Secrets management and data-plane encryption configurations

**Assumptions:**

- The reader will adapt these templates to their own standards.
- Real tenants will apply additional policies at the organization/management-group level.
- Identities referenced (ARNs, object IDs, emails, OCIDs) are placeholders and must be replaced.

---

## 2. Identity and Access Management

### 2.1 Human Access (Examples Only)

Each cloud implementation includes **example read-only roles or bindings**:

- AWS: An IAM role that can be assumed and grants `ReadOnlyAccess`.
- Azure: A `Reader` role assignment at the resource group scope.
- GCP: A project-level `roles/viewer` binding.
- OCI: Example policy statements granting read access (if extended).

**Considerations:**

- In real deployments, these roles should be mapped to **groups** managed by an IdP (Entra ID, Okta, Ping, etc.).
- MFA, conditional access, and strong authentication are expected at the identity provider level.
- Privileged roles (e.g., admin, security admin) are intentionally **not** created here to avoid promoting over-privileged patterns.

### 2.2 Workload Identities

These templates intentionally avoid hardcoded credentials. When extended:

- Use AWS IAM roles for compute/services (no access keys on instances).
- Use Azure managed identities for workloads.
- Use GCP service accounts with strict IAM bindings.
- Use OCI dynamic groups + policies for workload identity.

---

## 3. Network Security

**Patterns implemented:**

- Segmented networks using per-cloud constructs:
  - AWS VPC with public and private subnets.
  - Azure VNet with separate subnets.
  - GCP VPC with custom subnetwork and firewall rules.
  - OCI VCN with public and private subnets.
- Public internet access is confined to “public” tiers.
- Private subnets are created with no direct internet reachability.

**Additional considerations for real deployments:**

- Add explicit deny/allowlists via:
  - AWS Security Groups and NACLs.
  - Azure NSGs.
  - GCP firewall rules.
  - OCI security lists / NSGs.
- Introduce ingress/egress inspection (NGFW, WAF, IDS/IPS) at appropriate tiers.
- Consider private connectivity (VPN, Direct Connect, ExpressRoute, Cloud Interconnect, FastConnect) for enterprise networks.

---

## 4. Logging, Monitoring, and Visibility

Logging is enabled at the network/control plane level:

- **AWS:** VPC Flow Logs sent to CloudWatch Logs.
- **Azure:** Diagnostic Settings on VNets to Log Analytics (metrics enabled).
- **GCP:** VPC Flow Logs enabled on subnets (to Cloud Logging).
- **OCI:** Example log group for network resources.

**Considerations:**

- In production, forward logs to a central SIEM or log analytics platform.
- Define retention policies in alignment with regulatory and investigative needs.
- Add alerts for:
  - Unusual network patterns.
  - Changes to security groups/firewall rules.
  - Failed authentication / privilege escalation attempts.

---

## 5. Data Protection

These templates do **not** create storage or data services directly (databases, object storage, etc.). However, when extended:

- Enforce encryption at rest with cloud-native KMS/Key Vault/Key Management.
- Enforce encryption in transit (TLS) for application tiers.
- Apply appropriate key management policies (rotation, separation of duties, access control).

---

## 6. Organizational Guardrails (Layer 1)

These templates operate at **Layer 2**. They are intended to sit under stronger, org-level controls such as:

- **AWS:** Organizations, Service Control Policies (SCPs), Control Tower / Landing Zone Accelerator.
- **Azure:** Management Groups and Azure Policy.
- **GCP:** Organization node, Folder hierarchy, Org Policy constraints.
- **OCI:** Tenancy-level policies, compartment strategy.

Those controls should enforce:

- Allowed regions and services.
- Baseline logging and encryption defaults.
- Restrictions on creating internet-facing resources.
- Guardrails for IAM changes, key usage, and network changes.

---

## 7. Secrets and Configuration Management

These templates do not configure:

- Secrets stores (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, OCI Vault).
- Configuration management or drift detection.

In real environments:

- Provision a secret management solution early.
- Disallow storing secrets in Terraform variable files committed to Git.
- Consider configuration scanning and policy-as-code (e.g., tfsec, Checkov, OPA/Conftest).

---

## 8. Threat Model Highlights

The primary threats this Layer-2 design helps mitigate:

- **Unmonitored lateral movement** via lack of network logging.
- **Flat networks** without segmentation between public and private tiers.
- **Uncontrolled changes** via lack of baseline IAM/RBAC patterns.
- **Low visibility** into network flows and control-plane events.

Threats not fully addressed at this layer and requiring additional controls:

- Compromised privileged identities (handled via Layer-1 + IdP).
- Application-level vulnerabilities and OWASP Top 10.
- Supply-chain risks in CI/CD and dependencies.
- Advanced persistent threats and targeted campaigns.

---

## 9. Limitations and Responsibilities

This repository:

- Does not claim to be compliant with any specific standard.
- Does not replace formal security architecture, risk assessment, or threat modeling.
- Must be adapted, extended, and integrated into an organization’s broader security program.

The user is responsible for:

- Reviewing and modifying all configurations.
- Applying appropriate regulatory and internal controls.
- Performing testing, validation, and ongoing monitoring.

