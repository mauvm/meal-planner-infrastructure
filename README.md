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
```
