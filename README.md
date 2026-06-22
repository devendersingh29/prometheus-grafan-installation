# prometheus-grafan-installation

# Grafana Setup on Kubernetes 📊

## Prerequisites
- Kubernetes Cluster (1 Master + 2 Worker Nodes)
- Helm Installed
- kubectl Configured

---

## Architecture

```
K8s Cluster
    ↓
Node Exporter       ← Har node ki metrics collect karta hai
    ↓
Prometheus          ← Metrics store karta hai
    ↓
Grafana             ← Metrics visualize karta hai 📊
```

---

## Step 1: Helm Install Karo

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify karo:
```bash
helm version
```

---

## Step 2: Prometheus Community Repo Add Karo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## Step 3: Monitoring Namespace Banao

```bash
kubectl create namespace monitoring
```

---

## Step 4: Chart Pull Karo (Optional)

```bash
mkdir -p ~/monitoring
cd ~/monitoring
helm pull prometheus-community/kube-prometheus-stack --untar
```

Folder Structure:
```
monitoring/
└── kube-prometheus-stack/
    ├── Chart.yaml
    ├── values.yaml       ← Custom config yahan karo
    ├── templates/
    └── charts/
        ├── grafana/
        └── alertmanager/
```

---

## Step 5: Install Karo

### Default Values Se:
```bash
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```

### Custom Values Se (Agar values.yaml edit kiya ho):
```bash
helm install prometheus ~/monitoring/kube-prometheus-stack --namespace monitoring -f ~/monitoring/kube-prometheus-stack/values.yaml
```

---

## Step 6: Verify Karo

```bash
# Pods check karo
kubectl get pods -n monitoring

# Services check karo
kubectl get svc -n monitoring
```

Expected Output:
```
NAME                                      TYPE        PORT(S)
prometheus-grafana                        ClusterIP   80/TCP
prometheus-kube-prometheus-alertmanager   ClusterIP   9093/TCP
prometheus-kube-prometheus-prometheus     ClusterIP   9090/TCP
prometheus-node-exporter                  ClusterIP   9100/TCP
```

---

## Step 7: Grafana Access Karo

### NodePort Mein Change Karo:
```bash
kubectl patch svc prometheus-grafana \
  -n monitoring \
  -p '{"spec": {"type": "NodePort"}}'
```

### NodePort Number Lo:
```bash
kubectl get svc prometheus-grafana -n monitoring
```

### Admin Password Lo:
```bash
kubectl get secret prometheus-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

### Browser Mein Access Karo:
```
http://<NODE_PUBLIC_IP>:<NODEPORT>
Username: admin
Password: <upar wala command se mila>
```

---

## Step 8: AWS Security Group Ports Open Karo

EC2 → Security Groups → Inbound Rules → Add:

| Port | Protocol | Kya Hai |
|---|---|---|
| 10250 | TCP | Kubelet (port-forward) |
| 30000-32767 | TCP | NodePort Range |
| 9090 | TCP | Prometheus |
| 3000 | TCP | Grafana (agar port-forward) |

---

## Helm Useful Commands

```bash
# Installed charts dekho
helm list -A

# Chart upgrade karo
helm upgrade prometheus ~/monitoring/kube-prometheus-stack \
  --namespace monitoring \
  -f ~/monitoring/kube-prometheus-stack/values.yaml

# Chart uninstall karo
helm uninstall prometheus -n monitoring

# Chart status dekho
helm status prometheus -n monitoring
```

---

## Troubleshooting

### Port Forward Error:
```
error: dial tcp 172.31.x.x:10250: i/o timeout
```
Fix: AWS Security Group mein port 10250 open karo.

### Grafana Pod Not Ready:
```bash
kubectl describe pod <grafana-pod> -n monitoring
kubectl logs <grafana-pod> -n monitoring
```

### Public IP Find Karo:
```bash
curl ifconfig.me
# Ya
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

---

## What Gets Installed

| Component | Kaam |
|---|---|
| **Prometheus** | Metrics collect & store karta hai |
| **Grafana** | Dashboards & visualization |
| **AlertManager** | Alerts bhejta hai |
| **Node Exporter** | Node metrics (CPU, RAM, Disk) |
| **Kube State Metrics** | K8s resources metrics |

---

## Default Ports

| Service | Port |
|---|---|
| Grafana | 3000 |
| Prometheus | 9090 |
| AlertManager | 9093 |
| Node Exporter | 9100 |
