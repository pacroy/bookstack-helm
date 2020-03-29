# Bookstack Helm Chart

## Prerequisites

- A working Kubernetes cluster with following charts installed:
  - stable/nginx-ingress
  - jetstack/cert-manager
- [kubectl installed](https://kubernetes.io/docs/tasks/tools/install-kubectl/) with active context to the Kubernetes cluster
- [Helm 3 installed](https://helm.sh/docs/intro/install/)
- Domain name for your Wiki e.g. `www.testwiki.com` that resolvable to your nginx-ingress-controller public IP.

## Installation

Create a new namespace, if desired.

```
kubectl create namespace testwiki
```

Clone this repo and install with Helm

```
helm install testwiki --namespace=testwiki --set appHost=www.testwiki.com ./bookstack
```