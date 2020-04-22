# Meal Planner: Infrastructure

## Usage

```bash
minikube start
eval $(minikube docker-env)
kubectl config use-context minikube

minikube ip
minikube dashboard

kubectl apply

kubectl get service -n=meal-planner
kubectl port-forward -n=meal-planner service/app-service 8080:80
# http://localhost:8080/
```
