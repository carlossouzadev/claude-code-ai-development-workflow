---
name: llm-privacy-protection
description: "Assess and harden LLM systems against training-data memorization, extraction, and membership inference using differential privacy, PEFT, federated learning, and de-identification."
model: opus
metadata:
  version: 1.0.0
  category: ai-security
  source: "Privacy and Security for Large Language Models (O'Reilly)"
---

# LLM Privacy Protection

## Goal
Reduce the risk that an LLM's training data, fine-tuning corpus, or RAG store can be reconstructed, inferred, or leaked, by applying privacy-preserving training techniques (DP-SGD, PEFT, federated learning), PII de-identification pipelines, and defense-in-depth around inference, with concrete epsilon targets and regulatory alignment (GDPR, HIPAA, CCPA). This is distinct from llm-application-security, which covers prompt injection and app build-time hardening; this skill covers what happens to the DATA the model is trained on or retrieves.

## When to Use
- Before fine-tuning or RAG-indexing any corpus containing PII, PHI, or proprietary/sensitive records.
- When a privacy or compliance review asks "can this model regurgitate training data" or "can someone tell if record X was in the training set."
- When designing a federated or multi-party training setup across orgs/clients that cannot share raw data.
- When choosing between full fine-tuning, PEFT/LoRA, and RAG for a sensitive-data use case.
- When a healthcare, legal, financial, or other regulated-data LLM project needs a documented privacy budget and compliance rationale.

## When NOT to Use
- Prompt injection, jailbreaking defenses, agentic tool-permission design, or output content moderation at inference time only: use llm-application-security instead.
- Pure infrastructure/network security of the serving stack with no training-data privacy angle (API auth, rate limiting, TLS): treat as general infosec, reference this skill only for the data-handling layer.
- General dataset anonymization unrelated to model training (e.g., one-off CSV exports): use de-identification techniques directly, this skill's overhead is not needed.

## Authorization Check
- Confirm who owns the training/RAG corpus and that you have authority to inspect or modify it (data owner, DPO, or project sponsor sign-off).
- For regulated data (PHI under HIPAA, EU personal data under GDPR), confirm a legal/compliance stakeholder has reviewed the intended processing basis before any export or fine-tuning run proceeds.
- For red-teaming extraction or membership-inference attacks against a production model, confirm it is authorized under the project's security-scope (treat as an assessment, not a public attack).

## Methodology
1. **Classify the data and the risk surface**: identify whether sensitive data enters via fine-tuning corpus, RAG index, or both. Note that fine-tuning "bakes" information into weights (harder to remove, higher regurgitation risk) while RAG keeps it externally retrievable (easier to delete/audit, but still subject to retrieval-pattern and document-attribution leakage). Prefer a hybrid: DP fine-tuning for general knowledge, RAG with access controls for sensitive/specific records that should not be encoded in weights.
2. **Run a privacy/security evaluation before and after any mitigation**:
   - Membership inference: perplexity-based (low perplexity on a sample suggests training membership; use dynamic thresholds, not a fixed cutoff) and repeated-prompt-based (check whether generated output reproduces the probe text). Report attack success rate (ASR = successes / attempts) and false positive rate.
   - Data extraction / model inversion: probe with partial/prefix prompts and prompt variations, measure ASR and reconstruction error (MSE, Levenshtein/BLEU/ROUGE distance between reconstructed and true text; higher distance is safer).
   - Build a "Golden Dataset" of curated attack attempts (including canary/decoy examples planted by you) so ASR is comparable across model versions. If you lack ground-truth training-set access, use a proxy ground truth (public knowledge + synthetic examples + your own canaries) and label risk as high/medium/low rather than claiming exact membership-inference accuracy.
3. **Apply differential privacy to training, scoped to what you can afford**:
   - Use DP-SGD: per-example gradient clipping (bound L2 norm) plus calibrated Gaussian noise, via Opacus or TensorFlow Privacy. Track privacy loss with an RDP (Renyi DP) accountant rather than naive composition.
   - Target epsilon by risk tolerance: epsilon < 1 is strong privacy with notable utility loss; epsilon 1-3 is a reasonable privacy/utility balance; epsilon > 10 is weak privacy favoring performance. Always pair with a small delta (commonly 1e-5).
   - Combine DP-SGD with PEFT (LoRA, lower rank r=4-8 for stronger privacy): applying noise only to adapter parameters instead of the full model dramatically improves the privacy-utility tradeoff, and is the most practical path to epsilon <= 1.0 on real workloads. Expect 10-100x slower training and budget for 85-90% of the non-private model's task performance as the realistic ceiling.
   - For RAG, apply DP at the injection points that matter: noisy embeddings, randomized ("obfuscated") top-k retrieval via exponential/Gumbel-noise mechanisms instead of exact nearest-neighbor, and/or query perturbation before retrieval.
   - For data with small subgroups, use group-aware or adaptive privacy budgets: uniform DP under-protects minority groups because their smaller sample size makes them more identifiable at the same epsilon.
