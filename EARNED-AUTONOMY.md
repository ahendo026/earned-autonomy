# Earned Autonomy: A Portable Framework for Governed AI Self-Improvement

**Revision 3, September 30, 2026.** Base: revision 2 (Bullit repository commit 8c10291, July 29, 2026). This file, EARNED-AUTONOMY.md in github.com/ahendo026/earned-autonomy, is the single source of truth. The copy in the Bullit repository is a pointer to this file. The Drive copy is a snapshot exported from this file and supersedes all earlier exports in that folder.

**Patent pending.** Portions of this framework, including graduation and reversal, the never-earnable action class, and their combination with the Person-Sovereign Health Record, are the subject of a pending United States provisional patent application.

**What changed in revision 3.** Autonomy is now revoked as well as earned (automatic reversal, Section 2 step 9 and Section 32). A permanently non-earnable action class is defined (Section 5). Consent and autonomy are separated into two axes (Section 6). Consequence tiers are mapped to the American Medical Association's CPT Appendix S taxonomy (Section 9). Model selection becomes a governed action with a person-side posture (Section 22). A healthcare worked example is added (Section 7). The register gains rows for the healthcare implementations (Section 38). Em-dashes are removed throughout and technical terms are defined at first use.

-----

## At a glance

- **What it is:** a drop-in layer for any AI-using application that (1) **governs** what the AI is allowed to change and read, and (2) **observes** what the AI costs, how good it is, and how the app is actually used.
- **The core idea:** *autonomy is earned, not granted, and it can be taken back.* The AI proposes; a human approves; every decision is remembered; the accumulated record is the only thing that can justify more autonomy, and granting it is always a human act. When the record deteriorates, autonomy is withdrawn automatically. Some actions can never be earned at all.
- **Two terms used throughout.** The **fail-closed chokepoint** is the single function every AI action passes through; fail-closed means the default answer is no, so anything unknown, disabled, or over its limits is rejected. The **escalation guard** is the rule that the powers to mint agent credentials and to widen an agent's autonomy exist only on the human side of the system, so an agent can never grant itself more authority.
- **What it is not:** full or autonomous recursive self-improvement, and not certified "compliance." It is governed self-improvement, with controls *mapped* to government and industry frameworks.
- **Status:** built and live in two reference implementations, arh-tracker (Python/FastAPI) and Bullit (TypeScript/Next.js), and adopted as the governance posture of the IntelaCare clinical AI practice and the SHIRLe Health record platform (Section 38 lists exactly what is built where). Designed for extraction into a standalone open-core product.
- **How to read this document:** Part I explains the system and its loop in plain language, including what can never be automated. Parts II and III are the two technical contracts (governance, observability). Part IV is the business case and extraction path. Part V is the exact implementation contract, the byte-precise names an implementation must not vary. Part VI is the conformance and divergence register, where each live implementation differs from the contract and the direction of remediation.

-----

# Part I: The System

## 1. What the system is

A drop-in layer that gives any AI-using application two things:

**1. Governance.** Agent and AI writes flow through the fail-closed chokepoint, governed by a capability matrix with three modes (off, propose, auto). Proposals queue for human approval. Every action is attributed, audited, and reversible. Agent *reads* are governed through the same matrix (Section 13, Governed reads), making data egress as legible as mutation. Autonomy is **earned**: the system recommends promoting a capability to auto-apply only when its human-decision track record clears an explicit evidence gate, and promotion remains a human act. Autonomy is also **revoked**: when the track record falls below the gate, or the conditions that earned it change, the capability drops back to propose without waiting for a human (Section 32, The mode ladder and state machine). A defined class of actions is **never eligible** for auto regardless of track record (Section 5, The never-earnable class).

**2. Observability.** Every AI call is recorded with cost (tokens times price), classified by a **call-type taxonomy** (an app-local call site plus a standard cross-app task code), and joined to downstream outcomes for a **quality** signal per type. Human interaction with the app is recorded in its own stream (Section 19, Usage self-monitoring) so product decisions are grounded in real behavior. Cost times quality per type is the substrate for per-type model routing: *the cheapest model whose quality holds.*

**Provenance.** Born in arh-tracker (Python/FastAPI), ported into Bullit (TypeScript/Next.js), then adopted as the governance posture for IntelaCare and SHIRLe Health. This proves the central claim: the portable asset is the **contract** (table names, endpoint paths, state machine, mode ladder), not a codebase.

## 2. The loop: governed self-improvement in nine steps

1. **AI proposes.** A suggestion originates from one of four sources: a *capture*, a *backfill*, the *agent*, or a *routing change*.
2. **The chokepoint checks it.** One fail-closed gate validates identity, capability, consent scope, and limits. Anything unknown or disabled is rejected and logged.
3. **The proposal queues.** It is recorded but **not applied**; the data it would change is untouched.
4. **A human decides.** Approve executes the change; reject discards it. Either way, the decision is permanently recorded.
5. **Evidence accumulates.** The audit log keeps the full lifecycle; the acceptance track record tallies approve and reject per source and per capability; every AI call's cost is logged per type.
6. **The system recommends, only past an evidence gate.** Promote an eligible capability to auto. Promote emergent vocabulary. Inform the person or operator that a different model clears the gate for a task type. Below threshold, it stays silent.
7. **A human approves the recommendation.** The system cannot adopt its own suggestions (the escalation guard). Actions in the never-earnable class are never recommended for auto, because they cannot be promoted at all.
8. **Policy changes lead to better proposals.** More accurate, lower-friction, cheaper proposals generate new evidence, and the cycle repeats, each turn gated by a human.
9. **Autonomy reverses when the evidence no longer supports it.** If a promoted capability's acceptance falls below the gate, an auto-applied action is rolled back, the model behind it changes, its scope or limits change, or it goes unused for the dormancy window, it drops back to propose automatically and the reason is logged. Re-promotion requires a fresh track record. Reversal is the one policy change the system makes without a human, and the only direction it may move on its own is toward less autonomy.

**In one sentence:** AI suggestions from four sources all pass one no-exceptions gate, wait for a human yes or no, and every decision is remembered, until the accumulated record itself justifies the system asking for more trust, which only a human can grant, and which the system withdraws on its own the moment the record stops supporting it.

## 3. The loop, term by term

### The four proposal sources

| Source | Definition | Concrete example |
| :-- | :-- | :-- |
| **capture** | AI analysis of *new input* at the moment it enters the system | A photographed note says "Sara, Jeff's daughter" and yields a proposed child-of relationship |
| **backfill** | A *retroactive AI scan over existing data*, structuring facts recorded as text but never as data | "Friend of Jeff Morson" found on an entity page yields a proposed relationship |
| **agent** | The conversational assistant acting on a *user instruction* via its tools | "Lindsey is Jeff's friend" yields a relationship proposal |
| **routing change** | A proposed change to the *system's own configuration*: which AI model a call type runs on | "Move extraction to the smaller model" (designed) |

Each source is tagged as the **actor** on everything it proposes, which is what makes per-source track records possible.

### The control stages

- **Chokepoint (fail-closed).** The single function every governed write passes through: token valid? capability enabled? consent scope covers the target? limits respected? *Fail-closed means the default answer is no.* There is no second path, so the guarantees are structural, not conventional.
- **Approval queue.** The proposal is recorded (state: proposed) but not applied.
- **Human approve or reject.** Approve executes the stored change; only now does it actually happen. Reject discards it forever.

