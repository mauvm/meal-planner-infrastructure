# Meal Planner: Infrastructure

## Usage

```bash
minikube start
eval $(minikube docker-env)
kubectl config use-context minikube

minikube ip
minikube dashboard

kubectl apply -f .

kubectl config set-context --current --namespace=meal-planner
kubectl get service
kubectl port-forward service/app-service 8080:80
# http://localhost:8080/

# EventStore Admin UI
kubectl port-forward service/event-store-service 2113:2113

# Github Actions
# To get KUBE_CONFIG_DATA secret:
kubectl config view --flatten --minify
# Change the following:
# - Replace server IP in clusters.0.cluster.server to domain name
# - Base64 encode
```
