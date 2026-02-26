# Code Map

## Start here by goal

| Goal | Files in order | What to look for |
|---|---|---|
| Understand Argo bootstrap | `bootstrap/kustomization.yaml` -> `bootstrap/demo-application.yaml` | app registration and source path |
| Change shared app topology | `apps/demo/base/kustomization.yaml` -> `deployment-*.yaml` -> `service-*.yaml` -> `ingress.yaml` | core workload model |
| Promote stage | `apps/demo/overlays/stage/kustomization.yaml` | `newTag` and host patch |
| Debug config-only change | `apps/demo/base/configmap.yaml` | values mounted/exposed by workloads |
| Validate env namespace split | `apps/demo/overlays/local/namespace.yaml` and `apps/demo/overlays/stage/namespace.yaml` | env isolation |

## Stage -> module map

| Stage | Primary files | Artifact | Debug signal |
|---|---|---|---|
| Define desired state | `apps/demo/base/*` | rendered manifests | `kubectl kustomize ...` succeeds |
| Env specialization | `apps/demo/overlays/*/kustomization.yaml` | env-specific manifest output | namespace/host/tag differences |
| Argo binding | `bootstrap/*.yaml` | Argo Application CRs | `kubectl -n argocd get app` |
| Runtime effect | N/A (cluster) | rollout state | `kubectl -n <ns> get deploy,pods` |
