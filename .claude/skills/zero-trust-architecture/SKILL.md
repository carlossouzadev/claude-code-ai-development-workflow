---
name: zero-trust-architecture
description: "Design and migrate to a zero trust architecture using control/data plane separation, trust scoring, and per-request authorization."
model: opus
metadata:
  version: 1.0.0
  category: security-architecture
  source: "Zero Trust Networks, 2nd Edition (O'Reilly)"
---

# Zero Trust Architecture

## Goal
Replace implicit, network-location-based trust with continuous, per-request authentication and authorization, so that no device, user, application, or flow is trusted by default, and compromise of any single segment does not grant lateral movement.

## When to Use
- Designing a new network, application platform, or cloud landing zone that must not rely on perimeter firewalls or VPN as the primary control.
- Migrating an existing perimeter-based network (corporate LAN, flat VPC, legacy VPN) toward zero trust.
- Reviewing or building an authorization system (policy engine, enforcement points, trust scoring) for an enterprise resource.
- Evaluating device, user, application, or traffic trust end to end (BYOD, service-to-service auth, mTLS rollout).
- Mapping an organization's security posture against NIST SP 800-207, CISA Zero Trust Maturity Model, or DoD Zero Trust Reference Architecture for compliance or executive reporting.

## When NOT to Use
- Small, single-tenant systems with no sensitive data, no external users, and no realistic lateral movement risk, where the overhead is not justified.
- Pure physical-security or personnel-vetting questions unrelated to network/system access control.
- Low-level exploit development or offensive tooling. This skill is defensive architecture guidance only.

## Authorization Check
- Confirm sponsorship and authority to redesign network/access architecture (CTO, Head of Infrastructure, or equivalent), since zero trust touches identity, network, and application teams simultaneously.
- Confirm scope: which network zones, applications, or environments (dev/staging/prod) are in scope for this pass.
- For any live enforcement change (firewall rules, policy engine deployment, device re-imaging), confirm change management approval and a rollback plan before applying, per this repo's standard environment flow (Dev to Staging to Production, production requires approval).

## Methodology

1. **State the five zero trust assertions up front**, and use them as the design's acceptance criteria:
   - The network is always assumed to be hostile.
   - External and internal threats exist on the network at all times.
   - Network locality alone is not sufficient for deciding trust.
   - Every device, user, and network flow is authenticated and authorized.
   - Policies must be dynamic and calculated from as many data sources as possible.

2. **Separate control plane from data plane**:
   - Data plane: applications, firewalls, proxies, routers that carry traffic; keep this layer dumb and fast.
   - Control plane: the brains, where authentication, authorization, and policy decisions happen, and which reconfigures the data plane to admit only authorized flows in a time-bound (not permanent) way.
   - Never let data plane components (exposed, high-traffic) be able to compromise the control plane; isolate them as separate, independently securable systems.

3. **Build the authorization architecture from four isolated components** (map to NIST SP 800-207 terms in parentheses):
   - **Enforcement** (Policy Enforcement Point, PEP): placed as close to the workload/endpoint as possible; establishes, monitors, and terminates the connection; never let trust pool "behind" an enforcement point.
   - **Policy engine** (Policy Decision Point / Policy Engine + Policy Administrator): compares request context against versioned policy and returns allow/deny/revoke; keep it process-isolated from the PEP so a PEP compromise cannot reach it.
   - **Trust engine**: computes a numeric, continuously updated risk/trust score from device, user, and behavioral signals (a mix of ad hoc rules and machine learning); feeds the policy engine but is not itself the final decision maker.
   - **Data stores**: authoritative sources of truth (device inventory, identity directory, activity logs) that the trust and policy engines query by a small authenticated key (serial number, username).
   - Score both the network agent (the session/request bundle) and the underlying entities (device, user) separately; scoring only one misses attack classes (credential stuffing vs. compromised device vs. account-hopping insider). Never expose raw trust scores to end users; surface only infrequent, high-level feedback.

4. **Establish trust per entity type**, using the book's per-chapter model:
   - **Devices**: bootstrap trust via golden images, record last-imaged date, use secure boot where supported, generate a per-device certificate signed by a private CA, and store the private key in an HSM/TPM so it never leaves the security module. Maintain a searchable inventory as the secure source of truth; prefer periodic reimaging over indefinite patch-and-scan.
   - **Identities (users)**: strong multi-factor authentication (something you know/have/are), prefer security tokens (U2F/WebAuthn) over TOTP for phishing resistance, bind identity to a directory, and require stronger authentication for higher-risk actions (step-up auth) rather than a single static login event.
   - **Applications**: trust the full pipeline (source, build, distribution, runtime instance), not just the running binary; require code review, reproducible/verifiable builds, signed and attested artifacts, and runtime isolation plus active monitoring.
   - **Traffic**: authenticate every flow, prefer mutual TLS (mTLS) over one-way TLS, and treat encryption and authentication as separate properties (you can have one without the other, but zero trust needs both).

5. **Use private PKI, not public PKI, as the trust anchor** for internal device/application/user certificates (cost, foreign CA governance risk, and lack of programmability make public PKI unsuitable at zero trust scale); public PKI is acceptable only as a stepping stone with a migration path to private PKI.

