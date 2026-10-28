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
