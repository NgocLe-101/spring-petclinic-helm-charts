# spring-petclinic-helm-charts

NOTE to deploy:
setup the spring-petclincic-IaC repo first
# Create namespace and services for istio:
1. Add Helm Repo:
```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

2. Install base + istiod + ingress
```bash
# 1. Tạo namespace
kubectl create namespace istio-system

# 2. Cài đặt CRDs (Custom Resource Definitions) - Bắt buộc chạy trước
helm install istio-base istio/base -n istio-system --set defaultRevision=default

# 3. Cài đặt Control Plane (Istiod)
helm install istiod istio/istiod -n istio-system --wait

# 4. Cài đặt Ingress gateway
helm install istio-ingress istio/gateway -n istio-system
```

3. Install prometheus:
```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/prometheus.yaml
```

4. Install argocd:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

5. Apply argocd appication for kiali and services
```bash
kubectl apply -f dev/application.yaml
kubectl apply -f kiali/application.yaml
```