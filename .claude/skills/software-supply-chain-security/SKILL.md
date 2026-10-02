---
name: software-supply-chain-security
description: "Inventory software supply chains, produce SBOMs, secure build/CI/CD provenance, and assess third-party and consumption risk."
model: opus
metadata:
  version: 1.0.0
  category: devsecops
  source: "Software Supply Chain Security (O'Reilly)"
---

# Software Supply Chain Security

## Goal
Establish end-to-end trust and transparency across the software supply chain, from source code and dependencies through build, signing, distribution, and third-party suppliers, so compromises (SolarWinds, Codecov, Log4j, dependency confusion) are prevented, detected, or contained.

## When to Use
- Standing up or maturing a software supply chain security program from scratch.
- Introducing SBOM generation (SPDX/CycloneDX) for a product or release pipeline.
- Hardening source code, build, and CI/CD systems against tampering (post-incident or proactively).
- Assessing a supplier, open source dependency, or vendor product for supply chain risk.
- Preparing attestations for customers or US government procurement (CISA attestation form, NIST SSDF compliance).
- Evaluating whether to adopt or consume third-party/open source code safely (S2C2F-style intake).

## When NOT to Use
- Pure application-level vulnerability testing with no dependency/build/provenance angle (use a dedicated AppSec/pentest skill instead).
- One-off code review of a single pull request with no supply chain exposure.
- Physical logistics/shipping security unrelated to software, firmware, or hardware manufacturing.

## Authorization Check
- Confirm sponsorship and scope with the accountable security/product security leader (CISO, CPSO, or GRC owner) before auditing another team's pipeline or a supplier relationship; this is a governance assessment, not a pentest.
- For supplier assessments, confirm contractual "right to audit and assess" exists before requesting evidence, access, or on-site review.
- For CI/CD or source system changes, confirm change management approval; these are often production-adjacent systems.

## Methodology
1. **Scope the supply chain**: Map the specific chain you are securing using the book's model: people, processes, source code, build/CI/CD, distribution/deployment, and (if applicable) manufacturing. Identify every environment in play (developer, code repo/build, test/lab, preproduction/production, distribution, customer staging) since each has distinct risks.
2. **Classify source and dependencies**: Inventory code by type, open source, commercial, proprietary, OS/framework, low-code/no-code, generative-AI-produced, since each carries different integrity and licensing risk. Flag trusted vs. untrusted dependencies; quarantine and internally host (do not live-link) third-party packages to avoid dependency confusion (npm/pip/gems) and typosquatting.
3. **Secure source and build integrity**: Apply the SLSA (Supply-Chain Levels for Software Artifacts) framework to the build track (provenance generation, signed provenance, hardened build platform) and, where source track guidance is immature, apply NIST SSDF (SP 800-218) control PO.3.2 for securing tools, repos, and pipelines. Require: least-privilege, MFA-gated authentication/authorization on repos and build systems; peer code review (2+ reviewers for OSS inclusions); SAST/SCA/secrets-scanning in the pipeline; ephemeral (short-lived, hermetic, no-network) build environments; repeatable/reproducible builds verified by checksum comparison (SHA256, never MD5/SHA1).
4. **Sign and verify artifacts**: Require code signing (PKI, trusted CA) for all code, drivers, scripts, and application files; use Sigstore for free automated signing/verification/key management where a commercial CA is not in place. Validate signatures/hashes at every distribution and deployment hop, not just at the first build step (the Codecov hack was caught only because a customer manually diffed a hash).
5. **Produce software transparency artifacts**:
   - Generate an SBOM for every production release, in SPDX or CycloneDX (JSON), including at minimum the NTIA elements: author, timestamp, supplier, component name/version, other unique identifiers (CPE/PURL/SWID), and dependency relationships (including transitive).
   - Know the SBOM's limits: naming ambiguity, missed proprietary/commercial components, backported patches not reflected, no operational ingestion without VEX, and version drift against asset inventory.
   - Add adjacent BOMs where relevant: HBOM (hardware), SaaSBOM, OBOM (operational), AI-BOM/ML-BOM, CBOM (cryptography).
   - Attach provenance (in-toto attestation format referenced by SLSA: layout of required steps, recorded metadata per step, bit-for-bit verification against the layout) and, where required, a VDR (vulnerability disclosure report, signed) or VEX (vulnerability exploitability exchange, signed) record rather than only a CVE list.
   - Consider SCITT (Supply Chain Integrity, Transparency, and Trust) principles for append-only, independently auditable attestation logs, and GUAC for aggregating SBOM/provenance/vulnerability metadata into a queryable graph.
