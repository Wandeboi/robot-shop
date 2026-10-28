# Robot Shop: Availability SLO and Error Budget Policy

**Service:** payment (checkout) | **Owner:** Wande | **Review:** after every incident

## 1. SLI (what we measure)
The share of time the payment service has at least one ready pod.
Measured from Kubernetes readiness data:

    avg_over_time((max(kube_pod_status_ready{namespace="robot-shop",condition="true",pod=~"payment-.*"}) or vector(0))[7d:1m])

## 2. SLO (the target)
**99.5% availability** of the payment service over a rolling 30-day window.
Supporting target: web frontend 99.0%.

## 3. Error budget
The budget is the failure the SLO allows: 0.5% of 30 days.

| Window | Allowed downtime at 99.5% |
|---|---|
| 30 days | 216 minutes (3.6 hours) |
| 7 days | 50 minutes |
| 1 day | 7 minutes |

For the web frontend at 99.0%: 432 minutes per 30 days.
Note: Prometheus keeps about 10 days of data by default, so in this project the budget is checked over 7 days.

## 4. Alerting
`RobotShopPaymentAvailabilitySLOBreached` fires when no payment pod has been ready for 30 seconds
and posts to Slack with a link to the runbook (RUNBOOK.md).
`RobotShopHighCrashLoopRate` warns when a pod restarts more than twice in 10 minutes.

## 5. Error budget policy
| Budget used | What we do |
|---|---|
| Under 50% | Normal work. Releases go ahead. |
| 50% to 90% | Review recent incidents. Extra care on releases to payment. Fix the top cause of downtime. |
| Over 90% | Freeze non-urgent releases. Work only on reliability until the budget recovers. |
| 100% (SLO missed) | Full release freeze for payment. Write a blameless postmortem. Agree fixes before releasing again. |

Exceptions: planned maintenance announced in advance does not count against the budget.

## 6. Incident response
Mitigate first (restart, scale, or `helm rollback`), then find the root cause, then write the
postmortem. See RUNBOOK.md for the steps for each alert.
