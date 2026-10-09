---
name: serverless-security
description: "Audit and harden FaaS/serverless architectures against IAM over-privilege, event injection, insecure secrets, and supply chain risk."
model: opus
metadata:
  version: 1.0.0
  category: cloud-security
  source: "Learning Serverless Security (O'Reilly)"
---

# Serverless Security

## Goal
Audit and harden serverless (FaaS) architectures on AWS, Azure, and Google Cloud against the misconfigurations attackers chain together most often: over-privileged function identities, event-source injection, insecure secrets, exposed storage, and compromised dependencies or pipelines, while closing the shared-responsibility gaps the platform does not cover for you.

## When to Use
- Reviewing or designing a Lambda, Azure Functions, or Cloud Run functions/Cloud Run service architecture before or after launch.
- A function, execution role, managed identity, or service account appears to have broader permissions than its single task requires.
- Any event source (API Gateway, function URL, storage bucket upload, queue, webhook) feeds attacker-influenced data into function code.
- Investigating a suspected serverless incident: unexplained IAM users, disabled CloudTrail/audit logs, replaced function code, or anomalous Bedrock/LLM invocations.
- Setting up or reviewing CI/CD pipelines that build, scan, or deploy serverless workloads.
- Before adding or upgrading a third-party library/package used inside a function.

## When NOT to Use
- Traditional always-on VM or container-orchestration workloads with no FaaS or event-driven managed-service component (use container/K8s-focused guidance instead).
- Pure frontend/client-side security review with no backend serverless resources in scope.

## Authorization Check
- Confirm the account/subscription/project under review is one you are authorized to assess (reference `.claude/security-scope.yaml` for any active engagement scope).
- For production environments, confirm change windows and rollback plans before modifying IAM roles, trust policies, VPC attachments, or bucket ACLs; least-privilege tightening can break legitimate workflows if rolled out without validation.
- For incident investigation, confirm you have log read access (CloudTrail, Azure Activity Log/Monitor, Google Cloud Audit Logs) before concluding a resource is clean.

