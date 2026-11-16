# Meridian Mutual: AI Agent Platform Architecture Challenge

A realistic architecture challenge for engineers working towards **Senior, Staff, Solutions Architect or AI Systems Architect** level.

You've just joined a fictional UK/EU insurer as Principal Architect. It has about 2,300 AI agents, four model providers, a hard budget cap, overlapping regulators and stakeholders who disagree with each other. Your job is to design the platform that gets it all under control, then defend it.

There's no single correct answer. This tests judgement, trade-offs and how you handle conflicting requirements, not memory of tool names.

## How to take it

- **Timebox:** aim for up to **8 hours of focused effort**, spread over a week if you like.
- **You won't finish everything to full depth.** Choosing what to go deep on is part of the challenge.
- **Missing information:** state an assumption and keep going. Unstated assumptions count as gaps.
- **Contradictions:** the brief contains some deliberately, as real engagements do. Resolving or escalating them is part of the work.
- **Tools:** AI tools are fine, but you should be able to defend every number and decision yourself.
- **Practising alone?** Write your deliverables, then argue your design against the hardest stakeholder in Section 4.

## Disclaimer

Meridian Mutual, its people, providers, prices, contracts and incidents are entirely fictional. The regulatory regimes referenced are real, but the summaries are simplified for the exercise and aren't legal advice.

---

# Part A: Candidate Brief

## 1. The engagement

**From:** Priya Raman, Group CTO, Meridian Mutual Group
**To:** Incoming Principal Architect, AI Platform (you)
**Subject:** Getting AI agents under control before they get us into trouble

> Welcome aboard. I'll be blunt about why you're here.
>
> Over the past 20 months we've gone from a handful of pilots to roughly 2,300 "AI agents" across the Group. Nobody planned it. Fourteen teams are calling four model providers in at least five different ways. We've had a runaway agent that burned £41k over a weekend. We've had a near-miss where health data went to a provider we hadn't approved for it. Last month, during Storm Ciarán II, a provider outage took our claims-triage agents down at the worst possible moment.
>
> Our token bill is growing about 13% a month. My CFO has given me a hard cap. My CISO wants everything logged forever, and my DPO wants almost nothing kept. Claims wants the best models everywhere. Procurement has just signed a three-year commit with one vendor without asking me. Next year we plan to open agents to 1,200 external broker firms, and the year after we may white-label the platform for a banking partner.
>
> I need a platform: one way to build, run, govern, observe and pay for AI agents at Meridian. It has to survive three years of growth, two regulators' scrutiny and our own internal politics.
>
> I need your architecture and plan in a week. You'll defend it to me, the CFO and the CISO. I don't want a perfect answer. I want to know what you'd do, what you wouldn't do, and why.
>
> — Priya

---

## 2. Company background

**Meridian Mutual Group plc**
- Composite insurer and wealth manager
- Founded 1887 as a friendly society; demutualised 2004; FTSE 250
- **Gross written premium:** £6.8bn
- **Assets under management:** £41bn (wealth arm)
- **Employees:** 11,400
- **Customers:** 7.2m

**Legal entities and jurisdictions**

| Entity | Location | Regulator(s) | Business |
|---|---|---|---|
| Meridian Insurance UK Ltd | London, Swindon, Leeds | PRA, FCA | Motor, home, commercial lines |
| Meridian Life & Pensions Ltd | Edinburgh | PRA, FCA | Life, critical illness and income protection underwriting |
| Meridian Wealth Ltd | London | FCA | Investment management, advice |
| Meridian Europe DAC | Dublin | Central Bank of Ireland | EU motor/home, Irish life |
| Meridian Versicherung GmbH | Frankfurt | BaFin | German commercial lines (acquired 2023) |

**Strategic context**
- **"Meridian 2030":** the board strategy. It commits to **£4.5m annual benefit** from AI by end of FY2027, mainly claims handling time and underwriting throughput, and to a **15% cost-to-income reduction** by 2029.
- **Acquisition legacy:** the 2023 acquisition of Versicherung (formerly Rheinland Assekuranz) brought an Azure/Entra ID estate. It has never been fully integrated with the Group's AWS/Okta estate.
- **Board attention:** the Board Risk Committee has flagged "uncontrolled AI adoption" as an **emerging principal risk** and wants a credible control framework by **Q2 2027**.
- **Financial year:** January–December. FY2027 budgets are being finalised **now**.

