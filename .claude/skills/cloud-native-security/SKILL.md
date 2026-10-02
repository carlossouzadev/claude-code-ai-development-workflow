---
name: cloud-native-security
description: "Design and audit secure cloud landing zones, IAM, networking, encryption, logging, and policy-as-code guardrails across AWS, Azure, and GCP."
model: opus
metadata:
  version: 1.0.0
  category: cloud-security
  source: "Cloud Native Security Cookbook (O'Reilly)"
---

# Cloud Native Security Patterns (AWS, Azure, GCP)

## Goal
Stand up and harden a cloud estate using provider-native, infrastructure-as-code
guardrails so that security scales with the business rather than gatekeeping it:
scalable account/project hierarchies, least-privilege identity, segmented
networking, encryption and secrets management, centralized visibility, and
policy-as-code that prevents, detects, and remediates drift.

## When to Use
- Standing up a new cloud landing zone, account factory, or project/subscription
  hierarchy for a new team or business unit.
- Reviewing or hardening IAM, service accounts, or cross-account access patterns.
- Designing network segmentation (hub-spoke, shared VPC/VNet, private access).
- Implementing encryption at rest/in transit, key management, or secrets handling.
- Building centralized logging, a security operations center, or anomaly alerting.
- Writing organization-level guardrails (Organization Policy, SCP, Azure Policy)
  or policy-as-code checks (Checkov, OPA/Conftest) in CI/CD pipelines.
- Designing automated remediation for noncompliant cloud resources.

## When NOT to Use
- Application-level code review (SAST/dependency scanning) unrelated to cloud
  account, network, or resource configuration.
- Incident response or active breach investigation (use a DFIR-focused skill).
- On-premises-only infrastructure with no cloud provider involved.

## Authorization Check
- Confirm this falls under the user's existing infra/security decision authority
  (per project memory, the user owns security+infra decisions directly) before
  applying org-wide guardrails such as SCPs, Organization Policies, or Azure
  Policy assignments that affect every account/subscription in scope.
- For production changes, confirm the Dev -> Staging -> Production approval flow
  and who signs off before applying anything at the organization root, management
  group, or org/billing account level.
- Never embed real account IDs, subscription IDs, tenant IDs, or key material in
  generated code; use variables and confirm secrets flow through a secret manager
  (Secret Manager, AWS Secrets Manager/SSM, Key Vault), never plaintext.

## Methodology

1. **Establish the resource hierarchy (landing zone)**
   - GCP: Organization -> Folders (Bootstrap, Common, Production, NonProd, Dev)
     -> Projects. Each team gets four projects: production, preproduction,
     development, shared. Use `Import`/`Export` folders when migrating projects
     between organizations.
   - AWS: Organization -> Organizational Units (Security, Workload, Infrastructure)
     -> Accounts. Security OU holds log archive, security-tooling, read-only, and
     break-glass accounts plus a quarantine OU. Workload OU is the parent for
     per-team OUs with production/preproduction/development/shared accounts.
   - Azure: Management Groups -> Subscriptions -> Resource Groups, mirroring the
     same production/nonprod/dev/shared split.
   - Maintain two organizations/tenants (prod and test) so org-wide policy changes
     (a new Organization Policy, SCP, or Azure Policy) can be validated before
     rollout to the real estate.
   - Apply a region-locking guardrail (GCP Organization Policy constraint, AWS SCP
     deny-outside-region condition, Azure Policy allowed-locations) at the
     organization/management-group root so teams cannot provision outside
     approved regions regardless of console or API access.

2. **Centralize and harden identity**
   - Centralize human identity in one directory (Google Workspace/Cloud Identity,
     AWS IAM Identity Center with a dedicated identity account plus cross-account
     roles, Azure AD/Entra ID) and assign access via groups, never individual
     users, to keep management tractable at scale.
   - Apply permissions as high in the hierarchy as possible (e.g., `roles/viewer`
     at a folder/OU) and grant broader access only lower down for a specific
     need (e.g., `roles/editor` on a Development project), observing least
     privilege by default with explicit, auditable exceptions.
   - Treat service accounts/managed identities as first-class: scope each to a
     single purpose, avoid long-lived downloaded keys (prefer workload identity
     federation, managed identities, or short-lived impersonation), and give
     key-management identities and key-usage identities separate, narrower roles.
   - Reserve a security read-only account/role for investigation and a
     break-glass account/role for emergencies; both should be tightly audited.
   - Use SCPs (AWS) / Organization Policy (GCP) / Azure Policy plus RBAC (Azure)
     to lock down actions that must never be reversible by any in-account
     principal, such as deleting flow logs or modifying a protected IAM role,
     with narrow conditional exceptions for a privileged role only.

