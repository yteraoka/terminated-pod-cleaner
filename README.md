# terminated-pod-cleaner

Kubernetes の version によって Graceful Node Shutdown で停止させられた Pod の status は異なるため
どちらもカバーできる条件を指定する

```
kubectl get pods -o json \
  | jq -r '.items[]
           | select(.status.phase == "Failed")
           | select(.status.reason == "Shutdown" or .status.reason == "NodeShutdown" or .status.reason == "Terminated") 
           | .metadata.name'
```

## Install

```
helm upgrade terminated-pod-cleaner ./chart --install
```

## analysis

### kube-score

https://kube-score.com/

```
helm template terminated-pod-cleaner ./chart | kube-score score -
```

### kube-linter

https://docs.kubelinter.io/

```
helm template terminated-pod-cleaner ./chart | kube-linter lint -
```

### trivy

https://aquasecurity.github.io/trivy/

```
trivy config chart
```

## Kubernetes の version による status の違い

Kubernetes 1.20

```yaml
status:
  message: Node is shutting, evicting pods
  phase: Failed
  reason: Shutdown
```

```yaml
status:
  message: Pod Node is in progress of shutting down, not admitting any new pods
  phase: Failed
  reason: Shutdown
```

Kubernetes >= 1.21

https://github.com/kubernetes/kubernetes/commit/f1de598233e3f725016c820e2bf8b7ed78005705

```yaml
status:
  message: Pod Pod was rejected as the node is shutting down.
  phase: Failed
  reason: NodeShutdown
```

```yaml
status:
  message: Pod was terminated in response to imminent node shutdown.
  phase: Failed
  reason: Terminated
```