---

## 3. Current estate

### 3.1 Agent inventory (best available, 71% confidence)

Data comes from a two-week discovery exercise: API key scans, provider invoices and a team survey.

| Business area | Approx. "agents" | Nature | Customer-facing? | Data class |
|---|---|---|---|---|
| Claims triage and FNOL | 640 | Autonomous; tool use (ClaimCenter API, image analysis, fraud score) | Partly (FNOL chat) | C3–C4 (incl. injury/health) |
| Customer service | 420 | Conversational; RAG over policy docs | Yes (web, app, WhatsApp) | C3 |
| Underwriting assist (GI and Life) | 310 | Copilot; Life includes medical evidence summarisation | No | C4 (special category) |
| IT operations | 260 | Autonomous; runbook execution, ticket triage | No | C2–C3 |
| Developer agents | 250 | Code generation and review, CI bots | No | C2 (source code) |
| Finance reconciliation | 180 | Batch; overnight | No | C3 |
| Investment research (Wealth) | 150 | RAG plus web; analyst copilots | No | C3 (MNPI risk) |
| HR | 90 | Policy Q&A, CV screening (paused by Legal) | No | C3–C4 |
| **Total** | **~2,300** | | | |

**Definitional note.** "Agent" is used loosely. It covers:
- autonomous multi-step agents with tool access
- single-prompt cron jobs
- Copilot-style assistants with one human user each
- agents that call other agents (at least 3 known multi-agent chains in Claims)

### 3.2 How agents reach models today

| Path | Users | Notes |
|---|---|---|
| **"Gateway" (homegrown)** | ~38% of traffic | Node.js proxy written by one engineer, now on paternity leave. Single AWS region (eu-west-2). No HA. Provider A and B only. Basic API key auth. No per-agent attribution. |
| **LiteLLM fork** | ~22% | Run by Versicherung on Azure. Forked 14 months ago and not rebased since. |
| **Direct SDK calls** | ~31% | 63 distinct provider API keys found; 11 committed to Git (rotated after discovery). |
| **SaaS tools with embedded LLMs** | ~9% | Two claims-tooling vendors and one dev-tool vendor. Their models are invisible to us. |

### 3.3 Model providers

| Provider | Access route | Contract | Regions available to us | Notes |
|---|---|---|---|---|
| **Provider A** (frontier lab) | Direct API and via AWS | **New:** 3-year committed spend, **$2.5m/yr minimum**, signed Oct 2026 by Procurement. 18% discount above commit. | US; EU (subset of models); UK inference "on roadmap" | Best eval scores on claims tasks. 3 incidents in 6 months, longest 3h 40m. |
| **Provider B** (frontier lab via hyperscaler) | Azure-hosted | Pay-as-you-go. Provisioned throughput available as a monthly commit. | UK South, EU (Sweden, France) | Used by Versicherung. One 2-day latency degradation (p95 4s → 31s). |
| **Provider C** (hyperscaler-native models) | GCP | Pay-as-you-go; no GCP footprint today | EU (Belgium, Netherlands) | Cheapest frontier-class tier. Procurement has not onboarded them. |
| **Self-hosted open-weight** | Swindon data centre | n/a | UK (on-prem) | 32× A100 40GB, 3.5 years old. **Hardware support ends Q3 2027.** DC power-capped at +40 kW. Utilisation 22%. |

### 3.4 Enterprise platforms

- **Cloud**
  - AWS is primary (eu-west-2 London, eu-west-1 Dublin)
  - Azure (Germany West Central) from the Versicherung acquisition
  - On-prem DC in Swindon (lease ends 2029)
- **Identity:** Okta (Group); Entra ID (Versicherung). Federation partial. Agents use shared service accounts in 70% of cases.
- **Secrets:** HashiCorp Vault (Group). Azure Key Vault (Versicherung).
- **Integration:** Confluent Kafka (Group event backbone); MuleSoft (legacy APIs)
- **Core systems**
  - Guidewire ClaimCenter/PolicyCenter (GI)
  - Mainframe policy admin (Life; CICS/COBOL; batch windows 01:00–05:00)
  - Salesforce (service)
  - ServiceNow (ITSM)
