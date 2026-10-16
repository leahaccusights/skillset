---
name: reachability-filter
description: Separate exploitable findings with a confirmed attack path from theoretical noise, and show the path end to end. Use on vulnerability scan output, alert backlogs, pen-test findings, or when the user says "which of these actually matter", "too many findings", "is this exploitable", "what can an attacker actually reach". Read-only plus evidence write.
---

# reachability-filter

Licensed AccuSights skill. The instruction set is served by the AccuSights Control
Center and is not distributed with this package.

## Load

```bash
accusights-fetch "accusights-soc/reachability-filter"
```

`accusights-fetch` is installed by the AccuSights installer. It reads the entitlement
from the OS secret store and calls:

```
GET https://control.accusights.com/v1/skills/accusights-soc/reachability-filter/SKILL.md
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

Say: **"AccuSights skill `reachability-filter` is unavailable - <state>."** Then stop.

Evidence export and license status are never gated. A client can always retrieve their
own data.
