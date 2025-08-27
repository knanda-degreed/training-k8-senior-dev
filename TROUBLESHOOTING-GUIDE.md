# 🛠️ Kubernetes Tutorial - Troubleshooting Guide

## 🎯 Critical Fixes That Made Everything Work

This document captures the essential "hacks" and fixes discovered during the tutorial that are **crucial** for getting the application working in a Kind cluster.

---

## 🔧 **Fix #1: Ingress Controller Node Placement (CRITICAL)**

### **Problem:**
- Ingress controller runs on worker nodes by default
- Kind only maps ports 80/443 to the **control plane node**
- Traffic from `localhost:80` → control plane → ❌ (ingress on worker)

### **Solution:**
```bash
# Force ingress controller to run on control plane node
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"nodeSelector":{"node-role.kubernetes.io/control-plane":""}}}}}'

# Wait for restart
kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx

# Verify it's on control plane
kubectl get pods -n ingress-nginx -o wide
```

### **Why This Works:**
- Kind's port mapping: `localhost:80` → `control-plane:80`
- Ingress controller must be on control plane to receive external traffic
- This is **Kind-specific** - not needed in cloud Kubernetes

---

## 🔧 **Fix #2: Ingress Host Configuration**

### **Problem:**
- Original ingress had `host: k8s-tutorial.local`
- Accessing `localhost` didn't match the host rule

### **Solution Option A - Wildcard Host:**
```yaml
# 08-ingress.yaml
spec:
  rules:
  - http:  # No host specified = accepts any host
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 3000
```

### **Solution Option B - Specific Host + /etc/hosts:**
```yaml
# 08-ingress.yaml
spec:
  rules:
  - host: k8s-tutorial.local
    http:
      paths: [...]
```

```bash
# Add to /etc/hosts
echo "127.0.0.1 k8s-tutorial.local" | sudo tee -a /etc/hosts
```

---

## 🔧 **Fix #3: Ingress Resource Missing**

### **Problem:**
- Ingress YAML file exists but resource wasn't created
- `kubectl get ingress` returned "NotFound"
- Results in 404 errors from nginx

### **Solution:**
```bash
# Apply the ingress configuration
kubectl apply -f 08-ingress.yaml

# Verify it was created
kubectl get ingress -n microservices-app
```

### **Root Cause:**
- Ingress resource was deleted during troubleshooting
- Must be explicitly recreated after namespace deletion

---

## 🔧 **Fix #4: Namespace Cleanup Issues**

### **Problem:**
- Namespaces stuck in "Terminating" state
- Helm can't create namespace that "already exists"
- Finalizers prevent proper deletion

### **Solution:**
```bash
# Force remove finalizers
kubectl patch namespace microservices-app -p '{"spec":{"finalizers":[]}}' --type=merge

# Or complete cluster restart
kind delete cluster --name k8s-tutorial
cd 02-kind-setup && ./setup-kind.sh
cd ../scripts && ./build-images.sh
```

---

## 🔧 **Fix #5: Image Loading in Kind**

### **Problem:**
- Docker images built locally aren't available in Kind cluster
- Pods fail with "ImagePullBackOff"

### **Solution:**
```bash
# Build and load images into Kind cluster
cd scripts
./build-images.sh

# This script does:
# 1. docker build -t frontend:latest frontend/
# 2. kind load docker-image frontend:latest --name k8s-tutorial
# (repeat for all services)
```

---

## 📋 **Complete Working Deployment Sequence**

### **1. Setup Kind Cluster:**
```bash
cd 02-kind-setup
./setup-kind.sh
```

### **2. Build and Load Images:**
```bash
cd ../scripts
./build-images.sh
```

### **3. Deploy Application:**
```bash
cd ../03-k8s-manifests
kubectl apply -f .
```

### **4. Fix Ingress Controller Location:**
```bash
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"nodeSelector":{"node-role.kubernetes.io/control-plane":""}}}}}'
```

### **5. Verify and Test:**
```bash
# Wait for all pods to be ready
kubectl get pods -n microservices-app

# Test the application
curl http://k8s-tutorial.local/api/gateway/health
curl http://k8s-tutorial.local/
```

---

## 🚨 **Critical Debugging Commands**

### **Check Ingress Controller Location:**
```bash
kubectl get pods -n ingress-nginx -o wide
# Must show NODE = k8s-tutorial-control-plane
```

### **Check Service Endpoints:**
```bash
kubectl get endpoints -n microservices-app
# All services should have IP addresses listed
```

### **Test Direct Service Access:**
```bash
kubectl port-forward -n microservices-app svc/api-gateway 8080:8080 &
curl http://localhost:8080/api/gateway/health
pkill -f "kubectl port-forward"
```

### **Check Ingress Resource:**
```bash
kubectl get ingress -n microservices-app
kubectl describe ingress -n microservices-app app-ingress
```

---

## 🎯 **Key Learnings**

### **Kind-Specific Requirements:**
1. **Ingress controller MUST run on control plane** (for port mapping)
2. **Images must be loaded into Kind** (not just built locally)
3. **Host configuration matters** (wildcard vs specific host)

### **Kubernetes Fundamentals:**
1. **Services work independently** of ingress (test with port-forward)
2. **Ingress is just routing** - if services work, ingress is the issue
3. **Namespace deletion cascades** - removes all resources inside

### **Troubleshooting Approach:**
1. **Test services directly** first (port-forward)
2. **Check ingress controller location** (Kind-specific)
3. **Verify ingress resource exists** (kubectl get ingress)
4. **Check logs** for specific error messages

---

## 🏆 **Final Working State**

### **Application URLs:**
- **Frontend**: http://k8s-tutorial.local/
- **API Gateway**: http://k8s-tutorial.local/api/gateway/health
- **User Service**: http://k8s-tutorial.local/api/users/health
- **Product Service**: http://k8s-tutorial.local/api/products/health

### **Architecture:**
```
Browser (k8s-tutorial.local)
    ↓
Kind Control Plane (port 80 mapped)
    ↓
NGINX Ingress Controller (on control plane)
    ↓
Kubernetes Services (ClusterIP)
    ↓
Application Pods (across worker nodes)
```

### **Resource Count:**
- **7 Applications**: Frontend, API Gateway, User Service, Product Service, MongoDB, PostgreSQL, Redis
- **11 Pods**: High availability with 2 replicas for microservices
- **1 Ingress**: Routes external traffic to services
- **7 Services**: Internal cluster networking

---

**🎉 Congratulations! You now have a fully functional microservices application running on Kubernetes!**

This troubleshooting guide captures all the critical fixes that make the difference between a broken and working deployment. Keep this handy for future Kubernetes adventures! 🚀
