# ADR-0001: Overlay and Promotion Model

**Status:** Accepted  
**Date:** 2026-02-26

## Decision
Use Kustomize overlays per environment (`local`, `stage`) with promotion done by editing `newTag` values in overlay `kustomization.yaml`.

## Context
A single base with env-specific deltas keeps manifests DRY while maintaining explicit release control.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| Duplicate full manifests per env | simple mentally | drift and duplication risk |
| Single overlay for all envs | minimal files | no env isolation |
| Base + per-env overlays (chosen) | reuse + explicit env deltas | requires discipline on patches |

## Decision drivers

- Explicit promotion mechanics.
- Clear environment isolation in one cluster.
- Maintainable manifest reuse.

## Consequences

Pros:
- Easy to promote specific tags across envs.
- Lower drift risk than duplicated files.

Cons:
- Incorrect patching can break routing quickly.

Mitigation:
- Validate rendered output before push.

## Operational notes

- Dev: `apps/demo/overlays/local`
- Stage: `apps/demo/overlays/stage`
- Argo apps map directly to these paths.

## Validation

```bash
cd repos/infra
kubectl kustomize apps/demo/overlays/local >/tmp/local.yaml
kubectl kustomize apps/demo/overlays/stage >/tmp/stage.yaml
```

Expected outcome shape:
- both renders succeed
- host and namespace differ appropriately between envs
