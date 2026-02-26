# Glossary

| Term | Meaning | Software analogy | Where it appears |
|---|---|---|---|
| Overlay | Env-specific Kustomize customization | config profile | `apps/demo/overlays/*` |
| Base | Shared manifest set | common library | `apps/demo/base/*` |
| Argo Application | Git source-to-cluster mapping object | deployment subscription | `bootstrap/demo-application.yaml` |
| Promotion | Move tested image tag to higher env | release pointer change | overlay `newTag` fields |
