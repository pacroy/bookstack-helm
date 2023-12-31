# Bookstack Helm Chart

[![Lint Code Base](https://github.com/pacroy/bookstack-helm/actions/workflows/linter.yml/badge.svg)](https://github.com/pacroy/bookstack-helm/actions/workflows/linter.yml) [![Test and Publish Chart](https://github.com/pacroy/bookstack-helm/actions/workflows/test-and-publish.yml/badge.svg)](https://github.com/pacroy/bookstack-helm/actions/workflows/test-and-publish.yml)

## Local Installation

```sh
helm upgrade --install <release_name> . --namespace=<namespace> --set appHost=<www.yourdomain.com>
```

## Installation from Repository

```sh
helm repo add pacroy https://pacroy.github.io/helm-repo/
helm repo update
helm upgrade --install <release_name> pacroy/bookstack --namespace=<namespace> --values values.yaml
```
