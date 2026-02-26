# gitops_infra

Declarative infrastructure repository for the local GitOps workspace.  
Argo CD watches this repo and reconciles Kubernetes manifests from environment overlays.

## Why this repo exists

- Keep deployment state separate from app source.
- Make promotions explicit and auditable via Git commits.

## What you will do here

- Manage Kustomize base + overlays (`local`, `stage`).
- Update image tags for promotion.
- Control ingress hosts, resources, and config.

## Quickstart

### Requirements

- `kubectl` with access to the local kind cluster
- Argo CD installed in cluster (via `platform` repo)
- Optional: `kustomize` (or use `kubectl kustomize`)

### Validate Kustomize render

```bash
cd repos/infra
kubectl kustomize apps/demo/overlays/local > /tmp/demo-local.yaml
kubectl kustomize apps/demo/overlays/stage > /tmp/demo-stage.yaml
```

Expected output shape:
- both commands render YAML with Deployments, Services, Ingress, ConfigMap, Secret

### Promotion by tag change (stage example)

```bash
cd repos/infra
# edit apps/demo/overlays/stage/kustomization.yaml newTag values
git add apps/demo/overlays/stage/kustomization.yaml
git commit -m "chore(stage): promote demo tag"
git push origin main
```

## Mental model

```text
infra commit
  -> Argo detects new revision
  -> compares desired vs live state
  -> applies changed manifests
  -> Kubernetes rolls workloads (if pod template changed)
```

## Artifact locations

- Argo app bootstrap manifests: `bootstrap/`
- Demo app base manifests: `apps/demo/base/`
- Local overlay: `apps/demo/overlays/local/`
- Stage overlay: `apps/demo/overlays/stage/`
- Flow runbook: `runbooks/gitops-flows.md`

## Sanity checks

```bash
kubectl -n argocd get app demo-local demo-stage
kubectl -n demo get deploy,svc,ingress
kubectl -n demo-stage get deploy,svc,ingress
```

Expected output shape:
- Argo apps `Healthy` and `Synced`
- resources present in both namespaces

## Troubleshooting (fast)

- Argo app out of sync for long time:
  - `kubectl -n argocd annotate app demo-local argocd.argoproj.io/refresh=hard --overwrite`
- Comparison error on tag values:
  - ensure `newTag` values are quoted strings in overlay YAML.
- Ingress 404:
  - verify host/path in overlay patch and ingress-nginx availability.
- Image pull back-off:
  - verify registry tag exists for both images.

## Docs map

- System contract: `docs/architecture.md`
- ADRs: `docs/adr`
- Code navigation: `docs/code-map.md`
- Glossary: `docs/glossary.md`
- Learning checks: `docs/learning-checkpoints.md`
