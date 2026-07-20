# Changelog — Phoenix Runtime, Level 1 Architecture

## v1.0 — Frozen (this release)

The architecture that resulted from a full five-journey validation pass against the pre-validation Level 1 design. See `RELEASE_NOTES.md` for the complete, journey-by-journey account of what changed and why. In summary, validation added:
- A Turn Identity Contract, giving every runtime artifact a shared, traceable identity from the moment it is created.
- A dependency-aware Confidence aggregation rule for the Reasoning Engine.
- A Persistence Tail Principle guaranteeing every completed turn — normal, escalated, or blocked — is persisted identically.
- A documented, deliberate scope limitation on the Output Business Rules Gate, excluding pre-approved deterministic content.
- A Recommendation History artifact, closing the loop the Preconditions component was originally introduced to serve.

No Engine's core responsibility, boundary, or invariant from the pre-validation design was reversed. Every change was additive or clarifying.

This version is declared **architecturally frozen**. Further change requires evidence from implementation or production, not further hypothetical review.

---

## Pre-Validation State (superseded, not included in this release)

The architecture reached this frozen state through several rounds of design and review prior to formal validation:

1. **Foundation established.** The four-file Foundation (`README.md`, `01_Identity.md`, `02_Principles.md`, `03_Glossary.md`) was written first, defining MAYA's identity, timeless principles, and vocabulary independent of any implementation.

2. **Behavioral and cognitive layers added.** `Conversation.md`, `Psychology.md`, `Business_Rules.md`, `Consultation_Methodology.md`, `Consultation_Strategy.md`, and `Consultation_Principles.md` established how MAYA is expected to behave; `Architecture.md`, `Capabilities.md`, `Scope.md`, and `Decision_Framework.md` established how she reasons, what she can do, and where her authority ends.

3. **Mind models and knowledge system built out.** `Mental_Model.md`, `Knowledge_Model.md`, `Customer_Model.md`, `Conversation_State_Model.md`, and `Memory_Model.md` defined how understanding is held and updated; `Knowledge_Sources.md`, `Knowledge_Quality.md`, `Knowledge_Governance.md`, `Knowledge_Evolution.md`, and `Truth_Model.md` defined how knowledge itself is regarded, trusted, and allowed to change. `Project_Brain.md`, `Project_Knowledge.md`, `Project_Lifecycle.md`, `Customer_Knowledge.md`, and `Recommendation_Model.md` extended this into project- and recommendation-specific philosophy.

4. **Runtime Architecture drafted through three revisions (v1, v2, v3 — v1 and v2 superseded and not included in this release).**
   - v1 sketched the initial 18-stage Main Consultation Pipeline.
   - v2 corrected several structural issues identified on review: Context assembly was split into per-Engine production plus one late assembly step; a Contact Resolution stage was separated from role classification; the Project Engine was repositioned before the Customer Engine; a Decision Engine was split out from Planning; a Confidence Contract was formalized.
   - v3 (`Runtime_Architecture.md` in this release) resolved a final open question — keeping the Recommendation Engine independent rather than folding it into Planning — and added the Runtime Entry Events section, documenting that the Main Consultation Pipeline is one entry point among several anticipated future ones.

5. **The Engine Contract established.** `Engine_Contract.md` was written and approved as the single template — 26 fields, refined once to add Engine Invariants as a 26th field — against which every Engine would be documented.

6. **Runtime artifacts defined before their producing Engines.** `Reasoning_Conclusion.md`, `Recommendation.md`, and `Plan.md` were each defined and approved before the Engine that produces them was specified, ensuring every Engine was written against a settled output shape rather than an assumed one.

7. **All 22 Engines specified against the Engine Contract**, starting from the center of the Runtime (Decision Engine) outward, in batches once the pattern proved stable: the core consultation Engines (Reasoning, Decision, Planning, Recommendation), the execution-tail Engines (Reflection, Output Business Rules Gate, Response Composer, Memory & State Writer, WhatsApp Reply), and the context-and-infrastructure Engines (Communication Gateway, Normalization, Input Business Rules Gate, Contact Resolution, Memory Engine, Conversation Context Engine, Business Context Engine, Conversation State Recognition, Intent Understanding, Project Engine, Customer Engine, Context Builder), plus the two conditional branches (Handoff Composer, Safe Fallback Composer).

8. **Five-journey validation conducted**, producing this v1.0 release.
