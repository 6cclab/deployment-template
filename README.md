# Deployment Template

Copier template for scaffolding Kubernetes deployment repos that follow the 6cclab conventions: Kustomize overlays, InfisicalSecret CRD, Traefik ingress, and Prometheus annotations.

## Prerequisites

```bash
pipx install copier
```

## Usage

### Scaffold a new deployment repo

```bash
copier copy gh:6cclab/deployment-template ./my-service-deployment
```

### Update an existing repo when the template changes

```bash
cd my-service-deployment
copier update
```

## Variables

| Variable | Default | Description |
|---|---|---|
| `project_name` | *required* | Project name (e.g., `volttrack`) |
| `service_name` | *required* | Service name (e.g., `auth`, `api`) |
| `container_port` | `3000` | Container port the application listens on |
| `image_registry` | `ghcr.io/6cclab` | Container image registry |
| `domain_zone` | `volttrack` | Domain zone (`volttrack` / `homelab` / `bare`) |
| `domain` | `volttrack.io` | Base domain (only asked when zone=volttrack) |
| `dev_host` / `prd_host` | auto-derived | Ingress hostnames (see table below) |
| `infisical_project_id` | *required* | Infisical project ID |
| `infisical_identity_id` | `ccebb465-...` | Shared Kubernetes auth identity |
| `sentry_dsn` | `""` | Sentry DSN (omitted from yaml if blank) |
| `otel_service_name` | `{project}-{service}` | OpenTelemetry service name |
| `memory_request` / `cpu_request` | `1Gi` / `200m` | Container resource requests |
| `liveness_path` / `readiness_path` | `/uptime` / `/health` | Probe HTTP paths |
| `config_map_name` | `{project}-config` | ConfigMap name |
| `secret_name` | `{project}-{service}` | Secret name (from InfisicalSecret) |
| `has_disruption_budget` | `false` | Include PodDisruptionBudget |
| `has_service_account` | `false` | Include ServiceAccount |
| `service_type` | `ClusterIP` | Service type (`ClusterIP` / `LoadBalancer`) |
| `image_pull_policy` | `IfNotPresent` | Image pull policy |

## Host Auto-Derivation

`dev_host` and `prd_host` are automatically derived from `domain_zone` but can be overridden:

| Zone | Dev | Prd |
|---|---|---|
| `volttrack` | `{service}.dev.{domain}` | `{service}.{domain}` |
| `homelab` | `{service}.apps.dev-01.6cclab.dev` | `{service}.apps.prod-01.6cclab.dev` |
| `bare` | `{service}.6cclab.dev` | `{service}.6cclab.dev` |

## Generated Structure

```
base/
  deployment.yaml       # Rolling update, probes, envFrom, Prometheus annotations
  service.yaml          # ClusterIP or LoadBalancer (conditional sessionAffinity)
  config.yaml           # OTEL, Pyroscope, LOG_LEVEL
  infisical.yaml        # InfisicalSecret CRD (kubernetesAuth + autoCreateSAToken)
  auto-scaler.yaml      # HPA: min 2, max 4, CPU 80%
  kustomization.yaml    # Dynamic resource list
  disruption-budget.yaml    # (only if has_disruption_budget=true)
  service-account.yaml      # (only if has_service_account=true)
envs/
  dev/
    kustomization.yaml  # Image tag pin + patches + base ref + ingress
    deployment.yaml     # Patch: NODE_ENV, OTEL attributes (env=d)
    config.yaml         # Patch: CORS_CONFIG with dev origins
    ingress.yaml        # Traefik + cert-manager (letsencrypt-dev)
  prd/
    kustomization.yaml
    deployment.yaml     # Patch: NODE_ENV, OTEL attributes (env=p)
    config.yaml         # Patch: CORS_CONFIG with prd origins
    ingress.yaml        # Traefik + cert-manager (letsencrypt-prod)
```
