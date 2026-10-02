---
name: llm-application-security
description: "Design and harden LLM applications against prompt injection, data disclosure, hallucination overreliance, excessive agency, and supply chain risk using the RAISE framework."
model: opus
metadata:
  version: 1.0.0
  category: ai-security
  source: "The Developer's Playbook for Large Language Model Security (O'Reilly)"
---

# LLM Application Security (Build-Time Defense)

## Goal
Help development teams architect, build, and operate LLM-based applications (chatbots, copilots, RAG systems, agents) so they resist prompt injection, do not leak sensitive data, limit hallucination-driven harm, constrain agency and tool permissions, and maintain a trackable AI supply chain. This is the defensive, build-time counterpart to red-team/adversarial testing skills: it produces architecture decisions, guardrail code, SBOM artifacts, and process controls rather than exploit payloads.

## When to Use
- Architecting a new LLM-backed feature (chatbot, copilot, RAG pipeline, autonomous agent).
- Reviewing an existing LLM integration for trust boundary gaps, excessive permissions, or missing output filtering.
- Choosing or fine-tuning a foundation model and deciding what data (training, RAG, user interaction) it should have access to.
- Standing up LLMOps/DevSecOps pipelines, guardrail frameworks, or an ML-BOM/model card process.
- Responding to an incident involving prompt injection, data leakage, hallucinated output, or denial-of-wallet, and needing a remediation plan.
- Mapping LLM-specific risk to OWASP Top 10 for LLM Applications for a security review or compliance artifact.

## When NOT to Use
- Running offensive prompt-injection or jailbreak campaigns against a target system; use the dedicated llm-redteam-hunter skill for that (this skill is defensive only and must not be edited or merged with it).
- General web/API vulnerability classes unrelated to LLM behavior (SQLi, XSS, CSRF, IDOR as standalone web issues); use the standard web/API hunter skills.
- Pure model-training performance tuning with no security angle.

## Authorization Check
- For production changes (new tool permissions, database grants to the LLM, plug-in integrations, autonomy increases), confirm sign-off from the system/data owner before implementing, per the "least privilege is the best privilege" zero trust principle.
- For any proposed red-team or adversarial validation of the defenses built here, hand off to the authorized offensive skill/process; do not generate novel jailbreak payloads under this skill.
- When fine-tuning or RAG sources include regulated data (PII, PHI, financial), confirm data governance/compliance approval (GDPR, CCPA, HIPAA) before ingestion.

## Methodology

1. **Map the architecture and trust boundaries.** Diagram the application's data flows: user interaction, training data (foundation + fine-tuning), live external data (web/API access via RAG), internal services (databases, vector stores), and the model itself. Mark every point where data crosses from a less-trusted zone into the LLM or from the LLM into a downstream system. Each boundary needs explicit authentication, authorization, and validation controls. Public API-hosted models trade control for lower data-confidentiality assurance; privately hosted models trade convenience for supply-chain and maintenance responsibility.

2. **Limit the domain (RAISE step 1).** Scope the application to the narrowest functional domain that meets the business need. A narrow domain lets you build an allow-list of intended behavior instead of chasing an ever-growing deny-list (the "unconstrained domain" trap that general-purpose chatbots like ChatGPT face). Prefer smaller, domain-specific foundation models or models fine-tuned to reward staying on topic. A narrow domain also directly reduces hallucination risk and attack surface.

3. **Harden against prompt injection (direct and indirect).** Treat prompt injection as unsolvable in general, only mitigable in layers:
   - Add explicit prompt structure (tags/delimiters) to separate developer instructions from user- or RAG-supplied data so injected text is treated as data, not instruction.
   - Apply rule-based input filtering and rate limiting (IP-, user-, and session-based) as a first line of defense, understanding both are bypassable.
   - Consider a special-purpose filtering LLM or adversarial-trained model as an added layer, not a silver bullet.
   - Treat RAG inputs (scraped URLs, search results, databases) as equally untrusted as direct user input; indirect injection via a web page or document is just as dangerous as direct jailbreaking.
   - Adopt a pessimistic trust boundary: assume every LLM output is potentially adversarial, especially when any upstream input was untrusted.

