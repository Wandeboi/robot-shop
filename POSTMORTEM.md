# Postmortem: <short title>

**Date:** <date> | **Author:** Wande | **Status:** Draft
**Severity:** <Critical / Major / Minor> | **Duration:** <start to end>

## 1. Summary
<2-3 sentences: what broke, who was affected, how long, how it was fixed>

## 2. Impact
- <what users could not do>
- SLO affected: <which>, error budget used: <how much> (see SLO.md)

## 3. Timeline (one time zone)
| Time | Event |
|---|---|
| <hh:mm> | Incident started |
| <hh:mm> | Alert fired in Slack |
| <hh:mm> | First action taken |
| <hh:mm> | Mitigation applied |
| <hh:mm> | Service recovered |

## 4. Detection
<which alert fired, how long after the failure, did it point to the right place>

## 5. Mitigation
<what stopped the damage first, before the cause was known>

## 6. Root cause
- Why was the service failing? <answer>
- Why did that happen? <answer>
- Why was that possible? <the underlying cause>

## 7. What went well / badly / where we got lucky
- Went well: <...>
- Went badly: <...>
- Lucky: <...>

## 8. Action items
| Action | Type (prevent / detect / mitigate) | Owner | Due |
|---|---|---|---|
| <...> | <...> | Wande | <date> |

## 9. Evidence
<screenshots or links: Slack alert, Grafana, Loki search, Jaeger trace, commands run>
