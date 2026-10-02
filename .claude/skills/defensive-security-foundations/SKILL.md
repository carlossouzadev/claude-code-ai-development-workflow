---
name: defensive-security-foundations
description: "Build or mature a pragmatic blue team program spanning risk, asset management, IR/DR, segmentation, vuln management, and monitoring."
model: opus
metadata:
  version: 1.0.0
  category: blue-team
  source: "Defensive Security Handbook, 2nd Edition (O'Reilly)"
---

# Defensive Security Foundations

## Goal
Drive maximum security improvement in an organization's infrastructure using low-cost,
pragmatic, prioritized defensive controls, from program governance down to host hardening,
rather than chasing vendor-driven or compliance-only checklists.

## When to Use
- Standing up a new information security program or assessing/maturing an existing one.
- Building or reviewing asset management, policy/standard/procedure documentation, incident
  response, or disaster recovery/BCP processes.
- Designing network segmentation, hardening Windows/Unix/endpoint systems, or evaluating
  physical security controls.
- Scoping or tuning vulnerability management, IDS/IPS, or SIEM/logging programs.
- Prioritizing a long backlog of security gaps with little to no budget.

## When NOT to Use
- Active exploitation, penetration testing, or red team engagements (use an offensive/pentest
  skill instead; this skill is defensive and architectural).
- Deep dive on a single regulatory regime's legal text (reference the regulation directly;
  this skill maps controls to frameworks at a high level only).
- Writing application-level secure code (use a language-specific secure coding skill).

## Authorization Check
- Confirm sponsorship: an executive team or CISO/CIO role must back program-level decisions
  (budget, milestones, risk acceptance). Governance activities require stakeholder authority,
  not just technical buy-in.
- For any scanning, log collection, or configuration change against live systems, confirm
  change control approval and a prearranged engineering window, per `.claude/security-scope.yaml`
  or the organization's change management process where applicable.
- For physical security or DR/BCP tabletop exercises, confirm facilities, HR, legal, and
  management stakeholders are included; do not run live failover tests without authorized
  sign-off from named individuals empowered to declare a disaster.

## Methodology
1. **Baseline the program (NIST CSF)**: Map work to the six NIST CSF 2.0 functions
   (identify, protect, detect, respond, recover, govern). Baseline policies, endpoints,
   licensing/cert expirations, internet footprint, network devices, logging, ingress/egress
   points, vendors, and applications. Run the risk cycle: identify scope/assets/threats,
   assess (likelihood x impact), mitigate (avoid, remediate, transfer, or accept as last
   resort with documented annual review), monitor via a risk register, govern through
   policy maintenance and reporting. Prioritize into milestone tiers: tier 1 quick wins
   (hours/days), tier 2 this year, tier 3 next year, tier 4 long-term. Build use cases from
   the Cyber Kill Chain (recon, weaponization, delivery, exploitation, installation, C2,
   actions on objectives) and validate with tabletops (moderator, diverse stakeholders,
   evaluator producing an after-action report) and drills.
2. **Make asset management continuous**: Define a classification schema (public/internal/
   confidential/highly confidential) via stakeholder interviews per department. Tag every
   asset with criticality and risk tier (impact, role in critical processes, compliance
   scope, replacement cost) so patching, vulnerability remediation, monitoring, and IR can
   all be prioritized off one inventory. Track the lifecycle: procure, deploy (reset
   defaults, scan before production), manage, decommission (secure erase or physical
   destruction by data classification). Maintain a single source of truth; this is never
   a one-time exercise.
3. **Layer policy, standards, and procedures**: Policies state the "why" in imperative
   language (shall/must/will, never should/try), are management-endorsed, and reviewed
   annually. Standards give the technology-agnostic "what" (e.g., password complexity).
   Procedures give the platform-specific "how" (exact commands per OS). Keep the three
   separate for maintainability and audit mapping; each needs version info, owner/approver,
   scope, and related-document cross-references.
4. **Run incident response in three phases**: Pre-incident (use existing escalation paths,
   define what counts as an incident, set severity thresholds). Incident (name an incident
   manager, open a war room, set a communications cadence, split internal vs. external
   comms, prioritize removing attacker access and finding persistence, document everything).
   Post-incident (lessons-learned/postmortem within days, after-action report, update
   policies and tabletops). Centralize logs on a SIEM, not host-local, to prevent tampering;
   use EDR/XDR, disk/memory imaging, and PCAP tools (tcpdump, Wireshark, Zeek/Suricata) as
   needed.
5. **Separate DR from BCP, set RPO/RTO with the business**: BCP covers continuation of the
   whole business; DR covers the IT processes that achieve BCP objectives. Business owners,
   not IT, set recovery point objective (max tolerable data loss) and recovery time
   objective (max tolerable downtime) per system. Choose a strategy matched to RPO/RTO and
   budget: traditional backups, warm standby, high availability clustering, alternate
   system, system function reassignment, or cloud native DR (automated replication, IaC,
   multiregion, orchestration). Map dependencies (network, DNS, auth) so one slow system
   doesn't bottleneck another's RTO. Give DR sites the same data protection, patching, and
   physical security as production, and test with authorized failover drills plus a
   documented failback process.
