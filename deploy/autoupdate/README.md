# MatrixHub Internal Auto-Deploy

When a PR merges to `main`, GitHub Actions builds the image, then the `deploy-internal` job runs on a self-hosted runner inside the internal K8s cluster to execute `helm upgrade`.

## Architecture

```
PR merge → GitHub Actions → push image (cloud) → deploy-internal (self-hosted runner in K8s) → helm upgrade
```

The runner Pod makes an **outbound** HTTPS connection to GitHub (long-poll for jobs). No inbound ports needed.

## Setup

### 1. Create a GitHub PAT

Go to https://github.com/settings/tokens and create a token with `repo` scope (classic) or `Administration` read/write (fine-grained).

### 2. Edit the Secret in runner-deployment.yaml

Replace `ghp_REPLACE_WITH_YOUR_PAT` with your actual token.

### 3. Review values-internal.yaml

The ConfigMap in `runner-deployment.yaml` contains the Helm values for internal deployment. Adjust image registries and storage sizes as needed.

### 4. Deploy

```bash
kubectl apply -f deploy/autoupdate/runner-deployment.yaml
```

### 5. Verify runner registered

```bash
kubectl -n matrixhub logs -l app=github-runner -f
```

You should see the runner register and start listening. Also check:
- GitHub repo → Settings → Actions → Runners — the runner should appear as "Idle".

### 6. Test

Merge any PR to `main`. The `deploy-internal` job should appear in the Actions run and execute on the internal runner.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Runner not appearing in GitHub | Check PAT scope, check pod logs for registration errors |
| `helm upgrade` fails | Check RBAC (ServiceAccount needs ClusterRole), check values |
| Image pull fails | Verify `ghcr.m.daocloud.io` is accessible from the cluster |
| Runner shows "offline" | Pod might have restarted — check `kubectl get pods` |

## Files

- `runner-deployment.yaml` — Runner Pod + ServiceAccount + RBAC + Secret + ConfigMap
- `values-internal.yaml` — Source-of-truth for internal Helm values (also embedded in ConfigMap)
