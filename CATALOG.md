# Skill Catalog

Full catalog with build status. `BUILT` skills are written and reviewable now; `PLANNED`
skills are scoped but not yet authored.

Every skill, built or planned, is bound by [`DOCTRINE.md`](DOCTRINE.md) and reaches client
state only through the MCP tool contract, which ships with the licence.


---

## accusights-core — required base layer

| Skill | Status | Purpose |
|---|---|---|
| `data-boundary` | **BUILT** | Tokenise sensitive values before the model, rehydrate after. The Guardee-equivalent layer. |
| `local-reduce` | **BUILT** | Deterministic reduction ladder — 3,000 alerts to ~20 before any AI call. |
| `evidence-ledger` | **BUILT** | Versioned hash-chained determinations: inputs, method, determination, confidence. |
| `coverage-map` | PLANNED | What the telemetry can and cannot see. Every report's blind-spot statement comes from here. |
| `model-router` | PLANNED | Route each task to the cheapest model that can do it; record which model produced which determination. |
| `handoff` | PLANNED | Phase-5 handover: operator runbooks, config walkthrough, "run it without us" checklist. |

## accusights-compliance

| Skill | Status | Purpose |
|---|---|---|
| `framework-select` | **BUILT** | Industry + region + data types to in-scope frameworks, with exclusions stated. |
| `control-signal-map` | **BUILT** | Evaluate controls against live telemetry. Passing / failing / **unevidenced**. |
| `posture-score` | **BUILT** | Per-framework score, trend, and the specific controls that move it. |
| `control-drift` | **BUILT** | Passing-to-failing and coverage-loss detection, routed as alerts the day they break. |
| `policy-builder` | **BUILT** | Gap sweep, review cadence, gap-driven drafting, reg-change watch. Always pending-review. |
| `audit-readiness` | **BUILT** | SSP, POA&M, verified evidence packs, questionnaire pre-fill, auditor Q&A with citations. |
| `insurance-evidence` | **BUILT** | Underwriter-grade risk evidence from observed control operation. |
| `data-lineage` | PLANNED | Where personal data can actually go — flows, unintended destinations, data written to logs. |
| `vendor-risk` | PLANNED | Vendor intake, tiering, review cadence, fourth-party exposure. |
| `access-review` | PLANNED | Drive the review cycle from identity telemetry rather than a spreadsheet. |
| `retention-check` | PLANNED | Retention policy vs actual retention across stores. |
| `cross-border` | PLANNED | Transfer mechanisms, residency, and what moves where. |
| `control-design` | PLANNED | Propose the control + signal for an unevidenced requirement. |
| `readiness-gap` | PLANNED | Pre-certification gap assessment against a target framework. |

## accusights-soc

| Skill | Status | Purpose |
|---|---|---|
| `alert-triage` | **BUILT** | Alert to verdict with a stated reason. True positive / false positive / **unjudged**. |
| `verdict-memory` | **BUILT** | Indexed signature store — O(1) classification, analyst decisions as the training signal. |
| `reachability-filter` | **BUILT** | Confirmed path or not a finding. Shows every hop; reports the exclusions. |
| `compound-risk` | **BUILT** | Where accepted lows chain into a real path. Scored by endpoint, not by link. |
| `ioc-correlate` | **BUILT** | Intel against telemetry, deterministic match then judged. Reports feed health. |
| `containment-gate` | **BUILT** | Propose / approve / execute. The liability boundary. |
| `response-authority` | **BUILT** | Ownership, jurisdiction, third-party impact, preservation, notification clocks. |
| `incident-timeline` | PLANNED | Reconstruct an incident from the evidence chain, with gaps marked as gaps. |
| `behavior-baseline` | PLANNED | Per-entity normal, and what counts as a departure from it. |
| `campaign-correlate` | PLANNED | Link incidents across time and tenant assets into a campaign view. |
| `threat-playbook` | PLANNED | Match an incident to a response playbook; propose the steps, never run them. |
| `supply-chain-signal` | PLANNED | Vendor compromise signals reaching the client estate. |
| `injection-watch` | PLANNED | Detect and report attempts to steer the agent through ingested content. |
| `detection-tuning` | PLANNED | Which rules produce value, which produce volume, what to retire. |
| `purple-verify` | PLANNED | Confirm a control actually detects what it claims — evidence for `ir-active-response`. |

---

## Build order

**Tranche 1 — BUILT.** Core plus the two packs' load-bearing skills. Enough to run a real
compliance evaluation cycle and a real triage cycle end to end.

**Tranche 2 — next.** `coverage-map`, `incident-timeline`, `behavior-baseline`,
`data-lineage`, `access-review`. These close the honesty gaps: what we cannot see, what
happened in order, what normal is, where data goes.

**Tranche 3.** `model-router`, `handoff`, `detection-tuning`, `purple-verify`,
`campaign-correlate`. Operational maturity and the handover story.

**Tranche 4.** The remaining compliance depth — `vendor-risk`, `retention-check`,
`cross-border`, `control-design`, `readiness-gap` — driven by which frameworks clients
actually buy.

---

## What is not a skill

Deliberately excluded. These stay as MCP servers or as human process:

| Not a skill | Why |
|---|---|
| Log ingestion, parsing, normalisation | Transport. Belongs in an MCP server in the client tenancy. |
| Wazuh / SIEM polling | Transport. |
| Scheduling and sweep cadence | The client's scheduler, not a skill. |
| Notification delivery | Transport, with routing config the skills read. |
| Policy approval | A human act in the client's governance system. |
| Certificate issuance, audit opinion | Requires an accredited auditor. Nothing here substitutes for one. |
| Legal determination | `response-authority` surfaces the question; counsel answers it. |

---

## Skill authoring checklist

Before a skill moves from PLANNED to BUILT:

1. **Frontmatter** — `name` and a `description` that states when to invoke, including the
   phrases a user would actually type.
2. **Required tools** — named explicitly, from the tool contract. Behaviour when absent is stated.
3. **Doctrine reference** — which sections govern it.
4. **Read-only or gated-write** — stated up top.
5. **Reference data in `references/`** — not inline in the instructions.
6. **A worked output example** — the format an operator will actually see.
7. **Rules section** — what it always does.
8. **Refusals section** — what it never does, and what it does instead.
9. **Failure behaviour** — missing tool, empty result, low confidence. Bias to stop and say so.
10. **No client data, no secrets, no tenant identifiers** anywhere in the file.