3. **Segment the network with defense in depth**
   - Build a hub-and-spoke (or shared VPC/VNet) topology: a central
     networking/transit project or account routes traffic between spokes and to
     on-premises, while spokes hold workloads. Use Transit Gateway (AWS),
     Shared VPC (GCP), or hub VNet with peering (Azure) as the backbone.
   - Segment subnets by exposure (public, private, internal, firewall/transit)
     and attach a dedicated security group/firewall policy and route table to
     each. Route all internet-bound egress through a managed firewall
     (Cloud NAT + firewall rules, AWS Network Firewall, Azure Firewall); deny by
     default and allow narrowly.
   - Prefer private connectivity to internal resources over bastion/VPN-only
     trust: SSH/RDP via identity-aware proxy (GCP IAP, AWS SSM Session Manager,
     Azure Bastion) instead of open inbound ports, and Private Service
     Connect/PrivateLink/Private Endpoint for reaching managed services without
     traversing the public internet.
   - Defense in depth means never trusting network position alone: identity
     checks, firewall rules, and routing should be independent, layered
     controls so a compromise at one layer does not cascade.
   - Turn on flow logs and a network diagnostics service (VPC Flow Logs,
     Network Watcher, VPC traffic mirroring) per network, exported to the
     centralized logging destination, not left at default retention.

4. **Encrypt data and manage keys/secrets**
   - Default to provider-managed encryption at rest (SSE, Cloud KMS defaults,
     Azure Storage Service Encryption) for everything; layer customer-managed
     keys (CMEK/CMK) via Cloud KMS, AWS KMS, or Azure Key Vault for data with
     compliance or blast-radius requirements.
   - Use multiple keys scoped to sensitivity level or workload, not one shared
     key, to limit blast radius; separate the identity that manages keys
     (create/delete) from identities that only use keys (wrap/unwrap/get),
     granting each the minimum operations required.
   - Enforce in-transit encryption (TLS-only policies, HTTPS-only storage
     accounts, enforced SSL on managed databases) as a baseline, not optional.
   - Use DLP tooling (Cloud DLP, Macie-equivalent, Azure Purview/DLP) to find
     and classify sensitive data, then apply policy that flags or blocks
     resources handling that data when encryption/CMEK is missing.
   - Centralize secrets (Secret Manager, AWS Secrets Manager/SSM Parameter
     Store, Azure Key Vault secrets) with rotation and scoped access; never
     pass secrets through environment variables checked into IaC state.

5. **Get centralized visibility (cloud SOC and log aggregation)**
   - Stand up the provider's native SOC surface: Security Command Center (GCP),
     Security Hub + GuardDuty (AWS), Microsoft Defender for Cloud + Sentinel
     (Azure). Treat the free/standard tier (misconfiguration + anomaly findings)
     as a floor, and budget for the premium tier (threat detection, compliance
     benchmarks like CIS/PCI DSS/NIST 800-53/ISO 27001) where risk warrants it.
   - Wire findings into a real-time, low-noise notification path (Pub/Sub ->
     Cloud Function, EventBridge -> Lambda, Event Grid -> Azure Function) so
     findings reach a human or automation, not just a dashboard nobody watches.
     A single pane of glass with no alerting is a tree falling with no one
     around to hear it.
   - Centralize logs into one dedicated logging account/project/subscription as
     an immutable, append-only destination, separate from workload accounts, so
     a compromised workload cannot tamper with its own audit trail.
   - Build or adopt an infrastructure registry (Cloud Asset Inventory, AWS
     Config aggregator, Azure Resource Graph) as the queryable source of truth
     for what resources exist, their configuration, and their change history.
   - Configure log anomaly alerting (Event Threat Detection, GuardDuty,
     Sentinel analytics rules) tuned to minimize noise; alert fatigue defeats
     the purpose of centralization as surely as no alerting at all.

6. **Enforce guardrails as code (prevent, detect, remediate)**
   - Prevent first: apply organization-level guardrails (GCP Organization
     Policy, AWS SCP, Azure Policy with deny effect) wherever an exact policy
     exists for the requirement. These cannot be bypassed via IAM and apply
     identically whether the change comes from the console or automation; they
     are your strongest and first choice.
   - Shift left in the pipeline: run policy-as-code tools (Checkov, OPA/
     Conftest, Sentinel, cloud-specific linters) against Terraform/IaC before
     merge, so noncompliant infrastructure is rejected before it is ever
     applied, not found and fixed after the fact.
   - Detect what prevention missed: use the infrastructure registry and
     SOC/compliance tooling (Security Health Analytics, AWS Config Rules,
     Azure Policy audit mode) to continuously scan deployed resources against
     your compliance baseline, tagged/labeled by owner, environment, and data
     sensitivity for fast triage.
   - Tag/label every resource at creation time (GCP labels, AWS tags, Azure
     tags) with owner, environment, and cost center; compliance and
     remediation automation depends on being able to scope its blast radius by
     these tags.
   - Remediate automatically only where it is the right lever: trigger
     event-driven remediation (asset-change feed -> Cloud Function/Lambda/
     Azure Function) for high-security-risk, low-operational-risk findings such
     as a bucket becoming public. Reserve automated remediation as a last line
     of defense for critical issues, not a substitute for prevention.

