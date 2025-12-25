# Bookstack Helm Chart

[![Lint Code Base](https://github.com/pacroy/bookstack-helm/actions/workflows/linter.yml/badge.svg)](https://github.com/pacroy/bookstack-helm/actions/workflows/linter.yml) [![Test and Publish Chart](https://github.com/pacroy/bookstack-helm/actions/workflows/test-and-publish.yml/badge.svg)](https://github.com/pacroy/bookstack-helm/actions/workflows/test-and-publish.yml)

## Local Installation

```sh
helm upgrade --install <release_name> . --namespace=<namespace> --create-namespace \
    --set appHost=<www.yourdomain.com> \
    --set appKey=base64:VGhpc0lzQW5FeGFtcGxlQXBwS2V5Q2hhbmdlVGhpcyE= \
    --set azuread.enabled=false \
    --set smtp.enabled=false
```

> [!IMPORTANT]  
> `appKey` must be set to 32-character key. Container would end up in error if not set. App would show `An unknown error occurred` if set to blank.
`

## Installation from Repository

```sh
helm repo add pacroy https://pacroy.github.io/helm-repo/
helm repo update
helm upgrade --install <release_name> pacroy/bookstack --namespace=<namespace> --values values.yaml
```

## Configuration

### cert-manager Integration

This chart supports automatic TLS certificate provisioning using [cert-manager](https://cert-manager.io/). By default, cert-manager is enabled with a ClusterIssuer named `letsencrypt-prod`.

#### Using the Default Configuration

If you have a ClusterIssuer named `letsencrypt-prod` in your cluster, no additional configuration is needed. The chart will automatically request certificates.

#### Disabling cert-manager

To disable cert-manager and manage TLS certificates manually:

```sh
helm upgrade --install <release_name> pacroy/bookstack \
    --set ingress.certManager.enabled=false
```

#### Using a Custom ClusterIssuer

To use a different ClusterIssuer (e.g., `letsencrypt-staging` for testing):

```sh
helm upgrade --install <release_name> pacroy/bookstack \
    --set ingress.certManager.enabled=true \
    --set ingress.certManager.issuerName=letsencrypt-staging
```

#### Using a Namespace-Scoped Issuer

To use a namespace-scoped Issuer instead of a ClusterIssuer:

```sh
helm upgrade --install <release_name> pacroy/bookstack \
    --set ingress.certManager.enabled=true \
    --set ingress.certManager.issuerType=issuer \
    --set ingress.certManager.issuerName=my-issuer
```

#### Custom TLS Secret Name

To specify a custom name for the TLS secret:

```sh
helm upgrade --install <release_name> pacroy/bookstack \
    --set ingress.tls.secretName=my-custom-tls-secret
```

#### Additional Ingress Annotations

To add custom annotations to the ingress resource:

```yaml
ingress:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

### Configuration Options

The following values can be configured in `values.yaml` or via `--set` flags:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.certManager.enabled` | Enable cert-manager certificate provisioning | `true` (when `ingress` is configured) |
| `ingress.certManager.issuerType` | Type of issuer: `cluster-issuer` or `issuer` | `cluster-issuer` |
| `ingress.certManager.issuerName` | Name of the cert-manager issuer | `letsencrypt-prod` |
| `ingress.tls.secretName` | Custom name for the TLS secret | `<release-name>-bookstack` |
| `ingress.annotations` | Additional custom annotations for the ingress | `{}` |
