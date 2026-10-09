---
name: threat-intelligence-fundamentals
description: "Build defensible nation-state/APT threat assessments by auditing attribution assumptions, mapping adversary capability and intent, and translating findings into resilience guidance."
model: opus
metadata:
  version: 1.0.0
  category: threat-intel
  source: "Inside Cyber Warfare, 3rd Edition (O'Reilly)"
---

# Threat Intelligence Fundamentals

## Goal
Produce defensible, actionable threat intelligence on nation-state and APT actors by treating attribution as an inference problem, mapping adversary capability and intent across the cyber-conflict and cognitive-warfare landscape, and converting that analysis into resilience-focused defensive strategy rather than headline-driven blame assignment.

## When to Use
- Drafting or reviewing a threat actor profile, APT campaign writeup, or nation-state attribution assessment.
- Evaluating a vendor's or government's attribution claim before repeating or acting on it.
- Assessing exposure from operational technology (OT) / industrial control system (ICS) and critical-infrastructure threats, including supply-chain-enabled sabotage.
- Assessing exposure to disinformation/misinformation or cognitive-warfare campaigns targeting your organization, staff, or partners.
- Building a strategic risk brief for leadership on cyber-conflict posture, geopolitical threat landscape, or resilience investment priorities.
- Advising on whether civilian or volunteer cyber activity (e.g., "hacktivist" involvement) carries combatant/legal risk.

## When NOT to Use
- Hands-on incident response, malware reverse engineering, or log/forensic triage (use incident-response or forensics skills instead).
- Planning, authorizing, or conducting any offensive cyber operation, hack-back, or targeting activity. This skill is analytical and defensive only.
- Enterprise vulnerability management or patch prioritization (use defensive-security-foundations or software-supply-chain-security).

## Authorization Check
- Confirm the request is for defensive/strategic analysis (board brief, risk register, threat model, security review) and not for operational or offensive planning.
- If tied to a formal engagement or published assessment, confirm scope and sponsor authority; reference `.claude/security-scope.yaml` where one exists.
- Attribution and legal-risk content is sensitive and reputationally loaded: confirm who the assessment will be shared with and at what confidence level before distribution.

## Methodology
1. **Build the adversary profile (capability x intent)**: Identify sponsor type (state intelligence/military unit, state-aligned proxy/PMC, criminal-as-proxy, hacktivist), demonstrated capability (espionage, financial theft, OT/kinetic sabotage, information operations), and strategic intent (coercion, disruption, territorial conflict support, economic advantage). Use the book's case studies as reference patterns: PLA Unit 61398/Titan Rain (economic espionage), Russia's GUR vs Gazprom (sabotage via long-term supply-chain access), Israel/Stuxnet lineage at Natanz (sustained OT sabotage campaign), Predatory Sparrow at Khouzestan Steel (SCADA-triggered physical destruction with public claim), Wagner Group + Internet Research Agency under Prigozhin (paired kinetic and cognitive operations).

