# kyber-lonestar-deploy

Deployment configuration for [Kyber](https://github.com/matty-v/kyber) on the
**kyber-lonestar** cluster — an on-prem, single-node k3s install running on a
Proxmox VM. Public PWA served via Cloudflare Tunnel at
`https://kyber-lonestar.voget.io`.

This is a standalone deploy repo (separate from `matty-v/kyber-deploy`, which
tracks Matt's own fleet). ArgoCD on the box assembles two sources at sync time:
the Helm chart from `matty-v/kyber` (chart tracks `main`) + the per-cluster
values here (`environments/lonestar/values.yaml`).

## Version tracking

Images are **pinned to a Kyber release tag** (currently `v2.5.0`) in
`environments/lonestar/values.yaml` — the stable on-prem model, not the
head-of-main canary. Advancing to a new release is a value-bump commit here
(manual until the release pipeline is extended to open bump-PRs against this
repo).

## Layout

```
environments/lonestar/
  application.yaml   # ArgoCD Application (multi-source: chart + these values)
  values.yaml        # per-cluster Helm values
```

## Cluster facts

| | |
|---|---|
| Host | Proxmox VM `kyber-lonestar` (VMID 200), Ubuntu 24.04, 4 vCPU / 16 GB / 200 GB |
| Kubernetes | k3s (single-server), pod CIDR `10.42.0.0/16` |
| Node InternalIP | `192.168.176.244` |
| Public URL | `https://kyber-lonestar.voget.io` (Cloudflare Tunnel) |
| Compute provider | `mock` (single node; no cloud Machine Controller) |

## Hand-provisioned secrets (not chart-rendered, live in `kyber-system`)

Created out-of-band via `kubectl`; survive app teardown (`Prune=false`):

- `kyber-api-credentials` — `api-key`, `webhook-secret`, `k3s-join-token`, `k3s-server-url`
- `kyber-postgres-credentials` — `postgres-password`
- `kyber-internal-signing-key` — `signing-key` (per-agent :8082 authz)
- `kyber-cloudflared-creds` — `credentials.json` (added at the tunnel step)
