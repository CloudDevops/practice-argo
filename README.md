# ArgoCD Configuration Repository

This repository contains ArgoCD Application definitions that manage Kubernetes applications following GitOps principles.

## Repository Structure

```
practice-argo/
├── argocd-apps/              # ArgoCD Application definitions
│   └── hello-world-app.yaml  # Hello World application
├── hello-ingress.yaml        # Example hello app (standalone)
├── argocd-ingress.yaml       # ArgoCD server ingress
├── nginx-ingress-deploy.yaml # NGINX ingress controller
├── WARP.md                   # Warp AI assistant guidance
└── SETUP_SUMMARY.md          # Complete setup documentation
```

## ArgoCD Applications

### hello-world-app
- **Source Repo**: https://github.com/CloudDevops/hello-world-app
- **Destination**: `default` namespace
- **Sync Policy**: Automated with prune and self-heal

## Managing Applications

### Deploy an Application
```bash
kubectl apply -f argocd-apps/hello-world-app.yaml
```

### Check Application Status
```bash
kubectl get application -n argocd
kubectl get application hello-world -n argocd -o yaml
```

### Adding New Applications

1. Create a new application manifest in `argocd-apps/`
2. Point to the application's Git repository
3. Apply the manifest: `kubectl apply -f argocd-apps/<app-name>.yaml`

## GitOps Workflow

1. Application code/manifests live in their own repositories
2. ArgoCD configuration (this repo) defines which apps to deploy
3. Changes to application repos automatically sync to cluster
4. Changes to this repo update ArgoCD application configurations

## Infrastructure Components

- **NGINX Ingress Controller**: Deployed via `nginx-ingress-deploy.yaml`
- **ArgoCD Ingress**: Access ArgoCD UI via ingress

## Related Repositories

- [hello-world-app](https://github.com/CloudDevops/hello-world-app) - Sample application manifests
