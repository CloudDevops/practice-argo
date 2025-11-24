# ArgoCD Practice Setup Summary

## What We Built

### 1. GitHub Repository Setup
Two separate repositories following GitOps best practices:
- **ArgoCD Configuration**: `https://github.com/CloudDevops/practice-argo`
  - Contains ArgoCD Application definitions
  - Infrastructure components (ingress controller, etc.)
- **Application Manifests**: `https://github.com/CloudDevops/hello-world-app`
  - Contains Kubernetes manifests for the hello-world application
  - Managed and deployed by ArgoCD

### 2. Kubernetes Resources Deployed

#### NGINX Ingress Controller
- Deployed NGINX ingress controller to handle ingress traffic
- Service type: LoadBalancer with EXTERNAL-IP: `172.18.255.200`
- Namespace: `ingress-nginx`

#### Hello World Application (ArgoCD Managed)
Repository: `https://github.com/CloudDevops/hello-world-app`
- **Deployment**: 2 replicas running Google's hello-app
- **Service**: ClusterIP service on port 80
- **Ingress**: Accessible at `hello-world.local`
- **Image versions tested**:
  - `gcr.io/google-samples/hello-app:1.0` (current)
  - `gcr.io/google-samples/hello-app:2.0` (tested rollout)

#### ArgoCD Application
- **Name**: `hello-world`
- **Namespace**: `argocd`
- **Definition**: `practice-argo/argocd-apps/hello-world-app.yaml`
- **Source**: GitHub repo `https://github.com/CloudDevops/hello-world-app.git`
- **Target Revision**: `main`
- **Destination**: `default` namespace
- **Sync Policy**: Automated with prune and self-heal enabled

### 3. GitOps Workflow Demonstrated

1. **Initial Deployment**: Created hello-world app v1.0
2. **Version Update**: Changed image to v2.0 and pushed to GitHub
3. **Automatic Sync**: ArgoCD automatically deployed v2.0
4. **Rollback**: Reverted to v1.0 and pushed to GitHub
5. **Automatic Rollback**: ArgoCD automatically rolled back to v1.0

## Key Files

```
practice-argo/ (ArgoCD Configuration Repo)
├── argocd-apps/
│   └── hello-world-app.yaml     # ArgoCD Application definition
├── nginx-ingress-deploy.yaml    # NGINX ingress controller manifest
├── hello-ingress.yaml           # Example hello app (standalone)
├── argocd-ingress.yaml          # ArgoCD ingress configuration
├── README.md                    # Repository overview
├── SETUP_SUMMARY.md             # This file
└── WARP.md                      # Warp AI assistant guidance

hello-world-app/ (Application Repo)
├── deployment.yaml              # Deployment with 2 replicas
├── service.yaml                 # ClusterIP service
├── ingress.yaml                 # Ingress for hello-world.local
└── README.md                    # Application documentation
```

## Access Information

### Hello World App
- **URL**: `http://hello-world.local`
- **Hosts entry required**: `172.18.255.200 hello-world.local`

### ArgoCD Server
- **Service**: `argocd-server` in `argocd` namespace
- **Access via port-forward**: 
  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```
- **Get admin password**:
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```

## Common Commands

### Check ArgoCD Application Status
```bash
kubectl get application hello-world -n argocd
kubectl get application hello-world -n argocd -o yaml
```

### Watch Deployments
```bash
kubectl get pods -w
kubectl get application hello-world -n argocd -w
```

### Verify Ingress
```bash
kubectl get ingress --all-namespaces
kubectl describe ingress hello-world
```

### Test Application
```bash
curl http://hello-world.local
```

## GitOps Workflow

### Updating Applications
1. Make changes to application manifests in the `hello-world-app` repository
2. Commit and push to GitHub:
   ```bash
   cd ~/hello-world-app
   git add .
   git commit -m "Update application"
   git push
   ```
3. ArgoCD automatically syncs within ~3 minutes
4. Verify deployment: `kubectl get pods`

### Adding New Applications
1. Create a new application repository with Kubernetes manifests
2. Create ArgoCD Application definition in `practice-argo/argocd-apps/`
3. Commit and push to `practice-argo` repository
4. Apply the Application: `kubectl apply -f argocd-apps/<app-name>.yaml`

## Key Learnings

- **Automated GitOps**: Changes to GitHub automatically deploy to Kubernetes
- **Version Control**: All infrastructure/app configs are version controlled
- **Easy Rollbacks**: Reverting Git commits automatically rolls back deployments
- **Declarative Configuration**: Desired state defined in Git, ArgoCD ensures actual state matches
- **Self-Healing**: If manual changes are made to cluster, ArgoCD reverts them to match Git

## Repository Structure

### practice-argo (Configuration Repository)
Contains ArgoCD configuration and infrastructure components:
- ArgoCD Application definitions (argocd-apps/)
- Infrastructure manifests (ingress controller, etc.)
- Documentation

### hello-world-app (Application Repository)
Contains application-specific Kubernetes manifests:
- Deployment, Service, Ingress definitions
- Application code would typically live here too

## Benefits of Separation

- **Separation of Concerns**: Application teams manage app repos, platform teams manage ArgoCD config
- **Access Control**: Different permissions for app vs. infrastructure
- **Independent Updates**: App changes don't clutter infrastructure repo
- **Scalability**: Easy to add more applications with their own repositories
- **Best Practice**: Follows standard GitOps patterns