4. **Secure output handling (treat the LLM as a confused deputy).** Never pass LLM output to a downstream interpreter (shell, SQL, HTML renderer, code executor) without the same rigor as any other untrusted input:
   - HTML-encode output destined for a browser to prevent stored/reflected XSS.
   - Use parameterized queries/prepared statements if LLM output informs a database query; never string-concatenate.
   - Strip or escape shell metacharacters before any output reaches a shell context; avoid shell execution of LLM output entirely where possible.
   - Screen for toxicity (sentiment analysis, moderation APIs, custom classifiers) and PII (regex for structured identifiers like SSNs, NER for names/addresses, contextual analysis) before returning output to users.
   - Log every prompt and response pair before and after filtering for audit and incident response.

5. **Constrain excessive agency with least privilege.** Audit every permission, function, and autonomy level granted to the LLM as you would for a system user:
   - Excessive permissions: grant read-only database access by default; never expand to INSERT/UPDATE/DELETE without a specific, reviewed justification and audit trail.
   - Excessive autonomy: require human-in-the-loop approval for any safety-critical, financial, or irreversible action (trades, deletions, external communications) rather than full automation.
   - Excessive functionality: check new LLM-driven features against regulatory constraints (e.g., EU rules against automated hiring decisions) before shipping.
   - Map each excessive-agency finding to the confused-deputy pattern: an attacker exploiting a prompt injection to misuse privileges the LLM legitimately holds.

6. **Govern what the LLM is allowed to know (balance the knowledge base, RAISE step 2).** For each knowledge source (foundation training, fine-tuning data, RAG web/database access, learned user interactions), ask "what happens if this is disclosed?":
   - Scrub PII/secrets from training and fine-tuning datasets via anonymization, aggregation, masking, synthetic data, differential privacy, or tokenization; least-privilege data collection (don't gather what you don't need).
   - Apply role-based access control, data classification, views instead of raw tables, redaction/masking, and retention limits when connecting the LLM to relational or vector databases.
   - For direct web/RAG access, assume scraped content (comments, metadata, ads, author bios) can carry hidden PII; validate and sanitize retrieved content before it enters a prompt.
   - For user-interaction learning, disclose data retention policies clearly, sanitize inputs, avoid persistent learning from raw conversations, and consider session-scoped (non-persistent) memory.
   - Insufficient knowledge increases hallucination risk; excessive knowledge increases disclosure risk. Tune deliberately, don't default to either extreme.

7. **Reduce hallucination and overreliance.** Layer fine-tuning for domain expertise, RAG against vetted reference material, and chain-of-thought prompting for multi-step reasoning tasks. Add user-facing feedback loops (flagging, rating, comments) and clearly document intended use, limitations, and data handling so users calibrate trust appropriately. Never let an LLM's confident tone substitute for verification in legal, medical, financial, or safety-critical outputs.

8. **Manage DoS/DoW exposure.** Apply domain-specific guardrails, input validation/sanitization, robust rate limiting, resource-use capping (token/context limits), and financial threshold alerts to prevent resource-exhaustion and denial-of-wallet attacks against pay-per-use model APIs. Watch for context-window exhaustion and unbounded generation requests as attack vectors distinct from classic volumetric DoS.

9. **Track the AI supply chain.** Treat the foundation model, fine-tuning datasets, RAG data sources, and plug-ins as supply chain dependencies:
   - Vet model and dataset provenance (e.g., Hugging Face token/account compromise history) before adopting.
   - Build and maintain an ML-BOM (CycloneDX 1.5+, OWASP-maintained) alongside a model card for every model/dataset pair; store both in version-controlled, tamper-evident repositories and regenerate on every build.
   - Watch for digital signing and watermarking (C2PA, Sigstore + SLSA for models) to verify provenance and integrity as these practices mature.
   - Cross-reference MITRE CVE for traditional component vulnerabilities and MITRE ATLAS for AI-specific adversarial TTPs.

