# 04-helm-charts

This module demonstrates **Helm package management** for Kubernetes applications. Learn how to use pre-built Helm charts to deploy, manage, and scale your microservices application.

## 🎯 Learning Objectives

- **Package Management**: Deploy applications using Helm charts
- **Configuration Management**: Customize deployments with values.yaml
- **Release Management**: Install, upgrade, and rollback applications
- **Environment Management**: Deploy to multiple environments
- **Dependency Management**: Handle complex application dependencies

## 📁 Directory Structure

```
04-helm-charts/
├── README.md                    # This file
├── microservices-app/           # Helm chart for the microservices application
│   ├── Chart.yaml              # Chart metadata and dependencies
│   ├── values.yaml             # Default configuration values
│   └── templates/              # Kubernetes manifest templates
│       ├── _helpers.tpl        # Template helper functions
│       ├── namespace.yaml      # Namespace template
│       └── mongodb.yaml        # MongoDB deployment template
└── examples/                   # Example configurations
    ├── values-dev.yaml         # Development environment values
    ├── values-staging.yaml     # Staging environment values
    └── values-production.yaml  # Production environment values
```

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster running (Kind cluster from Module 2)
- Helm 3.x installed
- Docker images built (from Module 2: `./scripts/build-images.sh`)

### Deploy the Application

```bash
# Navigate to the Helm charts directory
cd 04-helm-charts

# Install the microservices application
helm install microservices-app ./microservices-app \
  --namespace microservices-app \
  --create-namespace

# Check the deployment
helm list -n microservices-app
kubectl get pods -n microservices-app
```

### Access the Application

```bash
# Test the API Gateway
curl http://k8s-tutorial.local/api/gateway/health

# Access the frontend
open http://k8s-tutorial.local
```

## ⚙️ Configuration

### Default Configuration

The chart comes with sensible defaults in `values.yaml`:

```yaml
replicaCount:
  frontend: 2
  apiGateway: 2
  userService: 2
  productService: 2

image:
  repository: 
    frontend: frontend
    apiGateway: api-gateway
    userService: user-service
    productService: product-service
  tag: "latest"

resources:
  frontend:
    limits:
      cpu: 250m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi
```

### Custom Configuration

Override values during installation:

```bash
# Install with custom replica counts
helm install microservices-app ./microservices-app \
  --set replicaCount.frontend=3 \
  --set replicaCount.userService=5 \
  --namespace microservices-app \
  --create-namespace

# Install with custom values file
helm install microservices-app ./microservices-app \
  -f examples/values-production.yaml \
  --namespace microservices-app \
  --create-namespace
```

## 🔄 Release Management

### Upgrade Applications

```bash
# Upgrade with new configuration
helm upgrade microservices-app ./microservices-app \
  --set replicaCount.frontend=4 \
  -n microservices-app

# Upgrade with new values file
helm upgrade microservices-app ./microservices-app \
  -f examples/values-production.yaml \
  -n microservices-app
```

### Rollback Applications

```bash
# View release history
helm history microservices-app -n microservices-app

# Rollback to previous version
helm rollback microservices-app -n microservices-app

# Rollback to specific revision
helm rollback microservices-app 1 -n microservices-app
```

### Monitor Releases

```bash
# Check release status
helm status microservices-app -n microservices-app

# List all releases
helm list --all-namespaces

# Get release values
helm get values microservices-app -n microservices-app
```

## 🌍 Multi-Environment Deployment

### Development Environment

```bash
# Create development values
cat > examples/values-dev.yaml << 'EOF'
replicaCount:
  frontend: 1
  apiGateway: 1
  userService: 1
  productService: 1

resources:
  frontend:
    limits:
      cpu: 100m
      memory: 128Mi
    requests:
      cpu: 50m
      memory: 64Mi

mongodb:
  persistence:
    size: 500Mi

postgres:
  persistence:
    size: 500Mi
EOF

# Deploy to development
helm install microservices-dev ./microservices-app \
  -f examples/values-dev.yaml \
  --namespace microservices-dev \
  --create-namespace
```

### Production Environment

