# Glossary

This is the official vocabulary of MAYA. Each concept has exactly one definition, written to be implementation-friendly rather than a dictionary entry. Other documents in this repository use these terms consistently rather than redefining them.

| Term | Definition |
|---|---|
| Lead | A person who has made initial contact or been referred into a conversation but whose needs, budget, and intent have not yet been understood or qualified. |
| Prospect | A lead whose intent and general situation have been partially understood, but who has not yet been fully qualified or matched to a recommendation. |
| Customer | A prospect actively engaged in an ongoing consultative relationship with Propify, regardless of whether a purchase has occurred. |
| Buyer | A customer whose intent is confirmed to be purchasing a property, as distinct from one still exploring intent. |
| Investor | A customer whose confirmed intent is financial return rather than personal use, requiring recommendations weighted toward investment performance. |
| Consultant | Any party, human or AI, responsible for guiding a customer through understanding, qualification, and recommendation toward a sound real estate decision. |
| Human Consultant | A human representative of Propify with the authority to finalize reservations, deposits, unit selections, and negotiated terms. |
| Conversation | The full exchange of messages between MAYA and a single customer across time, treated as one continuous relationship rather than disconnected sessions. |
| Conversation Stage | The sequential phase of the core sales sequence a conversation is in: understand, qualify, recommend, build trust, handle objections, create desire, or close. |
| Conversation State | The complete current snapshot of a conversation, combining its stage with everything understood about the customer so far. |
| Intent | The underlying goal behind a customer's inquiry, such as a primary residence, an investment, a vacation property, or general information. |
| Context | The full set of confirmed information relevant to a given moment in a conversation, including the customer's stated needs, prior exchanges, and applicable knowledge base content. |
| Memory | The retained record of information a customer has previously shared, carried forward so it does not need to be repeated. |
| Working Memory | The information relevant to the conversation currently in progress. |
| Long-Term Memory | Information about a customer retained across separate conversations over time, such as previously stated budget, preferences, or goals. |
| Knowledge Base | The confirmed body of factual information MAYA draws from — project facts, pricing, availability, and company policy — treated as the sole source of truth for factual claims. |
| Project Brain | The complete set of confirmed knowledge specific to a single real estate project, scoped so a conversation about that project draws only from information relevant to it. |
| Business Rule | An explicit, deliberately authorized constraint on MAYA's behavior that takes precedence over what would otherwise be a plausible generated response. |
| Recommendation | A specific project or unit suggested to a customer, grounded in their understood needs and accompanied by an explanation of why it fits. |
| Recommendation Engine | The capability responsible for selecting which project or unit to recommend to a given customer, based on their qualified context. |
| Qualification | The process of gradually establishing a customer's relevant details — budget, preferred area, unit type, timeline, and similar — through natural conversation. |
| Discovery | The earliest phase of qualification, focused on understanding the customer's goal and situation before any specific detail is requested. |
| Objection | A concern raised by a customer, often expressed as resistance to price, location, terms, or timing, reflecting an underlying unresolved question rather than a final refusal. |
| Follow-up | A subsequent outreach or continuation of a conversation after a pause, intended to maintain the relationship and move it forward without pressure. |
| Trust | The customer's confidence that the information and guidance received from MAYA is honest, accurate, and in their genuine interest. |
| Trust Signal | An observable element of a conversation — such as an honest acknowledgment of a limitation — that indicates trust is being built. |
| Voice Signal | An observable element of how a customer communicates — tone, urgency, hesitation, directness — that provides context beyond the literal content of their message. |
| Buying Signal | An observable indication that a customer is approaching a decision, such as requesting specific documents, asking about reservation steps, or narrowing to one option. |
| Confidence Score | A measure of how certain a piece of information or a customer-recommendation match is, used to determine whether it is safe to state as fact. |
| Risk | The potential for a conversation, recommendation, or claim to damage trust, accuracy, or the customer's outcome if handled incorrectly. |
| Priority | The relative importance assigned to competing objectives or actions within a single conversation, resolved according to MAYA's defined decision order. |
| Escalation | The act of raising a conversation or decision to a human consultant because it has reached the limit of MAYA's authority or scope. |
| Human Handoff | The deliberate transfer of a conversation from MAYA to a human consultant at the point a decision requires human authority or judgment. |
| Conversation Planner | The capability responsible for determining the next appropriate step in a conversation, given its current state and the customer's situation. |
| Decision | A conclusion reached about what to say, recommend, or do next, based on available context and applicable rules. |
| Reasoning | The explicit chain of justification connecting a customer's confirmed context to a given recommendation or response. |
| Constraint | A hard limit on the customer's situation or on MAYA's behavior that must be respected — such as a fixed budget ceiling or a business rule. |
| Goal | The outcome a customer is ultimately trying to achieve through the real estate decision, such as securing income, a home, or capital growth. |
| Requirement | A specific, stated condition a solution must meet to be acceptable to the customer. |
| Need | An underlying necessity behind a customer's request, which may be broader or different from what they explicitly asked for. |
| Pain Point | A specific difficulty or frustration in the customer's current situation that a recommendation should directly address. |
| Preference | A stated or inferred inclination that is not strictly required but should be weighted when selecting a recommendation. |
| Budget | The confirmed financial range a customer is able and willing to commit to a property. |
| Timeline | The confirmed period within which a customer expects to decide, purchase, or take delivery. |
| Project | A defined real estate development, with its own location, developer, unit types, pricing, and delivery details. |
| Unit | A specific, individually identifiable property within a project, such as an apartment or villa. |
| Developer | The company responsible for building and delivering a given project. |
| Market | The full set of projects, developers, and real estate options Propify covers, as distinct from any single project or developer. |
| LLM | Large Language Model — the underlying technology through which MAYA understands and generates natural language, treated as a reasoning and communication capability rather than a source of factual truth. |
| Inference | The process by which an LLM produces a response based on the context and instructions it is given. |
| Prompt | An instruction set given to an LLM to shape its behavior for a given task; an implementation detail outside the scope of this Foundation. |
| Specification | A precise statement of what must be true or must happen, used as the basis for building or evaluating a system, as distinct from a description written for general understanding. |
| Principle | A timeless, implementation-independent statement governing MAYA's behavior, defined in `02_Principles.md`. |
| Policy | A specific, authorized rule governing a defined situation, implemented as a business rule. |
