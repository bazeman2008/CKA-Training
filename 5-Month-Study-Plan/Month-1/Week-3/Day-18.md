# Day 18: Static Pods

## Overview (1.5 hours)
Master static pod configuration, management, and troubleshooting, including understanding how control plane components run as static pods and managing custom static pods.

## Why This Matters for the CKA Exam
- **Static pods appear in 15-20% of exam questions**
- **Control plane troubleshooting** requires understanding static pod behavior
- **kubelet direct management** bypasses the API server
- **Node-specific services** often implemented as static pods
- **Troubleshooting** static pod issues requires different approaches

## Task 1: Understanding Static Pods (30 minutes)

### What You'll Do
Learn how static pods work, where they're configured, and how they differ from regular pods.

### Step-by-Step Instructions

#### 1. Explore Existing Static Pods
```bash
# Find current static pods (control plane components)
kubectl get pods -n kube-system | grep $(hostname)
kubectl get pods -n kube-system -o wide | grep master

# Check kubelet configuration for static pod path
sudo cat /var/lib/kubelet/config.yaml | grep staticPodPath
# Alternative locations to check:
sudo ls /etc/kubernetes/manifests/
sudo ls /etc/kubelet.d/

# Examine a control plane static pod manifest
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | head -20
sudo cat /etc/kubernetes/manifests/etcd.yaml | head -20
```

#### 2. Understand Static Pod Behavior
```bash
# Try to delete a static pod (it will restart automatically)
kubectl delete pod kube-apiserver-$(hostname) -n kube-system
kubectl get pods -n kube-system | grep apiserver

# Static pods have node name suffix
kubectl get pods -n kube-system -o custom-columns="NAME:.metadata.name,NODE:.spec.nodeName" | grep $(hostname)

# Check ownership references
kubectl describe pod kube-apiserver-$(hostname) -n kube-system | grep "Controlled By"
```

#### 3. Locate Static Pod Directory
```bash
# Find the static pod directory from kubelet config
STATIC_POD_PATH=$(sudo cat /var/lib/kubelet/config.yaml | grep staticPodPath | awk '{print $2}')
echo "Static pod path: $STATIC_POD_PATH"

# Alternative method - check kubelet service
sudo systemctl cat kubelet | grep "\-\-pod-manifest-path"

# List existing static pod manifests
sudo ls -la /etc/kubernetes/manifests/
```

### CKA Exam Connection
- **Control Plane Management**: Understanding how core components run
- **Node-level Services**: Services that must run on specific nodes
- **Troubleshooting**: Debugging control plane component issues
- **High Availability**: Critical services independent of API server

### Validation Commands
```bash
# Verify static pod functionality
kubectl get pods -n kube-system --field-selector spec.nodeName=$(hostname)
sudo ls /etc/kubernetes/manifests/
ps aux | grep kubelet | grep manifest
```

## Task 2: Creating Custom Static Pods (35 minutes)

### What You'll Do
Create, manage, and troubleshoot custom static pods for node-specific services.

### Step-by-Step Instructions

#### 1. Create Basic Static Pod
```bash
# Create a simple static pod manifest
sudo tee /etc/kubernetes/manifests/static-webserver.yaml > /dev/null << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: static-webserver
  labels:
    app: static-webserver
    tier: system
spec:
  containers:
  - name: webserver
    image: nginx:1.20
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
    volumeMounts:
    - name: html-content
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html-content
    hostPath:
      path: /var/static-content
      type: DirectoryOrCreate
  restartPolicy: Always
EOF

# Create content directory and file
sudo mkdir -p /var/static-content
echo "<h1>Static Pod Web Server on $(hostname)</h1>" | sudo tee /var/static-content/index.html

# Wait for kubelet to detect and create the pod
sleep 10
kubectl get pods | grep static-webserver
kubectl describe pod static-webserver-$(hostname)
```

