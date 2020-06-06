# Meal Planner: Infrastructure

> Infrastructure as Code (IaC) for [Meal Planner project](https://github.com/users/mauvm/projects/1)

This repository contains:

- Kubernetes manifests for the [services](#services)
- Istio profile and manifest

## Services

- [App](https://github.com/mauvm/meal-planner-app/): the web interface (UI)
- [Shopping List](https://github.com/mauvm/meal-planner-shopping-list/): the API for managing shopping list items

These services build their own Docker images and push it to a registry that must be accessible to your Kubernetes cluster.

See [`kubernetes/production/kustomization.yml`](kubernetes/production/kustomization.yml) for setting the service versions.

## Architecture

The architecture is simple:

<p align="center">
  <img src="docs/kiali-service-graph.png" alt="Kiali service graph" />
</p>

See [`kubernetes/base/networking/gateway.yml`](kubernetes/base/networking/gateway.yml) for the routing configuration.

Tip: use an [`NGinX Proxy`](#letsencrypt-nginx-proxy-companion) in front of your Istio ingress gateway for automatic SSL certificates.

## Local Development

Prerequisites for host machine:

- [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Istioctl](https://github.com/istio/istio/releases/)

Follow the guides above to setup and configure your Minikube cluster.

Start Minikube VM and configure `docker` and `kubectl` commands:

```bash
minikube start
kubectl config use-context minikube
```

Then setup [Istio](https://istio.io/):

```bash
istioctl install -f istio/profile.yml
istioctl verify-install -f kubernetes/base/networking/istio.generated.yml
istioctl analyze
```

Finally deploy the Kubernetes resources and try out the app:

```bash
kubectl kustomize kubernetes/development/ | kubectl apply -f -

kubectl get -n istio-system service/istio-ingressgateway -o jsonpath="{$.spec.clusterIP}"
# Open http://{ip}/ in browser
```

### Helpful commands

```bash
# Get IP address
minikube ip

# Access to Minikube dashboard
minikube dashboard

# Access to Istio's Kiali dashboard
istioctl dashboard kiali # admin:admin

# Configure Docker CLI
eval $(minikube docker-env) # Or "minikube docker-env | Invoke-Expression" on Windows

# EventStore Admin UI
kubectl port-forward service/event-store-service 2113:2113 # admin:changeit
```

## Deploy to Production

Deployments to a remote Kubernetes cluster are done with Github Actions.

The Github [`deploy-on-push`](.github/workflows/deploy-on-push.yml) workflow does the following:

1. Install and verify Istio
2. Build Kubernetes manifest and replace environment variables (for improved configurability)
3. Apply manifest to Kubernetes cluster

### Access remote Kubernetes cluster

To access the remote Kubernetes cluster add a Kubectl context:

1. [Generate KUBE_CONFIG_DATA](#github-actions) (base64 encoding is not necessary)
2. Merge into Kubeconfig on host machine (`~/.kube/config`), see example below
3. You can now switch to the remote context: `kubectl config use-context remote-minikube`

```yml
apiVersion: v1
kind: Config
preferences: {}
current-context: minikube
clusters:
  - cluster:
      certificate-authority: /path/to/ca.crt
      server: https://{minikube ip}:8443
    name: minikube
  - cluster:
      insecure-skip-tls-verify: true
      server: https://{server domain}:8443
    name: remote-minikube
contexts:
  - context:
      cluster: minikube
      namespace: meal-planner
      user: minikube
    name: minikube
  - context:
      cluster: remote-minikube
      namespace: meal-planner
      user: remote-minikube
    name: remote-minikube
users:
  - name: minikube
    user:
      client-certificate: ~/.minikube/profiles/minikube/client.crt
      client-key: /.minikube/profiles/minikube/client.key
  - name: remote-minikube
    user:
      client-certificate-data: ...
      client-key-data: ...
```

### Github Actions

For deployments with Github Actions you must configure the following secrets:

- `KUBE_CONFIG_DATA`: base64 encoded Kubernetes config

  1. Get Kubeconfig: `kubectl config view --flatten --minify > /tmp/kubeconfig.yml`

     See [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) for Kubeconfig details.

  2. Replace `certificate-authority-data: ...` with `insecure-skip-tls-verify: true`
  3. Replace the server IP to your domain name (path: `clusters/0/cluster/server`)
  4. Encode config: `cat /tmp/kubeconfig.yml | base64`

- `DOCKER_REGISTRY`: Docker registry domain name and repository name

  Must end with `/`, for example: `registry.mydomain.eu/meal-planner/`.

  Leavy domain name blank when using Docker Hub, for example: `meal-planner/`.

When using the [`letsencrypt-nginx-proxy-companion`](#letsencrypt-nginx-proxy-companion) you should add these secrets as well:

- `LETSENCRYPT_HOST`: your domain name
- `LETSENCRYPT_EMAIL`: your (developer) email address

### Helpful commands

```bash
# Access to remote Minikube dashboard
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:80
# Note: "uid : unable to do port forwarding: socat not found" port forwarding errors can be resolved by installing socat on the remote machine (yum install socat)

# Access to remote Kiali dashboard
kubectl port-forward -n istio-system service/kiali 9000:20001
```

### LetsEncrypt NGinX Proxy Companion

You can put the [`nginx-proxy`](https://github.com/nginx-proxy/nginx-proxy) in front of your Istio ingress gateway. This allows you to:

- Host other Docker applications on the same server
- Run the [`letsencrypt-nginx-proxy-companion`](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion) on the side to automatically request and update LetsEncrypt SSL certificates