4. **Use multi-party or distributed training when raw data cannot be centralized**: federated learning (local training per client, FedAvg aggregation) keeps data on-premises at the cost of communication overhead, non-IID data handling, and poisoning/straggler risk; combine with DP-SGD and LoRA client-side for a defensible privacy story. Reserve homomorphic encryption (CrypTen, TF Encrypted, PySyft/MPyC, phe/tenseal) for narrow, specifically sensitive sub-pipelines or inference-only use, not full-scale training, given current compute cost.
5. **De-identify text before it enters training or retrieval**: run a hybrid pipeline (Presidio AnalyzerEngine/AnonymizerEngine plus spaCy NER) to detect PII (names, locations, email, phone, SSN, etc.) and replace with consistent synthetic placeholders (same entity to same token throughout a document) rather than blanking, to preserve linguistic structure and downstream utility. For structured quasi-identifiers (age, zip, etc.), apply k-anonymity (group size >= k), and strengthen with l-diversity or t-closeness if the data has few distinct sensitive values per group; prefer DP over k-anonymity alone when the attack model is sophisticated or the dataset is large. Stage the pipeline: automated detection, human review of a sample, adversarial re-identification validation, then release to training.
6. **Harden the inference boundary with layered defenses**: input validation and prompt-pattern filtering, output scanning for leaked sensitive data before returning a response ("sandwich" the model between pre- and post-processing), rate limiting and auth (JWT, role-based access), network segmentation into public/API/model trust zones, and encryption in transit (TLS 1.2+) and at rest (AES-256/AES-256-GCM). Treat this as defense-in-depth, not a substitute for the training-time mitigations above, since no single layer is foolproof (instruction-tuned refusals on PII prompts are not reliable alone).
7. **Map mitigations to regulatory obligations explicitly**: GDPR Article 25 (privacy/data-protection by design) is the strongest justification for building DP and minimization into the architecture rather than bolting it on; for PHI, document the epsilon target and anonymization steps as the technical control satisfying HIPAA's security and minimum-necessary expectations; note CCPA and the EU AI Act's risk-based posture as additional drivers but expect to need deeper article-level legal review than this skill provides, since the book's own regulatory coverage is high-level, not a full compliance crosswalk.
8. **Document readiness and ongoing operating costs**: record privacy budget consumption per training run, expected 2-10x compute/cost overhead and added latency from DP/FL/HE techniques, and assign clear ownership (privacy engineer, ML engineer, systems engineer) for re-running the evaluation in step 2 whenever the corpus or model changes.

## Output Format
- A privacy risk assessment (ASR, reconstruction error, membership-inference FPR) before and after mitigation, for the Golden Dataset of attack attempts.
- A documented privacy budget (epsilon, delta, accountant method used) tied to the training/fine-tuning run that produced the shipped model.
- A de-identification pipeline spec: which detector (Presidio/spaCy or equivalent), which entity types, placeholder/consistency strategy, and the human-review step.
- A short regulatory rationale memo mapping the chosen technique (DP epsilon, FL, minimization) to the applicable regulation (GDPR Art. 25, HIPAA, CCPA, AI Act) for the specific data type in scope.

## Quality Check
- Verify ASR and reconstruction-error metrics were measured on a held-out probe set, not just eyeballed on a few prompts.
- Verify the reported epsilon accounts for the full training run (all epochs/iterations) via a proper accountant (RDP or moments accountant), not a single-step calculation.
- Verify PII placeholders are consistent per entity across a document (same person stays the same token) rather than randomly re-masked each occurrence.
- Verify sensitive data intended to stay "RAG-only, never in weights" is actually excluded from the fine-tuning corpus, not accidentally included.
- Confirm defense-in-depth exists at inference (input/output filtering) even where training-time DP was applied, since DP bounds membership inference risk but does not guarantee zero leakage of any individual response.

## Common Issues
- Treating instruction-tuning refusals ("I can't share that") as a privacy control: the book's own example shows a DP-SGD-trained model can be made to regurgitate verbatim PII when RLHF/instruction alignment is absent or bypassed; refusals are not foolproof and are not a substitute for DP or minimization.
- Applying DP noise to the full model instead of LoRA/PEFT adapters, which wastes privacy budget on parameters that do not need protecting and tanks utility unnecessarily.
- Using k-anonymity alone on high-dimensional text data: quasi-identifier grouping that works for tabular data (age, zip) does not transfer cleanly to free text, and is vulnerable to homogeneity and background-knowledge attacks without l-diversity/t-closeness or DP.
- Fine-tuning on raw, unredacted sensitive corpora "because RAG will handle access control": retrieval pattern and document-attribution leakage still exist in RAG, so access controls and DP-style retrieval obfuscation are still needed there too.
- Assuming a single epsilon value is self-justifying: always state what delta, accountant, and threat model (which attacker, what auxiliary knowledge) the epsilon claim rests on, since epsilon without context is not comparable across systems.
- Ignoring subgroup effects: uniform DP budgets can leave minority subpopulations with materially weaker protection at the same nominal epsilon; check group sizes before claiming blanket privacy guarantees.