- **Data:** Snowflake (analytics); S3 data lake; an OpenSearch vector index owned by Customer Service (no data classification applied)
- **Observability**
  - **Splunk Enterprise:** 500 GB/day licence, currently at 440 GB/day. **Renewal in March 2027 is quoted at +38%.**
  - **Datadog:** Digital team only
  - **OpenTelemetry:** adopted by 3 teams
- **Platform engineering**
  - EKS (Group); AKS (Versicherung)
  - Terraform, ArgoCD, GitHub Enterprise

---

## 4. Stakeholders

| Name | Role | What they want | What they've said |
|---|---|---|---|
| **Priya Raman** | Group CTO (your sponsor) | One platform. Visible control within 6 months. A credible 3-year story for the board. | *"I'd rather have 80% of agents on a boring platform in six months than 100% on a beautiful one in eighteen."* |
| **Marcus Hale** | Group CFO | Hard cap on AI spend. Unit economics per claim, per policy, per customer contact. Chargeback to business units from 1 Jan 2027. | *"FY27 model spend is capped at £3.6m. Not a target, a cap. If you can't show me cost per claim by March, I'll freeze new agents."* |
| **Dr Elena Voss** | CISO | Zero trust for agents. Full prompt and response logging for forensics. No C4 data to any US-hosted endpoint. Kill switch per agent. | *"If an agent does something, I need to reconstruct exactly what it saw and why. Forever, ideally."* |
| **Siobhan O'Connell** | Group DPO | Data minimisation. Prompt content retained ≤30 days unless a specific lawful basis exists. DPIAs for every agent touching personal data. | *"Logging every prompt is building the largest unstructured personal-data store in the company. Elena's 'forever' is a GDPR breach waiting to happen."* |
| **Tomasz Wierzbicki** | Head of Model Risk Management | Every agent in the model inventory, tiered and validated. Decisions **reproducible for 7 years** for Life underwriting. | *"If a regulator asks why we declined Mrs Jones's critical illness cover in 2027, I need to replay it in 2033."* |
| **Daniel Okafor** | MD, Claims | Best model on every claim. Agents never down during surge events. Freedom to buy vendor tools. | *"My storm-surge agents saved 19,000 handler hours last winter. Throttle them and I go back to agency staff at £38/hour. I'll buy my own if I have to."* |
| **Hannah Liu** | Director, Digital & Customer | Customer chat at **99.95%** availability. Time to first token under 1.5s. Brand-safe responses. | *"Customers don't care which model is down. The chat is up, or it isn't."* |
| **Raj Patel** | Head of Platform Engineering | Build it in-house on Kubernetes. Fix on-call burnout. | *"My team is nine people, and four are on the on-call rota for the old gateway already. Don't hand us another pager without headcount."* |
| **Gemma Ashworth** | Chief Procurement Officer | Consolidate vendors. Meet the Provider A commit. Reduce supplier count by 20%. | *"The commit saved us 18%. Routing traffic elsewhere means we pay for tokens we don't use."* |
| **Alistair Brennan** | Group Chief Risk Officer | Concentration risk managed. DORA and operational resilience obligations met. Board-ready risk reporting. | *"Claims FNOL is an Important Business Service with a 24-hour impact tolerance. Show me how an AI vendor outage doesn't breach it."* |
| **Lena Fischer** | CEO, Meridian Europe | EU data stays in the EU. Versicherung's autonomy respected. Works council kept onside. | *"Our works council will block any system that logs how employees interact with agents unless they've agreed to it. That takes months."* |
| **Grace Mensah** | Chief Internal Auditor | Evidence that controls operate, not just that they exist. | *"I'll audit this platform in Q3 2027. Policy documents don't count; logs do."* |

### 4.1 Known conflicts (not exhaustive)