2. **Audit every attribution claim against its underlying assumptions, not just its conclusion**: Attribution is inferred (abductive reasoning), never deduced. Before accepting or publishing an attribution, explicitly test it against the known-weak assumptions documented in APT attribution literature (Steffens):
   - *Exclusive-use assumption*: malware/tooling is not proprietary to one actor. Code and implants get leaked, reverse-engineered, reused, or seized and redeployed by other actors (e.g., X-Agent used beyond APT28 after public disclosure; captured adversary malware reused by Ukraine's GUR against new targets).
   - *Working-hours assumption*: "9-to-5 in the actor's timezone" is weak evidence because multiple capable states and proxies share or overlap timezones (Russia spans 11; Iran, Israel, Ukraine, the UAE and others sit within one to two hours of Moscow time).
   - *Criminals-vs-spies assumption*: criminal and espionage activity are not mutually exclusive; espionage-as-a-service and hackers-for-hire blur the line (Su Bin case; EaaS vendors documented by TrendMicro).
   - For each claim, ask two questions out loud in the writeup: "What are the assumptions?" and "What is the evidence?" Flag claims resting on speculation (hat/photo-based identification, machine-translated forum content, WHOIS registrant names) as low-confidence.
   - Assign and state an explicit confidence level (low/moderate/high) for every attribution; never present an inference as a fact. Independent validation (e.g., Kaspersky's Equation Group assessment later confirmed by the Shadow Brokers leak) is rare and should be called out as such when it exists.

3. **Structure findings with standard CTI frameworks for actionability**: once the capability/intent profile and attribution confidence are established, organize observed or reported technical activity using the Diamond Model (adversary, capability, infrastructure, victim) to pivot across related incidents, and sequence TTPs against the Cyber Kill Chain or MITRE ATT&CK so defenders can map detections and gaps stage by stage. Treat the book's own intelligence cycle, F3EAD (Find, Fix, Finish, Exploit, Analyze, Disseminate), as the operational-intel analogue: reframed defensively, Find/Fix maps to surfacing and confirming a campaign's infrastructure and ownership, Finish maps to containment and remediation (never "kill/capture"), and Exploit/Analyze/Disseminate maps to extracting IOCs and TTPs from captured artifacts and pushing them to stakeholders fast enough to stay ahead of the adversary's next move.

4. **Map the OT/critical-infrastructure and supply-chain attack pattern**: cyber attacks with kinetic effects (fire, explosion, physical destruction) rarely show a malware signature and won't show up on VirusTotal, so signature-based defense fails here. Use the recurring pattern from documented cases (Aurora Generator Test; Stuxnet/Natanz; Gazprom pipeline ruptures; Khouzestan Steel) as a defender's checklist: (a) adversary profiles the supply chain and vendor list; (b) initial access via a trusted supplier/vendor (phishing); (c) lateral movement and network mapping inside the target; (d) identification of safety/alarm systems that are unmonitored, disconnected, or never finished (the Urengoy pipeline's alarm system had been non-functional for a decade before the explosion); (e) manipulation of the SCADA/protective-relay/PLC logic to force an out-of-sync or unsafe condition. Audit your own OT environment against each stage, especially stage (d): unfinished or silently-disabled safety instrumentation is the single most repeated root cause across these cases.

5. **Assess the cognitive-warfare / information-operations dimension as its own intelligence discipline**: campaigns from state-aligned actors (Internet Research Agency) typically pair with kinetic or cyber operations rather than standing alone. For each suspected influence operation, log actor, platform (Telegram, X, Facebook, TikTok, VK were the most abused in documented cases), narrative, and objective, using the EU EEAS FIMI taxonomy of common tactics: shifting blame, distorting context, distracting attention, and impersonating trusted organizations or individuals. When evaluating whether a specific piece of content targeting your organization or staff is disinformation, apply the source-reliability checklist (adapted from the Union of Concerned Scientists): does it separate fact from opinion, cite credible experts, have a traceable original source, confirm existing beliefs or play to emotion, come from a stakeholder with an undisclosed interest, require a secret conspiracy, scapegoat a group, or originate from a high-follower account with little history? Multiple "yes" answers indicate likely disinformation requiring independent verification before any internal or public response.

6. **Model legal and policy exposure for civilian/volunteer cyber activity**: when assessing whether a hacker, employee, or affiliated volunteer's offensive cyber activity could expose them (or the org) to legal or combatant-status risk during an armed conflict, apply the direct-participation-in-hostilities (DPH) test from international humanitarian law: threshold of harm (did the act negatively affect military operations/capability), causal link (is there a direct causal chain between the act and that harm), and belligerent nexus (is the act actually connected to the conflict, as opposed to unrelated crime such as ransomware or fraud that happens to occur during wartime). All three conditions must be met for DPH status to attach. Use this purely as a risk-assessment lens, never as guidance to conduct such activity.

7. **Evaluate sabotage/attack effectiveness against strategic objectives, not just technical success**: a technically successful OT/kinetic attack does not guarantee strategic benefit. Natanz suffered repeated, confirmed sabotage over a decade yet Iran's installed advanced-centrifuge count grew substantially over the same period. Include a cost/benefit framing in strategic assessments: time and resources sunk into offensive sabotage versus the demonstrated long-term effect on adversary capability, and use this to temper alarmist conclusions about any single incident.

8. **Translate analysis into resilience-first defensive recommendations**: because kinetic-effect and supply-chain-enabled attacks often have no signature to detect, the primary mitigation is architectural, not signature-based. Recommend redundant, independently cross-checking control paths for safety-critical functions (the "two is one, one is none" principle used in nuclear plant control systems, fly-by-wire aircraft, and redundant medical device controllers), strict segmentation of civilian/public from military/operational infrastructure, and supply-chain vendor security verification (confirm safety and alarm systems are actually connected and monitored, not just contractually specified).

## Output Format
- Adversary/threat-actor profile brief: sponsor, capability, intent, confidence-graded attribution, and relevant case precedent.
- Diamond Model / Kill Chain / ATT&CK-mapped campaign summary with explicit confidence levels per claim.
- OT/ICS and supply-chain risk checklist mapped to the five-stage attack pattern in step 4.
- Disinformation/information-operations incident log (actor, platform, narrative, objective, FIMI tactic).
- DPH legal-exposure assessment (step 6) when civilian/volunteer activity is in scope.
- Strategic risk brief for leadership: capability-vs-intent landscape, effectiveness evaluation, and resilience recommendations (not offensive countermeasures).

## Quality Check
- Every attribution claim in the output states both its assumptions and its evidence, and carries an explicit confidence level.
- No finding treats malware/tooling reuse, working hours, or "criminal vs. state" category as conclusive evidence on its own.
- OT/critical-infrastructure findings do not rely on "no malware signature found" as a clean bill of health.
- Legal-exposure content uses the three-part DPH test in full; no single condition is treated as sufficient.
- Recommendations are exclusively defensive/resilience-oriented; nothing in the output could serve as offensive operational guidance.

## Common Issues
- Target fixation: defaulting to the "usual suspect" nation-state (e.g., always China for IP theft, always Russia for financial crime) without weighing other capable states or non-state proxies.
- Treating a commercial vendor's attribution report as fact because it drove headlines; commercial incentives (deal valuation, differentiation from competitors) can bias published findings.
- Confusing a technically impressive sabotage event with strategic success; verify long-term effect on adversary capability before claiming the operation "worked."
- Ignoring the supply-chain/vendor reconnaissance phase when scoping OT defenses, and overlooking safety systems that were never finished or silently disconnected.
- Analyzing cyber and information operations in isolation when the same sponsor is running both in a coordinated, enmeshed campaign.
