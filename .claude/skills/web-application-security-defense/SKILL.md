---
name: web-application-security-defense
description: "Design, architect, and review web applications for secure-by-default defense against XSS, CSRF, XXE, injection, DoS, and dependency/business-logic risk."
model: opus
metadata:
  version: 1.0.0
  category: appsec
  source: "Web Application Security, 2nd Edition (O'Reilly, Andrew Hoffman)"
---

# Web Application Security Defense

## Goal
Build and review web applications using the book's recon-offense-defense arc distilled into the DEFENSE pillar only: secure architecture, defense in depth, and concrete mitigations for each major vulnerability class, so that security gaps are caught at the cheapest possible phase (architecture, 30 to 60 times cheaper than production per NIST).

## When to Use
- Architecting a new feature or product that handles authentication, PII, financial data, or search.
- Reviewing a pull request or codebase for security before merge or release.
- Writing or updating a threat model for a new feature.
- Choosing browser security headers, CSP, CORS, or cookie attributes for a web app.
- Building mitigations for XSS, CSRF, XXE, injection, DoS, mass assignment/IDOR, client-side attacks, or third-party dependency risk.
- Setting up a vulnerability discovery, triage, and regression pipeline (SSDL).

## When NOT to Use
- Active exploitation, payload crafting, or penetration testing against a live target (use an offensive hunter skill instead; this skill is defense-only).
- Pure infrastructure/network hardening with no application code surface (use container, cloud, or network-focused skills).

## Authorization Check
- Architecture and code review work needs no special scope, but confirm who owns the feature/codebase and that you are not making unilateral production changes without the usual review process.
- If this work intersects with an active assessment (pentest, bug bounty triage), confirm against `.claude/security-scope.yaml` and coordinate with the security-review owner rather than duplicating effort.

## Methodology

1. **Architect before coding (Secure Application Architecture)**:
   - For every new feature, analyze business requirements first and extract risk categories before any code is written: what credentials/PII/financial data is stored, what authentication/authorization tiers exist, whether a separate search index or cache exists (sync drift is a risk).
   - Mandate TLS for all data in transit; never fall back to plaintext HTTP.
   - Hash credentials with BCrypt (preferred) or PBKDF2 (key stretching, max iteration count your hardware tolerates). Never store or compare plaintext passwords. Reject passwords that are common, or contain the user's name/birthdate/address.
   - Offer MFA (TOTP app or hardware token) for any account holding sensitive data.
   - Apply Zero Trust Architecture (NIST SP-800-207): replace implicit trust ("already past the firewall/moat = trusted") with explicit, continuous verification at every privileged action. A valid session token is not enough: check role/entitlement at time of use, not just time of login.

2. **Harden browser-facing configuration (Secure Application Configuration)**:
   - Ship a Content-Security-Policy on every response (header, not meta-tag-only when avoidable). Minimum viable policy: `default-src 'self'`, explicit `script-src` allowlist, `frame-ancestors 'none'` (or explicit allowlist), `img-src data: https:`, a `report-uri`.
   - Prefer strict CSP: nonce-based (`'nonce-{random}' 'strict-dynamic'`) for server-rendered pages, hash-based (SHA-256 per inline script) for CDN-cached pages. Never ship `'unsafe-inline'` or `'unsafe-eval'` unless truly unavoidable, and treat each as a flagged exception.
   - Configure CORS with an explicit origin allowlist, never a wildcard combined with credentials. Understand simple vs. preflighted CORS requests and that the browser, not your server, enforces the block.
   - Set security headers: `Strict-Transport-Security` (HSTS, with `includeSubDomains` and `preload` where safe), `Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Resource-Policy: same-origin`, `X-Content-Type-Options: nosniff`. Retire legacy headers (`X-Frame-Options` -> CSP `frame-ancestors`; `X-XSS-Protection` removed; `X-Powered-By` disabled).
   - Cookie hardening: always set `Secure`, `HttpOnly`, and `SameSite=Strict` (or `Lax` only when a cross-site flow genuinely requires it) on session/auth cookies. Avoid the `domain` attribute unless subdomain sharing is truly required (it loosens scope to `*.domain.com`).
   - For third-party/untrusted code in-page, prefer sandboxed iframes (`sandbox` attribute) over same-context execution; use Subresource Integrity (`integrity` + `crossorigin`) for any third-party script tag; evaluate Web Workers or Shadow Realms when iframe UI coupling is too costly.

3. **Design a secure user experience**:
   - Treat verbose error messages and predictable identifiers as information disclosure. Use allowlisted, generic error messages (OWASP pattern: "authentication failed" rather than "wrong password" vs "user does not exist") to prevent enumeration.
   - Avoid iterable/guessable IDs and endpoint naming; rate-limit any endpoint that could be enumerated.
   - Use "light patterns" (the inverse of dark patterns): nudge users toward safer defaults (for example, warn about an unset transaction cap at the moment of a risky action, not just buried in settings) rather than relying on documentation alone. Secure-by-default beats a light pattern beats documentation.

