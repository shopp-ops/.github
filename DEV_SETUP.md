# Local Development Setup

## How It Works

The local development environment is split into two layers:

**Kind cluster** — runs in Docker and acts as the local Kubernetes environment. It hosts the infrastructure that applications depend on: the PostgreSQL operator (CNPG), the NGINX ingress controller, and the Shop operator. Once set up, this cluster stays running in the background.

**Shop and ShopHub applications** — can be run directly on local machine. They connect to the cluster over port-forward for database access, and talk to the cluster API directly to deploy Shop instances. 

**In production**, the cluster is permanent and managed through GitOps — Flux watches the `kube-state` repository and automatically applies any changes to the cluster. Applications are deployed as Docker images built and published by the CI/CD pipeline.

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows) or Docker Engine (Linux) — must be running before anything else

---

## 1. Install Required Tools

### Linux

```bash
# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/kubectl

# helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind && sudo mv kind /usr/local/bin/kind
```

### Windows (Chocolatey)

Install Chocolatey if you don't have it: https://chocolatey.org/install

Run in an elevated PowerShell prompt:

```powershell
choco install kubernetes-cli helm kind -y
```

Verify:

```bash
kubectl version --client
helm version
kind version
```

---

## 2. Create the Local Cluster

The `kind-config.yaml` is in the root of the `helm-charts` repository. From that directory:

```bash
kind create cluster --config kind-config.yaml
```

Verify:

```bash
kubectl get nodes
# Expected: one node in Ready state
```

---

## 3. Install Cluster Infrastructure

This installs the CNPG operator (PostgreSQL), NGINX ingress controller, and the Shop operator into the cluster. These are installed once and stay running.

```bash
helm upgrade --install shoppops-infra ./charts/shoppops-infra --create-namespace
```

Wait for CRDs to be registered before proceeding:

```bash
kubectl wait --for=condition=established \
  crd/clusters.postgresql.cnpg.io --timeout=60s
```

Verify everything is running:

```bash
kubectl get pods -A
# All pods should be Running or Completed
```

---

## 4. Deploy Application Dependencies

Create a local values override file — this is gitignored and never committed:

```bash
cp charts/shophub/values.local.example.yaml charts/shophub/values.local.yaml
# edit values.local.yaml with your local credentials
```

Deploy ShopHub's PostgreSQL instance into the cluster:

```bash
helm upgrade --install shophub ./charts/shophub \
  --namespace shophub \
  --create-namespace \
  -f charts/shophub/values.local.yaml
```

---

## 5. Run Applications Locally

### ShopHub

Port-forward the database so the app can reach it:

```bash
kubectl port-forward svc/shophub-db-rw 5432:5432 -n shophub
```

In a separate terminal, start the app:

```bash
cd shophub
cp .env.example .env.local
# edit .env.local — DATABASE_URL should point to localhost:5432
npm install
npm run dev
```

ShopHub will connect to the database in the cluster and use the local kubeconfig to deploy Shop instances when users create stores through the UI.

### Shop

Shop does not depend on the cluster during development. Run it standalone:

```bash
cd shop
cp .env.example .env.local
npm install
npm run dev
```

---

## Useful Commands

```bash
# Delete and recreate the cluster from scratch
kind delete cluster --name shoppops
kind create cluster --config kind-config.yaml

# Check what is running in the cluster
kubectl get pods -A

# Test deploying a Shop instance manually
kubectl apply -f examples/shop-cr.yaml
kubectl get shops -A
```
