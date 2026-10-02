---
name: kubernetes-security-observability
description: "Design and audit a holistic Kubernetes security and observability strategy across build, deploy, and runtime stages."
model: opus
metadata:
  version: 1.0.0
  category: cloud-security
  source: "Kubernetes Security and Observability (O'Reilly)"
---

# Kubernetes Security and Observability

## Goal
Help a platform, security, or SRE team design, review, or harden a Kubernetes cluster's defensive posture (identity, workload security, network segmentation, supply chain, secrets, runtime defense) together with the observability stack needed to detect and investigate incidents. This is a defensive architecture and governance skill, complementary to offensive container assessment tooling, not a replacement for it.

## When to Use
- Reviewing or designing infrastructure, cluster, or workload hardening for a new or existing Kubernetes environment.
- Defining RBAC, Pod Security Standards, admission control, or network policy for a cluster.
- Building or auditing a CI/CD image supply chain (scanning, signing, admission gating).
- Designing secrets management strategy for workloads.
- Standing up observability for security purposes (flow logs, audit logs, DNS logs, metrics, eBPF-based telemetry).
- Building a threat defense, intrusion detection, or SOC capability for a Kubernetes fleet.
- Mapping trust boundaries and RBAC responsibilities across platform, network, security, and application teams.

## When NOT to Use
- Active exploitation, live penetration testing of a running cluster, or red-team validation of container escapes (use `container-hunter` or an authorized offensive skill instead).
- Pure application-code vulnerability review unrelated to cluster or workload configuration.
- Non-Kubernetes container platforms with no orchestration layer (bare Docker hosts), though many principles still transfer.

## Authorization Check
- Confirm you are reviewing infrastructure the requester owns or has an architecture mandate for; this skill produces configuration guidance and policy artifacts, not live changes to production clusters.
- For any live cluster inspection (reading RBAC, PSP/PSA, NetworkPolicy, admission webhook config), confirm read-only access scope and that findings go through the normal change process before being applied.
- If asked to also validate exploitability of a finding (e.g., confirm a privilege escalation path works), hand off to an authorized offensive skill under `.claude/security-scope.yaml`; do not perform exploitation here.

## Methodology
1. **Anchor on the three-stage lifecycle**: Every control maps to build, deploy, or runtime. Build = image hardening, scanning, secrets-in-code avoidance. Deploy = cluster hardening, RBAC, admission control, network policy authoring. Runtime = Pod Security, kernel isolation, network enforcement, threat detection. Always identify which stage a gap belongs to before recommending a fix, and check whether the relevant team (dev, platform, security) has ownership.
2. **Harden infrastructure first**:
   - Host: prefer immutable, container-optimized OS (Flatcar, Bottlerocket), remove nonessential processes, apply host-based firewalling via the network plug-in (Calico, Weave Net, Kube-router) rather than raw iptables where possible.
   - Cluster: secure etcd with dedicated TLS/PKI for peer and client traffic, encrypt Kubernetes Secrets at rest (local or KMS/envelope encryption via cloud KMS or HashiCorp Vault), rotate credentials frequently, restrict alpha/beta feature gates, enable and tune Kubernetes audit policy (stages: RequestReceived/ResponseStarted/ResponseComplete/Panic; levels: None/Metadata/Request/RequestResponse), keep Kubernetes upgraded, use CIS Benchmarks and `kube-bench` to verify.
   - Network boundary: decide cluster internet-reachability deliberately; layer perimeter firewalls/security groups (coarse, node-level) with in-cluster NetworkPolicy (workload-aware); consider taints, nonoverlay/BGP-routable pod networks, or egress NAT gateways where perimeter devices need pod-level granularity.