### The evidence stages

- **Audit log.** Append-only lifecycle record: who proposed it, what it was, its before and after state, and its outcome. The kept before-state enables one-click rollback.
- **Acceptance track record.** Per-source, per-capability approved-versus-rejected tallies, the system's report card ("backfill's suggestions: 19 of 20 accepted"). A track record is bound to the actor, the capability, the data class, and the model that produced the proposals; change any of those and the record starts over.
- **Cost per call type.** Every AI call logged with token cost, by call type and model, the spend half of the evidence.

### The improvement stages (the recursive part)

- **The system recommends**, but only past explicit thresholds (in the reference implementations: at least 10 decisions and at least 90% acceptance): *promote capability to auto* (trusted work applies without per-item approval), *promote emergent vocabulary* (terms the AI coined get formally adopted and feed back into its prompts), *inform of a better model per type* (same quality, lower cost, or higher quality at acceptable cost).
- **A human approves.** Adoption is an Admin act the system cannot perform on itself.
- **Policy changes lead to better proposals.** The loop's payoff, and its recursion.

### The reversal stage

- **The system demotes.** On any reversal trigger (Section 32) a capability in auto returns to propose. Demotion never goes to off; off is an Admin decision. Demotion is logged with its trigger as a state transition in the audit log, so an auditor can see when and why autonomy was withdrawn without reading any domain data.

## 4. What this is, and is not (claims discipline)

Precision here is a feature: regulated-industry buyers and auditors reward accurate claims and punish inflated ones.

**This IS:**

- **Governed self-improvement, or earned autonomy.** The system observes its own performance and proposes changes to its own operating policy; every expansion of policy passes through a human; every contraction happens automatically. The recursion is real, but bounded and human-gated.
- **Compliance-aligned by design.** Controls mapped to NIST AI RMF, HIPAA, SOC 2, ISO/IEC 42001, FDA PCCP, the AMA CPT Appendix S taxonomy, DoD Responsible AI principles, and NIST SP 800-171, emitted machine-readably via the conformance manifest.

**This is NOT:**

- **Full or autonomous recursive self-improvement.** The system never modifies its own code, models, prompts (beyond the reviewed vocabulary feedback), or the governance rules themselves. Its improvement targets are enumerated, and the escalation guard makes self-granted autonomy structurally impossible.
- **A path to unsupervised clinical action.** The never-earnable class (Section 5) means the framework cannot be configured, even by an Admin with a perfect track record, to let an AI prescribe, share a record, alter consent, or delete data on its own.
- **Certified or "compliant."** No attestation has been pursued; compliance in the strict sense is a property of a deployed organization plus system, not of a toolkit. The claim is *alignment*: mapped controls that make later certification documentation, not re-architecture.

**Preferred product vocabulary:** "earned autonomy" and "governed self-improvement." Avoid unqualified "RSI" in outward-facing material.

## 5. The never-earnable class

Some actions are excluded from graduation by construction. A capability in this class carries a **cap** of propose in the capability matrix: the mode ladder for it is off or propose, and the promote-to-auto path rejects it at the chokepoint, whatever its track record and whoever asks. The cap is a column on the capability row (Section 29), so it is visible in the manifest and cannot be removed by the same Admin action that sets a mode.

The class is defined by consequence, not by domain. An action belongs in it when its effect is irreversible, when it moves data or rights outside the boundary the person or organization controls, or when a wrong action cannot be undone by rollback. The reference members:

| Action | Why it is never earnable |
| :-- | :-- |
| **Export or share** a record or any part of it outside the app | Once data has left, rollback cannot retrieve it |
| **Modify consent**: widen, narrow, or revoke what an agent or party may access | Consent is the person's act; an agent changing it would be granting itself access |
| **Prescribe, order, or sign** a clinical action | Legally and clinically a licensed human's act; the AI may draft, never sign |
| **Delete** a record or an audit entry | Destroys the evidence the whole model depends on |
| **Mint, revoke, or rescope** an agent token or capability | The escalation guard, restated as a capability |

Apps add members to the class; they cannot remove the reference members. In the Person-Sovereign Health Record (PSHR) this class is the **never-AI-approvable request**: a request to move data or change rights that only the person, by a recorded act, can approve (PSHR Architecture v0.20, Section 2.12, Earned Autonomy, and Section 5.5, Requests and participant identity).

The class is what lets a governance committee approve a deployment quickly. The question "what is the worst thing this AI could do on its own" has a fixed answer that no configuration, promotion, or track record can change.

## 6. Two axes: consent governs access, autonomy governs action

The framework separates two questions that are usually conflated:

- **Consent** answers *what may the agent see*. It is expressed as the per-resource, per-data-class allow-list on the data.read capability (Section 13). It is set by the person or the data owner, it can be withdrawn at any time, and withdrawal takes effect at the next chokepoint check.
- **Autonomy** answers *what may the agent do without a human, and to what*. It is expressed as the mode (off, propose, auto) on each write capability, scoped per data class, and it moves only through the evidence-gated promotion and automatic reversal path.

The two axes never move each other. Promotion to auto never widens read scope; a wider read scope never changes a mode. A capability at auto on one data class stays at propose on another. Withdrawing consent to a data class does not demote a capability, but it does remove the data that capability would act on, so the agent's proposals on that class simply stop. Both axes are enforced at the same chokepoint, in code.

**Per-data-class scoping.** Every capability row and every read allow-list is keyed by data class as well as by resource (Section 29). Protected classes carry the propose cap by default: in healthcare, substance use disorder records under 42 CFR Part 2, reproductive health, and behavioral health notes are seeded never-earnable and read-off until the person opens them. In the PSHR this is **consent-as-routing**: consent is not a flag checked after retrieval but the rule that decides whether a record is routed to the agent at all.

## 7. A worked example: medication titration at IntelaCare

IntelaCare PBC is a Medicare-enrolled clinical AI practice and a CMS ACCESS Model participant. Its governance posture, designed in before launch, reads as follows through the two axes.

**Consent (access).** The clinical agent holds a data.read allow-list covering the patient's medication list, vitals, and recent laboratory results for the condition under management. Behavioral health notes and Part 2 records are off unless the patient has opened them for this purpose.

**Autonomy (action), capability by capability.**

| Capability | Data class | Mode today | Eligible for auto? |
| :-- | :-- | :-- | :-- |
| Assemble the pre-visit brief | vitals, labs, meds | propose | Yes, after the gate |
| Flag a lab value outside the titration range | labs | propose | Yes, after the gate |
| Draft a titration proposal (dose change) | meds | propose | No: propose cap (consequential, needs clinical judgment) |
| Sign the order | meds | never | No: never-earnable (prescribing) |
| Share the brief with a receiving provider | all | never | No: never-earnable (export) |

**The loop in practice.** The agent reads within consent, drafts a titration proposal, and queues it. The clinician (the User role, Section 9) approves or rejects; the decision is logged with the before and after state of the medication record. Over weeks the pre-visit brief capability accumulates a track record. When it clears 10 decisions at 90% acceptance, the system recommends promotion; the Medical Director (the Admin role) decides. The titration proposal capability accumulates the same record and is never recommended, because its cap is propose. If the model serving the brief changes, the brief drops back to propose on the same day and earns its way back.

