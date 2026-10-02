# Security Knowledge Skills Library

19 advisory security skills distilled from 20 security books (O'Reilly and
others). These are DEFENSIVE / architecture / governance knowledge aids: they
help you design, assess, build, and review, following the same house-style
template as the architecture-book skills (`ddd-context-mapping`,
`architectural-fitness-functions`, etc.).

They are a separate family from the offensive / DFIR orchestrator set in
[SECURITY_SKILLS_README.md](SECURITY_SKILLS_README.md). They do NOT get
composed by `security-orchestrator`, do NOT write to `SECURITY_AUDIT.md`,
and are not bound by the offensive authorization contract. Where a skill
advises on active assessment work, it defers that execution to the matching
hunter skill and to `.claude/security-scope.yaml`.

Each skill follows: Goal / When to Use / When NOT to Use / Authorization Check
/ Methodology / Output Format / Quality Check / Common Issues.

## Inventory (19 skills)

### Blue team and operations (3)

| Skill | Covers | Source |
|---|---|---|
| [defensive-security-foundations](defensive-security-foundations/SKILL.md) | Build or mature a blue team program: risk, asset mgmt, IR/DR, segmentation, vuln mgmt, monitoring | Defensive Security Handbook, 2nd Ed |
| [blue-team-operations](blue-team-operations/SKILL.md) | Hands-on IR triage: live Windows/Linux collection, memory/packet analysis, PICERL execution | Blue Team Handbook: Incident Response Edition |
| [security-ops-bash](security-ops-bash/SKILL.md) | Collect, parse, baseline, triage host/log data from the CLI, then automate in bash | Cybersecurity Ops with bash |

### Security governance (2)

| Skill | Covers | Source |
|---|---|---|
| [cyber-risk-management-program](cyber-risk-management-program/SKILL.md) | Design/assess/mature an enterprise cyber risk program: governance, assessment, strategy, disclosure | Building a Cyber Risk Management Program |
| [security-program-management](security-program-management/SKILL.md) | Stand up or revitalize an InfoSec program via relationships, alignment, documentation, governance | The Cybersecurity Manager's Guide |

### Security architecture (2)

| Skill | Covers | Source |
|---|---|---|
| [zero-trust-architecture](zero-trust-architecture/SKILL.md) | Design/migrate to zero trust: control/data plane split, trust scoring, per-request authorization | Zero Trust Networks, 2nd Ed |
| [hybrid-cloud-security-architecture](hybrid-cloud-security-architecture/SKILL.md) | Zero-trust-based security for hybrid/multi-cloud: domains, threat modeling, shared responsibility, ADRs | Security Architecture for Hybrid Cloud |

### Cloud security (4)

| Skill | Covers | Source |
|---|---|---|
| [cloud-security-foundations](cloud-security-foundations/SKILL.md) | Cloud security program: shared responsibility, inventory, IAM, network, encryption, detection, IR | Learning + Practical Cloud Security, 2nd Ed |
| [cloud-native-security](cloud-native-security/SKILL.md) | Secure landing zones, IAM, networking, encryption, logging, policy-as-code across AWS/Azure/GCP | Cloud Native Security Cookbook |
| [serverless-security](serverless-security/SKILL.md) | Harden FaaS: IAM over-privilege, event injection, secrets, supply chain | Learning Serverless Security |
| [kubernetes-security-observability](kubernetes-security-observability/SKILL.md) | Holistic K8s security + observability across build, deploy, runtime | Kubernetes Security and Observability |

### DevSecOps (3)

| Skill | Covers | Source |
|---|---|---|
| [security-as-code](security-as-code/SKILL.md) | Policy-as-code gates across IaC, CI/CD, logging, IAM, resilience testing | Security as Code |
| [continuous-security-devsecops](continuous-security-devsecops/SKILL.md) | AI-augmented end-to-end Continuous Security operating model unifying DevSecOps and SecOps | Intelligent Continuous Security |
| [software-supply-chain-security](software-supply-chain-security/SKILL.md) | SBOMs, build/CI/CD provenance (SLSA, in-toto, Sigstore), third-party and consumption risk | Software Supply Chain Security |

### Identity (1)

| Skill | Covers | Source |
|---|---|---|
| [identity-security-for-developers](identity-security-for-developers/SKILL.md) | AuthN/AuthZ, secrets, and machine identity so no long-lived credential has to leak | Identity Security for Software Development |

### AI security (2)

| Skill | Covers | Source |
|---|---|---|
| [llm-application-security](llm-application-security/SKILL.md) | Build-time LLM app defense: prompt injection, data disclosure, excessive agency, supply chain (RAISE) | The Developer's Playbook for LLM Security |
| [llm-privacy-protection](llm-privacy-protection/SKILL.md) | Training-data privacy: memorization/extraction/membership inference, DP, PEFT, federated learning | Privacy and Security for Large Language Models |

### Application security (1)

| Skill | Covers | Source |
|---|---|---|
| [web-application-security-defense](web-application-security-defense/SKILL.md) | Secure-by-default web app design and review vs XSS/CSRF/XXE/injection/DoS/dependency risk | Web Application Security, 2nd Ed |

### Threat intelligence (1)

| Skill | Covers | Source |
|---|---|---|
| [threat-intelligence-fundamentals](threat-intelligence-fundamentals/SKILL.md) | Defensible nation-state/APT threat assessments: attribution auditing, capability/intent, resilience | Inside Cyber Warfare, 3rd Ed |

## Relationship to the offensive / DFIR skills

These advise; the hunter skills act. Pairings worth knowing:

- `blue-team-operations` complements the reactive `incident-response` reference
  and the DFIR hunters (`memory-forensics-hunter`, `disk-triage-hunter`,
  `log-timeline-hunter`).
- `kubernetes-security-observability` is the defensive counterpart to
  `container-hunter`.
- `llm-application-security` and `llm-privacy-protection` are the build-time
  counterparts to `llm-redteam-hunter`.
- `web-application-security-defense` is the secure-design counterpart to the
  web/API hunter set.

Existing skills were left untouched when these were added.