## Methodology
1. **Map the event-driven attack surface**: Inventory every trigger (HTTP API Gateway route, function URL, storage bucket event, queue/topic, scheduled event, webhook) per function. Flag function URLs and "temporary" staging endpoints first: they are easier to misconfigure than API Gateway and are often left public or embedded in frontend JavaScript/config bundles where anyone can extract identifiers (Cognito pool/client IDs, API keys, bucket names) via browser dev tools.
2. **Audit identity and permissions (least privilege)**: For every function execution role / managed identity / service account, list actual API calls needed versus granted. Specifically check for: wildcard `sts:AssumeRole` or `*` resource/action grants, IAM groups or roles that let a compromised function assume a more privileged role, self-registration left enabled on identity providers (e.g., Cognito `AdminCreateUserOnly: false`), and identity-pool-mapped roles broader than the authenticated user needs. Remediate by removing the escalation path (deny `AssumeRole` on the group, tighten the trust policy's `Principal`/`Condition`, scope IAM policies to specific resources/actions) and re-verify with the same credentials that escalation no longer works.
3. **Validate input handling across every event source, not just the API**: Treat all event payloads as untrusted, including file names/content in storage-upload triggers, webhook bodies, queue messages, and LLM-generated queries (text-to-SQL/prompt injection). Hunt for `eval()`/`exec()`, unsafe `yaml.load()`, `pickle.loads()`, string-built shell commands (`os.system`, `subprocess.run(shell=True)`), and server-side template injection (Jinja2/similar) reachable from any of these sources. Replace dynamic evaluation with safe parsers (e.g., an AST-restricted evaluator or a dedicated expression-parser library) and reject anything outside an explicit allowlist of operations.
4. **Secure secrets and credentials end to end**: Flag hardcoded keys in function code, frontend bundles, environment variables, and config files uploaded to storage. Require a managed secrets service (Secrets Manager, Key Vault, Secret Manager) with encryption at rest/in transit, fine-grained IAM-bound access, automatic rotation, and audit logging; confirm the function's role can only read the specific secrets it needs, not the whole vault or project.
5. **Lock down storage and network boundaries**: Check storage buckets for public read/write, missing bucket policies enforced via IaC, and dangling references to deleted buckets/domains that enable takeover. For network isolation, verify functions needing access to private resources are attached to a VPC/VNet with a NAT gateway or equivalent, rather than being moved to a public subnet for convenience; confirm outbound access is restricted so a successful injection cannot exfiltrate data or reach attacker infrastructure.
6. **Harden CI/CD pipelines and the software supply chain**: Verify build/deploy pipelines do not hold standing full-admin credentials, that container image builds pull from pinned, scanned base images, and that dependency installs are checked against known-malicious versions (maintain or run a bad-versions check against the lockfile) in addition to standard vulnerability scanning. Treat any pipeline capable of pushing to production as a high-value target equivalent to the production environment itself.
7. **Scan function code and dependencies with tooling**: Run Semgrep (prebuilt + custom rules for `eval()`-class sinks and hardcoded-secret patterns) against function source, and run OSV-Scanner (or equivalent SCA tool) recursively against lockfiles to catch known-vulnerable packages; triage true positives from false positives rather than disabling rules wholesale. Use Checkov or similar to audit IaC templates (Terraform/CloudFormation/Bicep) for storage, IAM, and network misconfigurations before they are ever applied.
8. **Instrument tracing, logging, monitoring, and alerting**: Confirm every function emits structured logs to a centralized, tamper-resistant sink (CloudTrail, Azure Monitor/Activity Log, Cloud Audit Logs), that log deletion/disabling itself raises an alert, and that you can answer: who accessed what data, what IAM actions ran, what permissions changed, and what code was deployed or replaced. Without this, privilege escalation, backdoored function versions, and credential exfiltration go undetected until the damage is done.
9. **Reconfirm the shared-responsibility boundary**: Document explicitly what the cloud provider secures (physical infrastructure, runtime patching, scaling) versus what remains your responsibility (IAM configuration, function code, event-source validation, secrets, network configuration, dependency hygiene, logging). New or preview serverless services often ship with incomplete security feature sets; evaluate a service's current feature set before using it in production and plan compensating controls for gaps.

## Output Format
- A prioritized finding list mapped to OWASP Serverless Top 10 categories (insecure deployment configuration, broken authentication, insecure serverless function permissions, event-data injection, improper exception handling, inadequate monitoring/logging, insecure third-party dependencies, insecure application secrets storage, denial of service/denial of wallet, function execution flow manipulation) with CWE references (CWE-78 command injection, CWE-94/95 code/eval injection, CWE-284/285/639 authorization, CWE-798 hardcoded credentials, CWE-502 insecure deserialization).
- Remediated IAM policy/trust-policy JSON, VPC/network configuration diffs, and secure code replacements (e.g., eval() to AST-based or library-based expression parsing).
- Semgrep rule files (prebuilt + custom), an OSV-Scanner or SCA report, and a bad-known-versions dependency check artifact.
- A tracing/logging gap list with the specific log sources needed to answer the nine audit questions in Methodology step 8.

## Quality Check
- Every function's execution role/managed identity grants only the specific actions and resources that function's code actually calls, nothing broader.
- No event source (HTTP, storage, queue, webhook, LLM-generated query) can reach a dynamic evaluation, deserialization, or shell-execution sink without passing through an allowlist-based validator.
- Re-run the exact escalation technique you found (e.g., AssumeRole chaining, Cognito self-registration, managed-identity token exchange) after remediation and confirm it now fails with an authorization error.
- Secrets scan and dependency/SCA scan both return zero findings, or remaining findings are explicitly risk-accepted and tracked.
- Log/audit trail exists for IAM changes, secret access, and function code deployment, and deleting or disabling that trail itself triggers an alert.

## Common Issues
- Treating a serverless function's ephemerality as a security control: attackers adapt persistence to IAM (new users, new access keys, backdoored function versions) instead of filesystem backdoors, so IAM audit trails matter more than host-based detection here.
- Assuming "no servers" means no network or IAM configuration to secure; VPC/VNet attachment, NAT, and trust policies still apply and are frequently skipped for convenience.
- Granting a function admin-level or wildcard permissions "just to get it working" during development and never revisiting it before production.
- Leaving default identity-provider settings (e.g., self-registration enabled) unreviewed, letting attackers create their own authenticated principal.
- Fixing the obvious code-injection vector (`eval()`) while leaving the same unvalidated event data reachable through deserialization, template rendering, or shell invocation elsewhere in the same function.
- Relying solely on automated scanners (Semgrep, OSV-Scanner, Checkov) without manual review; false positives and false negatives both occur, and scan noise can mask the one true positive that matters.
- Forgetting that CI/CD pipeline credentials are themselves a production-equivalent attack surface, especially for pipelines building custom container images with broad deploy permissions.
