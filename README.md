# EKS Canary Rollback — GitOps

Kubernetes manifests and Argo CD configuration for the [eks-canary-rollback](https://github.com/CloudDev-x/eks-canary-rollback-app) project.

## Structure

├── bootstrap/
│ └── root.yaml # Argo CD app-of-apps entry point
├── platform/
│ ├── argo-rollouts.yaml # Argo Rollouts Application (Helm chart)
│ ├── kube-prometheus-stack.yaml # Prometheus + Grafana Application
│ └── workloads.yaml # Workloads Application (recursive, auto-sync)
└── apps/
├── api/
│ ├── rollout.yaml # Argo Rollout with canary strategy
│ ├── analysis-template.yaml # Prometheus 5xx rate check
│ ├── podmonitor.yaml # Prometheus scrape config
│ ├── service.yaml # Main ClusterIP service
│ ├── stable-service.yaml # Stable pod selector
│ └── canary-service.yaml # Canary pod selector
├── frontend/
│ └── deployment.yaml
├── worker/
│ └── deployment.yaml
└── loadgen/
└── deployment.yaml # Traffic generator for canary metrics


## How it works

Argo CD watches this repo via the `root` Application. Changes here (typically image tag bumps committed by CI) trigger automatic sync.

The API service uses an Argo Rollouts canary strategy:
- **Steps**: 20% → pause 30s → analysis → 50% → pause 30s → 100%
- **Analysis**: Prometheus query checks `http_requests_total` 5xx ratio for the canary pods using `pod_template_hash` label
- **Rollback**: if 5xx rate ≥ 5%, the Rollout aborts and removes canary pods

Image tags are updated by the CI pipeline in the [app repo](https://github.com/CloudDev-x/eks-canary-rollback-app) via a GitHub App.

## Related repo

- [eks-canary-rollback-app](https://github.com/CloudDev-x/eks-canary-rollback-app) — Application code, Terraform infrastructure, and CI pipeline
