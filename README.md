# Bookstack Helm Chart

## Local Installation

```sh
helm install <release_name> . --namespace=<namespace> --set appHost=<www.yourdomain.com>
```

## Installation from Repository

```sh
helm repo add pacroy https://pacroy.github.io/helm-repo/
helm repo update
helm install <release_name> pacroy/bookstack --namespace=<namespace> --values values.yaml
```
