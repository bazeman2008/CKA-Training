# CKA Test Environment and Practical Scenarios

## Test Environment Details

The CKA exam runs on **Ubuntu Linux** systems accessed through a **browser-based terminal interface**. Here's what you'll see:

### Interface
- **No GUI** - Pure command-line interface only
- Browser-based terminal (looks like a black terminal window in your web browser)
- Standard bash shell with kubectl pre-installed
- Text-based editors available: `vim`, `nano`
- Copy/paste functionality works between terminal and documentation

### What You Get
- 6-8 pre-configured Kubernetes clusters
- kubectl already configured with contexts for each cluster
- Standard Linux command-line tools
- Access to kubernetes.io documentation in a separate browser tab

## Sample Practical Scenario

Here's what an actual exam question looks like:

---

**Question 7 (8 points)**

**Context:** Switch to cluster `k8s-cluster-1`

**Task:** 
Create a deployment named `web-app` in the `production` namespace with the following specifications:
- Use image `nginx:1.20`
- 3 replicas
- Set resource requests: CPU 100m, memory 128Mi
- Set resource limits: CPU 200m, memory 256Mi
- Add labels: `app=web`, `tier=frontend`
- Expose the deployment using a NodePort service on port 30080
- Ensure the service targets port 80 on the pods

Verify that all pods are running and the service is accessible.

---

### How You'd Solve This

```bash
# 1. Switch context (CRITICAL - always do this first!)
kubectl config use-context k8s-cluster-1

# 2. Create namespace if it doesn't exist
kubectl create namespace production

# 3. Create deployment with imperative command, then modify
kubectl create deployment web-app --image=nginx:1.20 --replicas=3 \
  --namespace=production --dry-run=client -o yaml > web-app.yaml

# 4. Edit the YAML file to add resources and labels
vim web-app.yaml
```

The YAML would need to be modified to include:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
  labels:
    app: web
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      tier: frontend
  template:
    metadata:
      labels:
        app: web
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
```

```bash
# 5. Apply the deployment
kubectl apply -f web-app.yaml

# 6. Create the NodePort service
kubectl expose deployment web-app --type=NodePort --port=80 \
  --target-port=80 --namespace=production --dry-run=client -o yaml > service.yaml

# Edit service.yaml to set nodePort: 30080
vim service.yaml
kubectl apply -f service.yaml

# 7. Verify everything works
kubectl get pods -n production
kubectl get svc -n production
kubectl describe deployment web-app -n production
```

## What the Terminal Interface Looks Like

```bash
student@terminal:~$ kubectl config current-context
k8s-cluster-1

student@terminal:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
k8s-master-node     Ready    control-plane   5d    v1.28.0
k8s-worker-node-1   Ready    <none>          5d    v1.28.0
k8s-worker-node-2   Ready    <none>          5d    v1.28.0

student@terminal:~$ kubectl create deployment web-app --image=nginx:1.20...
```

## Other Common Scenario Types

### Troubleshooting Example
"A pod named `broken-pod` in namespace `debug` is not starting. Identify and fix the issue."

### Storage Example
"Create a PersistentVolume of 5Gi and a PVC that uses it, then mount it to a pod at `/data`."

### RBAC Example
"Create a service account `app-sa` with permissions to only list and get pods in the `apps` namespace."

### Networking Example
"The service `frontend-svc` cannot reach `backend-svc`. Debug and fix the connectivity issue."

---

**Key Takeaway:** Every question gives you a specific, practical task to complete on real Kubernetes clusters. You're evaluated based on whether your solution actually works, not just whether you know the theory.