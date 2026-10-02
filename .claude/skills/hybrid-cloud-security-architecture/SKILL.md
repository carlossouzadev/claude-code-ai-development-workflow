---
name: hybrid-cloud-security-architecture
description: "Architect zero trust based security for hybrid and multi-cloud systems using enterprise domains, threat modeling, shared responsibility mapping, and ADRs."
model: opus
metadata:
  version: 1.0.0
  category: security-architecture
  source: "Security Architecture for Hybrid Cloud (O'Reilly)"
---

# Hybrid Cloud Security Architecture

## Goal
Drive a system from enterprise context through a documented, zero trust based solution architecture for hybrid or multi-cloud workloads, producing a traceable set of artifacts (threat model, shared responsibility map, deployment architecture, ADRs, operations runbooks) that a design authority and security operations team can act on.

## When to Use
- Designing or reviewing a new hybrid/multi-cloud workload spanning on-premises and one or more CSPs.
- An MVP is being proposed without architectural characteristics (security, resilience, scale) having been considered, and you need to distinguish it from a PoC.
- Onboarding a new landing zone, cloud account, or CSP and needing to define enterprise, CSP, application/CI-CD, and operations level controls.
- Integrating zero trust practices (ZTNA, microsegmentation, adaptive access, continuous authentication) into an existing deployment.
- A significant design choice needs a documented, defensible rationale (ADR) for a design authority.
- Handing a workload to security operations and needing RACI, threat detection use cases, and incident response runbooks defined before go-live.

## When NOT to Use
- Firewall rule specification, IaC coding, or other engineering-level configuration work; this is architectural thinking, not engineering.
- Single-tenant, single-cloud toy projects or PoCs that explicitly will never reach production.
- Pure enterprise architecture strategy work disconnected from any solution (use TOGAF directly for that).
- Exploit development, payload crafting, or active penetration testing; this skill produces defensive design artifacts only.

## Authorization Check
- Confirm you are mandated to produce or review solution architecture artifacts for the workload or landing zone in question (sponsor, design authority, or architecture lead).
- Confirm scope: is this an enterprise security architecture exercise (organization wide) or a solution architecture exercise (one workload)? Do not blend the two.
- For architectural decisions with budget, timeline, or risk acceptance impact, confirm the business sponsor, not the architect, owns the final decision and sign-off.
- For governance or compliance artifacts, confirm the CISO/BISO team structure and which control framework baseline applies before documenting requirements.

## Methodology

1. **Establish enterprise context and the security taxonomy**: Gather external requirements (laws, regulations, industry standards) and internal requirements (policies, standards, business strategy, risk tolerance). Classify every security capability as a service (technology + process + people with a service design), not a process tied to a control framework section. Decompose the organization's security capability into six domains (Governance at top; Identity and Access, Network, Application, Data, Endpoint left to right following the access-to-data transaction flow; Detect and Respond spanning all; Security Service Management at bottom), then into five categories per domain, then into individual services. Cross-check a control framework (e.g. CSA CCM) against the domains to expose gaps (architecture governance, threat modeling, secrets management, bastion/session recording are commonly missing).

2. **Define system context and data sensitivity**: Build a system context diagram identifying all human and system actors, external systems, and the boundary of the system in scope. Separate actors by threat environment (internal vs remote access are different risk profiles even for the same person). Classify data (confidentiality, integrity, availability categories; identify SPI/sensitive personal data; flag where aggregation or generation of data increases sensitivity beyond its source). Record assumptions and open questions in a RAID log (risks, assumptions, issues, dependencies) rather than guessing silently.

3. **Decompose into components and threat model each use case**: Produce a component architecture (logical then deployed level) and/or data flow diagram per key use case. Then run the six-step threat modeling loop, adding one labeled layer per step to the diagram: (a) identify trust boundaries, (b) identify and label assets (A0x) at rest, in transit, in use, (c) identify threat actors (external authorized/unauthorized, internal unauthorized, internal "honest but curious," inadvertent) (TA0x), (d) identify threats (T0x) using STRIDE (spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege) for security and LINDDUN (linking, identifying, nonrepudiation, detecting, data disclosure, unawareness, noncompliance) for privacy, supplemented by attack trees or MITRE ATT&CK/CAPEC for depth, (e) identify controls (C0x), classified detective/preventive/corrective, (f) prioritize controls using a qualitative risk rating (likelihood x impact, e.g. OWASP Risk Rating Methodology) and select a risk treatment (avoid, mitigate, transfer, accept), recording inherent vs residual risk in a risk register and risk matrix against the organization's acceptable risk line.

