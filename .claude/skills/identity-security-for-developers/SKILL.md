---
name: identity-security-for-developers
description: "Design authentication, authorization, secrets, and machine identity into software so no long-lived credential ever has to leak for an attacker to win."
model: opus
metadata:
  version: 1.0.0
  category: identity
  source: "Identity Security for Software Development (O'Reilly)"
---

# Identity Security for Software Development

## Goal
Build applications, pipelines, and workloads where every human and machine actor is authenticated, authorized to the minimum scope needed, and never dependent on a hardcoded or long-lived secret, so a single leaked credential cannot become a full compromise.

## When to Use
- Designing or reviewing AuthN/AuthZ for a new service, API, or SSO integration.
- Choosing how a service, script, CI job, or container gets its credentials (secrets manager, env vars, workload identity).
- Reviewing code, Dockerfiles, Kubernetes manifests, or CI pipelines for hardcoded secrets, API keys, or static credentials.
- Setting up a secrets manager, rotation policy, or workload identity federation (SPIFFE/SPIRE, cloud IAM roles, OIDC federation).
- Hardening a CI/CD pipeline or software supply chain (provenance, SBOMs, signing).
- Advising on OAuth 2.0 / OIDC flows, JWT usage, or service-to-service (machine) identity in Kubernetes/service meshes.

## When NOT to Use
- Pure UI/UX or performance work with no identity, secrets, or access-control surface.
- Low-level cryptographic algorithm design (defer to a dedicated cryptography reference); this skill covers key/secret lifecycle, not cipher internals.
- Incident response for an active breach (use an incident-response playbook; this skill is preventive/architectural).

## Authorization Check
- Confirm you have authority to change AuthN/AuthZ configuration, secrets stores, or IAM roles in the target environment (per this org's change-approval rules: Dev before Staging before Production).
- For anything touching secrets managers, cloud IAM, or Kubernetes RBAC, confirm who owns the resource and that production changes have approval.
- Never treat this skill's output as license to print, log, or transmit real secrets; redact examples.

## Methodology

1. **Classify the identity first**: human (end user, admin, developer, contractor) or machine (service, script, CI job, container, IoT device, AI agent). Machine identities now outnumber human ones by roughly 82:1 per industry data, and they fail differently: volume, variety, velocity, and (lack of) visibility. Pick the AuthN/AuthZ mechanism appropriate to the category rather than reusing human-style username/password for machines.

2. **Separate AuthN from AuthZ explicitly**:
   - AuthN answers "who are you" (username/password, MFA, biometrics, token-based, mTLS certificates). AuthZ answers "what can you do" (RBAC, ABAC, PBAC, ACLs, XACML) and only runs after AuthN succeeds.
   - Use OIDC as the AuthN/identity layer and OAuth 2.0 as the AuthZ/delegation framework; they are complementary, not interchangeable. OAuth 2.0 itself does not authenticate users.
   - For OAuth 2.0 integrations: store client credentials in a secrets manager (never hardcoded), request scopes incrementally (least privilege, not "ask for everything at login"), strip unused scopes before production, and handle refresh-token expiration/revocation explicitly rather than silently re-requesting access.
   - For JWTs: keep payloads light and claims flat, set short, reasonable `exp` times, sign with strong algorithms, transmit only over TLS, and always re-validate against the authorization server rather than trusting the token's self-contained claims blindly (a stolen or forged signature is a real attack class).

3. **Eliminate long-lived, static secrets wherever possible**:
   - Never hardcode credentials in code, scripts, config files, environment variables, build manifests, README files, or logs; these are the most common leak sources developers forget.
   - Use dynamic, short-lived secrets with an enforced time-to-live (TTL) instead of static ones whenever the secrets manager supports it.
   - Where a long-lived secret is unavoidable, externalize it to a secrets manager/vault (e.g., HashiCorp Vault, CyberArk Secrets Manager/Conjur, AWS/Azure/GCP secret services) and retrieve it at runtime rather than at build time.
   - Protect secrets in memory: encrypt memory where supported, zero it after use, and avoid immutable structures (e.g., Java `String`) that cannot be forcibly garbage-collected.

4. **Prefer workload identity federation over distributing secrets at all**. This is the strongest fix for "secret zero" (the master credential that unlocks everything else):
   - Use SPIFFE/SPIRE to give every workload a cryptographically verifiable identity (a SPIFFE ID, e.g. `spiffe://dev.example.com/pricingservice/api`) backed by X.509 or JWT-SVIDs, issued per-workload via a local workload API. This removes the need to retrieve API keys/passwords from a secrets manager at all for service-to-service calls, enables cross-cloud AuthN without provider-specific tokens, and can sign CI/CD artifacts for provenance.
   - In Kubernetes, use cert-manager (+ csi-driver / csi-driver-spiffe) to automate issuance and rotation of short-lived TLS/SPIFFE certificates per Pod, mounted via CSI volumes rather than stored in Kubernetes Secrets, so the private key never persists after the Pod is deleted.
   - In service meshes (Istio, Linkerd, Cilium), rely on the mesh control plane to issue and auto-rotate per-Pod X.509 identities for mTLS, and enforce AuthZ policies keyed on service identity (not network location or IP), starting from a default-deny policy and adding explicit allow rules.
   - In cloud environments, prefer the provider's workload identity federation (short-lived STS-issued credentials bound to a service account/role) over long-lived access keys.

5. **If a centralized secrets manager is still required, apply all seven management principles**: encryption (at rest and in transit, keys in a KMS/HSM, never roll your own crypto), access control (RBAC + MFA, least privilege, time-boxed privilege escalation with "break-glass" procedures), monitoring and auditing (log every create/use/rotate/revoke with who/what/when), compliance mapping (HIPAA, PCI DSS, GDPR, ISO 27001 Annex A 8.28 as applicable), testing (including simulating the secrets service being unavailable), automation (remove humans from rotation/creation/revocation to kill "easy password" behavior), and centralization (one repository or a federated set, to avoid secrets sprawl and shadow-IT "security islands"). Document explicitly how the resulting secret-zero (the master key protecting the vault itself) is bootstrapped and protected with its own multi-factor AuthN.

6. **Enforce least privilege and RBAC for every service identity**, not just human roles: define Role / Privilege / Resource policy statements (e.g., Conjur-style YAML policies) per machine identity, grant only read/execute on the specific secrets or resources a workload needs, and review/revoke access the moment a workload is decommissioned. Treat admin/root-equivalent access to build servers, CI runners, and Kubernetes service accounts as privileged access requiring the same scrutiny as human privileged accounts.

7. **Build rotation and revocation into the secret lifecycle from day one**: define a rotation schedule for every credential class (API keys, DB passwords, TLS certs, signing keys), automate it, and make sure rotation doesn't require a code change or rebuild (externalize, don't hardcode). Revoke immediately on compromise or role change, and expire tokens/sessions aggressively to shrink the attacker's window of opportunity.

