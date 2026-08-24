# Cadence Platform

Deployment settings for the Cadence app.

## Deployment

### Prerequisites

- `k3s` on target VM
- `kubectl`
- `kustomize` and `ksops`

### Applying prod manifests

```shell
kustomize build --enable-alpha-plugins --enable-exec k8s/prod | kubectl apply -f -
```
