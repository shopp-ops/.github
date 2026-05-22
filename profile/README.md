# shopp-ops

A platform for dynamically deploying e-commerce storefronts on Kubernetes. Users manage their stores through a central dashboard — each storefront is provisioned as an isolated Kubernetes workload with its own database, wallet, and notification channel.

## Repositories

| Repository | Description |
|---|---|
| [shophub](https://github.com/shopp-ops/shophub) | Management platform where users create and configure their storefronts |
| [shop](https://github.com/shopp-ops/shop) | E-commerce storefront — product browsing, cart, and Web3 crypto checkout |
| [shop-operator](https://github.com/shopp-ops/shop-operator) | Kubernetes operator managing the lifecycle of Shop deployments |
| [helm-charts](https://github.com/shopp-ops/helm-charts) | Helm charts for all platform components |
| [kube-state](https://github.com/shopp-ops/kube-state) | GitOps cluster state managed by Flux |

## Tech Stack

**Applications:** NestJS · Next.js · TypeORM · PostgreSQL  
**Blockchain:** Ethereum Sepolia · MetaMask · wagmi  
**Kubernetes:** Kubebuilder · CNPG · Flux · Helm  
**Observability:** Prometheus · Grafana · Loki · Tempo

## Getting Started

To set up a local development environment, see [DEV_SETUP.md](https://github.com/shopp-ops/.github/blob/main/DEV_SETUP.md).
