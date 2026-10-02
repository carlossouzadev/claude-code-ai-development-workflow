---
name: security-as-code
description: "Codify security controls as policy-as-code gates across IaC, CI/CD, logging, IAM, and resilience testing for AWS and open source stacks."
model: sonnet
metadata:
  version: 1.0.0
  category: devsecops
  source: "Security as Code (O'Reilly)"
---

# Security as Code (DevSecOps Patterns with AWS)

## Goal
Embed security as executable, version-controlled policy throughout the SDLC, preventing misconfigured infrastructure and insecure code from ever reaching production, instead of relying on after-the-fact manual review.

## When to Use
- Standing up or hardening a CI/CD pipeline that provisions cloud infrastructure (CloudFormation, Terraform, CDK) and needs automated security gates.
- Introducing IaC scanning (SAST/DAST/SCA/secrets-scanning equivalents for infra) before a resource can be deployed.
- Designing IAM/least-privilege automation, logging/monitoring baselines, or resilience (chaos/fault-injection) testing for a DevSecOps program.
- Building or auditing the people/process side of a DevSecOps team (roles, RACI, threat modeling cadence, SMART goals).

## When NOT to Use
- Application-level code vulnerability review unrelated to infrastructure or pipeline security (use an application security or code-review skill instead).
- One-off, unversioned manual cloud changes with no intent to automate or repeat them.
- Incident response during an active breach (use incident-response tooling, not this planning/prevention skill).

