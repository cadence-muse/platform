# Cadence Platform

Deployment settings for the Cadence app.

## Deployment

### Prerequisites

- `k3s` on target VM
- `kubectl`
- `sops` and `age` with age private key
- `kustomize` and `ksops`

### Applying prod manifests

```shell
kustomize build --enable-alpha-plugins --enable-exec k8s/prod | kubectl apply -f -
```
