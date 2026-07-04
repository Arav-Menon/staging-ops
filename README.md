# staging-ops

Personal GitOps repository for managing Kubernetes deployments across all applications.

## Purpose

This repository is the single source of truth for the desired state of my Kubernetes clusters. It is not scoped to one project — every application I build, current or future, has its deployment manifests or Helm charts stored here. ArgoCD continuously watches this repository and reconciles cluster state to match it.

There is no manual `kubectl apply`. If a change isn't in this repo, it isn't in the cluster.

## Architecture

```
+-------------+     +-----------+     +--------------+
|  Developer  | --> |    CI     | --> |  Docker Hub  |
+-------------+     +-----------+     +--------------+
                                              |
                                              v
                                     +------------------+
                                     |   staging-ops     |
                                     | (this repository)  |
                                     +------------------+
                                              |
                                              v
                                     +------------------+
                                     |     ArgoCD        |
                                     +------------------+
                                              |
                                              v
                                     +------------------+
                                     |    K8s Cluster   |
                                     +------------------+
```

## GitOps Workflow

1. Developer pushes code to an application repository.
2. GitHub Actions builds and pushes a container image to Docker Hub.
3. CI updates the image tag in this repository (manifest or Helm values).
4. ArgoCD detects the change and syncs it to the cluster.
5. NGINX Ingress routes traffic; cert-manager handles TLS; Sealed Secrets manages encrypted credentials.

## Folder Structure

```
staging-ops/
├── apps/
│   ├── app-one/
│   │   ├── base/
│   │   └── overlays/
│   ├── app-two/
│   └── ...
├── infrastructure/
│   ├── ingress-nginx/
│   ├── cert-manager/
│   └── sealed-secrets/
└── argocd/
    └── applications/
```

## Technology Stack

Kubernetes, ArgoCD, Helm, NGINX Ingress Controller, cert-manager, Sealed Secrets, GitHub Actions, Docker, DigitalOcean Kubernetes.

## Adding a New Application

1. Create a folder under `apps/<app-name>/` with manifests or a Helm chart.
2. Add a Sealed Secret if the app requires credentials.
3. Define an ArgoCD `Application` resource under `argocd/applications/`.
4. Commit and push — ArgoCD picks up the new application automatically.

## Philosophy

Cluster state is derived entirely from this repository. No exceptions, no drift, no undocumented changes. If it isn't here, it doesn't exist in the cluster.

## Future Improvements

- Per-environment overlays (staging / production)
- Automated rollback on failed health checks
- Centralized observability stack (Prometheus/Grafana)