**What the committee sees.** The manifest reports every capability, its mode, its cap, its approve rate, and the reversal events, with no patient data. The audit log answers "what did the AI do, who approved it, and when was its autonomy reduced" for any date.

## 8. What exists today (reference implementations)

| Piece | Status |
| :-- | :-- |
| Chokepoint plus capability matrix plus approval-queue log | Built (both apps) |
| Governance tile: batch review, audit, rollback, master and shadow switch | Built (both apps) |
| Governed AI writes with provenance plus acceptance metrics per source | Built (both apps) |
| Governed agent reads (data.read egress surface, Section 13) | Built (arh-tracker) |
| Earned autonomy: evidence-gated promotion recommendations; emergent-vocabulary promotion with prompt feedback | Built (Bullit) |
| Automatic reversal (demotion triggers, Section 32) | Designed |
| Never-earnable cap on the capability row (Section 5) | Designed |
| Per-data-class scoping of capabilities and read allow-lists (Section 6) | Designed |
| AI cost pillar: per-call cost, spend tiles, 30-day series | Built (both apps) |
| Usage self-monitoring (human interaction telemetry, Section 19) | Built (arh-tracker) |
| Call-type taxonomy plus cost per type by model matrix | Built (Bullit) |
| Quality per call type, closed-loop learning (outcome links plus decision join) | Built (both apps) |
| Conformance manifest: guarantees plus standards mappings plus call-type registry | Built (both apps) |
| Telegram review surface (inline approve and reject) | Built (both apps) |
| Installer skill (port-governance; TypeScript plus Python kits) | Built |
| Per-type model routing as governed config; model-change recommendations | Designed |
| Person-side governed model selection (Section 22) | Designed (PSHR local client) |
| Hardening roadmap (Section 37) | Designed |
| Hosted management and aggregator app | Designed |

Where each built piece lives, per app: the app's port note (BULLIT-PORT.md; arh-tracker's AGENT_AUTONOMY.md Section 6 serves this role until its port note exists). "Built (one app)" rows are divergences, tracked in Part VI. The healthcare implementations are listed in Section 38.

-----

# Part II: The Governance Pillar (the control contract)

## 9. Roles, separation of duties, and consequence tiers

| Role | Who | Authority |
| :-- | :-- | :-- |
| **Admin** (governor) | human | Changes governance config: mints and revokes tokens, defines capabilities, sets each capability's mode, cap, and limits. The only role that promotes to auto. Cannot remove a reference cap. |
| **User** (operator) | human | Acts within policy: approves and rejects proposals, reviews the log, rolls back. Cannot mint tokens or promote to auto. |
| **Agent** (actor) | AI or automation | Acts under a scoped token with granted capabilities. May propose always; auto-applies only where promoted. Never holds human credentials. |
| **Aggregator** | external management app | Read-only consumer of anonymized governance metrics only. |

**Escalation guard (invariant).** Token-minting and promote-to-auto live only on the human surface. A leaked agent token cannot escalate itself. The system may demote on its own; it may never promote.

**Single-user collapse.** One human can hold both Admin and User; the powers stay modeled distinctly so a multi-user or PHI deployment splits them cleanly later.

**Consequence tiers.** Each capability is assigned a tier at definition. The tier sets the highest mode the capability may reach and gives outside readers a recognized vocabulary: the American Medical Association's CPT Appendix S taxonomy for AI-enabled medical services, which classifies a service as **assistive** (the AI detects or surfaces data; the clinician interprets), **augmentative** (the AI analyzes or quantifies; the clinician interprets and acts), or **autonomous** (the AI interprets and reaches a clinically relevant conclusion without a clinician).

| Tier | Effect of the action | Highest mode | Appendix S level when it reaches that mode |
| :-- | :-- | :-- | :-- |
| 0 | Surfaces or organizes data; no state change | auto (after the gate) | assistive |
| 1 | Reversible state change with no clinical or rights consequence (tag, draft, schedule, summarize) | auto (after the gate) | augmentative |
| 2 | Consequential and reversible only with human review (a dose proposal, a coding suggestion, an order draft) | propose (capped) | augmentative, always human-interpreted |
| 3 | Irreversible, rights-bearing, or boundary-crossing (Section 5) | propose (never-earnable) | never autonomous |

A capability at tier 0 or 1 that reaches auto is, in Appendix S terms, an assistive or augmentative service operating without per-item review; the framework never produces an autonomous service at tier 2 or 3.

**Per-app posture.** The framework is identical across apps; each app dials the posture. Default everywhere: propose-first, fail-closed. Low-stakes (single-user, non-PHI): the Admin may promote tier 0 and 1 capabilities to auto once their track record justifies it; interactive in-chat tool use may stay live-gated by the user. High-stakes (healthcare, PHI, multi-user): every capability starts at propose, **all** agent writes including interactive ones route through the chokepoint, tiers 2 and 3 are capped, Admin and User split across real identities, protected data classes are read-off by default, and retention and right-to-delete are enabled on the audit log.

## 10. The three primitives (table contracts; names are fixed)

- **agent_tokens**: scoped, attributable agent identity. An app with an existing token layer reuses it via the identity adapter.
- **agent_capabilities**: the permission matrix; key is a control identifier, never a value or PHI. Carries mode, cap, tier, data class, and limits.
- **agent_action_log**: append-only; **doubles as the approval queue** via its state column. The kept before-state enables rollback. Records promotions and demotions as state transitions. **Local-only**: may hold domain data; never crosses the export seam.

Exact column contracts: Section 29.

## 11. The chokepoint (fail-closed semantics)

One function every agent write routes through, perform_agent_action(actor, capability, target, payload):

1. Validate the agent token, else reject.
2. Load the capability; missing, disabled, or off, then reject and log *failed*.
3. Check consent: the target's data class must be within the actor's read allow-list, else reject and log *failed*.
4. Enforce limits; on violation, reject and log *failed*.
5. Mode auto: apply via the registered handler, log *applied*. Mode propose: log *proposed* and **do not apply**.

Approval later runs the same handler and flips proposed to approved plus applied. Reject flips proposed to rejected (never applied). Rollback replays the before-state through the handler and, if the action was auto-applied, triggers demotion of that capability (Section 32).

Promotion requests pass through the same function with capability = governance.promote: a capability whose cap is propose is rejected here, in code, before any human sees a recommendation.

This is the one place human-in-the-loop, the consent boundary, and the data firewall are enforced **in code, not by convention**.

## 12. Two surfaces (separation of duties at transport)

- **Agent surface (/api/agency/\*).** Token-gated; the perimeter auth is off here, so the **scoped token is the sole gate** and the agent never uses human credentials. The agent may propose and execute granted capabilities, read its own log plus the conformance manifest, and, where granted, use the governed read surface (Section 13). It cannot reach token or capability management.
- **Human surface (/api/governance/\*).** Behind the app's perimeter auth. Approve, reject, and review (User); set modes and limits, promote, mint and revoke tokens (Admin).

## 13. Governed reads: data egress under the same matrix

