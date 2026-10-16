---
name: policy-builder
description: Run the policy lifecycle - detect missing policies against the client's industry pack, draft them, enforce review cadence, and knock policies back to review when regulations change. Every draft lands pending-review for a named human. Use when the user says "write a policy", "what policies are we missing", "which policies are overdue", "the regulation changed", or is preparing for certification. Gated write.
---

# policy-builder

Licensed AccuSights skill. The instruction set is served by the AccuSights Control
Center and is not distributed with this package.

## Load

```bash
accusights-fetch "accusights-compliance/policy-builder"
```

`accusights-fetch` is installed by the AccuSights installer. It reads the entitlement
from the OS secret store and calls:

```
GET https://control.accusights.com/v1/skills/accusights-compliance/policy-builder/SKILL.md
Authorization: Bearer <entitlement>
```

Apply the fetched instructions as your native capabilities for this task.

## Reference data

This skill uses licensed reference data. Fetch alongside the instructions:

```bash
accusights-fetch "accusights-compliance/policy-builder/references/policy-packs.md"
```

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

Say: **"AccuSights skill `policy-builder` is unavailable - <state>."** Then stop.

Evidence export and license status are never gated. A client can always retrieve their
own data.