- **Voss vs O'Connell vs Wierzbicki:** retention of prompt and response content. The positions are forever, ≤30 days and 7 years.
- **Hale vs Okafor:** cost cap vs surge throughput and frontier models everywhere
- **Ashworth vs Brennan:** single-vendor commit vs concentration risk
- **Liu vs provider reality:** 99.95% availability vs providers who publish no SLA or a 99.9% SLA
- **Patel vs Raman:** build in-house vs speed. Headcount allowed is +4 in FY27.
- **Fischer vs Voss:** employee-interaction logging vs works council co-determination
- **Okafor vs everyone:** threatens shadow procurement if constrained

---

## 5. Constraints

### 5.1 Technical

- **No new cloud provider:** no new provider for workloads without CTO and CRO approval. GCP counts as new. API-only use of Provider C is "under debate".
- **Data residency**
  - C4 data must not leave UK/EU.
  - EU-entity personal data must not leave the EU without a transfer mechanism. Legal currently prohibits this for C4.
- **Life mainframe:** only reachable via MuleSoft, at **≤ 120 TPS**. The batch window blocks reads from 01:00 to 05:00.
- **Peak rate limits:** Provider A's EU quota is **4.0M input tokens/minute** org-wide. Estimated storm-surge peak demand is **9–11M TPM**.
- **Identity:** existing agents can't easily be re-credentialed. 60% are owned by teams with no capacity this quarter.
- **Migration:** the platform must support **gradual migration**. A big-bang cut-over is ruled out by the CTO.

### 5.2 Financial

| Item | FY2027 |
|---|---|
| Model/token spend (all providers, incl. Provider A commit) | **£3.6m hard cap** |
| Platform build and run (people excluded; infra, licences, tooling) | **£1.4m** (opex and capex combined) |
| Headcount | Existing 9 platform engineers, **+4 approved hires** (start Q2 2027 at earliest) |
| Provider A minimum commit | **$2.5m/yr** (≈ £1.97m at budget FX of 1.27) for 3 years, take-or-pay |
| Splunk renewal (if renewed as-is) | £1.1m (from £0.8m) |
| GPU refresh (if pursued) | Not budgeted. Any capex >£500k needs Board approval (next window: **April 2027**). |

- 38% of current model spend is billed in **USD**.
- **Chargeback:** the CFO wants it live from 1 Jan 2027, at agent level. Business units will **resist** anything they see as unfair.

### 5.3 Regulatory and compliance

Summaries are from the Legal and Compliance memo. Where something is uncertain, the memo says so.

| Regime | Relevance |
|---|---|
| **UK GDPR / EU GDPR** | Lawful basis, minimisation and retention. Art. 9 special category data (health) in Life and Claims. Art. 22 automated decision-making in underwriting and claims. DPIAs required. |
| **EU AI Act** | Life and health insurance risk assessment and pricing are listed high-risk use cases. *Legal's note: the application date for high-risk obligations is subject to proposed EU deferral. Plan for obligations applying to Meridian Europe's Irish life book no later than **late 2027**, and possibly earlier.* GPAI provider obligations sit with the vendors, but deployer obligations sit with us. |
| **DORA** (EU entities) | ICT third-party register, incl. AI providers. Major ICT incident classification and reporting within tight regulatory clocks. Exit strategies for critical ICT providers. Resilience testing. |
| **PRA SS2/21** (UK) | Outsourcing and third-party risk. Material outsourcing needs notification, exit plans and audit rights. |
| **PRA/FCA operational resilience** | Important Business Services (IBS) with impact tolerances. Claims FNOL: **24 hours**. Customer service contact: **48 hours**. |
| **FCA Consumer Duty** | Good outcomes for retail customers. Evidence that AI-driven customer interactions aren't causing foreseeable harm. Vulnerable customer handling. |
| **Model risk** | Meridian has adopted SS1/23-aligned model risk management Group-wide. All "models", including LLM-based agents, must be inventoried, risk-tiered, validated and monitored. |
| **German Works Constitution Act** | The works council has co-determination rights over technical systems capable of monitoring employee behaviour or performance. |
| **Wealth: market abuse (UK MAR)** | Research agents may process material non-public information. Information barriers must be preserved. |
| **Future broker tenants** | Brokers will require SOC 2 Type II or ISO 27001 evidence, contractual data isolation and per-tenant audit. |

### 5.4 Organisational

