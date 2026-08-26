# VCF Demo Infrastructure (`vcf-demo-infra`)

Platform GitOps repository for managing VMware Cloud Foundation (VCF) Supervisor resources, VKS guest clusters, and VKS Add-ons (`cert-manager`, `istio`).

## Architecture Overview

This repository decouples **Platform Infrastructure** from **Application Delivery**:

1. **Control Plane Supervisor Namespace (`infra-fbhdn`)**:
   - Hosts ArgoCD Control Plane and `AppProject` declarations (`infra.yaml` and `tenant-apps.yaml`).
2. **Workload Supervisor Namespace (`prod-2r8k2`)**:
   - Hosts `vks-argo` VKS Guest Cluster Custom Resource and `AddonInstall` Custom Resources (`vks-argo-cert-manager`, `vks-argo-istio`).
3. **Application Workload Guest Cluster (`vks-argo`)**:
   - Workload applications (`bookstore`, `reader`, `chatbot`) are managed via `DemoApp/argocd-apps/apps.yaml` pointing to external application repositories.

## Directory Structure

```
vcf-demo-infra/
├── instance/
│   └── argo-instance.yaml    # ArgoCD Supervisor Service CR targeting infra-fbhdn
├── argocd/
│   ├── projects/
│   │   ├── infra.yaml            # Platform AppProject
│   │   └── tenant-apps.yaml      # Tenant AppProject
│   ├── appsets/
│   │   └── cluster-provisioning.yaml # ApplicationSet for VKS cluster CRDs & add-ons
│   └── root-app.yaml             # Root Application driving 100% GitOps
├── infrastructure/
│   ├── clusters/
│   │   ├── base/                 # CAPI Cluster base template
│   │   └── overlays/prod/        # Prod overlay (Supervisor NS: prod-2r8k2, Name: vks-argo)
│   └── addons/
│       ├── base/                 # cert-manager & istio AddonInstall base
│       └── overlays/prod/        # Prod overlay (Supervisor NS: prod-2r8k2, Prefix: vks-argo-)
└── README.md
```

## Quick Start

1. Deploy ArgoCD instance to the Supervisor namespace (`infra-fbhdn`):
   ```bash
   kubectl apply -f instance/argo-instance.yaml -n infra-fbhdn
   ```

2. Register destination cluster/namespace and apply Root Application (100% GitOps):
   ```bash
   kubectl apply -f argocd/root-app.yaml -n infra-fbhdn
   ```

3. Deploy cluster infrastructure & VKS Add-ons to `prod-2r8k2` (or let ArgoCD sync via `root-infra`):
   ```bash
   kubectl kustomize infrastructure/clusters/overlays/prod | kubectl apply -f -
   kubectl kustomize infrastructure/addons/overlays/prod | kubectl apply -f -
   ```