3. **Lock down workload identity and RBAC**: Use Kubernetes RBAC (Role/ClusterRole + RoleBinding/ClusterRoleBinding) as the default authorization mode; treat Node, ABAC, and AlwaysAllow/AlwaysDeny as legacy or dev-only. Grant least privilege via groups, not individual users. Use namespaces as the primary trust boundary between teams, with one namespace per microservice. Verify the RBAC privilege-escalation guard (users can't grant permissions on Roles/ClusterRoles they don't hold, absent explicit `escalate` verb). Restrict cloud metadata API access from pods (IRSA/Workload Identity/managed identity instead of node-wide credentials, plus NetworkPolicy blocking the metadata IP).
4. **Enforce Pod Security and kernel isolation**: Apply Pod Security Standards / SecurityContext (PSP is deprecated since v1.25; use Pod Security Admission or an admission controller such as Kyverno/OPA Gatekeeper for equivalent controls): non-root user, drop all capabilities and add back only what's needed, `allowPrivilegeEscalation: false`, read-only root filesystem, restrict hostNetwork/hostPID/hostIPC and hostPath volumes. Layer kernel defenses: seccomp (`runtime/default` profile, custom syscall allowlists for sensitive workloads), SELinux (MAC via MCS labels, blocks several historical container-escape CVEs such as CVE-2019-5736), AppArmor (per-microservice profiles, more flexible than SELinux), and sysctl (safe vs unsafe, scope to specific nodes via node affinity).
5. **Secure the image and secrets supply chain**: Minimize base images (distroless or `FROM scratch` multistage builds), pin image versions (no `latest`), verify provenance/signing, scan at multiple points in CI/CD (registry scan, post-build scan, inline/admission-time scan) and pick a failure mode (fail-open vs fail-closed) deliberately. Gate deployment with an admission controller (OPA Gatekeeper, Kyverno) to enforce image provenance, label/network-policy schema compliance, and to restrict dangerous fields (e.g., Service `externalIP`). For secrets: avoid plaintext etcd storage, prefer a secrets manager (Vault, AWS/GCP/Azure secrets services) or the Secrets Store CSI Driver, encrypt in transit and at rest, rotate automatically, use ephemeral/dynamic secrets where possible, keep secrets out of env dumps/logs, and recognize the "secret zero" problem (the KEK/root credential protecting everything else).
6. **Design network policy and microsegmentation as code**: Default-deny ingress and egress per namespace (and cluster-wide via a GlobalNetworkPolicy-style construct where supported), always define both ingress and egress for every pod, standardize label/policy schemas so teams can self-serve, and treat policy as reviewed code in CI. Use policy recommendation, policy impact preview, and policy staging/audit-mode tooling before enforcing changes. For exposing services externally, understand which mechanism (ClusterIP, NodePort, LoadBalancer with `externalTrafficPolicy: local`, eBPF-native service handling, or Ingress) preserves or obscures client source IP, since that determines whether NetworkPolicy can actually restrict by client. Use richer network policy implementations (Calico GlobalNetworkPolicy, tiers, service-account-based selectors) or hierarchical policy tiers when RBAC alone can't cleanly split responsibility across platform/security/app teams.
7. **Encrypt data in transit deliberately**: Choose between application-layer (code libraries, highest effort/risk), sidecar or service mesh (Envoy/Istio mTLS, consistent but adds operational complexity), or network-layer (IPsec/WireGuard or a CNI with built-in encryption, generally the best operational-simplicity/performance tradeoff). Map the choice to compliance drivers (PCI, HIPAA, GDPR, SOC 2).
8. **Build observability for security, not just ops**: Collect and correlate, with Kubernetes context attached at collection time (labels, namespace, service account, policy verdict), not joined after the fact: network flow logs, DNS activity logs (NXDOMAIN spikes, latency, DGA patterns), application-layer (L7/HTTP) flow logs, Kubernetes audit logs, and process/syscall telemetry via Linux kernel tools (eBPF, kprobes, NFLOG). Visualize via service graphs and flow-ring diagrams. Use distributed tracing (request-ID propagation via Envoy, or eBPF/kprobes for code-free tracing). Layer machine learning baselining (unsupervised, since workloads are ephemeral) for anomaly jobs: IP sweep detection, port scan detection, service bytes anomaly, process restart anomaly, DNS latency anomaly, L7 latency anomaly, HTTP connection spike anomaly. Feed high-fidelity alerts to a SIEM; consider UEBA for entities (services, service accounts) at scale (50+ clusters) where manual dashboard review stops working.
9. **Implement threat defense and intrusion detection mapped to the kill chain**: Use the cyber kill chain / Microsoft's Threat Matrix for Kubernetes (initial access, execution, persistence, privilege escalation, defense evasion, credential access, discovery, lateral movement, impact) to structure defenses; you only need to block one stage to thwart an attack. Implement IP/domain threat feeds (STIX/TAXII, open-source block lists) enforced via network policy and logged via a log-processing engine. Add deep packet inspection (DPI) inside the cluster (not just at ingress) for signature-based detection (OWASP Top 10, SANS Top 25), using a transparent proxy (Envoy) so DPI doesn't impact application traffic. Use canary pods/honeypots in sensitive namespaces to detect lateral movement with high-fidelity, low-false-positive alerts. Watch for DNS-based (DGA) command-and-control patterns that bypass IP/domain feeds since the domains are algorithmically generated.