- No central AI governance body exists. An "AI Steering Group" meets monthly and has no decision rights.
- Agent ownership is unclear for **41%** of discovered agents.
- **Versicherung:** engineering reports to Lena Fischer, not Group CTO. It has its own change advisory board.
- **SRE:** there is no SRE function. Platform Engineering is on call for infrastructure; application teams are on call for their own apps (in theory).

---

## 6. Growth projections (Strategy team, 3-year plan)

| Metric | Today (Q4 2026) | End FY2027 | End FY2028 | End FY2029 |
|---|---|---|---|---|
| Internal agents | 2,300 | 3,500 | 6,000 | 9,000 |
| External tenants | 0 | 0 (pilot with 20 brokers in Q4) | **1,200 broker firms** | 1,200 brokers + **Northgate Bank** white-label (bancassurance, ~2m end customers) |
| LLM requests / month | 38M | 70M | 160M | 340M |
| Tokens / month (input + output) | 61B | 115B | 260B | 560B |
| Peak-to-average ratio | 3.2× (storm: 6×) | 3.5× | 4× | 4× |
| Multi-agent chains | ~3 known | "many" | Strategy expects agent-to-agent the norm | — |
| Customer-facing share of traffic | 18% | 25% | 40% | 55% |

- **Assumption from Strategy:** blended price per token falls **25% per year**. Nobody has checked this against vendor roadmaps or Meridian's model mix.
- **Northgate Bank requirements** (from the term sheet):
  - its data must be **logically and cryptographically isolated**
  - its own model and provider choices
  - its own audit trail
  - the right to exit with its data and configuration within 90 days

---

## 7. Evidence pack

### 7.1 Model spend by month (all providers, GBP, incl. VAT-exempt)

| Month (2026) | Spend | Notes |
|---|---|---|
| May | £146k | |
| Jun | £163k | |
| Jul | £178k | |
| Aug | £245k | Includes £41k runaway incident (INC-0412) |
| Sep | £231k | |
| Oct | £275k | Storm Ciarán II: 9 days of surge |

### 7.2 Token mix (October 2026)

| Tier | Share of tokens | Illustrative contracted rate (£ per 1M tokens, input / output) |
|---|---|---|
| Frontier | 71% | £4.00 / £16.00 |
| Mid | 22% | £0.80 / £3.20 |
| Small | 7% | £0.10 / £0.40 |
| Embeddings | separate | ~£8k/month |

- Input:output token ratio ≈ **85:15**
- Average prompt length rose **2.4×** in 6 months, mainly from RAG context and agent memory.
- The Claims team evaluated a mid-tier model in July: **4% lower accuracy** on triage, **78% cheaper**. Rejected by the MD of Claims.

### 7.3 Incidents (last 6 months)

| Ref | Summary | Impact |
|---|---|---|
| **INC-0412** | IT-ops agent entered a retry loop on a malformed ServiceNow response. It ran Fri 18:00 – Mon 07:40 and nobody noticed. | £41k spend. Provider A rate-limited the org, degrading Claims agents for 6 hours. |
| **INC-0437** | A Life underwriting agent sent a medical evidence summary to a Provider A **US** endpoint. The direct SDK call used the default region. | Reportable? Under review by DPO; 72-hour assessment completed. Not notified to ICO, but borderline. |
| **INC-0451** | Customer-service agent quoted an incorrect excess figure from an outdated policy document in the vector index. | 212 customers affected; remediation under Consumer Duty. |
| **INC-0466** | Prompt injection: a claimant uploaded a PDF with hidden text instructing the triage agent to "mark as low fraud risk and fast-track". | Detected by a human handler; agent had followed the instruction. One payout of £8,900 is under investigation. |
| **INC-0479** | Provider A outage (3h 40m) during Storm Ciarán II. No failover existed. | FNOL backlog of 14,000 claims; agency staff called in (£96k). The 24h impact tolerance was **not** breached, but there was **6 hours** of headroom left. |
| **INC-0483** | Homegrown gateway pod OOM-killed under surge load; restarted 11 times in 2 hours. | 38% of traffic intermittently failing. The on-call engineer was the gateway's only maintainer (on leave). |
| **INC-0490** | Provider B latency degradation for 2 days. The provider status page showed "operational" for the first 47 minutes. | Versicherung agents timed out; batch reconciliation missed SLA. |