#### 2. Static Pod with System Monitoring
```bash
# Create monitoring static pod
sudo tee /etc/kubernetes/manifests/node-monitor.yaml > /dev/null << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: node-monitor
  labels:
    app: node-monitor
    component: system
spec:
  containers:
  - name: monitor
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "$(date): Node $(hostname) - Load: $(cat /proc/loadavg)"
        echo "$(date): Node $(hostname) - Memory: $(free -h | grep Mem)"
        echo "$(date): Node $(hostname) - Disk: $(df -h / | tail -1)"
        sleep 60
      done
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
    volumeMounts:
    - name: proc
      mountPath: /proc
      readOnly: true
    - name: sys
      mountPath: /sys
      readOnly: true
  volumes:
  - name: proc
    hostPath:
      path: /proc
  - name: sys
    hostPath:
      path: /sys
  hostNetwork: true
  hostPID: true
  restartPolicy: Always
EOF

# Wait and verify
sleep 10
kubectl get pods | grep node-monitor
kubectl logs node-monitor-$(hostname) --tail=5
```

#### 3. Static Pod with Init Container
```bash
# Create static pod with initialization
sudo tee /etc/kubernetes/manifests/init-static-pod.yaml > /dev/null << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: init-static-pod
  labels:
    app: init-example
spec:
  initContainers:
  - name: setup
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Initializing on $(hostname)"
      mkdir -p /shared/data
      echo "Node: $(hostname)" > /shared/data/node-info
      echo "Initialized at: $(date)" >> /shared/data/node-info
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
  containers:
  - name: main-app
    image: nginx:1.20
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
    volumeMounts:
    - name: shared-storage
      mountPath: /usr/share/nginx/html
    command:
    - sh
    - -c
    - |
      # Copy initialized content
      cp /usr/share/nginx/html/node-info /usr/share/nginx/html/index.html
      # Start nginx
      nginx -g 'daemon off;'
  volumes:
  - name: shared-storage
    hostPath:
      path: /var/init-static-data
      type: DirectoryOrCreate
  restartPolicy: Always
EOF

# Wait and verify initialization
sleep 15
kubectl get pods | grep init-static-pod
kubectl describe pod init-static-pod-$(hostname) | grep -A 10 "Init Containers"
```

#### 4. Static Pod Resource Management
```bash
# Create resource-constrained static pod
sudo tee /etc/kubernetes/manifests/resource-static.yaml > /dev/null << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: resource-static
  labels:
    app: resource-demo
spec:
  containers:
  - name: cpu-monitor
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "CPU usage check at $(date)"
        sleep 30
      done
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  - name: memory-monitor
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "Memory usage check at $(date)"
        free -h
        sleep 45
      done
    resources:
      requests:
        memory: "32Mi"
        cpu: "25m"
      limits:
        memory: "64Mi"
        cpu: "50m"
  restartPolicy: Always
EOF

# Wait and check resource usage
sleep 10
kubectl top pod resource-static-$(hostname) --containers
```

### CKA Exam Connection
- **Node Services**: Running critical services per node
- **System Integration**: Direct host system access
- **Resource Control**: Managing node-level resource consumption
- **Independence**: Services that work without API server

### Validation Commands
```bash
# Check all static pods
kubectl get pods --all-namespaces | grep $(hostname)
sudo ls -la /etc/kubernetes/manifests/

# Verify static pod logs
kubectl logs static-webserver-$(hostname)
kubectl logs node-monitor-$(hostname) --tail=10
```

## Task 3: Static Pod Management and Operations (25 minutes)

### What You'll Do
Practice managing static pods including updates, troubleshooting, and removal.

### Step-by-Step Instructions

#### 1. Static Pod Updates
```bash
# Update static pod by modifying manifest
sudo cp /etc/kubernetes/manifests/static-webserver.yaml /etc/kubernetes/manifests/static-webserver.yaml.backup

# Modify the static pod (change image version)
sudo sed -i 's/nginx:1.20/nginx:1.21/' /etc/kubernetes/manifests/static-webserver.yaml

# Watch for pod recreation
kubectl get pods -w | grep static-webserver &
sleep 20
# Stop watching with Ctrl+C

# Verify update
kubectl describe pod static-webserver-$(hostname) | grep Image:
```

