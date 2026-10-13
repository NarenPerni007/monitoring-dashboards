# Disk pressure

## Symptoms
- `DiskSpaceLow` (warning, predicted full in ~4h) or `DiskSpaceCritical` alert.
- Kubelet starts evicting pods with `DiskPressure` taint.

## Quick checks
```bash
# what's eating the disk
df -h /
du -sh /* 2>/dev/null | sort -rh | head

# docker/containerd leftovers (usual suspect)
docker system df
crictl images | head

# big logs
du -sh /var/log/* 2>/dev/null | sort -rh | head
journalctl --disk-usage
```

## Common fixes
1. **Containerd/docker bloat** - `docker system prune -a` or `crictl rmi --prune`. Old images pile up on long-lived nodes.
2. **Log rotation missing** - app writing GBs to stdout with no rotation, or journald unbounded. Set `SystemMaxUse` in journald.conf.
3. **EmptyDir abuse** - pod writing scratch data to emptyDir on the node disk. Move to a real volume.
4. **etcd growth** (control plane) - compact/defrag if `/var/lib/etcd` is the culprit.

After cleanup, confirm the node recovers: `kubectl describe node <node> | grep -A3 Pressure`. If kubelet already evicted pods, they reschedule on their own once pressure clears.