Writes are not the only thing worth governing: what the AI is allowed to **see** is a capability too, and it is the consent axis of Section 6. A scoped read surface (GET /api/agency/\<resource\>) under a **data.read** capability lets an agent ground itself on live app data **without holding the human perimeter credential**, the least-privilege alternative to sharing the app password.

Read-capability semantics (they differ from writes):

- propose is meaningless for a read; the routes treat mode == 'auto' (plus enabled) as on; anything else returns 403. Seed **off** (fail-closed).
- limits_json carries a **per-resource, per-data-class allow-list**: which named resources and which data classes this token may read. This allow-list *is* the consent record for the agent.
- **Every read is logged** to the audit trail (capability = 'data.read'), so data egress is as legible as mutation.
- **Retention split:** read-egress rows are high-volume and may be pruned after N days (arh-tracker: 30); **mutation rows are append-only and never pruned.** Log views return mutations by default, with reads opt-in (?include_reads=1), a read-time filter and retention policy, not a drop.

## 14. Export seam and conformance manifest

The action log is local-only. Only **aggregate governance metrics** may cross to an aggregator: counts by capability, approve and reject rates, auto-versus-propose mix, promotion and demotion counts, conformance flags. **Never:** before and after payloads, target ids, tokens, or free text.

The manifest endpoint (GET /api/agency/manifest) advertises the layer's guarantees (scoped_token, action_audit_log, fail_closed_gate, human_in_loop, separation_of_duties, never_earnable_cap, automatic_reversal), a schema version, the standards mappings (Section 16), and the call-type registry (Section 20).

## 15. The extension seam: what makes it a library

The core is domain-agnostic. Everything app-specific is confined to adapters, chiefly the **capability-handler registry**, the one piece of domain code. Each capability registers three functions:

- **apply**: performs the change; returns the after-state and target reference.
- **rollback**: reverses it from the logged before-state.
- **checkLimits**: throws on a limits violation.

Adding a capability means seeding a matrix row (with tier, cap, and data class) and registering a handler. Nothing else in the core changes.