8. **Harden the CI/CD pipeline and software supply chain as an identity surface**: eliminate credentials in environment variables and build scripts (the Codecov Bash Uploader breach pattern) in favor of a secrets manager; require code signing and signature verification for artifacts; target a SLSA build level (L1 reproducible/documented build through L3 tamper-resistant, automated, access-controlled build) appropriate to the project's risk; generate and store SBOMs (SPDX or CycloneDX) alongside artifacts; and use sigstore (Fulcio short-lived certificates + Cosign signing + Rekor transparency log) so artifact identity itself is backed by short-lived, verifiable credentials rather than a long-lived signing key sitting in CI secrets.

9. **Apply zero trust and risk-based AuthN as the default posture**: assume any code, repo, IDE, or pipeline step can be observed by an attacker; validate every request's identity and device health rather than trusting network location; and where supported, use adaptive/risk-based AuthN (step up to MFA or deny on anomalous location/device/time) instead of uniformly annoying all users with MFA on every login.

## Output Format
- Identity/secrets architecture recommendation naming the specific mechanism (OIDC provider, SPIFFE/SPIRE, cert-manager issuer, cloud workload identity, secrets manager) and why it fits the human-vs-machine, volume/variety/velocity profile of the target.
- Concrete config/code artifacts where relevant: SecretStore/ExternalSecret manifests, cert-manager Certificate/Issuer YAML, Istio/K8s AuthorizationPolicy or NetworkPolicy, OAuth2/OIDC client setup, Conjur-style RBAC policy YAML.
- A rotation/revocation plan per credential class with owners and TTLs.
- A short list of hardcoded-secret or long-lived-credential findings (file:line) with the externalization or federation fix for each.

## Quality Check
- No credential, API key, or password appears as a literal string in code, config, env files, or manifests reviewed; anything that must exist is in a secrets manager or replaced by workload identity federation.
- Every machine identity has a scoped policy (role/privilege/resource) rather than broad or admin-equivalent access.
- Every issued credential (token, cert, secret) has an expiration/TTL and a defined rotation path that does not require a rebuild.
- AuthN and AuthZ are implemented as separate, composable layers, not conflated in one check.
- CI/CD build artifacts have traceable provenance (signed, with an SBOM) if the project's risk profile warrants it.

## Common Issues
- Treating OAuth 2.0 alone as "login": it is an AuthZ delegation framework, not an AuthN protocol; pair it with OIDC for identity.
- Requesting all OAuth scopes up front instead of incrementally, leaving unused high-privilege access live in production.
- Storing JWTs or API keys with no expiration, turning a short-lived token into a de facto permanent credential if stolen.
- Centralizing all secrets behind one vault without hardening the vault's own AuthN, recreating a single secret-zero master-key failure point.
- Using Kubernetes Secrets (base64, not encrypted by default) as if they were a vault; prefer CSI-mounted, per-Pod short-lived certificates where possible.
- Letting CI pipelines pull credentials into environment variables that get logged or forwarded (the Codecov Bash Uploader pattern); externalize and scope pipeline secrets per job/step.
- Manual, human-driven secrets rotation that gets skipped under deadline pressure; automate rotation so it cannot be silently deferred.