```bash
# Create production values
cat > examples/values-production.yaml << 'EOF'
replicaCount:
  frontend: 5
  apiGateway: 3
  userService: 5
  productService: 3

resources:
  frontend:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 200m
      memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

mongodb:
  persistence:
    size: 10Gi
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi

postgres:
  persistence:
    size: 20Gi
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
EOF

# Deploy to production
helm install microservices-prod ./microservices-app \
  -f examples/values-production.yaml \
  --namespace microservices-prod \
  --create-namespace
```

## 🧪 Testing and Validation

### Template Testing

```bash
# Generate templates without installing
helm template microservices-app ./microservices-app \
  --namespace microservices-app

# Test with custom values
helm template microservices-app ./microservices-app \
  --set replicaCount.frontend=3 \
  --namespace microservices-app

# Validate against Kubernetes API
helm template microservices-app ./microservices-app | \
  kubectl apply --dry-run=client -f -
```

### Chart Validation

```bash
# Lint the chart
helm lint ./microservices-app

# Show chart information
helm show chart ./microservices-app
helm show values ./microservices-app
helm show readme ./microservices-app
```

## 🔧 Troubleshooting

### Common Issues

**1. Ingress Not Working**
```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress resource
kubectl describe ingress -n microservices-app

# Verify host entry in /etc/hosts
echo "127.0.0.1 k8s-tutorial.local" | sudo tee -a /etc/hosts
```

**2. Pods Not Starting**
```bash
# Check pod status
kubectl get pods -n microservices-app

# Check pod logs
kubectl logs -n microservices-app deployment/frontend

# Check events
kubectl get events -n microservices-app --sort-by='.lastTimestamp'
```

**3. Images Not Found**
```bash
# Rebuild and load images into Kind
cd ../scripts
./build-images.sh

# Check if images are loaded
docker images | grep -E "(frontend|api-gateway|user-service|product-service)"
```

### Debug Commands

```bash
# Get all resources
kubectl get all -n microservices-app

# Describe problematic resources
kubectl describe deployment frontend -n microservices-app

# Check Helm release status
helm status microservices-app -n microservices-app

# Get Helm release history
helm history microservices-app -n microservices-app
```

## 📊 Monitoring and Observability

### Health Checks

```bash
# Test all service health endpoints
curl http://k8s-tutorial.local/api/gateway/health
curl http://k8s-tutorial.local/api/users/health
curl http://k8s-tutorial.local/api/products/health
```

### Resource Usage

```bash
# Check resource usage
kubectl top pods -n microservices-app
kubectl top nodes

# Check resource limits
kubectl describe pods -n microservices-app | grep -A 5 "Limits\|Requests"
```

## 🚀 Advanced Usage

### Custom Helm Values

Create environment-specific configurations:

```yaml
# values-custom.yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: myapp.example.com

mongodb:
  enabled: false  # Use external MongoDB

externalMongoDB:
  connectionString: "mongodb://external-mongo:27017/myapp"
```

### Helm Hooks

Add pre/post-install hooks for database migrations:

```yaml
# templates/migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "microservices-app.fullname" . }}-migration
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
spec:
  template:
    spec:
      containers:
      - name: migration
        image: migrate/migrate
        command: ["migrate", "-path", "/migrations", "-database", "postgres://...", "up"]
```

## 🎓 Learning Outcomes

After completing this module, you'll understand:

- ✅ **Helm Chart Structure**: Chart.yaml, values.yaml, templates/
- ✅ **Package Management**: Install, upgrade, rollback applications
- ✅ **Configuration Management**: Environment-specific deployments
- ✅ **Release Lifecycle**: Managing application versions
- ✅ **Template System**: Basic understanding of Helm templating
- ✅ **Production Patterns**: Multi-environment deployment strategies

## 🔗 Related Modules

- **Module 3**: `/03-k8s-manifests/` - Raw Kubernetes YAML (what this chart packages)
- **Module 5**: `/05-helm-from-scratch/` - Creating charts from scratch
- **Module 2**: `/02-kind-setup/` - Kubernetes cluster setup
- **Sample App**: `/sample-app/` - Application source code

## 📚 Additional Resources

- [Helm Official Documentation](https://helm.sh/docs/)
- [Helm Chart Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Artifact Hub](https://artifacthub.io/) - Discover and share Kubernetes packages
- [Helm Chart Repository Guide](https://helm.sh/docs/topics/chart_repository/)

---

**Ready to master Helm package management? Start deploying!** 🚀
