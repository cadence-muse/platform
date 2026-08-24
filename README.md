# Cadence Platform

Deployment settings for the Cadence app.

## Prerequisites

- `k3s` on target VM.
- `kubectl` and `kustomize`.
- `sops` and `age` with the repo's age private key.

## Deployment

### prod

```shell
kubectl apply -f k8s/base/namespace.yaml
sops -d k8s/prod/secret.enc.yaml | kubectl apply -f -
kubectl apply -k k8s/prod
```
