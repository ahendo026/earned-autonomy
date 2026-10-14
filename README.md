# Earned Autonomy

A portable governance and observability framework for AI-using applications, published by Affinity Networks Inc.

The organizing principle: autonomy is earned, not granted. Every action an AI system can take is listed in advance, and each listed action has one of three settings: off (the system cannot do it), propose (the system drafts the action and a human must approve before anything happens), or auto (the system performs the action and records it). Everything starts at propose. The system may recommend moving an action to auto only after a track record of human approvals clears a stated evidence gate, and only a human can grant it. Autonomy is also revoked: an action at auto drops back to propose automatically when its quality falls below a stated threshold, on a rollback, on a change of the underlying model, or on a change of scope. Some actions are never earnable at all.

The complete framework is in [EARNED-AUTONOMY.md](EARNED-AUTONOMY.md) (revision 3, September 30, 2026). It covers the governance contract (capability register, fail-closed chokepoint, decision ledger, mode ladder with automatic reversal), the observability contract (cost, usage, and quality per call type), consent and autonomy as two separate axes, consequence tiers mapped to the American Medical Association's CPT Appendix S taxonomy, governed model selection, and mappings to the NIST AI Risk Management Framework, HIPAA, SOC 2, ISO/IEC 42001, FDA Predetermined Change Control Plan guidance, DoD Responsible AI principles, and NIST SP 800-171.

Reference implementations exist in Python/FastAPI and TypeScript/Next.js, with the Person-Sovereign Health Record as the healthcare application.

**Patent pending.** Portions of this framework are the subject of a pending United States provisional patent application.

**License.** The framework contract is released under the MIT License. See [LICENSE](LICENSE).

Affinity Networks Inc., Arlington, Virginia. www.TheANiGroup.com
