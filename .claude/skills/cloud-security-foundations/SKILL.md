---
name: cloud-security-foundations
description: "Assess or design cloud security programs across shared responsibility, asset/data inventory, IAM, network, encryption, detection, vulnerability, and incident response controls."
model: opus
metadata:
  version: 1.0.0
  category: cloud-security
  source: "Learning Cloud Security + Practical Cloud Security, 2nd Edition (O'Reilly)"
---

# Cloud Security Foundations

## Goal
Give a cloud program (AWS, Azure, GCP, or multicloud) a provider-agnostic, prioritized path to a defensible security posture, from understanding what you are and are not responsible for, through asset and data inventory, IAM, network, encryption, detection, vulnerability management, and incident response, governed by a risk and compliance program.

## When to Use
- Standing up security for a new cloud account, landing zone, or application, or reviewing an existing one with no clear control baseline.
- Someone asks "what should we fix first" in a cloud environment with limited security budget or headcount.
- Scoping a cloud risk assessment, security architecture review, or compliance readiness effort (SOC 2, ISO 27001, PCI DSS, HIPAA, FedRAMP).
- Investigating why a breach happened and the root cause traces to an unclear ownership boundary with the cloud provider.
- Building the control catalog or policy set (data classification, IAM, network, BCDR) that other teams will implement against.

## When NOT to Use
- Deep implementation work for a single named control that already has its own specialized skill or provider documentation (specific IAM role syntax, a particular WAF rule set, a specific SIEM query language).
- Offensive testing or exploit development; this skill is defensive and architectural only.
- Pure application-layer secure coding review (OWASP Top 10 remediation in source code) with no cloud infrastructure dimension.
- Decisions that are purely cost or performance optimization with no security implication.

## Authorization Check
- Confirm you have a mandate to review or design controls for the specific account, subscription, project, or organization in scope, and that the engagement is documented (ticket, SOW, or management request).
- For anything touching production credentials, encryption keys, or incident response tooling, confirm change windows and who can approve "break the glass" access.
- For compliance-scoped work, confirm whether legal/compliance owns the regulatory interpretation; this skill informs security design, not legal sign-off.

