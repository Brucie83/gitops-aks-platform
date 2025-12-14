# gitops-aks-platform

## GitOps Delivery with ArgoCD (Kubernetes / kind)

This repository represents the **GitOps delivery layer** of a Kubernetes platform using **ArgoCD**.

It is intentionally focused on **deployment and reconciliation**, not on application build or CI concerns.

---

## Purpose

The goal of this repository is to:

- Define the **desired state** of applications running in Kubernetes
- Allow ArgoCD to continuously **reconcile Git state vs cluster state**
- Enforce a **GitOps operating model**:
  - No manual deployments
  - No `kubectl apply` for applications
  - Git is the single source of truth

This repository **does not** build container images and **does not** run CI pipelines.

---

## Responsibilities Boundary

### What this repository DOES

- Stores Kubernetes manifests (`Deployment`, `Service`, etc.)
- Defines how applications should run in the cluster
- Is monitored by ArgoCD for changes
- Enables:
  - Automated sync
  - Drift detection
  - Self-healing
  - Git-driven rollback

### What this repository DOES NOT do

- Build container images
- Run tests
- Decide when a deployment should happen
- Manage application source code

Those responsibilities belong to the **application repositories and their CI pipelines**.

---

## Repository Structure

```text
apps/
  container-health/
    deployment.yaml
    service.yaml
```

Each folder under `apps/` represents a deployable application managed by ArgoCD.

---

## GitOps Flow

``` text
Application Repository (CI)
  └─ Build & publish container image (GHCR)

GitOps Repository (this)
  └─ Update Kubernetes manifests
     └─ ArgoCD detects Git change
        └─ Cluster reconciles automatically
```
Key principle:

ArgoCD does not deploy because CI finished.
ArgoCD deploys only because Git changed.

---

## Deployment & Reconciliation Model

ArgoCD Applications point to this repository

Auto-sync is enabled

Any manual change in the cluster is considered configuration drift

Drift is automatically detected and corrected

Rollbacks are performed by reverting Git commits

Example rollback flow:

```bash
git revert <bad-commit>
git push origin main
```
ArgoCD automatically reconciles the cluster back to the previous stable state.

---

## Operational Rules

❌ No kubectl apply for application resources

✅ All changes must go through Git

✅ Rollbacks are Git-driven

✅ The cluster is treated as immutable

---

## Example Application
container-health

- FastAPI-based health check service

- Deployed using:

  - Kubernetes Deployment

  - Kubernetes Service

- Health managed via readiness and liveness probes

Example manifest snippet:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## Local Environment

- Kubernetes distribution: kind

- ArgoCD installed in-cluster

- Target cluster used for GitOps and platform validation

---

## Design Philosophy

This repository intentionally keeps the GitOps layer simple and explicit.

Advanced patterns such as:

- App of Apps

- Progressive delivery (canary / blue-green)

- Environment promotion (dev → staging → prod)

are out of scope for this repository and would only be introduced when platform scale requires them.

---

## Summary

- Git defines the desired state

- ArgoCD enforces that state

- CI and GitOps are decoupled by design

- This repository represents the delivery and reconciliation layer, not the full SDLC