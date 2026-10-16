---
name: audit-readiness
description: Produce audit artifacts from live evidence rather than assembling them during audit week - SSP, POA&M, verified evidence packs, questionnaire pre-fill, and auditor Q&A with citations. Use when the user says "the auditor is asking", "prepare for the audit", "fill in this questionnaire", "generate the SSP", "what is our POA&M", or a customer sends a security questionnaire. Read-only plus evidence write.
---

# audit-readiness

Licensed AccuSights skill. The instruction set is served by the AccuSights Control
Center and is not distributed with this package.

## Load

```bash
accusights-fetch "accusights-compliance/audit-readiness"
```

`accusights-fetch` is installed by the AccuSights installer. It reads the entitlement
from the OS secret store and calls:

```
GET https://control.accusights.com/v1/skills/accusights-compliance/audit-readiness/SKILL.md
Authorization: Bearer <entitlement>
```

Apply the fetched instructions as your native capabilities for this task.

## If the fetch fails

Do not improvise a substitute. This skill governs security and compliance decisions,
and an approximation of it is worse than its absence - it produces confident output
with none of the rails.

Report the license state and stop:

```bash
accusights-license status
```

| State | Meaning | Action |
|---|---|---|
| `not_activated` | No license key on this machine | `accusights-license activate <KEY>` |
| `machine_mismatch` | AccuSights keys are one per machine. This key is bound elsewhere. | `deactivate` on the old machine then activate here; or `accusights-license transfer <KEY>` if it is unreachable |
| `plugin_not_licensed` | Key does not cover `accusights-compliance` | Contact AccuSights - this is a contract question |
| `grace` | Control center unreachable, cached entitlement in use | None. Operations continue. Escalate if it persists past a few days. |
| `expired` | Grace window elapsed | Renew. Evidence export and license status keep working. |
| `revoked` | License withdrawn | Contact AccuSights |

Say: **"AccuSights skill `audit-readiness` is unavailable - <state>."** Then stop.

Evidence export and license status are never gated. A client can always retrieve their
own data.