4. **Map shared responsibilities across every cloud and on-prem boundary**: For each compute platform, CSP, and SaaS/PaaS/IaaS layer in the hybrid environment, document a shared responsibility diagram or table showing who (CSP, infrastructure ops, security ops, consumer) owns each security service, split by operate vs administer. Never leave a one-sided agreement (only one party's responsibilities documented); gaps default silently to the unprepared party. Capture this in a document of understanding (DoU) or contract, not just a diagram.

5. **Architect zero trust across the boundaries, mapped by domain**: Translate the zero trust principles into practices per NIST SP 800-207's core logical model (subject, PEP, PDP = policy engine + policy administrator, control plane vs data plane): identity/data/transaction identification, continuous authentication, adaptive access control, least privilege (RBAC/ABAC/risk-based + microsegmentation), encryption in transit/at rest/in use, and threat detection and response (assume breach). Apply network-layer solutions (ZTNA for user-to-application edge traffic, microsegmentation for system-to-system traffic, service mesh/sidecar for container-to-container), endpoint solutions (posture checks, EDR), and identity solutions (PAM with just-in-time checkout, ITDR). Organize the work across four levels: enterprise (cross-environment controls), CSP/landing zone (instance-specific controls), application/CI-CD (pipeline quality gates), and operations (service levels, tooling distribution). Update the deployment architecture diagram and re-run the threat model for the infrastructure data flows (human/system actor flows, system-event flows, and threat-actor flows that should never occur).

6. **Apply and extend architecture patterns, then automate**: Reuse CSP well-architected frameworks and reference architectures before designing from scratch. Compose solution design patterns (n-tier separation, route-to-live environment separation, hub-and-spoke with edge/transit VPC and management VPC, resilient hub-and-spoke with duplicated transit/management VPCs for dev/test isolation from production) into the solution architecture. Once the design exceeds what a diagram can represent (large rule counts, many environments), move to a deployable architecture: a DVCS (e.g. Git) holding IaC (declarative or imperative), run through a CI/CD pipeline with security quality gates, published to a reusable catalog.

7. **Record every architecturally significant decision as an ADR**: For decisions with real cost-of-change (not routine engineering choices), write an ADR with subject area, decision title/question, problem statement, assumptions, motivation, alternatives (each with advantages, disadvantages, expected effort/cost), the decision, justification, consequences, derived requirements, and related decisions. Use a short-form (decision, rationale, implication) for simpler, well-understood choices. Scope decisions as enterprise guiding principle, program-level, or project-level, and route project-level security vs business tradeoffs to the accountable business sponsor, not the architect. Track ADRs in a living, auditable medium (kanban board plus a published site), not a static sign-off document alone.

8. **Define Day-2 security operations before sign-off**: For every security service touched by the solution, write a RACI (or RASCI when a team must both lead and support) describing provider vs consumer responsibilities, and embed it in an internal agreement or external contract. Layer processes (organization-wide, technology-independent), procedures (line-of-business specific), and work instructions (tool-specific keystrokes). Convert the threat model's identified threats into threat detection use cases and incident response runbooks, and verify complete coverage with a threat traceability matrix (threat ID to detection use case ID to incident response runbook ID to test ID). Only sign off the solution architecture once this coverage is in place.

## Output Format
- Enterprise security architecture diagram (domains, categories, services) with a control-framework gap mapping.
- System context diagram, RAID log, data classification/sensitivity register.
- Component architecture diagram, data flow diagram, layered threat model diagram, risk register, risk matrix.
- Shared responsibility diagram/table or DoU per CSP/platform.
- Deployment architecture diagram (with zero trust components layered in) and cloud architecture diagram per CSP.
- Architecture patterns referenced or created, deployable architecture (IaC repo + pipeline reference).
- ADR log (full or short-form) with status and approver.
- RACI/RASCI tables, processes/procedures/work instructions, threat detection use cases, incident response runbooks, threat traceability matrix.

## Quality Check
- Every actor in the system context diagram has a matching interface in the component architecture, and every identified data flow is traceable end to end.
- Every identified threat (T0x) has at least one documented control (C0x); every control maps back to a threat or a compliance requirement, not just "best practice."
- No shared responsibility is one-sided; every security service has both a provider and consumer row.
- Every architecturally significant choice has an ADR with alternatives considered, not just the chosen option.
- Every threat in the traceability matrix has a detection use case, a runbook, and a test ID; gaps are flagged, not left blank.
- Zero trust practices are applied starting from the highest-risk use cases, not uniformly diluted across everything with no clear priority.

## Common Issues
- Treating a PoC as an MVP: if architectural characteristics (scale, resilience, security, compliance) were never tested, it is still a PoC regardless of what it is called.
- Confusing enterprise architecture (organization-wide, vendor-neutral guidance) with solution architecture (one workload's implementable design); don't blend their artifacts.
- Using an industry control framework (e.g. CSA CCM) as a complete and final control list; frameworks are a baseline, not a substitute for threat modeling and risk tolerance review.
- Letting threat modeling stop after one pass; it is iterative and must repeat for Day-2 changes and for infrastructure flows after the deployment architecture is defined.
- Applying zero trust uniformly instead of prioritizing by risk; budget and time never cover full zero trust on every workload in one iteration.
- Letting architecture documentation drift from the deployed system; require updates through the change/incident/problem process, not as an afterthought project.
- Skipping security operations definition (RACI, runbooks, traceability matrix) until after go-live, leaving the operations team without a documented way to detect or respond to the threats the architecture assumed would be caught.
