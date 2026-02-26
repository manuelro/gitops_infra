# gitops_infra Architecture (System contract)

## What it is

`gitops_infra` is the desired-state repository consumed by Argo CD for local and stage environments.

## Goals

- Define Kubernetes state declaratively.
- Separate shared base from env-specific overlays.
- Support predictable promotion by image tag update.

## Non-goals (for now)

- Helm-based packaging.
- Multi-cluster fleet management.
- External secrets manager integration.

## Pipeline

```text
infra commit
  -> Argo Application source revision updates
  -> Argo sync (automated)
  -> Kubernetes apply/patch resources
  -> rollout/health checks in target namespace
```

## Artifacts and locations

| Artifact | Path | Produced by | Used by | Why it matters |
|---|---|---|---|---|
| Dev Argo app | `bootstrap/demo-application.yaml` | infra authors | Argo CD | binds Argo to local overlay |
| Stage Argo app | `bootstrap/demo-stage-application.yaml` | infra authors | Argo CD | binds Argo to stage overlay |
| Shared demo manifests | `apps/demo/base/*` | infra authors | overlays | single source for service topology |
| Local overlay | `apps/demo/overlays/local/kustomization.yaml` | infra authors/automation | Argo `demo-local` | current dev desired image tags/host |
| Stage overlay | `apps/demo/overlays/stage/kustomization.yaml` | infra authors | Argo `demo-stage` | controlled promotion target |

## Key invariants

| Invariant | Why true | What to do if broken |
|---|---|---|
| Argo source repo for app deploy is infra | bootstrap applications point to infra repo | inspect `spec.source` in Argo apps |
| `newTag` values are strings | Kustomize unmarshalling expectations | quote tags in overlays |
| Local and stage use separate namespaces | overlay `namespace` differs | check overlay namespace and Argo destination |
| Stage host differs from local | stage patch sets `stage.localtest.me` | patch ingress host in stage overlay |

## Common failure modes

| Symptom | Likely cause | Fast check | Fix |
|---|---|---|---|
| Argo app Healthy but old version | infra commit not pushed | `git log origin/main -n 3` | push latest commit |
| ComparisonError | malformed overlay yaml | `kubectl -n argocd get app demo-local -o yaml` | correct YAML shape/types |
| 404 on stage host | ingress host mismatch | `kubectl -n demo-stage get ingress demo -o yaml` | fix host patch, sync app |
| pod ImagePullBackOff | missing tag in registry | registry tags list endpoint | push image or change tag |

## Code entrypoints

- Bootstrap registry: `bootstrap/kustomization.yaml`
- Dev app binding: `bootstrap/demo-application.yaml`
- Stage app binding: `bootstrap/demo-stage-application.yaml`
- Demo base topology: `apps/demo/base/kustomization.yaml`
- Overlay image pointers: `apps/demo/overlays/*/kustomization.yaml`

## ADR index

- `docs/adr/0001-overlay-and-promotion-model.md`

## Updating locked decisions

Use append-only ADR updates; supersede with a new file when decisions evolve.
