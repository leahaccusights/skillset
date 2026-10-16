# Operating Doctrine

Every skill in every AccuSights pack is bound by these six rules. A skill that
breaks one of them is a bug, not a feature. When a skill's own instructions and this
document disagree, this document wins.

The doctrine is adapted from the CodeBridge delivery model — the same principles that
govern reachability security testing, compliance data tracing, and the Guardee privacy
layer — applied to continuous security operations and compliance monitoring.

---

## 1. Local first — do the heavy lifting before any AI involvement

The model is the most expensive, least deterministic, and least auditable component in
the pipeline. It goes last.

**Order of operations, always:**

```
raw feed
  -> deterministic filter   (rules, signatures, allowlists, dedup)
  -> correlation            (indexed lookup, not pairwise scan)
  -> reduction              (the set that survives is usually 1-5% of the input)
  -> [ model reasons here ]
  -> determination + evidence
```

**Why this is non-negotiable:**

- A 24h Wazuh window is 600–3000+ alerts. Sending all of them to a model is not
  analysis, it is a bill. Deterministic reduction gets that to a handful.
- Comparing every incoming alert against all stored alerts is O(N^2). It does not scale
  as history grows. Use an indexed signature lookup.
- Anything the model does not see cannot be leaked, hallucinated about, or mis-summarised.

**In practice:** if a skill's first action is "send the data to Claude and ask," it is
written wrong. Find the deterministic step that should have come first.

---

## 2. Tokenise before you reason

Sensitive values are replaced with stable stand-ins on the way into the model and the
real values are restored on the way out. The model reasons over *shapes and relationships*,
not over secrets.

This is the `data-boundary` skill in `accusights-core`. Every other skill that passes
client data to the model routes through it. No exceptions.

**What gets tokenised:** personal identifiers, credentials and key material, internal
hostnames and IPs where the client marks them sensitive, customer records, health and
financial data, contract terms.

**What does not:** severity, timestamps, control IDs, framework names, rule IDs, counts,
verdict labels. The model needs these and they carry no exposure.

**Why:** the client's tenancy boundary is only real if it holds at the model call. A
skill that ships raw PII into a prompt has moved the data out of the client's control
regardless of where the process is running.

---

## 3. Reachability, not volume

A finding without a confirmed path is not a finding. Traditional tooling lists every
possible weakness; that produces noise, and noise trains operators to ignore the tool.

**Applied to SOC:** an alert is actionable only when there is a traced path from an
entry point to something that matters. No path, no page. Suppress with a recorded reason,
never silently.

**Applied to compliance:** a control is failing only when the telemetry that evidences it
is absent or stale. A control nobody can produce evidence for is a finding; a control
that is merely undocumented in a spreadsheet is not.

**Applied to both:** report what an attacker can actually reach and what an auditor can
actually fail you on. Everything else goes in the appendix, not the alert.

**Compound risk is the exception to "low severity is not urgent."** Individually benign
findings that combine into a real path are reported at the severity of the combination,
not of the parts. See the `compound-risk` skill.

---

## 4. Evidence, not assumption

Every determination records:

| Field | Meaning |
|---|---|
| `inputs` | Exactly what was read, from which source, at what time |
| `method` | Which rule, model, prompt version, or correlation produced the result |
| `determination` | The conclusion, in unambiguous terms |
| `confidence` | And what would change it |
| `prev_hash` / `hash` | Position in the chain |

Records are **versioned** and **hash-chained**, so a determination made today survives
review in eighteen months and a tampered chain is detectable. The chain is independent
of the operational audit log — an attacker who compromises one has not compromised both.

**The test:** if a regulator, auditor, or underwriter asks "how do you know that?" six
months from now, the answer is a record, not a recollection. If a skill cannot produce
that record, it must not produce the finding.

---

## 5. Skills draft. Humans approve.

No skill executes a containment action, publishes a policy, closes a ticket, or mutates
client state on its own authority. Everything lands as **pending-approval**, attributed
to a named human on the way out.

**The three response tiers** (client-configured, per action class):

- `report-only` — the skill describes what it would do. Nothing happens.
- `approve-to-block` — the skill proposes; a signed human approval executes it.
- `auto-block` — pre-authorised for a narrow, explicitly enumerated action set only.

Approvals are cryptographically signed and rate-limited. A forged or replayed approval is
recorded as a security event in its own right.

**Why this is the liability boundary:** the client configures the tiers, the client names
the approvers, the client's audit chain records the decisions. If a client widens
`auto-block`, weakens a rail, or skips a review, that is a decision made in their tenancy
and the record shows it as theirs.

---

## 6. Handoff with backbone

The end state is the client's own team operating this without AccuSights in the loop.

- Every skill explains its reasoning in terms an operator can check, not just accept.
- Every threshold, rail, tier, and cadence is a client-editable config value, not a constant
  buried in a prompt.
- Every skill degrades honestly: missing tool, missing data, or low confidence produces a
  stated gap, never a confident guess.
- Documentation targets the client's operators, not AccuSights staff.

Designed for **AI-assisted operators, under human control** — not for autonomous operation
the client cannot inspect.

---

## Refusals — common to every skill

No skill may:

- Send un-tokenised sensitive client data to any model.
- Execute a write action without an approval that satisfies the configured tier.
- Report a finding it cannot evidence.
- Suppress a finding without recording the suppression and its reason.
- Transmit client data outside the client's environment for any purpose, including
  telemetry, quality improvement, or AccuSights support.
- Touch cloud security groups, install packages, write to production databases,
  force-push, merge, or read secrets. These are structurally excluded from the tool
  surface, not merely discouraged.

A skill that hits one of these stops and reports the block. It does not route around it.