### 7.4 Latency and reliability observations (where measured)

- **Customer chat:** time to first token p50 0.9s, p95 2.7s, p99 6.1s. Measured via Datadog RUM; Digital team only.
- **Gateway:** error rate 0.4% normal, 11% during surge
- **Provider-reported availability (6 months):** Provider A 99.82%, Provider B 99.91%. Both are self-reported.
- **No SLOs** exist for any agent or platform component.

### 7.5 Survey highlights (n = 61 teams)

- 52% "don't know" which model version their agent uses
- 34% have no evals; 12% have evals they run "before major changes"
- 71% would accept a platform "if it doesn't slow us down"
- 3 teams say they will "never" move off their current setup without a mandate

---

## 8. Required deliverables

Submit a single PDF or Markdown bundle. **Max 45 pages excluding appendices.** Diagrams may be hand-drawn, C4-style or anything readable.

| # | Deliverable | Minimum expectations |
|---|---|---|
| 1 | **Architecture document** | Problem framing, principles, target architecture, key decisions with alternatives considered (ADR style encouraged), explicit assumptions |
| 2 | **Context diagram** | System boundary, actors (internal users, agents, brokers, Northgate, regulators, providers), external dependencies |
| 3 | **Component diagram** | Major components, responsibilities, and which are built vs bought vs existing |
| 4 | **Data flow description** | At least: (a) a C4 Life underwriting request, (b) a storm-surge claims triage call during a primary provider outage, (c) a broker-tenant request in FY2028. Show where data is classified, transformed, logged, stored and how long it's kept. |
| 5 | **Threat model** | Method of your choice (STRIDE, attack trees, etc.). Must cover prompt injection, tool abuse, tenant breakout, data exfiltration via providers, credential compromise and insider risk. |
| 6 | **Cost model** | FY2027–FY2029. Your drivers, assumptions and sensitivity. How you stay within the FY27 cap (or why you can't, and what you'd ask for). Treatment of the Provider A commit. Unit economics the CFO asked for. |
| 7 | **Observability strategy** | Metrics, logs, traces and alerting. What is captured at which layer, who sees what, retention by data class, and the cost of observability itself. |
| 8 | **SLO/SLI definitions** | Platform and at least three agent classes. Error budgets and what happens when they're spent. |
| 9 | **Build vs buy analysis** | For at least: gateway/routing, guardrails/content safety, observability/LLM tracing, eval tooling, vector storage, agent runtime. Include total cost of ownership and exit cost. |
| 10 | **Risk register** | At least 15 risks, with likelihood, impact, owner (a named stakeholder) and mitigation. |
| 11 | **Incident runbook** | Full runbook for: **"Primary model provider unavailable during storm surge."** Outline runbooks for: runaway agent cost, suspected cross-tenant data exposure. |
| 12 | **Governance model** | Decision rights (who approves a new agent, a new model, a new provider), agent lifecycle, model risk tiering, and how you handle the retention dispute between Voss, O'Connell and Wierzbicki. |
| 13 | **Scaling roadmap** | Quarter-by-quarter for FY2027, half-yearly for FY2028–29. What ships when, what gets migrated when, what you deliberately defer, and the decision points where the plan could change. |

**Also required**
- **A one-page executive summary** written for the CFO and the CRO. No jargon.
- **A "What I'd say no to" list:** things stakeholders asked for that you won't deliver, or won't deliver as asked, with reasons.

---

## 9. Rules of engagement

- **Assumptions:** where information is missing, state an assumption and carry on. Unstated assumptions will be treated as gaps.
- **Contradictions:** the brief contains contradictions, because real engagements do. Resolving or escalating them is part of the work.
- **Vendors:** you may name real products and vendors. You'll be asked to justify them, including what you'd do if they disappeared.
- **No single right answer:** we're assessing judgement, not conformity to a reference architecture.
- **The defence:** in a hiring process, expect to present for about 20 minutes, then have at least one major constraint changed live and be asked how your design responds. Practising alone? Pick a constraint from the brief, change it, and test whether your design bends or breaks.

---

*Part of [The Giblets Challenges](https://github.com/The-Giblets-Challenges). I wanted it. So I forged it. Now forge yours.*
