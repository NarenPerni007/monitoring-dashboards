# Pod crash looping

## Symptoms
- `PodCrashLooping` alert fires, or `kubectl get pods` shows `CrashLoopBackOff`.
- Restarts counter climbing on the restarts-by-namespace panel.

## Quick checks
```bash
# which container and why
kubectl describe pod <pod> -n <namespace>

# last logs before the crash
kubectl logs <pod> -n <namespace> --previous

# recent events
kubectl get events -n <namespace> --sort-by=.lastTimestamp | tail -20
```

## Common fixes
1. **Bad image / config** — most common. Check the image tag actually exists and env vars / configmaps the app expects are present. `kubectl describe` shows `ImagePullBackOff` vs crash separately; don't mix them up.
2. **OOMKilled** — `kubectl describe pod` shows exit code 137. Bump `resources.limits.memory`, or fix the leak if it keeps climbing.
3. **Failing probe** — readiness/liveness probe misconfigured (wrong port/path). The app is fine; the probe is wrong. Check `kubectl describe pod` events for "probe failed".
4. **Crash on startup** — missing secret or DB connection. Logs `--previous` usually say exactly what.

If it started right after a deploy, `kubectl rollout undo deploy/<name> -n <namespace>` buys time while you figure it out.