## Methodology
1. **Map the shared responsibility model and trust boundaries**: For each workload, identify whether it is IaaS, PaaS, or SaaS and draw the line between provider-owned and customer-owned layers (physical infrastructure and virtualization are always the provider's; data access is always yours; operating system, middleware, application, and network are shared or yours depending on the delivery model). Most breaches trace back to an unstated assumption that "the provider handles that." Sketch the application as boxes and arrows (users, admins, each component), draw dotted trust boundaries around groups that implicitly trust each other, and flag every line that crosses a boundary as a place requiring authentication and authorization. Name likely threat actors (organized crime, hacktivists, insiders, state actors) so later control choices match real motives, not generic ones.
2. **Inventory and classify data and cloud assets**: Identify data assets first (customer data, credentials, keys, logs, source code) and classify into a small scheme (for example low/public, moderate/private, high/confidential), driven by regulatory triggers (GDPR, PCI DSS, HIPAA, ITAR). Then inventory cloud assets: compute (VMs, containers, aPaaS, serverless), storage (block, file, object, databases, queues, secrets and key stores, certificate stores), and network (VPCs/subnets, DNS, TLS certs, load balancers). Build an asset management pipeline and check it for leaks at each stage: procurement (missed providers or shadow IT), processing (missed asset types within a provider), tooling (assets not fed into scanners), and findings (findings generated but never remediated). Apply a single tagging standard (owner, data class, environment, function) across every provider so automation and audits stay consistent.
3. **Lock down identity and access management first**: IAM is the top-priority control because stolen or excessive credentials are the most common breach vector. Implement the full life cycle, not just login: request, approve, create/grant, authenticate, authorize, and revalidate (offboarding feeds, positive confirmation for high-impact access). Enforce least privilege (deny by default) and separation of duties. Require MFA or passwordless (FIDO2/passkeys) for all privileged and sensitive access; prefer phishing-resistant factors over SMS/TOTP where risk warrants it. Use federation and SSO (SAML/OIDC) instead of per-app passwords. Manage system-to-system secrets with a dedicated secrets manager or instance identity documents, never embedded in code or images; use privileged access management (PAM/PIM) for shared or break-glass IDs with session recording.
4. **Manage vulnerabilities and cloud security posture (CSPM)**: Cover every layer of the stack per the shared responsibility map: application code and dependencies (SBOM, SCA, SAST/DAST, OWASP Top 10), middleware/platform configuration (CIS Benchmarks, drift detection), operating system patching and hardening, container and orchestrator configuration (Kubernetes RBAC, pod security), and network device patching. Feed every discovered asset into scanners (the "tooling leak" from step 2) and track findings to closure or documented risk acceptance (the "findings leak"). Favor pipeline-based remediation (infrastructure as code, CI/CD, immutable images, blue/green or canary rollout) over manual patch-and-pray, since smaller, more frequent changes lower both security and availability risk.
5. **Apply network controls as defense in depth, not the primary control**: Treat network security as a secondary layer behind IAM and vulnerability management, since in cloud the perimeter is porous by design. Use zero trust networking principles (never trust, always verify; mutual TLS; per-identity microsegmentation) instead of relying on a flat trusted-internal-network assumption. Segment with VPCs/subnets and security groups, enforce default-deny with explicit allow rules, use WAFs and reverse proxies for internet-facing apps, encrypt all data in motion, and apply egress filtering and DLP to catch exfiltration. A zero trust architecture is an operating model (identity-based policy engine plus continuous device/posture verification), not a single product.
6. **Encrypt data and manage keys properly**: Classify and encrypt data at rest, in motion, and, for the highest-sensitivity workloads, in use (confidential computing). Never store an unwrapped key next to the data it protects; use a two-tier key hierarchy (data encryption keys wrapped by key encryption keys) via a cloud KMS or dedicated HSM, and prefer encrypting as close to the application layer as the performance/functionality trade-off allows for your most sensitive data. Use cryptographic erasure (destroy the key, not the bulk data) for fast, reliable deletion, and plan a "break the glass" process for emergency key access that is itself logged and alerted.
7. **Build detection, logging, and monitoring before you need it**: Instrument every layer (cloud control-plane/API logs, network flow logs, OS and middleware logs, application logs, secrets-server access, privileged session recordings) and ship them off-host to a separate account/credential domain so an attacker who compromises production cannot erase the trail. Aggregate, parse, correlate, and alert (SIEM, with SOAR for automated playbooks), tune aggressively to control false positives without going silent, and alert on logging gaps too. Map detection coverage against an attack model (Lockheed Martin Cyber Kill Chain or the Mandiant-style attack lifecycle: recon, foothold, privilege escalation, internal recon, lateral movement, persistence, mission) and against the NIST Cybersecurity Framework functions (Govern, Identify, Protect, Detect, Respond, Recover) to find coverage gaps.
8. **Prepare and run cloud incident response**: Before an incident, name primary/backup technical and business leads, legal, comms, and HR contacts; pre-approve a response budget; know your cloud provider's incident commitments; and keep an isolated incident-response cloud account with forensic tooling and runbooks. During an incident, run the OODA loop (observe, orient, decide, act) in rapid cycles: triage by severity, contain without tipping off the attacker before scope is understood, track indicators of compromise until no new ones appear, then eradicate (ideally by rebuilding compute from clean images rather than patching in place, since cloud compute/storage separation makes this cheap) and recover, closing with root-cause analysis and control remediation.
9. **Govern the whole program with risk management and a prioritized roadmap**: Run risk management as likelihood times impact (quantitatively with FAIR's Loss Event Frequency x Loss Magnitude, or with NIST RMF's prepare/categorize/select/implement/assess/authorize/monitor cycle, or ISO 31000/27005), keep a risk register with explicit treatment (avoid, mitigate, transfer, accept) and residual risk, and avoid listing raw problems ("unpatched server") as if they were risks. Map compliance obligations (SOC 2, ISO 27001, PCI DSS, HIPAA, GDPR, CSA CCM/STAR) to the controls already built in steps 1 to 8 rather than treating compliance as a separate program. Cascade policies (business intent) into standards (implementation requirements) into procedures (how-to), and maintain BCDR separately from security operations: business continuity keeps the business running, disaster recovery restores full capability, both driven by a business impact analysis defining MTO/RTO/RPO per function. Sequence remediation work using the priority order validated above: IAM first, vulnerability management second, network controls third, with encryption, detection, and incident response threaded through all three, because this order yields the most risk reduction per dollar spent.

## Output Format
- A shared responsibility matrix per workload/delivery model.
- A data and cloud asset inventory with a classification scheme and tagging standard.
- An IAM control set (life cycle, MFA policy, secrets management approach, PAM scope).
- A network control diagram (segmentation, zero trust policy engine, egress/DLP points).
- An encryption and key management design (what is encrypted where, key hierarchy, break-glass process).
- A logging/monitoring architecture and SIEM alert coverage mapped to the attack lifecycle and NIST CSF functions.
- An incident response plan (team, playbooks, tools, severity levels).
- A risk register and a prioritized control roadmap (policy, standard, procedure documents as needed).

## Quality Check
- Every workload has an explicit, named owner for each shared-responsibility layer; nothing is assumed to be "handled by the provider" without checking the actual terms of service.
- No cloud asset type (compute, storage, network) or data store is missing from the inventory pipeline, and every inventoried asset is confirmed to be reachable by the relevant scanner or monitoring tool.
- IAM review shows deny-by-default, MFA on all privileged paths, no long-lived secrets in code or images, and a working offboarding feed.
- The risk register expresses risk as likelihood x impact with a named treatment, not a bare list of technical findings.
- Detection coverage is checked against at least one attack-stage model (kill chain or attack lifecycle) to confirm no phase is unmonitored.
- The remediation roadmap is explicitly prioritized (IAM, then vulnerability management, then network, with encryption/detection/IR woven through), not an unordered checklist.

## Common Issues
- Treating "cloud provider says it's encrypted" as sufficient without controlling who holds the keys and who can access them; checkbox encryption without real key management stops almost nothing.
- Securing the network perimeter heavily while leaving IAM loose; in cloud, credential compromise bypasses network controls entirely.
- Building a vulnerability scanning program that never looks at container images, serverless code, or IaC templates because tools were chosen for VM-era assumptions.
- Logging locally only, so an attacker who gains host access can erase evidence; always ship logs to a separately credentialed account.
- Writing policies with no measurable statements, or standards that do not trace back to any policy statement, producing compliance artifacts that do not actually reduce risk.
- Confusing compliance with security: passing an audit (SOC 2, PCI DSS) is not evidence that controls actually stop a real attacker, only that documented controls exist and were sampled.
- Running incident response without a tested plan, so the first real incident is also the first rehearsal; tabletop exercises before an incident are far cheaper than learning during one.