6. **Write policy as fine-grained, logical-component based, version-controlled rules**, not IP/network based rules:
   - Define policy in terms of network services, device endpoint classes, and user roles, so policy survives autoscaling and workload rescheduling.
   - Apply the Kipling Method to every policy: Who (identity), What (application/API/service), When (time window), Where (resource location), Why (business justification, for compliance), How (how traffic should be processed).
   - Require code review on policy changes (policy as data in version control) and layer broad infrastructure guardrails (e.g., "only role X may accept internet traffic") on top of team-owned fine-grained policy so no team can widen the blast radius unilaterally.
   - Include a trust-score component in policy (e.g., require MFA when risk score is medium/high) in addition to static role/resource rules, to catch unknown unknowns.

7. **Migrate incrementally from a perimeter network**:
   - Capture a system diagram and all network flows first (SPAN/TAP, NetFlow/sFlow, cloud flow logs, or endpoint firewall log-only mode) before enforcing anything; categorize flows at the logical-system level, not IP/port level.
   - Move zone by zone: build a zero trust enclave within the existing perimeter and expand it, rather than attempting a single flag-day cutover.
   - Apply micro-segmentation: divide the network into small, independently monitored and policed zones so a breach in one segment cannot spread; this is the practical, incremental form of the full control-plane vision.
   - Where a full dynamic control plane is not yet built, use a Software-Defined Perimeter (default-deny, pre-vetted connections) or "cheat" with configuration management (version-controlled desired state, programmatically calculated firewall rules) as an interim step toward a dedicated controller.
   - Centralize application authN/authZ through an identity provider (SAML/OAuth2) rather than per-application credential stores; authenticate load balancers/proxies as applications in their own right, forwarding verified identity to backend systems so zero trust survives the client/server boundary.

8. **Pressure-test the design from an adversarial view** before calling it done: credential theft, privilege escalation/lateral movement, control plane compromise, endpoint enumeration, DDoS, MitM, and insider/physical coercion. Confirm which of these the design mitigates and which remain accepted residual risk.

9. **Map the result to NIST SP 800-207** for stakeholder communication and compliance framing:
   - PE (Policy Engine) + PA (Policy Administrator) = PDP (Policy Decision Point), in the control plane.
   - PEP (Policy Enforcement Point), client-side agent and/or resource-side gateway, in the data plane.
   - Data sources: Continuous Diagnostics and Mitigation (CDM), industry compliance system, threat intelligence, activity logs, data access policy, enterprise PKI, identity management system, SIEM.
   - Choose a deployment variation deliberately: device agent/gateway (needs strong device management, weak for BYOD), enclave gateway (fits legacy/datacenter perimeters), or resource portal/microsegmentation, and note that variants can coexist by maturity and business need.
   - Reference the core tenets: always assume breach, always least privilege, always per-request/session authorization, dynamic policy, no implicit trust from network location, and continuous monitoring of all resources including data sources themselves.

## Output Format
- A system/flow diagram (existing state) plus a target-state zero trust architecture diagram showing PEP placement, PDP (policy + trust engine), and data stores.
- A policy specification (JSON/YAML) per protected resource, written in Kipling Method terms (Who/What/When/Where/Why/How) and checked into version control.
- A trust-scoring approach note: which signals feed the trust engine (device posture, location, behavioral history) and which entities are scored (agent, device, user).
- A phased migration plan: flow discovery, zone-by-zone enclave rollout, identity provider centralization, PKI issuance plan, target state.
- A NIST SP 800-207 component-mapping table for the proposed design, usable in compliance/executive conversations.

## Quality Check
- Every enforcement point sits as close to the workload as possible; no zone exists where trust "pools" behind a single boundary device.
- Policy engine and enforcement are isolated as separate processes/services, not merged in a way that lets a PEP compromise reach the PDP.
- No policy rule depends solely on source IP or network zone membership; every rule references identity, device, or application attributes.
- Every credential (certificate, token, key) has a defined rotation period and a documented revocation path; no secret is "hard or impossible to rotate."
- Trust scores are never exposed raw to the entity being scored, and scoring covers both the agent and its underlying device/user entities.
- Migration plan shows an incremental, zone-by-zone path with a flow-discovery step before any enforcement change, not a flag-day cutover.

## Common Issues
- Treating zero trust as a product purchase instead of an architecture; a single vendor appliance rarely covers device, identity, application, and traffic trust together.
- Conflating encryption with authentication (TLS without mutual authentication secures confidentiality but still trusts the client's claimed identity implicitly).
- Centralizing too much in one component: merging the PEP and PDP into one process so a data-plane compromise directly reaches policy decisions.
- Static, IP-based policy left in place "temporarily" during migration, which recreates perimeter assumptions inside the zero trust design.
- Skipping the flow-discovery step, causing a frustrated rollback after enforcement breaks unknown-but-legitimate traffic.
- Over-rotating or over-prompting users for step-up authentication on low-risk actions, causing fatigue that erodes the value of step-up auth on genuinely high-risk actions.
- Assuming BYOD fits the device agent/gateway deployment model; it usually needs an enclave gateway or resource-portal variant instead.
