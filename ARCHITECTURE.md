# Architecture

How the AccuSights skillset is wired, where the tenancy boundary sits, and what
crosses it.

---

## 1. The inversion

**Today — AccuSights hosts, AccuSights carries the risk:**

```
client endpoints -> Wazuh (AccuSights) -> agent fleet on AccuSights EC2
                                            |
                                            +-> AccuSights Postgres
                                            +-> AccuSights dashboard

AccuSights holds: client logs, evidence artifacts, IOC hits, posture data,
                  policy drafts, containment history.
```

**This repo — the client hosts, the client carries the risk:**

```
                  +-----------------------------------------------+
                  |  CLIENT TENANCY                               |
                  |                                               |
  client stack ---+--> MCP servers --> [ accusights-core ]        |
  (SIEM, cloud,   |                          |                    |
   evidence store,|                    tokenise + reduce          |
   ticketing)     |                          |                    |
                  |                          v                    |
                  |              client's Claude account          |
                  |                          |                    |
                  |          [ compliance pack ] [ soc pack ]     |
                  |                          |                    |
                  |                          v                    |
                  |              evidence ledger (client disk)    |
                  +-----------------------------------------------+

AccuSights holds: nothing. License validation only.
```

---

## 2. Layer model

```
+---------------------------------------------------------------+
|  L4  SKILL PACKS        compliance / soc                       |
|      What to conclude. Framework logic, triage logic, policy   |
|      logic. This is the licensed expertise.                    |
+---------------------------------------------------------------+
|  L3  CORE               data-boundary / local-reduce /         |
|                         evidence-ledger                        |
|      What may cross to the model, and what gets written down.  |
|      Mandatory. Sits in front of every L4 skill.               |
+---------------------------------------------------------------+
|  L2  MCP TOOL SURFACE   inbound (read) / outbound (gated write)|
|      The only way a skill touches client state. Client-owned,  |
|      client-configured, client-revocable.                      |
+---------------------------------------------------------------+
|  L1  CLIENT STACK       Wazuh / SIEM, cloud config, evidence   |
|                         store, ticketing, identity             |
|      Never AccuSights-operated.                                |
+---------------------------------------------------------------+
```

A skill at L4 cannot reach L1 except through L2, and cannot reach the model except
through L3. That is the whole security model, and it is enforced by which tools exist
in the surface — not by asking the model nicely.

---

## 3. The boundary — what crosses, in each direction

### Inbound (client stack -> skill), read-only

| Data | Tokenised before model? |
|---|---|
| Alert metadata: rule id, severity, timestamp, agent id | No — needed, non-sensitive |
| Alert detail: process image, command line, file paths, user | **Yes** |
| Network indicators: source/dest IP, domain, hash | **Yes** (except public IOC feeds) |
| Control telemetry: signal key, boolean status, last-seen | No |
| Client profile: industry, region, frameworks, cadence, tiers | No |
| Crown-jewel asset list | **Yes** — asset names are sensitive |
| Policy bodies, evidence artifacts | **Yes** |
| Cloud/config state | **Yes** |

### Outbound (skill -> client stack), gated write

| Action | Gate |
|---|---|
| Append evidence record | Always allowed — append-only, hash-chained |
| Record alert verdict | Always allowed — reversible, recorded |
| Draft policy | Lands `pending-review`. Never `approved`. |
| Open ticket | Always allowed |
| Notify a channel | Subject to severity routing + quiet hours |
| Propose containment | Always allowed — proposal only |
| Execute containment | **Signed approval required**, per configured tier |

### Crossing to AccuSights

License validation only. No logs, no evidence, no indicators, no posture data, no
tenant identifiers, no prompts, no completions. There is no telemetry channel and no
support channel that carries client data — support happens on the client's screen.

---

## 4. Data flow through core

Every L4 skill call takes this path:

```
  1. FETCH        MCP read tool returns raw client data
  2. REDUCE       local-reduce: dedup, signature-index, rule filter,
                  allowlist, staleness check
                     -> typically 1-5% of input survives
  3. TOKENISE     data-boundary: sensitive values -> stable stand-ins
                  (T_HOST_01, T_USER_04, T_IP_11 ...) held in a
                  session-local map that never leaves the process
  4. REASON       the L4 skill's logic runs against the tokenised,
                  reduced set on the client's Claude account
  5. REHYDRATE    data-boundary: stand-ins -> real values in the output
  6. RECORD       evidence-ledger: inputs, method, determination,
                  confidence, prev_hash, hash
  7. ACT          gated write, if any, per the response tier
```

Steps 2, 3, 5 and 6 are `accusights-core`. Steps 1 and 7 are MCP. Step 4 is the
licensed IP.

---

## 5. Memory tiers

The skillset assumes four tiers on the client's own storage. Sizing is the client's call;
these are the shapes.

| Tier | Contents | Access pattern | Retention |
|---|---|---|---|
| **Hot (RAM)** | Verdict memory index, token map, active correlation window | O(1) signature lookup, per-session | Session |
| **Warm (SSD)** | Event store, control state, policy register, baselines | Indexed query, days–weeks | 30–90 days, configurable |
| **Cold (object storage)** | Archived events, expired evidence, historical posture | Batch export, rarely read | Per retention policy |
| **Immutable (append-only)** | Evidence chain, audit log | Append + verify, never mutate | Life of the compliance obligation |

**The hot tier is what makes triage scale.** Verdict memory is keyed on an alert
*signature* — description + agent + process image + process id + target file + GUID —
so classifying a new alert is an indexed lookup, not a pairwise scan against history.
That is the difference between O(1) and O(N^2) as the client's alert history grows.

**The immutable tier is what makes evidence defensible.** The evidence chain is
hash-linked and kept independent of the audit log, so compromising one does not let an
attacker rewrite the other.

---

## 6. Failure behaviour

| Condition | Behaviour |
|---|---|
| Required MCP tool absent | Skill states which tool is missing and stops. No partial run. |
| Tool present, returns empty | Reported as a **coverage gap**, not as "clean". A control with no telemetry is failing, not passing. |
| Tokenisation map incomplete | Hard stop. Never send partially-tokenised data. |
| Model unavailable or low confidence | Deterministic-only result, explicitly labelled as such. |
| Evidence write fails | The determination is not published. No record, no finding. |
| Approval invalid, expired, or replayed | Action refused **and** recorded as a security event. |

The bias throughout: **stop and say so, rather than proceed and guess.** A silent
degradation in a security control is worse than a loud failure.

---

## 7. What this replaces

The skillset is a repackaging of the AccuSights managed SOC and compliance platform,
which has run these determinations as hosted services since before this existed. What
moved is the reasoning: control-to-signal evaluation, policy lifecycle, audit
readiness, verdict memory, IOC correlation, drift detection, and the guarded response
rails. What stayed a service is transport — pollers, indexer clients, ingestion
pipelines, active-response executors — and those now run in your tenancy, not ours.

**The pattern:** reasoning and judgement become skills; transport, polling, and
ingestion stay as MCP servers running in the client's tenancy.