6. **Apply physical security in depth**: Combine physical controls (badge/PIN/biometric
   access, locked racks, tamper-resistant camera placement, secure media handling and
   disposal, floor-to-ceiling datacenter walls) with operational controls (visitor sign-in
   and escort, distinguishable visitor/contractor badges with expiry, background-checked
   contractors). Train staff against tailgating, badge cloning, malicious USB drops, and
   pretexting; require two-factor access for the most sensitive areas.
7. **Segment the network, harden hosts by default**: Combine physical segmentation
   (firewalls at every ingress/egress, between production and dev/test/DMZ/guest) with
   logical segmentation (VLANs by risk tier or role, ACLs with explicit matches and a
   logged deny-all, NAC/802.1X for port-level auth, IPsec or SSL/TLS VPNs with no
   unnecessary split tunneling). Default deny, then allow list; block unneeded egress;
   separate application tiers (web, app, DB) so one compromised component doesn't yield
   the stack. Segregate roles and duties (admins get separate privileged/standard accounts,
   developers don't touch production, DBAs don't get root). Harden Windows and Unix hosts
   alike: patch via package management, disable unneeded services, enforce host firewalls,
   use file integrity monitoring, apply least-privilege file permissions, and use MAC/chroot
   or endpoint protection where available.
8. **Run vulnerability management as a program**: Distinguish authenticated scans (more
   accurate, fewer false positives/negatives, but need tightly controlled, time-boxed
   scanner credentials) from unauthenticated scans (the attacker's view, banner-grab
   based, more false positives). During initialization, batch remediation by operations
   team or host group to make large backlogs tractable; in business-as-usual mode, cycle
   discover, prioritize, assign, track/verify closure. Prioritize by combining severity
   with the asset's criticality/risk rating from step 2, not vendor advice alone. Document
   any accepted risk with an annual review.
9. **Instrument detection (IDS/IPS and a tuned SIEM)**: Deploy network-based IDS/IPS
   (Snort, Suricata, Zeek, or an NGFW) and host-based IDS/EDR (OSSEC, osquery, honeypots);
   in cloud, use the platform-native service (GuardDuty, Defender/Sentinel, GCP Event
   Threat Detection) plus its log sources (CloudTrail/VPC Flow Logs, Azure resource/AD
   logs, Cloud Audit Logs). Design the SIEM deliberately: define coverage scope, build
   threat use cases mapped to MITRE ATT&CK or the kill chain, prioritize alerts by
   relevance to your actual data, proof-of-concept test detections with a purple team, and
   document retention in a Record of Authority. Enrich Windows logging with Sysmon (process
   creation, network connections, registry persistence, DNS) and Group Policy auditing.
   Tune continuously for false positives and alert fatigue; centralize logs off-host so a
   compromised system can't erase its trail.

## Output Format
- Risk register and tiered milestone roadmap (quick wins through long-term).
- Asset inventory schema with criticality/risk fields and lifecycle definitions.
- Policy, standard, and procedure document set (hierarchical, version-controlled).
- Incident response plan with pre/during/post-incident processes and contact tree.
- DR/BCP plan with RPO/RTO per system, chosen recovery strategy, and test results.
- Physical security control checklist (physical and operational).
- Network segmentation diagram (VLANs, ACLs, NAC zones, DMZ) and host hardening checklist.
- Vulnerability management program definition (cadence, prioritization, remediation SLAs).
- SIEM/IDS-IPS design document (coverage scope, use cases, alert priorities, retention).

## Quality Check
- Every milestone traces back to an identified risk or asset criticality rating, not just
  vendor recommendation.
- RPO/RTO values were set or signed off by business owners, not unilaterally by IT.
- Policies use mandatory language (shall/must) with no unresolved ambiguity about scope.
- SIEM alerts have been proof-of-concept tested, including evasion attempts, before being
  relied upon.
- Accepted risks are documented with owner, justification, and a scheduled review date.
- Segmentation design enforces default-deny with explicit, logged exceptions.

## Common Issues
- Treating asset management as a one-time inventory instead of a continuous lifecycle
  process; data goes stale within months.
- Letting IT alone set RTO/RPO, producing unrealistic expectations or unjustified spend.
- Relying on VLANs alone without ACLs or firewalls; VLAN hopping is a known risk in
  default configurations.
- Running vulnerability scans without change control, crashing legacy devices during a
  scan window.
- Deploying a SIEM or IDS/IPS with default rules and no tuning, burying real incidents
  in alert fatigue.
- Backup and DR environments left unpatched or with weaker access controls than
  production, making them the softest target in the environment.
- Writing policies so vague ("should," "try") that they cannot be enforced or audited.
