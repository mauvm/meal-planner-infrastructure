# Meal Planner: Infrastructure

This repository contains the Infrastructure as Code (IaC) aspect of the [Meal Planner project](https://github.com/users/mauvm/projects/1):

- Kubernetes manifests for the [services](services)
- Istio profile and manifest

## Services

- [App](https://github.com/mauvm/meal-planner-app/): the HTTP web interface (UI)
- [Shopping List](https://github.com/mauvm/meal-planner-shopping-list/): the API for managing shopping list items

See [`kubernetes/production/kustomization.yml`](kubernetes/production/kustomization.yml) for setting the service versions.

## Local Development

Prerequisites for host machine:

- [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Istioctl](https://github.com/istio/istio/releases/)

Follow the guides above to setup and configure your Minikube cluster.

Start Minikube VM and configure `docker` and `kubectl` commands:

```bash
minikube start
eval $(minikube docker-env) # Or "minikube docker-env | Invoke-Expression" on Windows
kubectl config use-context minikube
```

Then setup [Istio](https://istio.io/):

```bash
istioctl manifest apply -f istio/profile.yml
istioctl verify-install -f kubernetes/base/istio.yml
istioctl analyze
```

Finally deploy the Kubernetes resources and try out the app:

```bash
kubectl kustomize kubernetes/development/ | kubectl apply -f -

kubectl config set-context --current --namespace=meal-planner
kubectl get service
kubectl port-forward service/app-service 8080:3000
# http://localhost:8080/
```

### Helpful commands

```bash
# Get IP address
minikube ip

# Access to Minikube dashboard
minikube dashboard

# Access to Istio's Kiali dashboard
istioctl dashboard kiali

# Direct access to container IPs
minikube tunnel # Run as Administrator

# EventStore Admin UI
kubectl port-forward service/event-store-service 2113:2113 # admin:changeit
```

## Deploy to Production

Deployments to a remote Kubernetes cluster are done with Github Actions.

### Github Actions

For deployments with Github Actions you can configure the following secrets:

- `KUBE_CONFIG_DATA`: base64 encoded Kubernetes config

  1. Get Kubernetes config: `kubectl config view --flatten --minify > /tmp/config.yml`
  2. Replace the server IP to your domain name (path: `clusters/0/cluster/server`)
  3. Encode config: `cat /tmp/config.yml | base64`

When using the [`letsencrypt-nginx-proxy-companion`](letsencrypt-nginx-proxy-companion) you should add these secrets as well:

- `LETSENCRYPT_HOST`: your domain name
- `LETSENCRYPT_EMAIL`: your (developer) email address

### Helpful commands

```bash
# Access to remote Minikube dashboard
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:80

# Access to remote Kiali dashboard
kubectl port-forward -n istio-system service/kiali 9000:20001
```

### LetsEncrypt Nginx Proxy Companion

You can put the [`nginx-proxy`](https://github.com/nginx-proxy/nginx-proxy) before your Istio Ingress Gateway. This allows you to:

- Run other Docker applications on the same VPS
- Run the [`letsencrypt-nginx-proxy-companion`](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion) as well, to automatically request and update LetsEncrypt SSL certificates
