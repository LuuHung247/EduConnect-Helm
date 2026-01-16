# EduConnect Helm Chart - Test Commit 

Helm chart for deploying EduConnect microservices platform on Kubernetes (K3s).

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Traefik Ingress                      │
│                      (educonnect.domain.com)                 │
└───────────────────────┬─────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
    ┌───▼───┐      ┌────▼────┐    ┌────▼────┐
    │Backend│      │  User   │    │Frontend │
    │ :5001 │◄─────┤ Service │    │  :80    │
    └───┬───┘      │  :5002  │    └─────────┘
        │          └────┬────┘
        │               │
        └───────┬───────┘
                │
         ┌──────▼──────┐
         │ MongoDB     │
         │ Atlas       │
         └─────────────┘
```

## Services

- **Backend** (Port 5001): API Gateway, handles /api/v1/* routes
- **User Service** (Port 5002): User management microservice
- **Media Service** (Port 8001): File upload and media processing
- **Tracking Service** (Port 8002): User activity tracking
- **Frontend** (Port 80): React SPA

## Prerequisites

1. **K3s Cluster** running with Traefik enabled
2. **Helm 3** installed
3. **kubectl** configured to access your cluster
4. **MongoDB Atlas** account with connection string
5. **AWS Account** with credentials (S3, SES, Cognito, SQS)

## Quick Start

### 1. Generate values file from .env

```bash
chmod +x generate-values.sh
./generate-values.sh
```

This creates `values-generated.yaml` with your credentials from `../.env`.

### 2. Review generated values

```bash
cat values-generated.yaml
```

**⚠️ WARNING**: This file contains secrets! Never commit it to git.

### 3. Deploy to K3s

```bash
chmod +x deploy.sh
./deploy.sh install
```

### 4. Check deployment status

```bash
./deploy.sh status
```

### 5. View logs

```bash
# Backend logs
./deploy.sh logs backend

# User Service logs
./deploy.sh logs userService
```

## Configuration

Edit `values.yaml` for default configuration.

Key configuration sections:

- `global.domain`: Your domain name
- `global.mongodb`: MongoDB Atlas connection
- `global.aws`: AWS credentials and services
- `global.cognito`: AWS Cognito configuration
- `backend.replicaCount`: Number of backend pods
- `userService.replicaCount`: Number of user service pods

## Deployment Commands

```bash
# Install
./deploy.sh install

# Upgrade after changes
./deploy.sh upgrade

# Check status
./deploy.sh status

# View logs
./deploy.sh logs [service-name]

# Uninstall
./deploy.sh uninstall
```

## Manual Helm Commands

```bash
# Install
helm install educonnect . \
  --namespace educonnect \
  --values values.yaml \
  --values values-generated.yaml \
  --create-namespace

# Upgrade
helm upgrade educonnect . \
  --namespace educonnect \
  --values values.yaml \
  --values values-generated.yaml

# Uninstall
helm uninstall educonnect --namespace educonnect

# Status
helm status educonnect --namespace educonnect
```

## Accessing the Application

### Via Ingress (Production)

```
https://yourdomain.com           → Frontend
https://yourdomain.com/api/v1    → Backend API
```

### Via Port Forward (Development)

```bash
# Backend
kubectl port-forward -n educonnect svc/backend 5001:5001

# User Service
kubectl port-forward -n educonnect svc/userService 5002:5002

# Frontend
kubectl port-forward -n educonnect svc/frontend 8080:80
```

## Troubleshooting

### Pods not starting

```bash
# Check pod status
kubectl get pods -n educonnect

# Describe pod
kubectl describe pod -n educonnect <pod-name>

# Check logs
kubectl logs -n educonnect <pod-name>
```

### Connection issues

```bash
# Check services
kubectl get svc -n educonnect

# Check endpoints
kubectl get endpoints -n educonnect

# Test service connectivity
kubectl run -n educonnect test-pod --image=curlimages/curl -it --rm -- sh
curl http://backend.educonnect.svc.cluster.local:5001/health
```

### MongoDB connection issues

```bash
# Check secrets
kubectl get secret -n educonnect educonnect-secrets -o yaml

# Verify MONGODB_URI is correct
kubectl get secret -n educonnect educonnect-secrets -o jsonpath='{.data.MONGODB_URI}' | base64 -d
```

## Security Notes

1. **Never commit** `values-generated.yaml` or `values-production.yaml`
2. Use Kubernetes secrets for sensitive data
3. Enable RBAC in production
4. Use NetworkPolicies to restrict pod communication
5. Rotate credentials regularly

## Scaling

```bash
# Scale backend manually
kubectl scale deployment backend -n educonnect --replicas=5

# Enable HPA (auto-scaling)
# Already configured in values.yaml:
# - backend: 2-10 replicas based on CPU
# - userService: 2-5 replicas based on CPU
```

## Monitoring

```bash
# Watch pods
kubectl get pods -n educonnect -w

# Resource usage
kubectl top pods -n educonnect
kubectl top nodes
```

## Development

To modify the chart:

1. Edit files in `templates/` or `charts/*/templates/`
2. Test locally: `helm template educonnect . --values values-generated.yaml`
3. Validate: `helm lint .`
4. Deploy: `./deploy.sh upgrade`

## Support

For issues, contact the EduConnect team or check logs:

```bash
./deploy.sh logs backend
./deploy.sh logs userService
```