4. **Threat model every new feature or significant change**:
   - Collect logic design (what the feature does, in product terms) and technical design (languages, frameworks, data flow, third-party/in-network services, network config, authn/authz model, schema) separately; logic design surfaces business-logic vulnerabilities that technical design alone misses.
   - Enumerate threat actors broadly: external authenticated/unauthenticated users, internal admins, support staff, and machine/service accounts. Internal and machine actors are the most commonly forgotten.
   - Cross-reference logic x technical x actors to build an attack-vector table, each row scored by severity (P0 to P4 or CVSS).
   - List existing mitigations per attack vector, then compute the delta (unmitigated vectors). A feature should not ship until the delta list has an assigned mitigation or an accepted-risk sign-off.
   - Keep the threat model as a living document; revisit it whenever scope changes.

5. **Review code for security, not just functionality**:
   - Sequence: architecture review first, then code review (never the reverse) at merge-request granularity so the full feature scope is visible.
   - Start the review at the client, follow client calls into the API layer, then trace dependencies (DB, helper libs, logging, file/conversion utilities), then hunt for unlinked/forgotten endpoints, then sweep the remainder by risk.
   - Look past archetypal vulnerabilities (XSS, SQLi, CSRF) for business-logic vulnerabilities that require knowledge of the specific feature's rules (for example, a hidden `isMember: true` field bypassing a paid-tier gate).
   - Flag these secure-coding anti-patterns on sight:
     - **Blocklists** instead of allowlists (blocklists require perfect, permanent knowledge of all bad inputs; they always erode).
     - **Boilerplate/default framework code** shipped unreviewed (default configs are often insecure-by-default, e.g., unauthenticated MongoDB, framework version-fingerprinting 404 pages).
     - **Trust-by-default / shared service accounts** where one compromised module (DB, disk, logging) grants access to all three; instead give each module its own least-privilege credentials.
     - **Client/server coupling** (templated HTML mixed with auth logic) instead of a clean data-format boundary (JSON over a defined API contract).

6. **Mitigate each major vulnerability class at the architecture and code level**:
   - **XSS**: never pass unsanitized user data into the DOM; prefer `innerText`/`textContent` over `innerHTML`; avoid dangerous sinks (`innerHTML`, `document.write`, `DOMParser.parseFromString`, `Blob`-as-script, raw SVG with `<script>`); HTML-entity-encode the "big five" characters (`& < > " '`) for text contexts (note: entity encoding does not protect `<script>`, CSS, or URL contexts); disallow or tightly constrain user-uploaded CSS (CSS can exfiltrate via `background:url()` attribute selectors); enforce CSP `script-src` allowlisting plus strict CSP (nonce/hash) as defense in depth, not a substitute for sanitization.
   - **CSRF**: never allow HTTP GET to mutate state; verify `Origin`/`Referer` headers against an allowlist as a first line of defense; implement CSRF tokens (cryptographically random, bound to session/user, time-boxed) as the primary defense, including for stateless APIs (embed user id + timestamp + server-side nonce, HMAC/encrypt); enforce both checks via centralized middleware, never per-route ad hoc.
   - **XXE**: disable external entity resolution and DOCTYPE declarations in every XML parser (verify per-language/parser default, do not assume disabled); prefer JSON or another non-XML format when the payload is just structured data rather than markup/mixed content; remember XXE is a recon foothold that can escalate to RCE, so treat it as high severity even when it first looks read-only.
   - **Injection (SQL/command/code)**: use prepared statements/parameterized queries everywhere (first line of defense, nearly eliminates the class); layer database-specific escaping functions as defense in depth, never as the sole defense; apply principle of least authority so each service/module (DB, disk, logging, CLI wrappers) runs under its own minimally scoped account; never let client input become a literal command or query, instead allowlist the specific operations/commands a client may invoke.
   - **DoS**: log request timing and async job performance so logic/regex DoS is detectable after the fact; statically scan regexes for catastrophic backtracking patterns (e.g., `(a[ab]*)+`) and never accept user-supplied regexes; rank exposed functionality by DoS risk (high/medium/low) rather than binary vulnerable/safe; mitigate DDoS via bandwidth-management/scrubbing services and blackholing, understanding these are mitigations, not prevention, and can also drop legitimate traffic if misconfigured.
   - **Data/object attacks (mass assignment, IDOR, serialization)**: never pass a raw client payload into a DB update; allowlist updatable fields or wrap input in a Data Transfer Object that drops unknown keys; never expose direct/sequential object references, mask and authorize on every file/object access, use high-entropy random identifiers as a stopgap; use well-audited, popular serialization formats (JSON/YAML) and sanitize/allowlist types rather than trusting a deserializer with arbitrary objects.
   - **Client-side attacks (prototype pollution, clickjacking, tabnabbing)**: sanitize/allowlist object keys before merge operations (block `__proto__`, `constructor`, `prototype`); use `Object.freeze()` or `Object.create(null)` selectively, not as a bulk operation (breaks unrelated APIs); set CSP `frame-ancestors 'none'` (or explicit allowlist) as the primary clickjacking defense, framebuster script as fallback only; set `Cross-Origin-Opener-Policy: same-origin` plus `rel="noopener noreferrer"` on all dynamically generated links to stop tabnabbing; where supported, layer in Fetch Metadata request headers (`Sec-Fetch-Site/Mode/Dest/User`) for additional server-side request-context checks.

