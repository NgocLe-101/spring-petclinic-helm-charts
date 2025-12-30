# Spring Petclinic Helm Charts

Helm charts for deploying the Spring Petclinic microservices application with Istio service mesh and ArgoCD.

## Prerequisites

Before starting, ensure you have:
- Kubernetes cluster running (1.24+)
- kubectl configured to access your cluster
- Helm 3 installed
- The `spring-petclinic-IaC` repository set up first

## Setup Instructions

Follow these steps in order to set up the complete environment:

### Step 1: Install Istio Service Mesh

Add the Istio Helm repository:
```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

Install Istio components (base, control plane, and ingress gateway):
```bash
# Create the istio-system namespace
kubectl create namespace istio-system

# Install Istio base (CRDs) - must be installed first
helm install istio-base istio/base -n istio-system --set defaultRevision=default

# Install Istio control plane (istiod)
helm install istiod istio/istiod -n istio-system --wait

# Install Istio ingress gateway
helm install istio-ingress istio/gateway -n istio-system
```

### Step 2: Install Prometheus

Deploy Prometheus for metrics collection:
```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/prometheus.yaml
```

### Step 3: Install ArgoCD

Create ArgoCD namespace and install ArgoCD:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 4: Deploy Applications via ArgoCD

Apply the ArgoCD Application resources to deploy Kiali and the Spring Petclinic services:
```bash
# Deploy Spring Petclinic services (dev environment)
kubectl apply -f dev/application.yaml

# Deploy Kiali for service mesh visualization
kubectl apply -f kiali/application.yaml
```

## Verification

After deployment, verify that all components are running:
```bash
# Check Istio components
kubectl get pods -n istio-system

# Check ArgoCD status
kubectl get pods -n argocd

# Check application deployments
kubectl get applications -n argocd
```

## Accessing Services

Get the ArgoCD admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Port-forward to access ArgoCD UI:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access at: https://localhost:8080 (username: `admin`)