#### 2. Static Pod Scaling and Multiple Instances
```bash
# Create second static pod with different name
sudo cp /etc/kubernetes/manifests/static-webserver.yaml /etc/kubernetes/manifests/static-webserver-2.yaml
sudo sed -i 's/name: static-webserver/name: static-webserver-2/' /etc/kubernetes/manifests/static-webserver-2.yaml
sudo sed -i 's/containerPort: 80/containerPort: 8080/' /etc/kubernetes/manifests/static-webserver-2.yaml

# Wait and verify both pods
sleep 10
kubectl get pods | grep static-webserver
```

#### 3. Static Pod Troubleshooting
```bash
# Create a problematic static pod
sudo tee /etc/kubernetes/manifests/broken-static.yaml > /dev/null << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: broken-static
spec:
  containers:
  - name: broken-app
    image: nonexistent-image:latest
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# Wait and diagnose
sleep 10
kubectl get pods | grep broken-static
kubectl describe pod broken-static-$(hostname) | grep -A 10 Events

# Check kubelet logs for static pod errors
sudo journalctl -u kubelet -f --since "1 minute ago" | grep -i static &
sleep 5
# Stop log monitoring with Ctrl+C
```

#### 4. Static Pod Removal
```bash
# Remove static pods by deleting manifests
sudo rm /etc/kubernetes/manifests/broken-static.yaml
sudo rm /etc/kubernetes/manifests/static-webserver-2.yaml

# Wait and verify removal
sleep 10
kubectl get pods | grep static-webserver
kubectl get pods | grep broken-static

# Attempting to delete via kubectl (will restart)
kubectl delete pod static-webserver-$(hostname)
sleep 5
kubectl get pods | grep static-webserver
```

### CKA Exam Connection
- **Maintenance Operations**: Updating critical system components
- **Troubleshooting**: Diagnosing static pod failures
- **Recovery**: Restoring static pod functionality
- **Version Management**: Upgrading system components

### Validation Commands
```bash
# Monitor static pod changes
kubectl get events --field-selector involvedObject.name=static-webserver-$(hostname)
sudo journalctl -u kubelet --since "5 minutes ago" | grep static
```

## Task 4: Control Plane Static Pod Analysis (20 minutes)

### What You'll Do
Analyze control plane static pods and understand their configuration for troubleshooting scenarios.

### Step-by-Step Instructions

#### 1. Control Plane Component Analysis
```bash
# Examine API server static pod
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -A 5 -B 5 "image:\|command:\|args:"

# Check scheduler configuration
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml | grep -A 3 -B 3 "command:\|args:"

# Examine controller manager
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep -A 3 -B 3 "command:\|args:"

# Check etcd configuration
sudo cat /etc/kubernetes/manifests/etcd.yaml | grep -A 5 -B 5 "command:\|args:"
```

#### 2. Control Plane Resource Configuration
```bash
# Check resource allocations for control plane
for component in kube-apiserver kube-scheduler kube-controller-manager etcd; do
    echo "=== $component ==="
    sudo cat /etc/kubernetes/manifests/${component}.yaml | grep -A 10 resources: | head -15
    echo ""
done
```

#### 3. Control Plane Troubleshooting Simulation
```bash
# Backup and temporarily break API server
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/kube-apiserver.yaml.backup

# Create a backup with intentional error (for learning)
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/broken-apiserver.yaml
# Don't actually apply broken config - just demonstrate the concept

# Show how to check kubelet logs for static pod issues
sudo journalctl -u kubelet --since "5 minutes ago" | grep -i "api-server\|apiserver"
```

#### 4. Static Pod Certificate Management
```bash
# Check certificate mounts in static pods
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -A 2 -B 2 "certificate\|cert\|key"

# Examine certificate locations
sudo ls -la /etc/kubernetes/pki/
sudo cat /etc/kubernetes/manifests/etcd.yaml | grep -A 2 -B 2 "cert\|key"

# Verify certificate validity
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -A 2 "Validity"
```