7. **Secure third-party dependencies**:
   - Model the full dependency tree (including transitive/"fourth-party" dependencies), not just direct imports; use tooling (`npm ls`, Snyk, or equivalent) rather than manual review once the tree exceeds a handful of packages.
   - Continuously diff the dependency tree against a CVE database (NIST NVD or equivalent) in CI, not just at onboarding time.
   - Lock exact versions (remove `^`/`~` ranges) and shrinkwrap/lockfile the full tree; for maximum integrity pin to Git SHAs or run an internal package mirror, since version-number reuse by a malicious or compromised maintainer bypasses simple version pinning.
   - Apply separation of concerns: run risky or loosely trusted third-party integrations on their own isolated service/process with a narrow JSON-over-HTTP contract, rather than importing them directly into the main application process.

8. **Mitigate business logic vulnerabilities**:
   - Use worst-case, not best-case/median-case, design for every architectural decision; a feature that is fast or convenient in the common case but has an exploitable edge case is a worse choice than a slightly slower feature with no edge case.
   - Consider malicious/unintended use for every functional component during architecture, before code is written; this is the cheapest point to catch logic vulnerabilities.
   - For high-value or high-complexity flows, apply statistical/model-based testing: model realistic and edge-case input distributions and user action sequences, replay them via headless-browser automation (e.g., Puppeteer), and log all responses/errors for anomalies that reveal unintended logic paths.

9. **Run an ongoing vulnerability discovery, triage, and regression pipeline (SSDL)**:
   - Automate three layers: static analysis (pre-execution syntax/pattern scanning, strong for statically typed languages, noisier on dynamic languages like JS), dynamic analysis (post-execution behavior in a production-like environment, better for logic/runtime issues), and vulnerability regression tests (functional-test-style assertions that a fixed vulnerability stays fixed, written in the same framework as your normal tests).
   - Stand up a responsible disclosure program and consider a bug bounty program (HackerOne, Bugcrowd, or equivalent) and third-party penetration testing for coverage your internal team will not get to.
   - Reproduce every reported vulnerability in a staging environment before paying a bounty or committing engineering time; this filters false positives and sharpens root-cause understanding.
   - Score every confirmed vulnerability with CVSS v3.1 (Base: Attack Vector, Attack Complexity, Privileges Required, User Interaction, Scope, Confidentiality/Integrity/Availability impact; optionally Temporal and Environmental) or an equivalent system suited to your business/data model; use the score to prioritize, not just to document.
   - Never close a vulnerability with only a partial fix; if a full fix is not yet possible, open a new tracked issue for the remaining exposure. Ship a regression test with every fix.

## Output Format
- Threat model documents (logic design, technical design, threat actor table, attack-vector table with severity, mitigation table, delta table).
- Secure architecture / design review notes mapped to the risk categories above.
- CSP / CORS / security-header / cookie configuration snippets ready for implementation.
- Code review findings categorized as archetypal vulnerability vs. business-logic vulnerability vs. secure-coding anti-pattern, each with a concrete fix.
- CVSS-scored vulnerability triage records and paired regression test specs.
- Dependency-tree risk reports (outdated/vulnerable packages, lockfile/shrinkwrap gaps).

## Quality Check
- Every externally reachable input that reaches the DOM, a SQL/command interpreter, an XML parser, or an object-merge operation has a named mitigation from Methodology step 6, not just "we'll sanitize it."
- CSP, CORS, HSTS, and cookie flags are present and explicit (no silent defaults relied upon) in the final configuration.
- The threat model's delta table is empty, or every remaining row has an owner and a ticket.
- No blocklist is the sole defense for any security-relevant filter; allowlists are used wherever feasible.
- Every closed vulnerability has a corresponding regression test in the suite.
- Dependency lockfile/shrinkwrap is committed and CI fails on new CVEs in the resolved tree.

## Common Issues
- Treating CSP as a substitute for input sanitization rather than defense in depth; CSP does not stop DOM-based XSS that occurs purely client-side without a network round trip.
- Relying on `Referer`/`Origin` header checks alone for CSRF defense; an XSS on an allowlisted origin defeats header-only checks, so CSRF tokens are still required.
- Assuming XML parsers disable external entities by default; many (notably several Java parsers) do not.
- Using median/best-case performance or UX reasoning when a worst-case/adversarial framing is needed, especially for business logic and algorithmic complexity decisions.
- Pinning only the top-level dependency version while transitive dependencies float; use a full lockfile/shrinkwrap, not just a version pin on direct imports.
- Writing detailed, literal error messages or sequential IDs that enable enumeration, even when no other vulnerability is present.
- Doing code security review before architecture review, or skipping architecture review entirely under deadline pressure; this is consistently the most expensive ordering mistake per the book's NIST-cited cost data (30 to 60x cheaper in architecture phase than in production).
