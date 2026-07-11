# Copilot Instructions for bookstack-helm

## Overview
This repository is a Kubernetes Helm chart for deploying BookStack (a wiki application) with MySQL on Kubernetes. The chart handles the entire BookStack application stack including the web app, database, storage volumes, ingress, and certificate management.

## Build, Test, and Lint Commands

### Test Helm Template
Validate that the Helm chart templates render correctly without syntax errors:
```bash
helm template test-release . --values values-test.yaml
```

### Lint YAML and Code
Linting runs automatically via GitHub Actions on push/PR. The workflow uses a shared linter workflow from `pacroy/gh-common-workflows`. Linting includes:
- YAML validation (via `.github/linters/.yaml-lint.yml`)
- Markdown linting (via `.github/linters/.markdown-lint.yml`)
- Copy detection (via `.github/linters/.jscpd.json`)

To lint locally, use `yamllint` and `markdownlint` on files matching the configuration files in `.github/linters/`.

### Package Helm Chart
Packages the chart for distribution to the Helm repository:
```bash
helm package .
helm repo index .
```

## High-Level Architecture

### Repository Structure
- **Chart.yaml** - Helm chart metadata (name, version, appVersion, maintainers)
- **values.yaml** - Default configuration values (commented examples)
- **values-test.yaml** - Test values for validation
- **templates/** - Kubernetes resource manifests:
  - `bookstack.yaml` - BookStack Deployment with init containers, environment variables, persistent volume mounts
  - `mysql.yaml` - MySQL Deployment with persistent volume
  - `pvc.yaml` - PersistentVolumeClaim for uploads and storage
  - `ingress.yaml` - Nginx Ingress with cert-manager TLS integration

### Deployment Architecture
The chart deploys a complete stack:
1. **BookStack Container** - Runs the BookStack wiki application
   - Single replica (Recreate strategy)
   - Init containers: volume permission setup, MySQL readiness check
   - Environment variables: APP_KEY, APP_URL, DB credentials, storage type
   - Volumes: uploads, storage (for persistent data)

2. **MySQL Container** - Dedicated database instance
   - Single instance (not HA)
   - Database name: `bookstack`
   - Default credentials configured in template

3. **Storage** - Two PersistentVolumeClaims
   - `bookstack-uploads` - User-uploaded files
   - `bookstack-storage` - Application storage data

4. **Ingress** - Nginx with cert-manager
   - TLS via Let's Encrypt (letsencrypt-prod ClusterIssuer)
   - HTTP01 challenge with edit-in-place annotation
   - Client max body size: 10m
   - Host configured via `appHost` value

### Key Integration Points
- **cert-manager** - Automatically issues/renews Let's Encrypt certificates
- **Azure AD** - Optional Azure AD authentication (controlled via `azuread.enabled`)
- **SMTP** - Optional email support (controlled via `smtp.enabled`)
- **Storage Backend** - Supports local_secure or Azure storage via `storageType` value

## Key Conventions

### Required Values
These values must be provided during installation or in values file:
- `appHost` - Domain name for the wiki (e.g., `www.yourdomain.com`)
- `appKey` - 32-character base64-encoded encryption key; must be set to non-blank value or app shows "An unknown error occurred"

### Optional Features
- `azuread` - Set `.enabled: true` to enable Azure AD integration with `tenantId`, `appId`, `appSecret`
- `smtp` - Set `.enabled: true` to enable email with host, port, credentials, sender address

### Template Naming Conventions
Resource names follow the pattern: `{{ .Release.Name }}-{{ resource_type }}`
- Examples: `my-release-bookstack`, `my-release-mysql`, `my-release-bookstack-uploads`
- This allows multiple installations (different release names) in the same cluster

### Init Container Pattern
The `bookstack.yaml` Deployment uses init containers for:
1. Volume initialization - Fixes ownership and removes lost+found directories (critical for storage access)
2. MySQL readiness - Waits for MySQL to be available before starting the app

### Resource Specifications
- Uses specific image tags (mysql:8.4, busybox)
- Session lifetime: 1440 minutes (24 hours)
- App view default: grid
- Secure cookie enforcement: enabled for HTTPS

### Version Management
- **Chart version** (Chart.yaml) - Incremented when chart changes
- **appVersion** (Chart.yaml) - Reflects BookStack application version being packaged
- **Packaging** - Test workflow checks package size against `HELM_MAX_PACKAGE_SIZE` secret

## Development Notes

### Testing Changes
1. Modify templates or values
2. Run: `helm template test-release . --values values-test.yaml`
3. Verify output contains expected resources and no errors

### Adding New Features
- Add new fields to `values.yaml` (commented if optional)
- Reference values in templates using `{{ .Values.fieldName }}`
- Update version in `Chart.yaml`
- Ensure template renders correctly with `helm template` before PR

### Pushing Chart to Repository
Test/publish workflow automatically packages and publishes to `pacroy/helm-repo` on push to main. Manual trigger available via workflow_dispatch.