The remaining adapters: **identity** (reuse an existing token table if one exists), **storage** (the contract is the DDL; the driver varies), **surface** (mount the two route groups in the app's router), and **tile or UI** (the queue is a headless data contract; GET /api/governance/queue returns grouped proposals; each app renders its own tile).

**Fixed (the contract):** table and column names, state machine, mode ladder, caps on the reference never-earnable members, endpoint paths, JSON envelope, export-seam rules, fail-closed semantics, escalation guard, reversal triggers. **Free to vary (per app):** capability keys and limits, additional never-earnable members, handler implementations, posture, UI, storage driver.

## 16. Standards alignment: government and industry compliance

*Alignment by design, not certification.* Each control maps to established frameworks; the manifest emits the mappings machine-readably, the substrate for a future "compliance packs" product tier.

| Layer control | NIST AI RMF | HIPAA | SOC 2 | ISO/IEC 42001 | FDA (if SaMD) | AMA Appendix S |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Capability matrix plus roles plus escalation guard | GOVERN | 164.308(a)(3)/(4) | CC6.1 to 6.3 | A.2 policy, A.3.2 roles | none | tier ceiling per level |
| Scoped agent tokens | GOVERN / MANAGE | 164.312(a)(1)/(d) | CC6.1 | A.3.2 accountability | none | none |
| Fail-closed chokepoint | MANAGE | 164.306(a) | CC8.1 | A.9.2, Cl. 8 | none | none |
| Human-in-the-loop (propose then approve) | GOVERN-3 / MANAGE | none | CC8.1 | A.9.2 human oversight | GMLP 7 | assistive, augmentative |
| Never-earnable class (Section 5) | GOVERN-1, MANAGE-2 | 164.508 authorization; 164.502(b) | CC6.1 | A.5 impact assessment | GMLP 7; PCCP scope exclusion | never autonomous |
| Append-only audit log plus rollback | MEASURE / MANAGE | 164.312(b) audit controls | CC7.2 to 7.3 | A.6.2.8 event logs | 21 CFR 11-style traceability | none |
| Governed reads (data.read egress audit, Section 13) | MEASURE / GOVERN-6 | 164.312(b); minimum necessary | CC6.1, CC7.2 | A.6.2.8 | none | none |
| Earned autonomy (evidence-gated change) | MANAGE-4 | none | CC8.1 | A.5 impact assessment, Cl. 10 | PCCP (see note) | level change requires human act |
| Automatic reversal (Section 32) | MANAGE-4, MEASURE-4 | none | CC7.4 | Cl. 10.2 nonconformity | PCCP rollback plan | none |
| Export seam (aggregate-only) | GOVERN-6 | 164.502(b) minimum necessary | CC6.7, Confidentiality | A.7 data, A.8 interested parties | none | none |
| Cost, usage, and quality observability (Part III) | MEASURE | none | CC7.1 | Cl. 9 performance evaluation | GMLP 9 to 10 | none |

**ISO/IEC 42001.** The certifiable AI-management-system standard, the closest single framework to what this layer *is*, and the strongest certification fit. The table covers the governance-relevant Annex A controls; certification would add the organizational management-system clauses.

**FDA PCCP.** FDA's Predetermined Change Control Plan guidance (final, December 2024) pre-authorizes planned AI changes that follow a declared protocol and impact assessment. The earned-autonomy loop is structurally the same contract: pre-declared, evidence-gated, human-approved, logged, reversible. Automatic reversal supplies the half a change control plan usually leaves to procedure: the condition under which a change is undone, and the record that it was. The never-earnable class defines what is outside the plan's scope by construction. For a future Software-as-a-Medical-Device, this layer is the natural PCCP substrate. These positions are set out in Precision Health Ventures' comment to FDA docket FDA-2026-N-7874 on generative-AI-enabled devices (in preparation, October 2026).

**AMA CPT Appendix S.** The taxonomy gives the consequence tiers of Section 9 an external vocabulary. It also gives the audit log a provenance level per AI-derived entry (assistive, augmentative, autonomous), which the PSHR records on every AI-contributed fact.

**Defense and federal.** CMMC 2.0 (DFARS 252.204-7021; phased rollout November 2025 to 2028; built on NIST SP 800-171) gates deployments touching FCI or CUI. The layer's tokens, audit log, and access separation map to the 800-171 AC and AU families, so readiness is a deployment posture, not a redesign. DoD AI Ethical Principles: *Traceable* is the provenance-tagged audit log; *Governable* is the master switch plus fail-closed gate plus rollback plus automatic reversal. FedRAMP applies only to a future hosted control plane sold to federal agencies.

**Posture.** Emit the mappings now; pursue attestations (SOC 2 first, ISO/IEC 42001 as the differentiator) only when a customer warrants. Harden early: true append-only enforcement on the audit log; encryption-at-rest guidance in the porting kits. The full hardening roadmap is Section 37.

## 17. Porting the governance pillar to another app

- **New app:** create the three tables; seed capabilities with tier, cap, data class, and posture; register handlers; mount the two surfaces; carve the token-only agent path at the perimeter; keep the log local; render the tile from the queue endpoint.
- **Existing app (retrofit):** additive migrations only. **Wrap, don't rewrite:** route only agent and AI writes through the chokepoint first. Use **shadow mode** (log-only, non-blocking) until confident. Master switch inert by default. Reuse existing identity.
- **Cross-language rule:** never share a codebase across stacks. Share the contract plus a thin per-stack kit (TypeScript/Next and Python/FastAPI kits exist); keep names, paths, and the state machine identical. That is what lets a management app roll up N apps with zero per-app adapters.
- **Installer:** the port-governance skill detects the stack, creates the tables, installs the chokepoint, registry, surfaces, and a tile stub, seeds a starter capability at propose, seeds the reference never-earnable members with their caps, and leaves the master switch inert. Bullit's own install was performed through this procedure, validating the extraction path.

-----

# Part III: The Observability Pillar (the measurement contract)

## 18. The pillars

Each follows the same portable posture (Section 34): a stable table, one cached read-time aggregator, a documented JSON endpoint, an aggregate-only export seam, a master switch, bounded retention.

| Pillar | Question it answers | Store |
| :-- | :-- | :-- |
| **AI Cost** | What is each AI call, type, and model costing? | usage_events (fn, provider, model, tokens, cost_usd, ok, ref_type, ref_id) |
| **AI Volume** | How much, how often; trend and forecast? | same table, time-bucketed |
| **Usage Self-Monitoring** | How do *humans* actually use the app: pages, features, actions, channels? | human_interactions (Section 19) |
| **Closed-Loop Learning** | How good is each type; are its outputs accepted and kept? | outcome links plus decision joins |
| **App Health** (optional) | Is it up; error and refusal rate? | heartbeat plus the ok flag |

Cost is computed at write time from a per-model price table and stored; it is an estimate (unknown models cost $0; the caveat stays visible). Aggregations: spend tiles (month-to-date, forecast, today, last 30 days), a 30-day daily series, and breakdowns by model, by call type, and by call type by model (the routing matrix).

## 19. Usage self-monitoring: human interaction telemetry

The app records how *humans* use it (page views, feature clicks, actions taken) so product and UX decisions, and AI-mined improvement suggestions, are grounded in real behavior rather than intuition. This is a separate stream from AI-call telemetry, with its own table: **human_interactions** (Section 29).

The rules that make it work:

- **Multi-channel, always.** Any app with more than one interaction surface (web plus bot, web plus mobile, web plus CLI) routes **all** of them into the same stream, browser surfaces via a JS tracker and everything else via a server-side writer, with the channel namespaced into the target (telegram:send, never bare send). A channel that doesn't emit isn't "unused," it's *invisible*; suggestions built on a partial stream are wrong.
- **Central mutation capture.** Instrument the app's shared fetch wrapper so every successful non-GET emits a mutation event automatically: full write coverage with zero per-handler instrumentation. Add explicit track('feature', ...) calls only at notable interactions.
- **Labels are control identifiers, never values.** A target must be a UI control name (range:day), not a domain value; strip value suffixes before export. This is what keeps the stream exportable as aggregate counts.
- **Privacy posture:** data stays on the app's server; a user-visible toggle; bounded retention (reference: 180 days) plus a clear or delete endpoint.

Read endpoint and reference export shape: Section 31.

## 20. The call-type taxonomy (hybrid: app call site plus standard task code)

Each app's **fn** is a call site, app-local, the unit of routing and measurement. Every registered type carries the dimensions that drive the cost and quality tradeoff (shape, output as structured or freeform, stakes as reviewed-suggestion, applied-directly, or user-facing, volume, latency, and a routing **lean** of cost, quality, or fixed) plus a **standard task code and category** from the app-independent vocabulary (Section 33).

**Task and category are the cross-app comparability keys.** A management app aggregates by them (comparing "extraction workloads" across apps whose call sites differ); the fn never needs to leave the app. Each app registers only the subset its call sites use.

## 21. Quality per call type: the hard 80%

There is no single quality metric. Use behavioral proxies already collected, per type; invest in heavier evals only where a model change is high-stakes.

| Signal (cheapest first) | Applies to |
| :-- | :-- |
| Acceptance rate of governed proposals | extraction and suggestion types |
| Edit rate (did the user change the parsed fields?) | parse types |
| Correction or delete rate on emitted items | vision and capture types |
| Follow-up-correction rate, or explicit thumbs | conversational types |
| The ok flag (errors and refusals) | all |
| Golden-set score per type by model; LLM-as-judge sampling | structured types; open generation |

The plumbing: usage_events carries outcome links (ref_type, ref_id), and the governance log's decisions join back to call types (proposing actors map to types). Quality per type equals accepted divided by decided.

**Honest framing (carry it into the UI and docs):** follow-through is **correlation, not causation**, co-occurrence within a window rather than attribution. The calibration loop tunes confidence; it does not prove impact.

## 22. Model routing, governed model selection, and emergent vocabulary

**The binding of a task type to a model is a governed action.** It is stored as governed config (the config.set capability) and moves only through the propose, approve, audit, rollback path. Every AI call records the model that served it (usage_events.model), so the track record behind any promoted capability is attributable to a specific model. The framework offers two postures for who decides the binding.

**Operator-side routing (designed).** Replace coarse model config with a per-type map: each call type gets a model and an optimize-for mode (cost or quality). Policy: pick the cheapest model whose quality stays above the type's threshold. When a candidate model's evidence for a type clears the gate, the system recommends the change; an Admin approves it.

**Person-side governed model selection (designed; PSHR local client).** In a deployment where the person holds the record and runs or chooses the inference, the person is the Admin for model choice on their own data:

1. The person selects, per task type, a local model or a hosted model. The selection is recorded as a governance event.
2. Every call records which model was used, so the person can see what handled each task.
3. When a different model (an updated local model, a new hosted one) clears the evidence gate for a task type, the person is **informed**. The system never switches on its own.
4. A change of model happens only by the person's recorded act. The prior selection is kept as before-state, so the change is reversible like any other.

This posture is what makes the local-first Person-Sovereign Health Record client governable without an operator: the person's own recorded choices are the policy.

**Model change and reversal (both postures).** A promoted capability's track record belongs to the model that earned it. Changing the model serving a capability's call type is a reversal trigger (Section 32): the capability returns to propose and begins a fresh record on the new model. This is what keeps "the cheapest model whose quality holds" honest: a swap cannot inherit trust it has not earned.

**Emergent vocabulary (built).** Terms the AI coins outside the registry surface with usage counts. Promoting one adds it to a runtime registry (proper label, inverse, symmetry) and feeds it back into the extraction prompts, so the AI converges on your vocabulary. Remap instead rewrites those edges onto an existing term. The same propose-then-approve pattern, applied to the system's own taxonomy.

## 23. Where the two pillars compose

1. **Routing config is a governed capability.** Model-map changes flow through the governance pillar's propose, approve, audit, rollback path (the config.set capability), and a model change demotes the capabilities it serves.
2. **Cost times quality drives earned autonomy.** The Learning pillar's track record is the evidence behind every promotion and every model recommendation. The system proposes; a human approves.
3. **Quality decline drives reversal.** The same acceptance signal that earned auto withdraws it when it falls below the gate.

-----

# Part IV: The Standalone Offering

## 24. Why this is a product (and for whom)

- **The problem is universal.** Every team wiring LLM or agent capabilities into an app faces the same questions: what is the AI allowed to see and change, who approved it, what can it never do, what is it costing us, and how good is it? Most answer with ad-hoc flags and log lines. This layer answers with a reusable contract.
- **Differentiators.** Fail-closed by construction. Human-in-the-loop as code, not convention. *Earned*, evidence-gated, auditable, and *revocable* autonomy rather than configured autonomy. A never-earnable class that gives governance committees a fixed answer. A PHI-safe export seam. A compliance-mapped, machine-readable conformance manifest. The FDA PCCP structural parallel is a genuine moat for healthcare buyers.
- **Buyers.** AI-feature teams in regulated or risk-conscious domains (healthcare, government and defense-adjacent, finance); agencies shipping many AI apps (the multi-app roll-up is the hosted product's core value); solo builders via the free core.

## 25. Target repo shape

One monorepo (github.com/ahendo026/earned-autonomy) with publishable packages:

- **packages/governance**: core: the three tables, chokepoint, surfaces, manifest.
- **packages/observability**: core: usage_events plus human_interactions, taxonomy, aggregators, pillars.
- **kits/typescript** and **kits/python**: per-stack kits. *Already exist as the installer skill's references.*
- **skills/port-governance**: the installer skills. *Already exists.*
- **management-app**: the hosted control plane (the paid layer).
- **docs**: this document plus each consumer app's port note.

## 26. Open-core split: the export seam IS the product boundary

- **Free (MIT):** the framework contract, per-stack kits, installer skills. Apps keep all raw data local. Drives adoption.
- **Paid (hosted):** the management and aggregator app consuming each app's aggregate-only exports and conformance manifests: cross-app spend and quality dashboards, governance-posture overview, promotion and reversal history, alerting, org policy, model recommendations. Pricing axes: per-app, per-seat, or per-monitored-AI-spend.
- **Premium add-ons:** the quality-eval harness (golden sets, LLM-as-judge); compliance packs (HIPAA, SOC 2, ISO 42001, CMMC reporting generated from the manifest); advanced earned-autonomy policies.

## 27. Extraction path (ordered)

1. **Harden the seams in place:** dependency-light cores, versioned envelopes, append-only enforcement, the cap column and reversal triggers.
2. **Second port is the proof:** install the pillars into a second app via the installer skills; divergences become kit fixes.
3. **Extract the packages** into the standalone repo; existing apps become consumers and the TypeScript and Python demo references.
4. **Build the management app** against the manifests and export endpoints of the live apps; that app is the monetized product.
5. **Certify when a customer warrants it:** SOC 2 first; ISO/IEC 42001 as the AI-management differentiator. The controls already map.

## 28. Invariants that must survive productization

- Fail-closed chokepoint; escalation guard: admin powers never on agent tokens.
- Human-in-the-loop for every promotion and policy expansion; the system proposes, never self-edits its policy upward.
- Automatic reversal: the system may only move its own autonomy downward, and must.
- The never-earnable class: reference members cannot be uncapped by configuration.
- Consent and autonomy on separate axes; neither moves the other.
- Append-only, attributable, reversible audit; raw data stays local to each app.
- Aggregate-only export seam: the privacy guarantee and the product boundary are the same line.
- Contract stability: identical table, endpoint, and envelope names across every app.

-----

# Part V: The Exact Contract (implementation appendix)

*The byte-precise part. An implementation that varies anything in this part does not interoperate and cannot roll up. Everything else in this document can be paraphrased; this part cannot. Where a live implementation currently deviates, the deviation is listed in Part VI; the contract does not bend to match it.*

## 29. Table contracts (DDL; keep these column names)

**agent_tokens**: id, name, token_hash (sha256), app_id, scopes_json, active, created_at, last_used_at. *(Reuse an existing token table via the identity adapter if one exists; Bullit reuses hub_tokens.)*

**agent_capabilities**: key, description, mode ('off'|'propose'|'auto'), cap ('auto'|'propose'), tier (0|1|2|3), data_class, limits_json, enabled, updated_at. *Revision 3 adds cap, tier, and data_class as additive columns (no schema_version bump). cap defaults to 'auto'; the reference never-earnable members are seeded with cap = 'propose' and tier = 3, and the human surface rejects any update that changes their cap. A capability row is keyed by (key, data_class); a capability with no data-class distinction uses data_class = '\*'.*

**agent_action_log** *(append-only for mutations; data.read egress rows prunable per Section 13)*: ts, actor, capability, target_table, target_id, before_json, after_json, mode, state ('applied'|'proposed'|'approved'|'rejected'|'failed'|'promoted'|'demoted'), detail. *Revision 3 adds the promoted and demoted states (additive). A demoted row carries the trigger in detail: acceptance_below_gate, rollback, model_change, scope_change, or dormancy.*

**usage_events** *(one row per AI call)*: created_at, fn, provider, model, input_tokens, output_tokens, cost_usd, ok, ref_type, ref_id

**human_interactions** *(one row per human interaction; the Section 19 pillar)*: ts, session_id, event_type ('page_view'|'feature'|'chart'|'chat'|'mutation'), target, page, meta_json. *(Channel is namespaced into target, e.g. telegram:command:/todo. Bounded retention; reference 180 days.)*

## 30. Chokepoint and writer signatures

```
perform_agent_action(actor, capability, target, payload)
  -> rejects (fail-closed) | logs 'proposed' | applies + logs 'applied'

register(capability_key, {
  apply(payload)               -> performs the change, returns after_json + target ref
  rollback(before_json)        -> reverses it (uses the logged before-state)
  checkLimits(limits, payload) -> boolean/throws
})

promote(capability_key, data_class)      -> Admin only; rejected if cap == 'propose'; logs 'promoted'
demote(capability_key, data_class, trigger) -> system or Admin; propose only, never off; logs 'demoted'

recordUsage({ fn, provider, model, inputTokens, outputTokens, ok, refType, refId })
  -> one row in usage_events; cost_usd computed from PRICES at write time

recordInteraction({ eventType, target, page, sessionId, meta })
  -> one row in human_interactions; silent-on-error (telemetry never breaks a
     real request); browser surfaces batch via a JS tracker, other channels
     call the server-side writer directly
```

PRICES is a per-model table (model to $ per 1M input tokens, $ per 1M output tokens). Ships with defaults; the app overrides. Unknown model costs $0, caveat kept visible.

## 31. Endpoint, envelope, and reference response shapes

| Surface | Path | Auth | Purpose |
| :-- | :-- | :-- | :-- |
| Agent | GET /api/agency/manifest | agent token | Guarantees plus standards mappings plus call-type registry plus schema_version |
| Agent | POST /api/agency/actions (plus reads of own log) | agent token | Propose and execute granted capabilities |
| Agent | GET /api/agency/\<resource\> | agent token | Governed read surface (data.read, Section 13) |
| Human | GET /api/governance/queue | perimeter auth | Grouped pending proposals (headless tile contract) |
| Human | POST /api/governance/review (approve, reject, rollback) | perimeter auth | Decisions |
| Human | POST /api/governance/promote and /demote | perimeter auth | Mode changes (promote is Admin-only) |
| Human | capabilities plus relationship-types management under /api/governance/\* | perimeter auth | Admin config |
| Observability | GET .../ai-usage | perimeter auth | AI cost plus volume |
| Observability | GET .../usage/analytics | perimeter auth | Human interaction analytics (Section 19) |
| Observability | GET .../follow-through | perimeter auth | Closed-loop learning |
| Observability | GET .../health | perimeter auth | App health |

Every read endpoint returns the envelope { app_id, schema_version, pillar, captured_at, ... }. Shared range vocabulary: 24h, 7d, 30d, 90d, mtd, ytd, all (support a subset, never redefine). Optional discovery: GET /api/platform/manifest returns { app_id, pillars, schema_versions, ranges }.

**Reference aggregate shapes.** Apps may add fields freely; these keys must be present with these names and meanings (this is what the management app relies on):

```
// cost: GET .../ai-usage
{ "app_id","schema_version","pillar":"cost","range",
  "total_cost_usd","total_calls",
  "by_model":   [{ "model","calls","cost_usd" }],
  "by_endpoint":[{ "endpoint","calls","cost_usd" }] }   // endpoint = fn/call-site label

// usage: GET .../usage/analytics
{ "app_id","schema_version","pillar":"usage","range",
  "total_events","active_days","sessions",
  "by_page":      [{ "page","count" }],
  "top_features": [{ "target","count" }] }

// learning: GET .../follow-through  (AGGREGATE-ONLY for export)
{ "app_id","schema_version","pillar":"learning","days",
  "total","followed","follow_rate",
  "by_kind": { "<kind>": { "n","followed","rate" } } }
// NO per-entity array in the exported payload: the per-entity dimension
// (by_ticker / by_document / by_patient) is the classic leak vector; it is
// computed and used LOCALLY only, and the export strips it.

// governance: AGGREGATE-ONLY for export (raw agent_action_log is LOCAL-ONLY)
{ "app_id","schema_version","pillar":"governance","days",
  "by_capability": [{ "capability","data_class","tier","cap","mode",
                      "actions","approved","rejected","failed",
                      "promotions","demotions" }],
  "approve_rate","auto_share",
  "conformance": { "scoped_token","action_audit_log","fail_closed_gate",
                   "human_in_loop","separation_of_duties",
                   "never_earnable_cap","automatic_reversal" } }

// health: GET .../health
{ "app_id","schema_version","pillar":"health",
  "version","uptime_s","last_tick_at","tick_lag_ok","db_size_bytes",
  "errors_24h": [{ "type","code","count" }] }
// Error COUNTS by type/code only, never error messages or stack traces
// (those are a classic PHI/financial leak).
```

**Export seam rule (all pillars):** only aggregate, anonymized metrics cross to an aggregator: counts, rates, cost by type and model, forecast, promotion and demotion counts. Never raw rows, prompts or outputs, before and after payloads, target ids, tokens, free text, session ids, or per-entity dimensions.

## 32. The mode ladder and state machine

**Modes:** off (rejected), propose (queued for a human), auto (applied immediately plus logged). Each capability row carries a **cap**, the highest mode it may reach; the reference never-earnable members (Section 5) are capped at propose and their cap cannot be raised.

**Promotion (propose to auto).** Human-only, Admin-only, rejected in code when cap == 'propose'. The system *recommends* it from the acceptance track record (reference gate: at least 10 decisions, at least 90% acceptance, on the current model and scope). Logged as state 'promoted'.

**Reversal (auto to propose).** System-initiated on any of these triggers, or Admin-initiated at will; logged as state 'demoted' with the trigger:

| Trigger | Definition (reference values) |
| :-- | :-- |
| acceptance_below_gate | Rolling acceptance over the last 20 decisions on this capability and data class falls below 90% |
| rollback | Any auto-applied action of this capability is rolled back |
| model_change | The model bound to the capability's call type changes (Section 22) |
| scope_change | The capability's limits_json, data_class, or the actor's read allow-list for that class changes |
| dormancy | No decisions on this capability and data class for 90 days |

Reversal never moves a capability to off; off is an Admin act. After a reversal the track record restarts at the demotion timestamp; earlier decisions remain in the log but do not count toward the next gate.

**States:** proposed to approved (applied) | proposed to rejected | applied (auto) | failed (chokepoint rejection) | promoted | demoted. Append-only: state transitions happen only through the review, promote, and demote paths; no UPDATE outside the state machine.

**Read capabilities** (Section 13) collapse the ladder: auto equals on, anything else returns 403; propose has no meaning for a read. Reads have no cap and no track record; consent alone governs them.

## 33. The standard task vocabulary (cross-app comparability keys)

| Category | Task codes |
| :-- | :-- |
| **Knowledge** | qa (qa_open, qa_closed, qa_rag) · reading (reading_extract, reading_abstract) · extract (extract_ner, extract_kv, extract_table, extract_relation) |
| **Reasoning** | reason (reason_math, reason_logic, reason_common, reason_causal, reason_analogical, reason_multihop) |
| **Creation** | generate (generate_email, generate_report, generate_story, generate_marketing, generate_technical) · code (code_generate, code_complete, code_debug, code_refactor, code_explain, code_test, code_review) · translate (translate_literal, translate_natural) · summarize (summarize_extract, summarize_abstract, summarize_multi) · rewrite (rewrite_grammar, rewrite_tone, rewrite_simple, rewrite_expand, rewrite_condense) |
| **Analysis** | classify (classify_binary, classify_multi, classify_labels) · rank (rank_retrieval, rank_preference) · decision (decision_compare, decision_recommend, decision_risk, decision_tradeoff) · plan (plan_project, plan_travel, plan_study, plan_strategy) |
| **Interaction** | dialogue (dialogue_assistant, dialogue_coach, dialogue_tutor, dialogue_brainstorm) · agent (agent_search, agent_db, agent_api, agent_files, agent_workflow) |
| **Infrastructure** | retrieval_embed (embedding and retrieval plumbing) |

Per-type registered dimensions: task, category, shape (extract | classify | generate | converse | vision | parse | embed), output (structured | freeform), stakes (reviewed-suggestion | applied-directly | user-facing), volume (hi | lo), latency (interactive | background), lean (cost | quality | fixed).

## 34. Portable posture (every pillar, every port)

- Master switch, **inert by default**.
- Additive migrations only (CREATE TABLE IF NOT EXISTS, additive ALTERs).
- **Silent-on-error writers:** telemetry and audit writers swallow their own failures; observability must never break a real request.
- Mutation tables append-only and never pruned; high-volume read and telemetry tables pruned after N days (bounded retention; reference: data.read egress 30 days, human_interactions 180 days).
- Lazy read-time aggregation behind a 30 to 60 second cache; no write-path overhead.
- A stable JSON read contract with the Section 31 envelope.
- Shadow mode for retrofits: log-only (record proposed without blocking) until enforcement is switched on.
- Reference never-earnable members seeded capped at install; reversal triggers active whenever any capability is at auto.

## 35. Versioning and compatibility rules

- schema_version per pillar; bump **only** on a breaking change (a removed, renamed, or repurposed contract field).
- **Additive** changes (new fields, new states, new columns with defaults) do **not** bump the version; consumers ignore unknown fields. Revision 3's additions (cap, tier, data_class, promoted, demoted, the two new conformance flags) are additive.
- **Never reuse a field name with a different meaning** across apps; that is what breaks aggregation silently.
- The management app depends on these names across many tenants; contract stability is the last invariant in Section 28.

## 36. Fixed versus free-to-vary (the one-table summary)

| Fixed (the contract) | Free to vary (per app) |
| :-- | :-- |
| Table plus column names, state machine, mode ladder, caps on reference never-earnable members, reversal triggers | Capability *keys*, tiers for app-specific capabilities, limits, additional never-earnable members |
| Endpoint paths, JSON envelope plus reference shapes, export-seam rules | Handler implementations (apply, rollback) |
| Fail-closed chokepoint semantics, escalation guard, consent check | Posture (which caps start where), UI, storage driver |
| recordUsage and recordInteraction signatures, task vocabulary codes, range vocabulary | PRICES values, the fn registry, outcome predicates, routing policy, reversal reference values, tracked pages and features |

## 37. Hardening roadmap (design-stage; non-optional once PHI is behind the seam)

Ordered by how much they matter the moment a healthcare app feeds the seam:

1. **Code-enforced export gate.** The Section 31 export seam as a *validator*, not a convention: allowlist keys against the reference shapes, regex-scan values for disallowed patterns (dollar amounts, SSN or MRN-shaped strings, long free text), refuse to emit on any violation (fail-closed). Ship it in the shared module so every app inherits it; back it with a conformance test.
2. **A scoped, read-only seam token.** The aggregator pulls with a dedicated per-app observability token: read-only, rotatable, valid only on the export endpoints, never the human perimeter credential.
3. **An access-audit log of every export.** Append-only, local: timestamp, token id, pillar(s), schema_version, aggregate counts returned.
4. **De-identification robustness.** Coarse time buckets (export counts per day, never per-event timestamps) and small-count suppression (drop or bucket n below k), the line between "aggregate" and "still identifiable."
5. **Error counts, never error messages.** Messages and stack traces leak PHI and financial data; scrub at the source (Section 31 health shape).
6. **Cost anomaly alerts plus cost-by-feature attribution.** Cross-app budget caps and spike alerts; *expensive plus low-follow-through* features are the clearest prune candidates. All anonymous, so it crosses the seam cleanly.
7. **Standardized retention plus right-to-delete** across pillars, with a purge mechanism; exported aggregates shouldn't outlive their basis.
8. **Cap enforcement test.** A conformance test that attempts to promote each reference never-earnable member through every surface and asserts rejection; a build that fails this test is not conformant.
9. **Human-in-the-loop in high-stakes domains.** Carry the calibration loop's confidence-tuning into a clinical app, but **never its auto-action** for tier 2 or 3: the loop tunes confidence; a human acts (Section 9 posture ladder).

-----

# Part VI: Conformance and Divergence Register

*The framework applies its own discipline to itself: every known gap between this contract and a live implementation is named here, with a direction. The rules: divergences are remediated* ***toward the contract***; *the contract changes only via an explicit schema_version decision or an additive revision like this one, never by silently bending to an implementation. Each app's port note owns the detail and the remediation plan; this register is the roll-up. The machine-readable counterpart is each app's conformance manifest (Section 14).*

## 38. The register (as of 2026-09-30)

| # | App | Divergence from contract | Contract target | Direction |
| :-- | :-- | :-- | :-- | :-- |
| 1 | arh-tracker | AI-cost table named ai_usage | usage_events (Section 29) | Rename at arh-tracker's next migration window, coordinated with #2 |
| 2 | arh-tracker | Human interactions stored in a table named usage_events | human_interactions (Section 29) | Rename together with #1 (a coordinated swap; do not run half-migrated) |
| 3 | arh-tracker | No call-type taxonomy; cost is labeled by free-text endpoint only | fn registry plus standard task codes (Sections 20, 33) | Register arh-tracker's call sites; map each to task and category |
| 4 | Bullit | Usage self-monitoring pillar not implemented (no human_interactions, no JS tracker) | Sections 19, 29 | Implement when product questions warrant it; until then Bullit reports the pillar as absent in its manifest |
| 5 | Bullit | No governed read surface; the in-process agent reads via its own tools, not /api/agency/\<resource\> | data.read (Section 13) | Adopt when any *external* agent needs live-data access; in-process reads are inside the perimeter and out of scope |
| 6 | Bullit | Cost read endpoint is GET /api/usage, without the {app_id, schema_version, pillar} envelope (the manifest does carry them) | GET .../ai-usage plus Section 31 envelope | Add the envelope plus a reference-shape view (or ?export=1) before the management app consumes it |
| 7 | Bullit | Emergent-vocabulary promotion exists only for relationship types | Section 22 (generic pattern) | Acceptable specialization; generalize in the extracted package, not in Bullit |
| 8 | arh-tracker | No emergent-vocabulary promotion | Section 22 | Optional; adopt if and when arh-tracker grows an open vocabulary |
| 9 | Both | Hardening roadmap items (Section 37) are design-only | Section 37 | Build export gate plus seam token first, at extraction step 1 (Section 27) |
| 10 | Both | GET /api/platform/manifest (cross-pillar discovery) not implemented | Section 31 | Add when the management app exists; per-pillar manifests suffice today |
| 11 | Both | No cap, tier, or data_class columns; no promoted or demoted states; no reversal triggers | Sections 29, 32 | Additive migration in both apps at extraction step 1; seed reference never-earnable members capped |
| 12 | IntelaCare | Governance posture (titration proposals, clinician approval, prescribing and export never autonomous) is operating policy; chokepoint, matrix, and audit log conformance to Sections 29 to 32 to be confirmed in its port note | Part V | Write INTELACARE-PORT.md; report built versus designed per row of Section 8 |
| 13 | SHIRLe Health | Record-side controls (confidence-gated admission, models operate on derived indexes only, rebuild-and-diff test, consent-as-routing) are specified in PSHR Architecture v0.20; the governance tables and surfaces are not yet instantiated | Sections 6, 13, 29 | Instantiate at the PSHR reference implementation; the person-side model selection of Section 22 lands here first |

**How to use this register:** before the second port (Section 27 step 2), items 1 to 3, 6, and 11 should be resolved or consciously deferred; they are the ones that break cross-app roll-up. New divergences discovered during any port get a row here and a section in that app's port note; a row is removed only when the divergence is gone or the contract formally absorbs it with a version bump.

-----

## Revision history

- **Revision 3, September 30, 2026.** Base: revision 2. Automatic reversal; never-earnable class with cap column; consent and autonomy as two axes with per-data-class scoping; consequence tiers mapped to AMA CPT Appendix S; governed model selection with a person-side posture; IntelaCare worked example; register rows 11 to 13; standards table gains the Appendix S column and the reversal and never-earnable rows; patent-pending notice; source of truth moved to github.com/ahendo026/earned-autonomy; em-dashes removed; terms defined at first use. All contract changes are additive (no schema_version bump).
- **Revision 2, July 29, 2026 (Bullit repository commit 8c10291).** Restored the full arh-tracker capability set (usage self-monitoring, governed reads, reference export shapes, hardening roadmap); added Part VI, the conformance and divergence register.
- **Revision 1.** Initial extraction from arh-tracker's AGENT_AUTONOMY.md and Bullit's port note.
