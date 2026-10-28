# Robot Shop Runbook

## RobotShopPaymentAvailabilitySLOBreached
**Meaning:** the payment pod is not ready or does not exist. Checkout fails.
**Check:**
- `kubectl get pods -n robot-shop -l service=payment`
- `kubectl describe pod -n robot-shop -l service=payment`
- `kubectl logs -n robot-shop deploy/payment --tail=50`
**Mitigate:** `kubectl rollout restart deploy/payment -n robot-shop`; if it was scaled to zero, `kubectl scale deploy/payment -n robot-shop --replicas=1`. After a bad release, `helm rollback robot-shop -n robot-shop`.
**Escalate:** if not recovered in 15 minutes.

## RobotShopWebAvailabilitySLOBreached
**Meaning:** the web frontend is not ready or does not exist. The shop is unreachable.
**Check:**
- `kubectl get pods -n robot-shop -l service=web`
- `kubectl logs -n robot-shop deploy/web --tail=50`
**Mitigate:** `kubectl rollout restart deploy/web -n robot-shop`, or `kubectl scale deploy/web -n robot-shop --replicas=1`.

## RobotShopHighCrashLoopRate
**Meaning:** a pod restarted more than twice in 10 minutes.
**Check:**
- `kubectl get pods -n robot-shop`
- `kubectl describe pod <pod> -n robot-shop` (look at Last State and Events)
- `kubectl logs <pod> -n robot-shop --previous`
**Mitigate:** fix the cause. OOMKilled means raise the memory limit; a bad config or release means roll back.

## Day 5 Incident Drill: Shipping Readiness Probe

**Incident date:** 28 October 2026  
**Service:** shipping  
**Namespace:** robot-shop  
**Status:** Intermittent issue observed; currently not reproduced during the latest checks.

### Symptoms
The shipping pod intermittently failed its Kubernetes readiness probe:
`GET http://<pod-ip>:8080/health` timed out while awaiting response headers.

### Investigation
- Confirmed the shipping container was Running and Ready during inspection.
- Container restart count was zero.
- Manual health checks returned HTTP 200, although some earlier responses took approximately one second.
- Readiness probe configuration: 10-second initial delay, 5-second period, 1-second timeout, and failure threshold of 30.
- Prometheus CPU throttled-period measurements were 6.42% and later 14.33%.
- CPU usage in a sampled five-minute window was approximately 78 millicores.
- The latest 15-minute readiness-failure query returned zero.
- No Kubernetes events were present during the latest inspection.

### Findings
The intermittent readiness timeout is confirmed by the observed probe errors. CPU throttling may contribute to response delays, but available evidence does not establish it as the root cause. Application logs showed normal shipping calculations and cart confirmations during the sampled period.

### Mitigation and Recovery
No deployment changes or pod restarts were performed during this investigation. The service was Ready during the latest inspection. Continue monitoring readiness failures, probe duration, CPU throttling, and application logs. If failures recur, correlate timestamps across these signals before changing CPU limits or readiness-probe settings.

### Follow-up Actions
1. Monitor the shipping readiness-probe failure count.
2. Investigate CPU throttling alongside CPU usage and application latency.
3. If failures recur, evaluate a controlled increase of the readiness timeout and validate the result.
4. Confirm service availability against the 99% availability SLO.
5. Record the verified cause and outcome after sufficient monitoring.

### Blameless Review
The incident exposed a potential gap between the one-second readiness timeout and intermittent service response latency. Further evidence is required to identify the underlying cause. The follow-up focuses on improving observability and validating probe settings rather than attributing blame.
