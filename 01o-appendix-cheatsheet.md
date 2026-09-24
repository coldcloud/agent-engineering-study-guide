---
title: Appendix — The cheatsheet (interview day)
type: study-guide
created: 2026-09-23
tags:
  - interview-prep
---

# Appendix — The cheatsheet (interview day)

← Previous: [Part G (2 of 2) — Mocks, feedback and final prep](01n-part-g2-fde-mocks-and-final-prep.md) · Index: [Full Read-Through](01-full-read-through.md)

*Source: google-genai-fde-quick-cheatsheet.md*

> **Why this is here, and why it is not expanded.** The cheatsheet is deliberately terse. It is the only thing you open on interview day, so every line should trigger a full explanation you already hold from Parts A to G. Use it as a final test: cover each section and reconstruct it from memory, then check. If a line does not expand in your head, go back to the step that teaches it (SAFE COST and SCOPE are [Step 81 · SAFE COST and SCOPE](01m-part-g1-fde-interview-knowledge.md#81-memorize-safe-cost-and-scope); the architecture is [Step 79 · Architecture from memory](01m-part-g1-fde-interview-knowledge.md#79-draw-architecture-from-memory-you-already-know-every-box-from-parts-b-to-f); the decisions table is [Step 80 · The eight decisions they will probe](01m-part-g1-fde-interview-knowledge.md#80-read-the-eight-decisions-they-will-probe-write-one-sentence-of-your-own-for-each); the Google Cloud mapping is [Step 82 · Google Cloud vocabulary](01m-part-g1-fde-interview-knowledge.md#82-read-google-cloud-vocabulary-map-each-product-to-a-box-you-already-know); the coding plan is [Step 83 · The coding interview protocol](01m-part-g1-fde-interview-knowledge.md#83-read-the-coding-interview-protocol-then-solve-q1-q7-q23-q40-with-a-25-min-timer-each-no-running-code)).

## Google Cloud GenAI FDE — Quick Interview Cheatsheet

Full guide: [[google-genai-fde-targeted-interview-prep]]. Supporting depth: [[anthropic-agent-systems-fde-notes-2025-present]], [[agentic-ai-system-design-interview-guide-2026]], and [[google-fde-python-coding-prep]].

---

### Confirmed loop

- **RRK — 60 min:** customer discovery → GenAI/system design → deployment, security, scale, cost, evaluation, troubleshooting
- **30-minute break**
- **Coding — 60 min:** Python, approximately 30–50 lines, static editor, algorithm + OOP/design fundamentals, test and optimize aloud
- AI tools are prohibited during the interview.

Recent reports agree on the two-round shape but disagree on coding difficulty: prepare for **medium through hard**.

---

### RRK minute plan

| Time | Goal |
|---:|---|
| 0–2 | Restate and set agenda |
| 2–15 | Discovery and stakeholder alignment |
| 15–18 | Requirements, assumptions, MVP, non-goals |
| 18–35 | End-to-end happy path |
| 35–48 | Security, state, failure, eval, scale, cost |
| 48–56 | Follow-ups and changed constraints |
| 56–60 | Tradeoffs, rollout, handover, recap |

Opening:

> “I’ll first clarify the customer outcome, workflow, data and risk, then define a narrow MVP, design the end-to-end flow, and close with operations, evaluation, rollout and tradeoffs.”

---

### Discovery — ask what changes the design

#### VALUE

- **V — Value:** What metric and baseline? Who owns it?
- **A — Actors:** User, buyer, domain expert, security owner, operator?
- **L — Limits:** Scale, latency, availability, budget, residency?
- **U — User authority:** Read, recommend, write, communicate, money?
- **E — Evidence:** Data sources, quality, ownership, permissions, golden set?

Then summarize:

> “For the MVP I assume **[user/scale]**, **[allowed action]**, **[latency]**, and **[data]**. Success is **[business metric + quality/safety floor]**. I will exclude **[high-risk/nonessential scope]**.”

---

### Architecture from memory

```mermaid
flowchart LR
    U["User"] --> G["Gateway<br/>auth + quota"]
    G --> R{{"Route"}}
    R --> W["Workflow"]
    R --> A["Agent"]
    A --> C["ACL-filtered<br/>context + RAG"]
    A --> P{{"Policy<br/>risk + budget"]}
    W --> P
    P --> T["Typed tools / MCP"]
    P --> H["Human approval"]
    H --> T
    T --> S["Customer systems"]
    A <--> D[("Durable state<br/>event log")]
    W <--> D
    A --> O["Trace + eval"]
    T --> O
```

Narrate: authenticate → route → retrieve permission-safe context → model proposes → policy validates → tool executes idempotently → verify state → checkpoint → observe/evaluate.

---

### Decisions interviewers probe

| Choice | Rule |
|---|---|
| Workflow vs agent | If the path is drawable, encode it. Use an agent where the next action depends on new evidence. |
| Single vs multi-agent | Split only independent, high-value parallel branches or distinct trust/tool boundaries. |
| RAG vs long context | RAG for large, fresh, permissioned corpora; long context for bounded holistic reading. |
| API vs UI automation | API first; computer use only when legacy constraints justify fragility. |
| Sync vs async | Async for approvals, long tools, variable latency, and resumability. |
| Automatic vs HITL | Automate bounded/reversible/measured actions; gate high-blast-radius actions. |

Always define maximum steps, tools, tokens, dollars, wall time, fan-out, repeated-state detection, and programmatic completion.

---

### Production checklist: SAFE COST

- **S — Security:** identity propagation, least privilege, ACL before retrieval, tenant isolation
- **A — Availability:** SLOs, timeouts, retries with jitter, circuit breakers, degraded mode
- **F — Failure/state:** idempotency, checkpoints, compensation, reconciliation, resume
- **E — Evaluation:** real customer cases, outcome + trajectory, repeated trials, safety suite
- **C — Cost:** model routing, context/tool budgets, caching, batching, human-review cost
- **O — Observability:** request trace across retrieval/model/tools/policy; redacted logs
- **S — Scale:** stateless workers, queues, partitioning, backpressure, quotas, locality
- **T — Transfer:** rollout, runbooks, dashboards, training, customer ownership, product feedback

---

### Security answer

Threats: cross-tenant leakage, prompt injection, credential theft, excessive action, exfiltration, generated code, audit/retention failure.

Controls:

- Authenticate user and agent; propagate tenant and purpose
- Apply document/row ACL **before retrieval**
- Treat web/email/docs/tool output as untrusted data
- Keep authorization and policy outside the model
- Put tools behind schemas, scopes, idempotency, destination rules, and audit
- Keep reusable secrets out of prompts and sandboxes; use a vault/proxy
- Combine filesystem and network isolation for generated code
- Human approval for irreversible, financial, production, or external actions
- VPC Service Controls for service/data perimeters; DLP/redaction; regional storage

Say:

> “I assume the model can be convinced. Security rests on what IAM, policy, network and execution layers make impossible.”

---

### Evaluation answer

Build the golden set with customer SMEs.

1. **Outcome:** Did the database/artifact/business state become correct?
2. **Trajectory:** Correct tools/arguments, policy, turns, recovery, no loops?
3. **Quality:** Groundedness, citation correctness, completeness, tone?
4. **Safety:** Injection, leakage, unauthorized action, harmful output?
5. **Operations:** p50/p95 latency, tokens, cost, tool errors, escalation?
6. **Business:** Time saved, resolution, conversion, reopen/correction, adoption?

Use deterministic graders first, calibrated LLM rubrics for subjective quality, and sampled human review. Separate capability from near-100%-pass regression suites. Run multiple trials.

---

### Scale and cost answer

#### Scale

Quantify → locate bottleneck → stateless workers → partition state → queue long work → backpressure/fairness → cache safely → batch ingestion → load-test dependencies → regionalize where required.

#### Cost

Optimize **cost per successful task**, including inference, retrieval, tools, compute, retries, storage, observability, and human review.

Order: remove calls → route models → shrink context/results → cache → parallelize safe branches → batch → cap retries/steps/fan-out → re-evaluate quality.

---

### Troubleshooting: SCOPE

- **S — Scope:** who, where, when, percentile, baseline, business impact
- **C — Compare:** affected vs healthy cohort; recent changes
- **O — Observe:** client → edge → app → agent → model → retrieval → tools → data
- **P — Prove:** one falsifiable hypothesis; change one variable
- **E — Ease impact:** safest mitigation, verify recovery, then prevent recurrence

Never invent a cause. Communicate symptom, scope, impact, mitigation, evidence, and next update.

Official sample “website is slow”: clarify page/user/region/device/time → waterfall/Core Web Vitals → network/edge → backend trace → DB/cache/dependencies → compare rollout cohorts → mitigate → verify → postmortem.

---

### Google Cloud mapping

- Models: Gemini on Vertex AI
- Agent framework: ADK
- Runtime/session/eval/trace: Vertex AI Agent Engine
- Enterprise product: Gemini Enterprise Agent Platform
- Tool/data: MCP; agent interoperability: A2A
- RAG: Vertex AI Search / Vector Search; BigQuery or AlloyDB where appropriate
- Serving: Cloud Run for simple stateless; GKE for deeper control
- Async: Pub/Sub, Cloud Tasks, Workflows
- API: Apigee / API Gateway
- Security: IAM, Workload Identity Federation, Secret Manager, Sensitive Data Protection, CMEK, VPC Service Controls, Model Armor
- Operations: Cloud Logging, Monitoring, Trace, Error Reporting

Names do not score by themselves. Explain why the service matches the workload and verify launch stage, region, quota, SLA, residency, CMEK, VPC-SC, and Access Transparency requirements.

---

### Reported RRK shape/questions

Confidence labels matter:

- **[candidate report]** Choose agent/workflow automation or generative media/content creation.
- **[candidate report]** About 15 min discovery, 30–40 min design, 5–6 follow-ups.
- **[candidate/recruiter notes]** Enterprise modernization over CRM, Drive, Office 365, and sensitive data; MVP → production → deployment/scale/monitoring/eval.
- **[candidate report]** Creative story-writing agent, with usefulness evaluation.
- **[aggregated practice]** Marketing research/outreach agent.
- **[aggregated practice]** Bank fraud investigation assistant.
- **[aggregated practice]** Airline operations copilot.

Expect twists: 100× scale, tool 500s, conflicting systems, prompt injection, regional data, wrong model conclusion, async approval, cost reduction.

---

### Coding minute plan

| Time | Action |
|---:|---|
| 0–5 | Restate; clarify size, ordering, duplicates, invalid input, mutation |
| 5–10 | Example + edge cases |
| 10–15 | Baseline → bottleneck → invariant → optimized approach |
| 15–38 | Code readable Python, narrating intent |
| 38–47 | Dry-run line by line |
| 47–55 | Tests + exact time/space |
| 55–60 | Scale/constraint follow-up |

Priority: graphs/trees/trie → strings/parsing/hash/stack/sliding window → intervals/heap/binary search → tracker/cache/rate limiter/registry → mixed hard practice.

#### Python tools

- `dict`, `set`, `defaultdict`, `Counter`
- `deque` for O(1) queue ends
- `heapq` for top-k / k-way merge
- `sorted(..., key=...)`
- `enumerate`, `zip`
- DFS colors: unseen/visiting/done
- BFS: mark visited when enqueued
- Trie: dict children + terminal flag/count

#### Edge cases

Empty · one · duplicates · missing · ties/order · cycles/self-loops · disconnected · deep/skewed · malformed · Unicode/case · huge input.

Say:

> “Because this editor cannot execute, I’ll dry-run the example against each state mutation and then test the boundary that is most likely to break the invariant.”

---

### Final five signals

1. I discover the customer's real problem before selecting AI.
2. I use deterministic software for deterministic decisions.
3. I place security, policy, budgets, and completion outside the model.
4. I evaluate actual outcomes under realistic failure and scale.
5. I ship a narrow MVP the customer can safely operate and expand.

---

← Previous: [Part G (2 of 2) — Mocks, feedback and final prep](01n-part-g2-fde-mocks-and-final-prep.md) · Index: [Full Read-Through](01-full-read-through.md)
