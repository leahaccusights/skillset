# AccuSights Skillset

The AccuSights SOC + compliance agent, packaged as **Claude Code skills** that run
inside the **client's own tenancy** — the client's Claude account, the client's data,
the client's infrastructure.

Built on the CodeBridge delivery model: **do the heavy lifting locally, before any AI
involvement**; report **exploitability, not volume**; ship **evidence, not assumption**;
hand the running system back to the client's own team.

Nothing in this repo phones home. The skills are instructions plus reference data.
They reach the client's stack through MCP tools the client installs and points at
their own systems. AccuSights ships the expertise. The client keeps the data and
carries the risk.

---

## Documentation

| Doc | For |
|---|---|
| [`DOCTRINE.md`](DOCTRINE.md) | The six rules every skill obeys. |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Where client data sits and what crosses the boundary. |
| [`CATALOG.md`](CATALOG.md) | Every skill, built and planned. |
| [`LICENSING.md`](LICENSING.md) | Why the skill files in this package are stubs, and what the installer adds. |

The full tool contract, installer spec, and client onboarding guide come with the
licence. Ask us: <https://accusights.com>

---

## Why this exists

A managed SOC holds your logs, your evidence, your indicator hits, and your posture
data — which means it also holds your risk. Change vendors, and the record of how your
controls actually operated stays with them.

This inverts that. The reasoning — control-to-signal evaluation, hash-chained evidence,
policy lifecycle, verdict memory, guarded containment — arrives as a skill pack you
install and run inside your own tenancy. The data never leaves it. See
[`ARCHITECTURE.md`](ARCHITECTURE.md).

---

## The three packs

| Plugin | Required | What it does |
|---|---|---|
| **`accusights-core`** | Yes — prerequisite for both others | The layer that sits *in front of* the model: sensitive-data tokenisation, local-first reduction, and the versioned evidence ledger every other skill writes to. |
| **`accusights-compliance`** | Optional | Framework selection, control-to-signal evaluation, posture scoring, drift detection, policy lifecycle, audit readiness, insurer evidence. |
| **`accusights-soc`** | Optional | Alert triage, verdict memory, reachability filtering, IOC correlation, compound-risk detection, approval-gated containment, response-authority checks. |

A client can license compliance without SOC, or SOC without compliance. Neither runs
without `accusights-core` — that is deliberate, because core is what keeps client data
out of the model and every determination on the record.

---

## Install (client side)

The skillset is **licensed**. The marketplace plugin carries stubs; the installer carries
the product.

```
1. INSTALLER    accusights-skillset-<version>.exe   (Windows, Authenticode-signed)
                accusights-skillset-<version>.pkg   (macOS, notarised)
                .deb / .rpm / docker                (Linux — where most SIEM stacks live)

   Ships the MCP tool servers, the license CLI, and the skill stubs for the
   licensed packs only. Prompts for the license key and activates the machine.

2. ACTIVATE     accusights-license activate ACC-XXXX-XXXX-XXXX-XXXX
                accusights-license status

3. CONFIGURE    Point the MCP servers at the client's own stack — SIEM/Wazuh
                endpoint, evidence store, ticketing, identity. The installer
                cannot do this and does not guess.
```

Installing the marketplace plugin without the installer gets you stubs that fetch nothing
and tools that do not exist. That is intended, and it says so plainly rather than failing
cryptically.

Every skill declares the tools it needs.
A skill whose tools are absent says so and stops. It never guesses at data it cannot read.

### Licensing


**One license key, one machine.** Node-locked, no seat pools. A client needing five machines
is issued five keys. Moving a key is self-service — `deactivate` then `activate`, or
`transfer` if the old machine is unreachable (24h cooldown, 3 per 90 days).

Two independent gates: **skill content** is served by the control center against an
entitlement, and **MCP tools** refuse to register without one — and refuse again if the
entitlement's `machine_id` does not match this machine. Skills without tools read nothing;
tools without skills decide nothing.

Entitlements are signed and **verified offline**, refreshed by a 6-hourly heartbeat, with a
14-day grace window if the control center is unreachable. A vendor licence server must never
be able to blind a client's SOC.

**Never gated, at any license state including after cancellation:** evidence export, and
license status. The compliance record is the client's, not ours.

---

## Client prerequisites

| Requirement | Why |
|---|---|
| Claude Max (or Team/Enterprise) account | Skills run on the client's own subscription. AccuSights never proxies inference. |
| 1–2 dedicated projects | One for compliance, one for SOC. Keeps memory, context, and audit trails separate. |
| MCP servers pointed at their own stack | Wazuh/SIEM, cloud config, evidence store, ticketing. The client configures these. |
| A named human approver | Every write action lands pending-approval. Skills draft; humans approve. |
| A local scratch store | The evidence ledger and verdict memory are files on the client's disk, not a service. |

---

## Operating doctrine

Every skill in every pack is bound by [`DOCTRINE.md`](DOCTRINE.md). The short version:

1. **Local first.** Deterministic filtering, correlation, and dedup happen before a
   single token goes to the model. The model reasons over the reduced set, never the raw feed.
2. **Tokenise before you reason.** Sensitive values are swapped for stand-ins on the way
   in and restored on the way out. The model sees shapes, not secrets.
3. **Reachability, not volume.** A finding without a confirmed path is not a finding.
   Report what an attacker can actually reach and what an auditor can actually fail you on.
4. **Evidence, not assumption.** Every determination records its inputs, its method, and
   its conclusion, versioned and hash-chained, so it survives review months later.
5. **Skills draft. Humans approve.** No containment, no published policy, no state
   mutation without an explicit signed human decision.
6. **Handoff with backbone.** The end state is the client's team running this without
   AccuSights in the loop — AI-assisted operators, under human control.

---

## Delivery model

Aligned to CodeBridge: **one-off delivery cost, not perpetual subscription fees.**

| | |
|---|---|
| Scoping to handover | 2–6 weeks |
| Deliverable | Installed packs, configured MCP surface, tuned control map, trained operators |
| Post-handover | 30 days of monitoring included |
| Ongoing | Client-run. Optional retainer for new-threat skill updates. |

The recurring revenue is not hosting. It is the **skill updates** — new threat
coverage, new framework versions, new regulator guidance — shipped as marketplace
updates the client pulls.

---

## Repo conventions

- One skill = one directory = one `SKILL.md` with YAML frontmatter (`name`, `description`).
- Reference data (control catalogs, policy packs, framework maps, rails) lives in
  `references/` beside the skill, never inline in the instructions.
- Every skill states up top: required MCP tools, read-only vs gated-write, and what it refuses to do.
- No secrets, no tenant identifiers, no client data ever enters this repo.
