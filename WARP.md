# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Overview

This is a Kubernetes practice repository containing example manifests for:
- **NGINX Ingress Controller** - Full deployment manifest for ingress-nginx v1.14.0
- **ArgoCD Ingress** - Example ingress configuration for ArgoCD server access
- **Hello App** - Simple demo application with Pod, Service, and Ingress

All manifests use the `hello.local` hostname for local testing/development.

## Common Commands

### Applying Manifests
```bash
# Apply NGINX ingress controller (creates namespace, RBAC, deployment, services)
kubectl apply -f nginx-ingress-deploy.yaml

# Apply the hello demo application
kubectl apply -f hello-ingress.yaml

# Apply ArgoCD ingress (requires ArgoCD to be installed separately)
kubectl apply -f argocd-ingress.yaml
```

### Viewing Resources
```bash
# Check ingress controller status
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx

# Check hello app
kubectl get pod hello
kubectl get svc hello
kubectl get ingress hello-ingress

# Check ArgoCD ingress
kubectl get ingress argocd-ingress -n argocd

# View all ingresses
kubectl get ingress --all-namespaces
```

### Testing Ingress Access
```bash
# Test hello app locally (requires hello.local in /etc/hosts)
curl http://hello.local

# Port-forward to test without ingress
kubectl port-forward pod/hello 5678:5678
curl http://localhost:5678
```

### Debugging
```bash
# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Describe resources for troubleshooting
kubectl describe ingress hello-ingress
kubectl describe ingress argocd-ingress -n argocd
kubectl describe pod hello
```

### Cleanup
```bash
# Delete specific resources
kubectl delete -f hello-ingress.yaml
kubectl delete -f argocd-ingress.yaml
kubectl delete -f nginx-ingress-deploy.yaml

# Or delete by resource type
kubectl delete ingress hello-ingress
kubectl delete pod hello
kubectl delete svc hello
```

## Architecture Notes

### Ingress Configuration
- All ingress resources use `ingressClassName: nginx`
- The NGINX ingress controller is deployed with LoadBalancer service type
- Controller uses hostPort binding (80, 443) for direct access
- Ingress controller version: v1.14.0

### ArgoCD Ingress Specifics
- **Namespace**: `argocd` (must exist before applying)
- **Annotations**: Configured for non-SSL HTTP backend
  - `nginx.ingress.kubernetes.io/force-ssl-redirect: "false"`
  - `nginx.ingress.kubernetes.io/ssl-passthrough: "false"`
  - `nginx.ingress.kubernetes.io/backend-protocol: "HTTP"`
- Routes to `argocd-server` service on port 80

### Hello App Structure
The hello-ingress.yaml contains three resources in one file:
1. **Pod** - Runs hashicorp/http-echo image on port 5678
2. **Service** - ClusterIP service exposing port 5678
3. **Ingress** - Routes hello.local traffic to the service

## Local Development Setup

To use these manifests locally with a Kubernetes cluster (minikube, kind, etc.):

1. Add hostname to /etc/hosts:
   ```bash
   echo "127.0.0.1 hello.local" | sudo tee -a /etc/hosts
   ```

2. For ArgoCD, install ArgoCD first:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. Apply manifests in order:
   - nginx-ingress-deploy.yaml (ingress controller)
   - hello-ingress.yaml (demo app)
   - argocd-ingress.yaml (after ArgoCD is installed)
