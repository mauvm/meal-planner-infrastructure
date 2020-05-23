# Meal Planner: Infrastructure

## Usage

```bash
minikube start
eval $(minikube docker-env) # Or "minikube docker-env | Invoke-Expression" on Windows
kubectl config use-context minikube

minikube ip
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:80 # Or "minikube dashboard"
minikube tunnel # Run as Administrator

# Create meal-planner namespace
kubectl kustomize .\development\ | kubectl apply -f -

# Inspect and test if running
kubectl config set-context --current --namespace=meal-planner
kubectl get service
kubectl port-forward service/app-service 8080:3000
# http://localhost:8080/

# EventStore Admin UI
kubectl port-forward service/event-store-service 2113:2113

# Github Actions
# To get KUBE_CONFIG_DATA secret:
kubectl config view --flatten --minify
# Change the following:
# - Replace server IP in clusters.0.cluster.server to domain name
# - Base64 encode

# Istio
# Install: https://github.com/istio/istio/releases
kubectl label namespace meal-planner istio-injection=enabled
istioctl install -f development/istio-install/profile.yml
# Kiali dashboard
kubectl port-forward -n istio-system service/kiali 9000:20001 # Or "istioctl dashboard kiali"
# Verify Istio installation
istioctl verify-install -f development/istio-install/manifest.yml
istioctl analyze
```
