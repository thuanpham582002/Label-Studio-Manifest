# Label Studio Kubernetes Manifests

Quick deployment of Label Studio using the official Helm chart.

## 🚀 Deploy

```bash
# Apply the manifest
kubectl apply -f label-studio.yaml

# Port forward to access
kubectl port-forward -n label-studio svc/label-studio-label-studio 8080:8080
```

Visit: http://localhost:8080

Default credentials: `admin/admin`

## 📦 Components

- **Label Studio**: Web UI & API
- **PostgreSQL**: Database (1Gi storage)
- **Redis**: Message broker
- **Workers**: Background task processing

## ⚙️ Customize

Edit `label-studio.yaml` directly or generate with custom values:

```bash
helm template label-studio heartex/label-studio \
  --namespace label-studio \
  --values your-values.yaml > label-studio.yaml
```

## 🗑️ Cleanup

```bash
kubectl delete -f label-studio.yaml
kubectl delete namespace label-studio
```

---

**Note**: Uses official [heartex/label-studio](https://github.com/HumanSignal/label-studio) Helm chart.