10. **Operationalize via LLMOps/DevSecOps.** Integrate LLM-specific security testing (e.g., TextAttack, Garak, Giskard LLM Scan, Responsible AI Toolbox) into CI/CD alongside standard AST tooling. Deploy a guardrails framework (open source: NeMo-Guardrails, Llama Guard, Guardrails AI; commercial: Lakera Guard, WhyLabs LangKit, Cloudflare Firewall for AI) for input validation (injection detection, domain limiting, secret/PII redaction) and output validation (toxicity, sensitive-data, code-injection, compliance, fact-checking), supplemented with custom domain-specific rules. Log every prompt/response into a SIEM, layer UEBA for anomaly detection, and stand up or contract an AI red team (human-led, optionally augmented with automated tooling like PyRIT) to continuously validate these controls (coordinate with, but do not replace, the dedicated red-team skill).

11. **Run the RAISE checklist before shipping.** Limit domain -> balance knowledge base -> implement zero trust (screen input, screen output, guardrails) -> manage supply chain (provenance, ML-BOM, secure DevOps pipeline) -> build/engage an AI red team -> monitor continuously (log everything, centralize in SIEM, hunt for anomalies). Treat it as a living gate, not a one-time form.

## Output Format
- Architecture diagram or written trust-boundary map (data flow + boundary list) for the application under review.
- RAISE checklist (filled in, with gaps flagged) as a markdown or ticket-tracked artifact.
- Guardrail configuration or code (input/output filters, PII regex, toxicity thresholds, rate-limit policy).
- ML-BOM (CycloneDX JSON) and/or model card for each model and dataset dependency.
- OWASP Top 10 for LLM Applications mapping table for any findings (LLM01 through LLM10) to standardize communication with security/compliance stakeholders.
- Monitoring/alerting spec: what gets logged, where (SIEM), and what anomaly thresholds trigger review.

## Quality Check
- Every trust boundary in the architecture has a named control (filter, permission check, human approval) rather than an implicit assumption of safety.
- No LLM output reaches a shell, SQL engine, or HTML renderer without encoding/parameterization appropriate to that sink.
- Database/tool permissions granted to the LLM are the minimum required (verify by attempting to justify each permission against a concrete use case).
- Every model and dataset dependency has a corresponding ML-BOM entry and, where available, a model card.
- Guardrails cover both directions: input (injection, domain, secrets) and output (toxicity, PII, code, compliance, hallucination).
- Logging captures full prompt/response pairs, not just metadata, and feeds a system capable of anomaly detection.

## Common Issues
- Keyword/deny-list filtering alone (e.g., blocking "bomb" or "napalm") degrades legitimate functionality while remaining trivially bypassed by rephrasing (reverse psychology, misdirection, roleplay framing); always pair with structural and output-side defenses.
- Treating RAG-retrieved content as trusted because it "came from your own pipeline" ignores that the underlying web page, document, or database record may itself be attacker-controlled (indirect prompt injection).
- Expanding LLM database permissions from read to read-write for a single feature without re-reviewing least privilege is the single most common excessive-agency failure mode.
- Assuming a fine-tuned or RAG-augmented model can't hallucinate; CoT prompting and reference grounding reduce but do not eliminate hallucination, so user-facing feedback and clear limitation disclosures remain necessary.
- Skipping ML-BOM/model card maintenance until an incident forces a scramble to determine what training data or model version was in production at a given time.
- Conflating this defensive skill with red-team/exploit work; keep adversarial validation in the dedicated llm-redteam-hunter skill and treat its outputs as input to this skill's remediation steps, not the other way around.
