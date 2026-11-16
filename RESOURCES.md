# What You Can Use

## Allowed

- **AI assistants:** any you like. Declare your usage in your submission, and be ready to defend every number yourself.
- **Public documentation:** vendor docs, cloud provider docs, open-source projects, books, blogs and papers.
- **Real products and vendors:** name them in your design. Justify each one, including what you'd do if it disappeared.
- **Real pricing:** you may use real public prices **as a sensitivity check**. Your main cost model must use the illustrative rates in the brief (Section 7.2), so submissions stay comparable.
- **Any diagramming tool:** a photo of a hand-drawn diagram is fine if it's readable.
- **Spreadsheets or code** for your cost and capacity models. Include them, or link to them.

## Not allowed

- **Help from another person** on the content. Proofreading for spelling and grammar is fine.
- **Copying another candidate's submission.**
- **Contacting real companies or vendors** as if Meridian Mutual were real.
- **Changing the brief's facts** to make the problem easier. You may *challenge* a fact with a stated assumption, but not ignore it.

## Diagram tools

- [Mermaid](https://mermaid.js.org): diagrams as code, renders natively on GitHub
- [draw.io / diagrams.net](https://app.diagrams.net): free, works in the browser
- [Excalidraw](https://excalidraw.com): hand-drawn style, fast
- [Structurizr](https://structurizr.com): C4 models as code

## Useful frameworks

These are starting points, not requirements. Use whatever method you can defend.

| Area | Starting point |
|---|---|
| Architecture diagrams | [C4 model](https://c4model.com) |
| Decision records | [Architecture Decision Records](https://adr.github.io) |
| SLOs and error budgets | [Google SRE books](https://sre.google/books/) (free online) |
| LLM security | [OWASP Top 10 for LLM Applications](https://genai.owasp.org) |
| AI threat modelling | [MITRE ATLAS](https://atlas.mitre.org) |
| AI risk management | [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) |
| Observability | [OpenTelemetry](https://opentelemetry.io), including its GenAI semantic conventions |
| FinOps | [FinOps Foundation](https://www.finops.org) |
| Cloud architecture | Any cloud provider's Well-Architected framework |

## Minimal templates

Use them, adapt them or ignore them.

**ADR**
ADR-NNN: <decision>
Status: Proposed | Accepted | Superseded
Context: <forces at play, constraints from the brief>
Options considered: <A, B, C with trade-offs>
Decision: <what and why>
Consequences: <what gets easier, what gets harder, reversal cost>
**SLO**
Service / agent class:
SLI: <what is measured, where, how>
Objective: <target over window>
Error budget: <amount>
Budget policy: <what happens when it's spent, who decides>
**Risk register row**
ID | Risk | Likelihood (1-5) | Impact (1-5) | Owner (named stakeholder) | Mitigation | Residual risk | Trigger to revisit
## A note on regulation

The brief summarises real regimes: UK/EU GDPR, DORA, PRA SS2/21, the FCA Consumer Duty, the EU AI Act and German works council rights. You **don't** need to read the legislation.
- Work from the brief's summaries.
- Where you rely on an interpretation, state it as an assumption.
- You're being assessed on architectural judgement, not legal knowledge.

---

*I wanted it. So I forged it. Now forge yours.*
