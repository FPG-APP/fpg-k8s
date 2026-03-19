# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Kubernetes manifests for the FPG (Fantasy Premier League prediction game) infrastructure. Manages deployments across two FPG environments (prod/testing) plus two unrelated apps (house-finance, ghost blog).

## Common Commands

```bash
# Restart deployments after image push
kubectl rollout restart deployment fpg-api -n testing
kubectl rollout restart deployment fpg-api -n prod

# Debug pods
kubectl get pods -n testing
kubectl logs <podname> -n testing
kubectl describe pod <podname> -n testing

# Apply manifests
kubectl apply -f fpg-api/prod/
kubectl apply -f fpg-api/testing/

# Apply secrets (not tracked in git)
kubectl apply -f fpg-api/prod/prod-secrets.yaml
kubectl apply -f fpg-api/testing/testing-secrets.yaml
```

## Structure & Namespaces

| Namespace | Directory | Purpose |
|-----------|-----------|---------|
| `prod` | `fpg-api/prod/` | Production FPG API + engine cron |
| `testing` | `fpg-api/testing/` | Testing FPG API + engine cron |
| `house` | `house-finance/house/` | Separate Streamlit finance app |
| `ghost` | `ghost/` | Ghost blog platform |
| `monitoring` | `monitoring/` | Placeholder only |

## Key Architecture Details

**FPG API** (`tobygaskell/fpg:latest` / `:testing`): FastAPI app, 2 replicas, port 8000. Testing environment pins to `fpg-worker` node with `imagePullPolicy: Always`.

**FPG Engine**: Runs as a `CronJob` (not a long-running deployment). Prod runs at `:30 * * * *`, testing at `:17 * * * *`. Args control season/round/mode — the testing manifest has commented-out args for manual run configurations.

**Ingress**: All apps use nginx ingress with Let's Encrypt TLS via the `letsencrypt-prod` ClusterIssuer (defined in `issuer.yaml`).

**Secrets**: Secret files are gitignored. `db-secrets` holds MySQL credentials; `engine-secrets` additionally holds the external football API key and FPG API host/key.

## Environments

- Production image: `tobygaskell/fpg:latest`, pull policy `IfNotPresent`
- Testing image: `tobygaskell/fpg:testing`, pull policy `Always`
- Engine follows the same pattern: `tobygaskell/fpg-engine:latest` vs `:testing`
