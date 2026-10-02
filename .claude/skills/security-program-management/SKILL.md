---
name: security-program-management
description: "Stand up or revitalize an InfoSec program by cultivating relationships, aligning to company risk culture, and building it on documentation, governance, architecture, and communications."
model: opus
metadata:
  version: 1.0.0
  category: security-governance
  source: "The Cybersecurity Manager's Guide (O'Reilly)"
---

# Security Program Management

## Goal
Build or rescue an InfoSec program by treating security leadership as a relationship-driven, culture-change discipline first and a technical discipline second: cultivate alliances, align the program to the company's actual risk tolerance, lay down the four cornerstones (documentation, governance, architecture, communications), then distribute ownership across the organization (the "neighborhood watch") and prove progress with a handful of metrics that matter.

## When to Use
- Taking over a new InfoSec/security leadership role, whether building a program from zero or inheriting one from a predecessor.
- The existing program is sumo-style (InfoSec imposing controls by force) and generating friction, stalled initiatives, or CISO churn.
- You need a charter, information security policy, or incident response plan drafted or refreshed.
- Leadership is asking "are we more secure than last year" and you have no simple metric to answer with.
- You need to decide what to centralize in the security team versus hand off to IT/engineering/DevOps.
- Preparing for or recovering from a misaligned audit engagement.

## When NOT to Use
- Doing a specific cyber-risk assessment, risk register, or quantitative risk model: use `cyber-risk-management-program` instead. This skill is about running the overall security function and its relationships; that one is about the risk discipline specifically.
- Writing or reviewing a single technical control, architecture diagram, or pentest: hand off to the relevant technical or offensive-security skill.
- Pure compliance-checklist work with no governance/relationship component.

## Authorization Check
- Confirm you have (or are seeking) executive sponsorship and a reporting line; this methodology assumes you may have neither, so proceed even without a signed charter, but flag to the user when a recommendation (for example, publishing a charter or policy) requires a senior executive signature (CEO/COO/CIO) before it carries weight.
- Confirm who the stakeholder/sponsor is for any document you draft (charter, policy, SIRP) so review and sign-off routing is clear before circulating.
- For anything touching live audits, legal holds, or HR investigations, confirm scope with legal/HR before acting; those are the one domain (staff investigations) the book says InfoSec should own exclusively and handle with care.

## Methodology
1. **Read the operating environment before building anything**. Assume, until proven otherwise, that nobody in the company cares much about InfoSec, nobody fully understands the job, and the industry's fear-based messaging distorts budget and tooling decisions. Gather evidence: how did the company react to its last breach or incident (money, attention, accountability), does it treat InfoSec policy violations like HR violations, is asset enumeration actually maintained. This diagnostic sets realistic expectations before step 2.

2. **Step 1: Cultivate relationships, continuously**. Treat relationships as the core deliverable of the job, not a soft skill on the side; most breaches are first identified by staff, not the security team, so every employee is a customer worth investing in. Use the "judo, not sumo" posture: don't force controls on unwilling, uneducated teams through positional power; roll with the business's momentum toward a better shared position. Never use vulnerability scans, audits, or findings to publicly embarrass another team; report findings to the system owner first and let them fix it before management hears "bad news." Amplify others' contributions, downplay your own.

3. **Step 2: Ensure alignment to the company's actual risk tolerance**. Interview widely to find where the company's risk needle sits (0 to 10), using signals such as: did anyone get disciplined after the last incident, is spend following InfoSec's recommendations, do information owners engage voluntarily. Calibrate your policy, architecture, and controls to that reading, not to NIST/ISO/OWASP defaults (those assume near-maximum risk aversion) and not to your personal risk appetite. Recognize misalignment early: chronic complaints from clients, repeated reorganizations of the security function, high CISO turnover, or using the audit team as a weapon are all red flags. Revisit alignment on a cadence; don't treat it as a one-time exercise.

