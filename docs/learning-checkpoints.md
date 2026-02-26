# Learning Checkpoints

| Checkpoint | Commands | Success looks like | If it fails |
|---|---|---|---|
| Render local overlay | `cd repos/infra && kubectl kustomize apps/demo/overlays/local >/tmp/local.yaml` | command exits 0, YAML file produced | malformed kustomization or patch |
| Render stage overlay | `cd repos/infra && kubectl kustomize apps/demo/overlays/stage >/tmp/stage.yaml` | command exits 0 | fix overlay patch structure |
| Argo app healthy | `kubectl -n argocd get app demo-local demo-stage` | `Healthy` and `Synced` | inspect app events and source revision |
| Dev resources live | `kubectl -n demo get deploy,svc,ingress` | expected resources listed | app path or namespace mismatch |
| Stage resources live | `kubectl -n demo-stage get deploy,svc,ingress` | expected resources listed | stage app missing or unsynced |