## Output Format
- Gap analysis mapped to build/deploy/runtime stage and to MITRE ATT&CK / Microsoft Threat Matrix for Kubernetes stages.
- RBAC Role/ClusterRole + binding manifests, Pod SecurityContext / Pod Security Admission policies, NetworkPolicy (default-deny plus per-microservice allow rules), and admission controller policy (OPA/Rego or Kyverno) drafts.
- Secrets management recommendation (CSI driver, Vault, or cloud KMS) with rotation and encryption-at-rest configuration.
- Observability architecture: what to log (flow, DNS, audit, L7, process), where to correlate, and which anomaly-detection jobs to run.
- Threat defense runbook: threat feed sources, DPI placement, canary/honeypot plan, alert routing to SIEM.
- A security and observability checklist tailored to the cluster's stage of Kubernetes adoption (learning / pilot / production).

## Quality Check
- Every NetworkPolicy change has gone through impact preview or staging/audit mode against real or historical flow data before enforcement.
- Default-deny policies exist in every namespace, including ones to be created in the future (cluster-wide policy or provisioning template).
- No pod runs privileged, as root, or with hostNetwork/hostPID/hostPath unless explicitly justified and scoped.
- Secrets never appear in plaintext in etcd dumps, audit logs, or CI logs; encryption at rest is confirmed by rewriting existing secrets after enabling it.
- RBAC bindings use groups and namespaced Roles wherever possible; ClusterRole/ClusterRoleBinding use is deliberate and reviewed.
- Audit logging, flow logs, and DNS logs are actually flowing to the SIEM/observability platform and are enriched with Kubernetes metadata, not just raw IPs and five-tuples.
- Failure mode of every admission controller and scanning gate (fail-open vs fail-closed) is a conscious decision, documented against the organization's risk tolerance.

## Common Issues
- Treating shift-left image scanning as a complete security strategy; it does not replace deploy-time and runtime controls.
- Forgetting egress rules: default-deny ingress alone leaves lateral movement and exfiltration paths open.
- Assuming NodePort/LoadBalancer services preserve client source IP by default; without `externalTrafficPolicy: local` or an eBPF-native dataplane, kube-proxy NATs away the original IP and NetworkPolicy client restriction silently stops working.
- Enabling cluster-wide default-deny policy without pre-staging failsafe ports and control-plane allow rules, which can break the cluster.
- Relying on perimeter firewalls/security groups alone; they rarely have pod-level granularity and cannot see same-node east-west traffic.
- Treating PSP as current guidance; PSP is deprecated since v1.25. Use Pod Security Admission or a general-purpose admission controller (OPA Gatekeeper, Kyverno) instead, but the underlying SecurityContext fields and kernel-hardening concepts (seccomp, SELinux, AppArmor) still apply.
- Alert fatigue from rule-based thresholds in a system where workloads are ephemeral; prefer ML baselining (unsupervised) over static thresholds.
- Running DPI only at ingress; attacks that originate inside the cluster (compromised sidecar, node OS vulnerability) never cross the ingress point and will be missed.
- Not rotating the "secret zero" (KEK or root credential); every envelope-encryption scheme is only as strong as its root of trust.
