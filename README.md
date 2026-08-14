# Cadence Platform

Deployment settings for the [Cadence app](https://github.com/nightnoryu/cadence-backend).

## Layout

- `k8s/base` — shared manifests: backend, frontend, Postgres, Redis.
- `k8s/cluster` — cluster-scoped resources (cert-manager `ClusterIssuer`), applied once per cluster.
- `k8s/staging` — staging overlay, namespace `cadence-staging`, hosts `api.beta.cadence.app` / `beta.cadence.app`.
- `k8s/production` — production overlay, namespace `cadence`, hosts `api.cadence.app` / `cadence.app`.

Secrets are committed as `secret.enc.yaml` files encrypted with [sops](https://github.com/getsops/sops) (age). They are
**not** wired into kustomize — decrypt and apply them manually before applying the overlay.

## Prerequisites

- `k3s` on target machine with certResolvers set up.
- `kubectl` pointed at the target cluster.
- `kustomize` (or `kubectl kustomize`).
- `sops` with the repo's age private key available, either exported as `SOPS_AGE_KEY_FILE` or placed at
  `~/.config/sops/age/keys.txt`.

## Deployment

### Staging

```bash
sops -d k8s/staging/secret.enc.yaml | kubectl apply -f -
kubectl apply -k k8s/staging
```

### Production

```bash
sops -d k8s/production/secret.enc.yaml | kubectl apply -f -
kubectl apply -k k8s/production
```

## Editing secrets

```bash
sops k8s/production/secret.enc.yaml   # or k8s/staging/secret.enc.yaml
```

Opens the decrypted secret in `$EDITOR` and re-encrypts on save. Never commit a decrypted secret file.
