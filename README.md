# Label Studio Kubernetes Manifests with Kustomize

This directory contains Kubernetes manifests for deploying Label Studio using Kustomize.

## Directory Structure

```
label-studio-manifests/
├── base/
│   ├── kustomization.yaml    # Base configuration
│   └── label-studio.yaml     # All Label Studio resources (Deployment, Services, PVCs, Jobs)
├── overlays/
│   ├── dev/                  # Development environment overrides
│   │   ├── kustomization.yaml
│   │   ├── storage-patch.yaml      # 5Gi storage
│   │   └── resources-patch.yaml    # Smaller resource limits
│   └── prod/                 # Production environment overrides
│       ├── kustomization.yaml
│       ├── storage-patch.yaml      # 100Gi storage
│       ├── resources-patch.yaml    # Larger resource limits
│       └── replica-patch.yaml      # 2 replicas for HA
└── README.md
```

## Components

The deployment includes the following components:

### Label Studio Application
- **Main Application**: Web UI and API server
- **Worker**: Background task processing for data labeling
- **Initialization Job**: Database migration, static files collection, and admin user creation

### Dependencies
- **PostgreSQL**: Primary database for storing projects, tasks, and annotations
- **Redis**: Message broker for Celery workers

## Deployment Options

### Development Environment

```bash
# Deploy to dev namespace
kubectl apply -k overlays/dev

# Check the deployment
kubectl get pods -n label-studio-dev
kubectl get svc -n label-studio-dev
```

### Production Environment

```bash
# Deploy to prod namespace
kubectl apply -k overlays/prod

# Check the deployment
kubectl get pods -n label-studio-prod
kubectl get svc -n label-studio-prod
```

### Base Configuration

```bash
# Deploy with base configuration
kubectl apply -k base

# Access Label Studio using port-forward:
kubectl port-forward -n default svc/label-studio-service 8080:80
# Then visit: http://localhost:8080
# Default credentials: admin/admin
```

## Accessing Label Studio

### Application (Port 8080)
- Internal cluster: `label-studio-service.namespace.svc.cluster.local:80`

### External Access Options
For external access, consider using:
- **Ingress**: Configure an Ingress controller to route traffic to the ClusterIP service
- **kubectl port-forward**: `kubectl port-forward -n namespace svc/label-studio-service 8080:80`
- **LoadBalancer**: Change service type to LoadBalancer if your cluster supports it

### Default Credentials
- Username: `admin`
- Password: `admin`

**Important**: Change the default credentials before using in production!
- Update the `label-studio-secrets` Secret with your desired credentials
- Use `kubectl create secret generic label-studio-secrets --from-literal=LABEL_STUDIO_USERNAME=youruser --from-literal=LABEL_STUDIO_PASSWORD=yourpass --from-literal=POSTGRES_PASSWORD=yourdbpass`

## Configuration

### Environment Variables

The following environment variables can be customized:

#### Label Studio Configuration
- `LABEL_STUDIO_USERNAME`: Default admin username
- `LABEL_STUDIO_PASSWORD`: Default admin password

#### Database Configuration
- `DJANGO_DB_ENGINE`: Database backend (PostgreSQL recommended)
- `POSTGRES_HOST`: PostgreSQL service hostname
- `POSTGRES_PORT`: PostgreSQL port (default: 5432)
- `POSTGRES_USER`: PostgreSQL username
- `POSTGRES_PASSWORD`: PostgreSQL password
- `POSTGRES_DB`: PostgreSQL database name

#### Celery Configuration
- `CELERY_BROKER_URL`: Redis broker URL
- `CELERY_RESULT_BACKEND`: Redis result backend URL

### Customization

#### Environment-Specific Variables

Edit the patch files in overlays/dev or overlays/prod to customize:
- Storage size for PostgreSQL
- Resource limits/requests for all components
- Replica count for high availability
- Environment variables

#### Adding New Environments

1. Create a new directory under `overlays/`
2. Create a `kustomization.yaml` that references `../../base`
3. Add patch files as needed

### Monitoring and Health Checks

All deployments include health checks:
- **Liveness Probe**: Ensures the container is running
- **Readiness Probe**: Ensures the container is ready to serve traffic

### Persistent Storage

- PostgreSQL uses a PersistentVolumeClaim for data persistence
- Default storage sizes: 5Gi (dev), 100Gi (prod)
- Storage class can be customized based on your cluster setup

## Scaling

### Horizontal Scaling
- **Label Studio Workers**: Can be scaled horizontally by adjusting replicas
- **Label Studio Application**: Production overlay includes 2 replicas for HA

### Resource Scaling
- CPU and memory limits are configured per environment
- Adjust based on your workload and usage patterns

## Maintenance

### Database Backups
Implement regular backups for PostgreSQL:
```bash
# Example backup command
kubectl exec -n namespace deployment/postgresql -- pg_dump -U labelstudio labelstudio > backup.sql
```

### Updates
```bash
# Apply updates
kubectl apply -k overlays/environment

# Check rollout status
kubectl rollout status deployment/label-studio -n namespace
```

## Troubleshooting

### Common Issues

1. **Migration Job Fails**: Check database connectivity and credentials
2. **Workers Not Processing**: Verify Redis connection and Celery configuration
3. **High Memory Usage**: Adjust resource limits and monitor task complexity

### Logs
```bash
# Label Studio logs
kubectl logs -n namespace deployment/label-studio -f

# Worker logs
kubectl logs -n namespace deployment/label-studio-worker -f

# Database logs
kubectl logs -n namespace deployment/postgresql -f
```

## Security Considerations

1. **Change Default Credentials**: Update admin username and password in the `label-studio-secrets` Secret
2. **Network Policies**: Implement network policies to restrict traffic between components
3. **RBAC**: Use proper RBAC for Kubernetes access and service accounts
4. **Secrets Management**: The manifests use Kubernetes secrets for sensitive data
5. **TLS**: Enable TLS for external access using Ingress with SSL termination
6. **Image Security**: Images are pinned to specific versions and use `IfNotPresent` pull policy

## Cleanup

```bash
# Delete dev deployment
kubectl delete -k overlays/dev

# Delete prod deployment
kubectl delete -k overlays/prod

# Delete base deployment
kubectl delete -k base
```

## Version Information

- **Label Studio**: `heartexlabs/label-studio:1.14.0`
- **PostgreSQL**: `postgres:15-alpine`
- **Redis**: `redis:7-alpine`

These manifests use pinned versions for better stability and security in production deployments.