# Kubernetes Senior Developer Tutorial

## 🎯 Tutorial Overview

This comprehensive tutorial teaches Kubernetes from a senior developer perspective, covering development operations and automation workflows.

## 📁 Project Structure

```
k8s-senior-dev-tutorial/
├── README.md                          # This file
├── 01-docker-compose/                 # Docker Compose baseline
├── 02-kind-setup/                     # Local Kubernetes cluster
├── 03-k8s-manifests/                  # Kubernetes YAML files
├── 04-helm-charts/                    # Helm package management
├── 05-helm-from-scratch/              # Creating Helm charts
├── sample-app/                        # Microservices application
└── scripts/                           # Automation scripts
```

## � Getting Started

### Prerequisites

- Docker Desktop
- kubectl
- kind
- helm
- Node.js 16+
- Python 3.8+

### Workshop Modules

1. **Docker Compose Foundation** - Start with familiar containerization
2. **Kind Cluster Setup** - Local Kubernetes environment
3. **Kubernetes Manifests** - Deploy with YAML configurations
4. **Helm Charts** - Package management and templating
5. **Helm from Scratch** - Create custom charts

## 🏗️ Sample Application

A complete microservices application featuring:

- **Frontend**: React application (port 3000)
- **API Gateway**: Express.js proxy (port 8080)
- **User Service**: Node.js + MongoDB (port 3001)
- **Product Service**: Python Flask + PostgreSQL (port 5001)
- **Databases**: MongoDB, PostgreSQL, Redis

## 🛠️ Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd k8s-senior-dev-tutorial

# Start with Docker Compose
cd 01-docker-compose
docker-compose up --build

# Or jump to Kubernetes
cd ../02-kind-setup
./setup-kind.sh

cd ../scripts
./build-images.sh
./deploy.sh
```

## 📚 Learning Path

### Module 01: Docker Compose
- Containerized microservices
- Service discovery
- Volume persistence
- Environment configuration

### Module 02: Kind Setup  
- Multi-node Kubernetes cluster
- Ingress controller setup
- Image loading strategies
- Cluster management

### Module 03: Kubernetes Manifests
- Namespace isolation
- Deployments and services
- Persistent volumes
- Secrets management
- Health checks

### Module 04: Helm Charts
- Package management
- Template reuse
- Value overrides
- Release management

### Module 05: Helm from Scratch
- Chart creation
- Go templating
- Helper functions
- Best practices

## 🎯 Key Learning Objectives

- **Container Orchestration**: Understanding Kubernetes architecture
- **DevOps Workflows**: From development to production
- **Infrastructure as Code**: Declarative configurations
- **Service Discovery**: Internal networking and communication
- **Scaling Strategies**: Horizontal pod autoscaling
- **Storage Management**: Persistent volumes and claims
- **Package Management**: Helm charts and templating
- **Debugging & Monitoring**: Troubleshooting production issues

## 🛡️ Production Considerations

- Resource limits and requests
- Health checks and probes
- Security contexts
- RBAC policies
- Network policies
- Monitoring and logging

## 🔧 Automation Scripts

- `setup-kind.sh` - Automated cluster setup
- `build-images.sh` - Docker image building
- `deploy.sh` - Application deployment
- `debug.sh` - Debugging utilities

## 🎓 Target Audience

**Senior Developers** who want to:
- Transition from development to DevOps
- Understand Kubernetes internals
- Implement production-ready deployments
- Master container orchestration
- Learn infrastructure automation

## � Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kind Documentation](https://kind.sigs.k8s.io/)

## 🛠️ Troubleshooting

If you encounter issues during the tutorial, check the **[TROUBLESHOOTING-GUIDE.md](./TROUBLESHOOTING-GUIDE.md)** which documents all the critical fixes and "hacks" discovered during development.

**Key Issues & Solutions:**
- **Ingress 404 errors** → Move ingress controller to control plane node
- **Namespace stuck in Terminating** → Remove finalizers or restart Kind cluster
- **Images not found** → Run `./scripts/build-images.sh` to load into Kind
- **Host routing issues** → Configure ingress host or use wildcard

---

**Ready to master Kubernetes? Start your journey here!** 🚀
