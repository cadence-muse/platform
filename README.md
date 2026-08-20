# Cadence Platform

Deployment settings for the [Cadence app](https://github.com/nightnoryu/cadence-backend).

## Layout

- `k8s/base` — shared manifests: backend, frontend, Postgres, Redis.
- `k8s/production` — production overlay, namespace `cadence`, host `cadence.app` (routes `/api` to the backend,
  `/` to the frontend). Uses Traefik (`ingressClassName: traefik`) with the `myresolver` cert resolver for TLS —
  no cert-manager involved.
- `k8s/logging` — standalone logging stack, namespace `logging`: Loki (log storage), Promtail (DaemonSet, tails
  pod logs cluster-wide), Grafana (view/query logs, Loki datasource auto-provisioned). Not part of the `cadence`
  overlay — applied separately. Grafana is `ClusterIP`-only, reachable via port-forward.

Secrets are committed as `secret.enc.yaml` files encrypted with [sops](https://github.com/getsops/sops) (age). They are
**not** wired into kustomize — decrypt and apply them manually before applying the overlay.

## Prerequisites

- `k3s` on target machine with certResolvers set up.
- `kubectl` pointed at the target cluster.
- `kustomize` (or `kubectl kustomize`).
- `sops` with the repo's age private key available, either exported as `SOPS_AGE_KEY_FILE` or placed at
  `~/.config/sops/age/keys.txt`.

## Deployment

### Production

```bash
sops -d k8s/production/secret.enc.yaml | kubectl apply -f -
kubectl delete job -n cadence cadence-migrate --ignore-not-found
kubectl kustomize k8s/production | kubectl apply -f - -l app=cadence-migrate
kubectl wait --for=condition=complete job/cadence-migrate -n cadence --timeout=180s
kubectl apply -k k8s/production
```

The `cadence-migrate` Job runs the backend image with `migrate` and must exit `0` before the rest of the
overlay (including the `cadence` Deployment) is applied — `kubectl wait` blocks on it, and the sequence must
stop if any step fails.

### Logging

```bash
sops -d k8s/logging/secret.enc.yaml | kubectl apply -f -
kubectl apply -k k8s/logging
```

View logs:

```bash
kubectl port-forward -n logging svc/grafana 3000:3000
```

Open `localhost:3000`, log in `admin` / the decrypted `GF_SECURITY_ADMIN_PASSWORD`, and query via Explore
(datasource `Loki` is preconfigured), e.g. `{namespace="cadence"}`.

## Editing secrets

```bash
sops k8s/production/secret.enc.yaml
```

Opens the decrypted secret in `$EDITOR` and re-encrypts on save. Never commit a decrypted secret file.
