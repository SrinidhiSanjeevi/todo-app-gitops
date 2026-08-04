# Todo App GitOps Infrastructure Repository

Infrastructure and GitOps deployment manifests for the MERN Todo application on **Azure Kubernetes Service (AKS)** using **Helm**, **ArgoCD**, **Prometheus**, and **Grafana**.

---

## 📁 Repository Structure

```
todo-app-gitops/
├── kubernetes/
│   ├── namespace.yaml                # Isolated todo-app namespace
│   ├── configmap.yaml                # Application environment configuration
│   ├── secret.yaml                   # Base64 encoded secrets
│   ├── backend-deployment.yaml       # Backend Pod spec, non-root USER, health probes
│   ├── backend-service.yaml          # ClusterIP service exposing backend on port 5000 & 9100
│   ├── frontend-deployment.yaml      # Frontend Pod spec, Nginx unprivileged USER
│   ├── frontend-service.yaml         # ClusterIP service exposing frontend on port 80
│   ├── ingress.yaml                  # NGINX Ingress rules routing / and /api
│   └── hpa.yaml                      # Horizontal Pod Autoscaler (2-5 replicas)
├── helm/
│   └── todo-app/                     # Production Helm Chart
│       ├── Chart.yaml                # Helm chart metadata
│       ├── values.yaml               # Parameterized configurations (image tag, resources, etc.)
│       └── templates/                # DRY Kubernetes templates
│           ├── _helpers.tpl
│           ├── namespace.yaml
│           ├── configmap.yaml
│           ├── secret.yaml
│           ├── backend-deployment.yaml
│           ├── backend-service.yaml
│           ├── frontend-deployment.yaml
│           ├── frontend-service.yaml
│           ├── ingress.yaml
│           └── hpa.yaml
├── argocd-application.yaml           # ArgoCD GitOps Application CRD
├── servicemonitor.yaml               # Prometheus Operator ServiceMonitor
├── grafana-dashboard.json            # Pre-configured Grafana Dashboard JSON
└── README.md
```

---

## ⚡ Deployment Workflows

### Option 1: Direct Manual Deployment with kubectl
```bash
# Apply namespace
kubectl apply -f kubernetes/namespace.yaml

# Apply configuration and secrets
kubectl apply -f kubernetes/configmap.yaml
kubectl apply -f kubernetes/secret.yaml

# Apply workloads and services
kubectl apply -f kubernetes/backend-deployment.yaml
kubectl apply -f kubernetes/backend-service.yaml
kubectl apply -f kubernetes/frontend-deployment.yaml
kubectl apply -f kubernetes/frontend-service.yaml
kubectl apply -f kubernetes/ingress.yaml
kubectl apply -f kubernetes/hpa.yaml
```

### Option 2: Deployment via Helm
```bash
# Install / Upgrade Helm release
helm upgrade --install todo-app ./helm/todo-app \
  --namespace todo-app \
  --create-namespace \
  --set backend.image.tag=latest \
  --set frontend.image.tag=latest
```

### Option 3: GitOps Deployment with ArgoCD
```bash
# Apply ArgoCD Application CRD
kubectl apply -f argocd-application.yaml -n argocd

# ArgoCD automatically detects changes in values.yaml and syncs with AKS!
```

---

## 📊 Observability Setup

### 1. Prometheus Scrape Configuration
Apply `servicemonitor.yaml` to allow Prometheus Operator to scrape metrics from backend `/metrics` endpoint:
```bash
kubectl apply -f servicemonitor.yaml -n todo-app
```

### 2. Grafana Dashboard Import
1. Open Grafana Dashboard UI.
2. Navigate to **Dashboards** -> **Import**.
3. Copy and paste the contents of `grafana-dashboard.json` or upload the JSON file.
4. Select your Prometheus data source and click **Import**.