6. **Map to SSDF and SLSA levels**: Translate findings into NIST SP 800-218 SSDF's four practice areas, PO (Prepare the Organization), PS (Protect Software), PW (Produce Well-Secured Software), RV (Respond to Vulnerabilities), and state the target/current SLSA build level (0 through 3) explicitly in any report. Cross-reference NIST SP 800-161 (C-SCRM) controls (access control, configuration management, incident response, supply chain risk assessment) for organization-wide risk management, not just the pipeline.
7. **Assess suppliers and third-party risk**: Run supplier cyber assessments covering IT/environmental security, product security org maturity, SDL/testing practices, build/DevSecOps/release management, vulnerability SLAs, cloud environment controls, and manufacturing where applicable. Require cyber agreements/contract addendums (including right-to-audit) and establish ongoing monitoring, not a one-time assessment. For consuming open source, apply S2C2F-style intake: check OpenSSF Scorecard and Best Practices Badge, pin versions, quarantine before use, re-vet on every update (assume compromise on update, same as first intake).
8. **Attest for regulated/government consumers**: Where selling to the US government, prepare the CISA Secure Software Development Attestation Common Form (secure dev environment, trusted source code supply chain, provenance maintenance, automated vulnerability tooling) and align with OMB Memo M-22-18.
9. **Close the loop with people and process controls**: Ensure change management (logged, reviewed, least-privilege) covers code, systems, and environments; build security champions and role-specific training (development, DevSecOps, manufacturing, field services) so controls are sustained, not just documented once.

## Output Format
- A supply chain risk inventory mapped to the book's environments (developer, repo/build, test, preprod/prod, distribution, manufacturing).
- An SBOM (SPDX or CycloneDX JSON) per release, plus any applicable HBOM/SaaSBOM/AI-BOM.
- Provenance/attestation artifacts (in-toto layout + recorded steps, signed VDR/VEX) and a stated SLSA build level with gap list to the next level.
- A control set (numbered, e.g., SCBD-xx/ST-xx style) mapped to NIST SSDF practice areas and, where relevant, NIST SP 800-161 or SLSA, ready to merge into an existing controls framework.
- A supplier/dependency risk register with assessment status, contract/audit-right status, and monitoring cadence.

## Quality Check
- Every production release has a generated, retained SBOM; spot-check that transitive (n-2+) dependencies are present, not just top-level.
- Build artifacts are reproducible: independently rebuilding produces a matching checksum.
- No code, driver, script, or application file ships unsigned; signature/hash verification is enforced at distribution, not assumed from the build step alone.
- Every third-party/open source dependency is internally hosted (quarantined and mirrored), never pulled live from a public registry at build time.
- Supplier risk register shows an assessment date, a right-to-audit clause, and a next-review date, not just a one-time intake.
- Controls are traceable to a named framework (SSDF, SLSA, 800-161, ISA/IEC 62443-4-1) so a customer or auditor can verify coverage.

## Common Issues
- Treating an SBOM as sufficient on its own: without VEX/VDR it cannot drive vulnerability operations, and without asset inventory matching it goes stale the moment a version patches.
- Signing software without protecting the signing keys themselves (Nvidia's stolen private keys were used to sign malware); key custody is part of the control, not an afterthought.
- Assuming SLSA build-level controls stop SolarWinds-class attacks; the build platform was hardened enough that only the highest SLSA level (and separate source integrity controls) would have helped, don't oversell lower levels.
- Letting "repeatable build" mean "same script ran twice" instead of verified checksum equality, that is not reproducibility.
- Re-trusting an open source dependency on update without re-running the same due diligence (code review, SCA, scan) as the original intake, treat every update as a new, unverified artifact.
- Skipping the people/training layer: controls documented in a policy but never trained into security champions or developers will not hold under schedule pressure.
