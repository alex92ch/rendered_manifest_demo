# GitOps Workflow for Grafana Deployment

## Overview

This repository implements a GitOps workflow where changes to Grafana configurations are automatically rendered and deployed through a review process.

## Workflow

### 1. Source Changes
- Developers make changes to the Grafana configurations in `{env}/grafana/` directories on the `main` branch
- This includes modifications to `kustomization.yaml`, `values.yaml`, or any resources

### 2. Automated Rendering
When changes are pushed to `main`, the GitHub workflow automatically:
- Renders the Kustomize manifests with Helm charts for each environment (int, tst, prd)
- Creates review branches with timestamp: `{env}/grafana-review-YYYYMMDD-HHMMSS`
- Pushes the rendered manifests to these review branches
- Creates pull requests from review branches to live branches

### 3. Review Process
- Pull requests are created with detailed information about the changes
- Team members can review the rendered manifests before deployment
- PRs are labeled with environment and automation tags for easy filtering

### 4. Deployment
- Once PRs are approved and merged to live branches (`{env}/grafana`), ArgoCD automatically syncs the changes
- ArgoCD applications watch their respective live branches:
  - `int/grafana` branch for INT environment
  - `tst/grafana` branch for TST environment  
  - `prd/grafana` branch for PRD environment

## Branch Structure

```
main                    # Source configurations
├── int/grafana/        # INT environment configs
├── tst/grafana/        # TST environment configs
└── prd/grafana/        # PRD environment configs

# Live branches (ArgoCD watches these)
int/grafana            # Rendered manifests for INT
tst/grafana            # Rendered manifests for TST  
prd/grafana            # Rendered manifests for PRD

# Review branches (temporary)
int/grafana-review-*   # Review branches for INT
tst/grafana-review-*   # Review branches for TST
prd/grafana-review-*   # Review branches for PRD
```

## ArgoCD Applications

Each environment has its own ArgoCD application that:
- Watches the corresponding live branch (`{env}/grafana`)
- Deploys to the `monitoring` namespace
- Has automated sync with self-healing enabled
- Creates the namespace if it doesn't exist

## Benefits

1. **Safety**: No direct deployment - all changes go through review
2. **Transparency**: Rendered manifests are visible before deployment
3. **Audit Trail**: Git history shows exactly what was deployed when
4. **Rollback**: Easy to revert by reverting commits on live branches
5. **Environment Isolation**: Each environment has its own branch and review process

## Required Secrets

- `DEPLOY_PAT`: GitHub Personal Access Token with repository and PR creation permissions

## Labels on PRs

- `automated`: Indicates this is an automated PR
- `manifests`: Indicates this contains rendered manifests
- `{env}`: Environment-specific label (int, tst, or prd)
