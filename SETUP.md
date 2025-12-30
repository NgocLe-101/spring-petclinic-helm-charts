# Complete Setup Guide: Spring PetClinic on GKE

This guide walks you through the complete setup process for deploying the Spring PetClinic microservices application on Google Kubernetes Engine (GKE) with Istio service mesh and ArgoCD for GitOps deployment.

## Overview

The setup consists of two main phases:
1. **Infrastructure Provisioning**: Create a GKE cluster using Terraform
2. **Application Deployment**: Deploy Istio, ArgoCD, and the Spring PetClinic application

## Prerequisites

Ensure you have the following tools installed:
- [Terraform](https://developer.hashicorp.com/terraform/downloads) >= 1.5.0
- [Google Cloud CLI (gcloud)](https://cloud.google.com/sdk/docs/install)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/) 3.x
- A Google Cloud Platform account with an active project

---

## Phase 1: Infrastructure Provisioning

### Step 1: Install Google Cloud CLI

Download and install the Google Cloud CLI for your operating system:

**Follow instructions at: https://cloud.google.com/sdk/docs/install**

### Step 2: Authenticate with Google Cloud

Log in to your Google Cloud account:
```bash
gcloud auth login
```

Set your GCP project ID:
```bash
gcloud config set project <PROJECT_ID>
```

### Step 3: Create Application Default Credentials (ADC)

Terraform requires ADC to authenticate with Google Cloud:
```bash
gcloud auth application-default login
```

### Step 4: Clone and Configure the IaC Repository

Clone the infrastructure repository:
```bash
git clone https://github.com/NgocLe-101/spring-petclinic-IaC.git
cd spring-petclinic-IaC
```

Create a `terraform.tfvars` file:
```bash
cat > terraform.tfvars <<EOF
project_id   = "your-gcp-project-id"
zone         = "us-central1-a"
cluster_name = "spring-petclinic-cluster"
EOF
```

### Step 5: Deploy the GKE Cluster

Initialize Terraform:
```bash
terraform init
```

Review the infrastructure plan:
```bash
terraform plan
```

Apply the configuration to create the cluster:
```bash
terraform apply
```

Type `yes` when prompted to confirm.

### Step 6: Configure kubectl Access

Once the cluster is created, configure kubectl:
```bash
gcloud container clusters get-credentials spring-petclinic-cluster --zone=us-central1-a
```

> **Note:** Use the output command from `terraform apply` for the exact parameters.

Verify cluster access:
```bash
kubectl get nodes
```

---

## Phase 2: Application Deployment

### Step 7: Clone the Helm Charts Repository

Clone this repository:
```bash
cd ..
git clone https://github.com/NgocLe-101/spring-petclinic-helm-charts.git
cd spring-petclinic-helm-charts
```

### Step 8: Install Istio Service Mesh

Add the Istio Helm repository:
```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

Install Istio components:
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

Verify Istio installation:
```bash
kubectl get pods -n istio-system
```

### Step 9: Install Prometheus

Deploy Prometheus for metrics collection:
```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/prometheus.yaml
```

### Step 10: Install ArgoCD

Create ArgoCD namespace and install ArgoCD:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 11: Deploy Applications via ArgoCD

Apply the ArgoCD Application resources:
```bash
# Deploy Spring Petclinic services (dev environment)
kubectl apply -f dev/application.yaml

# Deploy Kiali for service mesh visualization
kubectl apply -f kiali/application.yaml
```

---

## Verification

### Check All Components

Verify that all components are running:

**Istio Components:**
```bash
kubectl get pods -n istio-system
```

**ArgoCD Status:**
```bash
kubectl get pods -n argocd
```

**ArgoCD Applications:**
```bash
kubectl get applications -n argocd
```

**Application Pods (Dev Environment):**
```bash
kubectl get pods -n dev
```

---

## Accessing Services

### ArgoCD UI

Get the ArgoCD admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

Port-forward to access ArgoCD UI:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access at: **https://localhost:8080**
- Username: `admin`
- Password: (use the password retrieved above)

### Kiali Dashboard

Port-forward to access Kiali:
```bash
kubectl port-forward svc/kiali -n istio-system 20001:20001
```

Access at: **http://localhost:20001**

### Spring PetClinic Application

Get the Istio Ingress Gateway external IP:
```bash
kubectl get svc istio-ingress -n istio-system
```

Access the application at: **http://\<EXTERNAL-IP\>**

---

## Infrastructure Details

The deployment includes:

**GKE Cluster:**
- Node Pool: 3 nodes (autoscaling 3-6 nodes)
- Machine Type: e2-standard-2
- Disk Size: 20GB per node
- Auto-repair and Auto-upgrade: Enabled

**Istio Components:**
- Control Plane (istiod)
- Ingress Gateway
- Service Mesh with mTLS

**Monitoring:**
- Prometheus for metrics
- Kiali for service mesh visualization

**GitOps:**
- ArgoCD for continuous deployment

---

## Troubleshooting

### Pods not starting
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

### ArgoCD applications not syncing
```bash
kubectl describe application <app-name> -n argocd
```

### Istio sidecar not injected
Ensure namespace has the label:
```bash
kubectl label namespace dev istio-injection=enabled
```

---

## Cleanup

### Remove Applications
```bash
kubectl delete -f dev/application.yaml
kubectl delete -f kiali/application.yaml
```

### Uninstall ArgoCD
```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```

### Uninstall Istio
```bash
helm uninstall istio-ingress -n istio-system
helm uninstall istiod -n istio-system
helm uninstall istio-base -n istio-system
kubectl delete namespace istio-system
```

### Destroy GKE Cluster
```bash
cd ../spring-petclinic-IaC
terraform destroy
```

Type `yes` when prompted to confirm.