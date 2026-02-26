# Local GitOps Runbook (infra + app)

## Scope

This runbook covers two local GitOps flows with Argo CD and the `demo-local` Application.

- Flow A: infra-only change (no new image build)
- Flow B: app code change (rebuild images and bump tags in infra)

## Baseline assumptions

- Argo CD is installed and `infra-bootstrap` is applied from platform.
- `demo-local` Argo Application exists from `bootstrap/demo-application.yaml`.
- Local registry is reachable at `localhost:5001` from kind.

## Flow A: infra-only change -> Argo sync -> visible change

Example change: update banner text in `apps/demo/base/configmap.yaml` key `OPS_BANNER_TEXT`.

High-level sequence:

1. Edit infra manifest only (for example `OPS_BANNER_TEXT: "demo-banner=v2"`).
2. Commit and push to infra main branch.
3. Argo reconciles automatically (or manually sync if needed).
4. Verify visible change at `http://localhost/ops-banner.txt`.

Expected result: output reflects new banner text without building/pushing new client or api images.

## Flow B: app code change -> build/push images -> bump tags -> Argo sync -> visible change

Example change: update API version response in app repo.

High-level sequence:

1. Change app code in `repos/app` (client or api).
2. Build and push images to local registry with new immutable tag (`git-sha` preferred).
3. In infra repo overlay `apps/demo/overlays/local/kustomization.yaml`, update `newTag` for `demo-client` and/or `demo-api`.
4. Commit and push infra changes.
5. Argo reconciles to new image tags.
6. Verify visible change through ingress:
   - API: `http://localhost/api/version`
   - Client: `http://localhost/`

Expected result: traffic serves updated behavior from the new image tag(s).

## Troubleshooting

### Argo cannot reach Gitea

- Validate Gitea host endpoint from a debug pod: `host.docker.internal:3000`.
- Check Argo repo-server logs for DNS/connect/auth errors.
- Verify repository URL and credentials configured in Argo match Gitea.
- Confirm Gitea container is running and exposed on host port 3000.

### Pods stuck Pending

- Inspect pod events for scheduling failures (CPU/memory/taints/PVC issues).
- Compare requested resources in Deployments vs available node capacity.
- Reduce requests/limits temporarily if local nodes are constrained.

### Ingress routing failures

- Confirm ingress-nginx controller pod is Running.
- Check `demo` Ingress host/path rules and backend service names/ports.
- Ensure requests use the expected host (`localhost`) and paths (`/`, `/api`).
- Inspect ingress controller logs for 404/502 upstream errors.