## Authorization Check
- Confirm you have authority to modify the CI/CD pipeline, IAM policies, and AWS account configuration involved (this book's model: AWS-native tooling plus open source scanners).
- Confirm with security/compliance stakeholders which baseline standard applies (CIS Benchmarks, NIST 800-53, Cloud Security Alliance CCM) before codifying new gates, since conformance packs and SCPs can become org-wide and hard to reverse.
- For production environments, confirm change management and rollback procedures per this repo's Environments & Deployment policy (Dev to Staging to Production, approval required for Production).

## Methodology
1. **Secure the pipeline's front door (prevent unwanted access)**
   - Store all IaC in a remote, versioned repository (CodeCommit/Git).
   - Scope IAM policies to the repository and region with explicit conditions (`aws:RequestedRegion`), not blanket `*` access.
   - Treat the pipeline's own security checks as a protected asset: nobody should be able to toggle scanners off via a commit. Any "urgent bypass" must be logged and remediated, never silently allowed.

2. **Detect misconfigurations before correcting them (shift left)**
   - Identify a baseline standard (CIS Benchmarks for cloud-agnostic hardening, NIST 800-53 for compliance, CSA Cloud Control Matrix for a control library) and track its updates.
   - Build a STRIDE threat model per workload: Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege. Map each risk to a named control.
   - Classify every control as preventive, detective, or corrective (in that priority order). Preventive stops a bad resource from ever launching; detective (SecurityHub, GuardDuty, Config) finds it after the fact; corrective restores known-good state. Favor preventive first, then layer detective as a feedback loop that tells you where preventive controls are missing.

3. **Wire preventive checks into a CI/CD decision gate**
   - Pipeline shape: commit to main triggers a build stage; the build stage runs an IaC scanner (for example cfn-nag for CloudFormation, equivalent linters for Terraform/CDK); only a passing scan reaches the deploy stage; a failing scan cancels deployment and returns actionable, line-numbered findings to the developer.
   - Attributes every automated security check needs: idempotence (same input, same result every run), baseline coverage (applies broadly, not to a rare resource variant, to avoid alert fatigue), and a recommended fix (never leave a failure unexplained).
   - Extend the same pattern to IAM policy review: lint submitted IAM JSON (for example with Parliament or IAM Access Analyzer's policy validation API) for over-broad `Resource: "*"` or action wildcards before granting.

4. **Instrument logging and monitoring as a detective layer**
   - Separate logging (capturing events) from monitoring (acting on them) from metrics (benchmarked measurements) from dashboards (visualization). Standardize log schemas (structured JSON, consistent fields) to enable correlation.
   - Cover both infrastructure logs and application logs; route them through a SIEM-equivalent, dashboards, and incident-management tooling.
   - Enable anomaly detection on metric filters (for example CloudWatch anomaly detection) and encrypt log storage at rest (KMS); remember key rotation does not retroactively encrypt old data.
   - Use CloudTrail (or equivalent control-plane audit log) correlation to catch unauthorized or unreviewed changes, such as a security group opened outside the approved pipeline, and trigger automated remediation (SSM automation documents or equivalent) rather than manual reversion.
   - Enable VPC Flow Logs (or network equivalent) to validate security group (stateful) and network ACL (stateless) behavior matches intent.

5. **Automate least-privilege IAM as code**
   - Apply the principle of least privilege to both human and machine identities; machine identities should do the overwhelming majority of production work, with human access minimized closer to production.
   - Use a tagging system (key/value, for example `team:gamedev`) as the basis for attribute-based access control and to scope automated actions (so a cleanup/deletion automation cannot delete resources tagged as core infrastructure).
   - Layer service control policies (account/org-wide allow or deny) and permissions boundaries (caps an IAM role's maximum possible permissions) as preventive controls; pair with IAM Access Analyzer and GuardDuty as detective controls.
   - Pipeline this: policy change committed, scanner lints it, failures block merge with a specific remediation (for example, replace `Resource: "*"` with a scoped ARN).

6. **Validate resilience with fault injection and chaos engineering**
   - Define a measurable steady state (technical metrics: latency, CPU, error rate; business metrics: failed logins, failed transactions) before testing.
   - Form a hypothesis that steady state holds under injected failure, inject a real-world variable (instance termination, API throttling, security group rule removal, EBS volume detachment, CPU stress), then try to disprove the hypothesis by comparing control vs. experimental behavior.
   - Advance from staging to production experiments only once confidence is established; automate experiments to run continuously (scheduled, like cron); minimize blast radius with canary-style rollout (2% then 5%, 10%, 25%... to 100%).
   - Codify each experiment as an IaC template (for example AWS Fault Injection Simulator JSON) in version control, wired into the CI/CD pipeline with stop conditions (CloudWatch alarms) so runaway experiments self-terminate.

7. **Operationalize people and process, not just tooling**
   - Staff a DevSecOps function with security engineers (generalists over narrow specialists), developers (own and maintain the toolchain long-term, not a one-off side project), a compliance team member (engaged from day one, not only at audit time), and a product-owner-equivalent role to prioritize and communicate.
   - Hold product/dev teams accountable for remediating their own findings; security assists and looks for systemic patterns that justify a new preventive control, rather than becoming a remediation bottleneck.
   - Maintain a control matrix (start from CCM, extend with threat-modeling findings) and reusable secure patterns (for example a standardized secure SSH pattern) to avoid re-deriving the same guidance per team.
   - Report status consistently using highlights / lowlights / trends / upcoming objectives / blockers, and set SMART, quarterly, metrics-backed goals.

## Output Format
- A CI/CD pipeline definition (CodePipeline/CodeBuild, GitHub Actions, GitLab CI, or equivalent) with a scanning stage that fails the build on misconfiguration.
- IaC templates (CloudFormation/Terraform/CDK) for the protected repository, IAM roles, and the resource under test, plus a scanner buildspec/config.
- A STRIDE threat model table and a control matrix (control to risk to control type: preventive/detective/corrective) per workload.
- Logging/monitoring configuration (log group, metric filters, anomaly detectors, alarms) and an IAM tagging/permission-boundary policy set.
- Fault-injection experiment templates (JSON/YAML) with defined steady-state metrics and stop conditions.
- A RACI-style status report template and a documented control matrix for compliance handoff.

## Quality Check
- Every new IaC resource type has at least one preventive check wired into the pipeline before it can be merged to main.
- Security checks are idempotent, apply to a meaningful baseline of resources (not a rare edge case), and emit an actionable recommendation on failure.
- No one (including security) can disable a pipeline's scanning stage via a normal commit; emergency bypasses are logged and followed by remediation.
- IAM policies contain no unscoped `Resource: "*"` or wildcard actions without a documented, reviewed exception.
- Chaos/fault-injection experiments have explicit stop conditions tied to real alarms, and have been run in staging before being promoted to production.
- Compliance/control matrix is current and traceable to a named standard (CIS, NIST 800-53, CCM) plus any threat-modeling-derived controls beyond that baseline.

## Common Issues
- Treating security as "the security team's problem": remediation ownership belongs to the team that built the resource, security augments and finds systemic fixes.
- Detective-only posture: detection services (GuardDuty, Config, SecurityHub) without preventive gates leave a window where misconfigurations are live and exploitable.
- Over-permissive "temporary" access that is never revisited; permissions granted under deadline pressure tend to become permanent without a tagging or review cadence.
- Treating DevSecOps tooling as a side project for rotating developers; unmaintained scanners and checks silently go stale and get bypassed.
- Running chaos experiments without a defined steady state or stop condition, turning a controlled test into an unplanned outage.
- Buying security tools without a clear risk statement ("Lack of X leads to loss of Y because of Z"); unused or half-configured tools add cost without reducing risk.
- KMS/encryption applied only going forward: rotating or adding a key does not retroactively encrypt previously stored data.