7. **Weigh the tradeoffs of automated remediation before building it**
   - Automated remediation fights infrastructure as code: it introduces drift
     between declared state and live state. Where teams already use IaC,
     prefer blocking the bad change at the pipeline (Checkov/Conftest) over
     silently reverting it afterward.
   - It carries an ongoing maintenance burden of its own (the remediation
     function is itself a piece of production code with its own permissions
     and failure modes); do not build it for findings that are rare or easily
     fixed by hand.
   - It can rob the end user of the learning opportunity to configure the
     resource correctly next time; for findings where correct configuration
     knowledge matters (e.g., public bucket exposure), favor guided prevention
     (policy-as-code feedback) over silent auto-fix. For low-stakes,
     high-volume housekeeping (e.g., deactivating stale unused credentials),
     silent remediation is fine.
   - Segment remediation scope and privilege tightly: the remediation
     function's service account/role should hold only the specific permissions
     needed to fix the one finding class it targets, scoped to the target
     projects/accounts, nothing broader.

8. **Treat the pipeline as the DevSecOps control plane**
   - Security's goal is enablement, not gatekeeping: minimize the size of each
     change and the lead time to fix issues, since total risk exposure from a
     vulnerability is bounded by how long it stays live, not purely by whether
     it existed.
   - Embed security checks (dependency scanning, SAST, policy-as-code, secret
     scanning) directly into CI/CD so they run on every change automatically,
     rather than as a manual gate (e.g., a mandatory multiweek penetration test
     per release) that is incompatible with frequent deployment.
   - Measure the program, not just individual findings: track percentage of
     compliant infrastructure, time to notify and time to fix known
     vulnerabilities, percentage of changes rejected on security grounds (should
     trend down as secure-by-default becomes the path of least resistance), and
     attempted-breaches-prevented at a component level.

## Output Format
- Terraform (or equivalent IaC) modules implementing the chosen pattern:
  organization/folder/account/subscription resources, IAM/role bindings,
  network topology, KMS/Key Vault keys and access policies, logging sinks,
  and policy/guardrail resources (Organization Policy, SCP, Azure Policy
  assignment).
- A short architecture note naming the provider-specific services used and
  mapping them to the pattern (e.g., "hub-spoke via Shared VPC + Cloud NAT",
  "SCP denying flow-log deletion at the org root").
- Policy-as-code rule files (Checkov custom checks, Rego/OPA policies) wired
  into the CI/CD pipeline where prevention is the chosen control.
- A compliance/remediation design note stating which findings are
  prevent-only, detect-and-alert, or auto-remediated, and why.

## Quality Check
- Every account/project/subscription in the hierarchy maps to a named owner,
  environment, and the four-tier split (production/preproduction/dev/shared)
  unless explicitly justified otherwise.
- No long-lived, downloaded service-account/access keys where workload
  identity federation or managed identity is available.
- Every network segment has an explicit default-deny posture verified, not
  assumed; internet egress is traceable through a named firewall/NAT resource.
- Every encryption-at-rest resource either uses the provider default or has an
  explicit, access-scoped CMEK/CMK with separate manage vs. use identities.
- Guardrails are applied as close to the root (org/management group) as
  possible when available, with pipeline-level policy-as-code as the
  complementary, not substitute, control.
- Any proposed auto-remediation has a documented scope, a named trigger, and
  an explicit answer to "does this fight the team's IaC."

## Common Issues
- Treating a SOC/dashboard (SCC, Security Hub, Defender) as done once enabled,
  without wiring real-time, low-noise alerting into an actual response path.
- Granting broad roles (Editor/Owner/Contributor) at a high scope "to unblock
  the team" and never revisiting it; permissions are easy to extend and rarely
  revoked, so audit grants on a schedule, not only when things break.
- Sharing one encryption key across unrelated workloads, defeating blast-radius
  containment, or mixing key-management and key-usage permissions on one
  identity.
- Building automated remediation before prevention: fix the pipeline gate
  first, and only add remediation for findings that genuinely recur despite
  prevention.
- Applying org-wide policy changes (SCP, Organization Policy, Azure Policy)
  directly to the single production organization/tenant instead of validating
  in a second test organization/tenant first.
- Forgetting that SCP/Organization Policy/Azure Policy denies do not cover
  every surface (e.g., AWS SCPs do not apply to the organization management
  account itself, service-linked roles, or external principals); complement
  with resource-level IAM conditions where the guardrail has gaps.