4. **Step 3: Lay the four cornerstones**, started roughly in this order and spanning the first 6 to 18 months:
   - **Documentation**: draft (don't own the enforcement of) three documents: the InfoSec charter (one page, RACI-style, states what leadership wants from the security team and what IT/engineering owes in return, signed by the CEO/COO), the information security policy (behavioral expectations for all staff, calibrated to the risk tolerance found in step 2, reviewed every six months, enforced by line managers not InfoSec), and the security incident response plan (SIRP, with roles/responsibilities and a post-incident lessons-learned step). Build each collaboratively ("drag it through the streets") so ownership is shared before it is ever enforced.
   - **Governance**: stand up three advisory councils rather than make unilateral decisions: the Security Business Council (business-unit reps; strategy, purchases, policy, phishing program), the Extended Security Council (the most technical staff in the company; technical standards and the "tasty topics"), and the Executive Security Council (senior leaders; final review, board-presentation preview, tone from the top). Co-chair each with someone outside InfoSec.
   - **Security architecture**: present a simple defense-in-depth model as concentric layers (cloud, perimeter, internal network, data center/servers, endpoints, data, people) and populate each layer's current controls with the owning team, without judging gaps yet. Use it to educate owning teams (for example, show them the NIST control list for their layer) rather than to audit them.
   - **Communications, education, and awareness**: budget a dedicated communications/training role if at all possible; the charter and policy are inert until taught. Repeat messages (rule of seven) across channels, and prioritize training IT/engineering staff directly, since a trained ally effectively becomes an extension of your team.

5. **Step 4: Give your job away and build the "neighborhood watch"**. Recognize that security functions migrate to the teams that operate the systems (this has been the multi-decade industry trend: firewalls to network services, endpoint controls to IT, identity to operations, cloud security to DevOps) and lean into it rather than resist it. Use the charter and governance councils to make this transfer explicit, not silent. Involve system owners in tool selection and proofs of concept before buying anything. When a team owns a piece of security poorly, respond with patience, praise, and education (bring an industry framework to the table) rather than public correction; reserve escalation (for example, bringing in the audit team) as a last resort for teams that habitually ignore the partnership ("invisible middle finger").

6. **Step 6: Organize the team around people skills, not just technical depth**. Hire or develop engineers who can both operate technically and represent the security function credibly to five to seven client groups; a technically brilliant but socially unable team member actively damages the relationship-first model. Cultivate an "extended security team" of engineer-advocates outside your reporting line, and recognize and reward them visibly and often.

7. **Step 7: Measure what matters and report it as ROI**, resisting the urge to track dozens of vanity metrics. Track two: (a) can staff recognize a security-policy violation in their own area and do they know how to report it, and (b) can staff recognize and report a phishing attempt. Both are proxies for the same underlying goal: is the organization becoming self-defending. Run continuous phishing simulation with a target failure rate under about 3%, track "repeat offenders" for targeted remediation, and test awareness at the office farthest from headquarters to avoid a headquarters-bias blind spot. Present these metrics (not raw vulnerability counts) to the board as evidence the program is working.

8. **Partner with, don't fight, the audit function**. Audit teams frequently lack InfoSec depth and default to generic checklists (often a SOX/compliance legacy) that waste cycles on low-value findings while missing systemic gaps. Invest in the relationship with the chief auditor specifically; try to get a seat at annual audit planning so you can steer scope toward the real gaps. Treat audit as a tool of last resort to force action from teams that have repeatedly ignored partnership, not as a weapon for routine use.

9. **Operate as a cultural change agent, not an asset owner**. Internalize that you do not own the company's systems and will never be resourced to secure them directly; your job is to get every department to secure its own information assets. Every meeting is a chance to move the culture, and success is measured by the organization's growing ability to defend itself, not by the size of your team or toolset. Stay technically current (SANS, vendor-neutral industry threat reports) even if your core lane is now cultural/organizational.

## Output Format
- A short written alignment assessment (risk-tolerance read, misalignment signals found) to ground later decisions.
- Draft charter (one page, RACI-style), information security policy, and SIRP, each with a clear review/signature routing plan.
- A governance model: three councils with charter, membership, cadence, and a standing agenda template.
- A defense-in-depth architecture diagram (layers, current controls, owning team per control).
- A communications and training plan (channels, cadence, audiences, the "rule of seven").
- A two-metric dashboard: policy-violation recognition/reporting rate and phishing simulation failure rate, trended over time, with board-ready framing.
- A staffing/role profile emphasizing the technical-plus-relational hybrid skill set.

## Quality Check
- Every cornerstone document names who is accountable for enforcement (hint: usually not InfoSec) and who signed off.
- The policy's control requirements are traceable back to the alignment read in step 3, not copy-pasted from a framework at a stricter risk posture than the company's.
- Governance councils have members and a cadence, not just a slide describing them.
- The two core metrics are actually being measured, trended, and presented, not merely proposed.
- No finding or scan result from InfoSec has been delivered in a way that shames a system owner in front of their management without warning.
- The program's plan distinguishes what InfoSec will own outright (staff investigations, incident command, log analysis, policy, SIRP) from what it will hand off or co-own (endpoint controls, network security, IAM, cloud/DevOps security).

## Common Issues
- Treating this as a technology-procurement exercise: buying tools before relationships and alignment exist produces an architecture nobody uses and a bigger budget ask nobody will approve.
- Running a pentest or formal risk assessment in the first weeks on the job; it reads as a "love letter" of bad news before trust exists and can alienate the very teams you need as partners.
- Writing policy at NIST/ISO/government-grade strictness for a company whose actual risk tolerance is far lower; this produces unenforceable rules and credibility loss.
- Letting InfoSec scan or audit other teams' systems directly instead of administering the tool and letting system owners run their own scans; this manufactures "friendly fire" resentment.
- Chasing dozens of security metrics to look rigorous instead of the two that map to the program's actual goal (a self-defending organization).
- Showing up to audit kickoff meetings passively; without InfoSec steering scope, audits default to generic checklists and miss the real gaps.
- Confusing "giving your job away" with abdication: the transfer must be explicit (charter, RACI) and still governed through the councils, not a silent drop of responsibility.
