---
name: compound-risk
description: Find where individually low-severity findings combine into a real attack path, and report the combination at the severity of the whole. Use after triage or reachability filtering, on backlogs of accepted low findings, or when the user says "these are all lows", "we accepted that risk", "how could someone chain this", "is this campaign related". Read-only plus evidence write.
---

# compound-risk

Licensed AccuSights skill. The instruction set is served by the AccuSights Control
Center and is not distributed with this package.

## Load

```bash
accusights-fetch "accusights-soc/compound-risk"
```

`accusights-fetch` is installed by the AccuSights installer. It reads the entitlement
from the OS secret store and calls:

```
GET https://control.accusights.com/v1/skills/accusights-soc/compound-risk/SKILL.md
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
| `plugin_not_licensed` | Key does not cover `accusights-soc` | Contact AccuSights - this is a contract question |
| `grace` | Control center unreachable, cached entitlement in use | None. Operations continue. Escalate if it persists past a few days. |
| `expired` | Grace window elapsed | Renew. Evidence export and license status keep working. |
| `revoked` | License withdrawn | Contact AccuSights |

Say: **"AccuSights skill `compound-risk` is unavailable - <state>."** Then stop.

Evidence export and license status are never gated. A client can always retrieve their
own data.