### CKA Exam Connection
- **Control Plane Troubleshooting**: Understanding component configuration
- **Certificate Management**: Static pod certificate mounts
- **Recovery Procedures**: Restoring control plane components
- **Configuration Management**: Control plane parameter tuning

### Validation Commands
```bash
# Check control plane pod status
kubectl get pods -n kube-system --field-selector spec.nodeName=$(hostname)
sudo systemctl status kubelet | grep -i static
```

## Common Issues and Solutions

### Issue 1: Static Pod Not Starting
**Problem**: Static pod manifest exists but pod doesn't appear
**Solution**: 
```bash
# Check kubelet logs
sudo journalctl -u kubelet -f --since "2 minutes ago"

# Verify static pod path
sudo cat /var/lib/kubelet/config.yaml | grep staticPodPath

# Check manifest syntax
sudo cat /etc/kubernetes/manifests/<pod-name>.yaml | yaml lint
```

### Issue 2: Static Pod Restart Loop
**Problem**: Static pod continuously restarting
**Solution**:
```bash
# Check pod events and logs
kubectl describe pod <static-pod-name>-$(hostname)
kubectl logs <static-pod-name>-$(hostname) --previous

# Check resource constraints
kubectl describe node $(hostname) | grep -A 5 "Allocated resources"
```

### Issue 3: Cannot Delete Static Pod
**Problem**: Static pod recreates after deletion
**Solution**:
```bash
# Remove the manifest file instead of using kubectl delete
sudo rm /etc/kubernetes/manifests/<pod-name>.yaml

# Wait for kubelet to detect removal
sleep 10
kubectl get pods | grep <pod-name>
```

### Issue 4: Control Plane Component Failure
**Problem**: Control plane static pod not functioning
**Solution**:
```bash
# Check kubelet status and logs
sudo systemctl status kubelet
sudo journalctl -u kubelet --since "5 minutes ago"

# Verify manifest file integrity
sudo cat /etc/kubernetes/manifests/<component>.yaml

# Check certificate validity
sudo openssl x509 -in /etc/kubernetes/pki/<cert>.crt -text -noout | grep Validity
```

## Time Management Tips

### For CKA Exam Success
- **Quick Static Pod Check**: Use `sudo ls /etc/kubernetes/manifests/`
- **Manifest Location**: Remember `/etc/kubernetes/manifests/` is default
- **kubelet Logs**: Use `sudo journalctl -u kubelet` for troubleshooting
- **Control Plane**: Static pods have node name suffix

### Daily Practice Goals
- [ ] Create static pod in under 3 minutes
- [ ] Troubleshoot static pod issues efficiently
- [ ] Update static pod configuration safely
- [ ] Understand control plane static pod structure

## Next Day Preparation

### Review Before Day 19
- [ ] Static pod vs regular pod differences
- [ ] kubelet static pod directory location
- [ ] Control plane component static pod structure
- [ ] Static pod troubleshooting methodology

### What's Coming Next
Day 19 will cover Workload Troubleshooting, building on static pod knowledge:
- Static pod troubleshooting techniques ✓
- kubelet log analysis skills ✓  
- Control plane component understanding ✓

## Study Notes Section
_Use this space to document static pod patterns specific to your environment_

**Static Pod Configuration:**
- Static pod path: /etc/kubernetes/manifests/
- kubelet config: /var/lib/kubelet/config.yaml
- Log monitoring: sudo journalctl -u kubelet

**Control Plane Components:**
- API Server: ________________
- Scheduler: __________________
- Controller Manager: __________
- etcd: ______________________

**Common Static Pod Patterns:**
```yaml
# System monitoring static pod:
apiVersion: v1
kind: Pod
metadata:
  name: system-monitor
spec:
  containers:
  - name: monitor
    image: busybox
    # Add your monitoring commands
```

**Troubleshooting Checklist:**
- [ ] Check kubelet logs
- [ ] Verify manifest syntax  
- [ ] Check resource constraints
- [ ] Verify image availability

**Personal Static Pod Patterns:**
_Record your preferred static pod configurations and management approaches_