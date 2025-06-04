# Day 9: Pods Deep Dive

## Overview (1.5 hours)
Master pod lifecycle, multi-container patterns, and advanced pod configurations that are essential for CKA troubleshooting questions.

## Why This Matters for the CKA Exam
- **Pod troubleshooting** accounts for ~40% of troubleshooting domain (30% of exam)
- **Multi-container patterns** appear in workload management questions
- **Pod lifecycle knowledge** enables systematic debugging
- **Init containers** are commonly tested scenarios

## Task 1: Study Pod Lifecycle and Phases (30 minutes)

### What You'll Do
Understand the complete pod lifecycle to diagnose issues and predict behavior.

### Pod Phases

#### 1. Pending Phase
```bash
# Pod is accepted but not yet running
# Common reasons:
# - Image pulling
# - Scheduling constraints
# - Resource limitations
# - Volume mounting issues

# Troubleshooting Pending pods
kubectl get pods | grep Pending
kubectl describe pod <pending-pod> | grep -A 10 "Events:"

# Common Pending scenarios for CKA:
# - Insufficient cluster resources
# - Node taints without tolerations
# - PVC not bound to PV
# - Image pull secrets missing
```

#### 2. Running Phase
```bash
# At least one container is running
# Pod may still have failed containers

# Check running pod details
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name> | grep -A 5 "Conditions:"

# Monitor running pods
kubectl top pod <pod-name>
kubectl logs <pod-name> -f
```

#### 3. Succeeded Phase
```bash
# All containers terminated successfully
# Typical for Job/CronJob pods

# Check completed pods
kubectl get pods --field-selector status.phase=Succeeded
kubectl describe pod <succeeded-pod> | grep "Exit Code"
```

#### 4. Failed Phase
```bash
# At least one container terminated with failure
# Critical for CKA troubleshooting

# Debug failed pods
kubectl get pods --field-selector status.phase=Failed
kubectl describe pod <failed-pod>
kubectl logs <failed-pod> --previous  # Previous container logs
```

#### 5. Unknown Phase
```bash
# Pod state cannot be determined
# Usually communication issues with kubelet

# Troubleshoot Unknown pods
kubectl get nodes  # Check node status
kubectl describe node <node-name>
ssh <node> "sudo systemctl status kubelet"
```

### Container States Within Pods

#### 1. Waiting State
```bash
# Container not yet running
# Reasons: ImagePullBackOff, CrashLoopBackOff

# Check waiting containers
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state.waiting.reason}'

# Common waiting reasons for CKA:
# - ErrImagePull / ImagePullBackOff
# - CrashLoopBackOff
# - CreateContainerConfigError
# - InvalidImageName
```

#### 2. Running State
```bash
# Container executing normally
# Check container processes and health

kubectl exec <pod-name> -- ps aux
kubectl exec <pod-name> -c <container-name> -- ps aux  # Multi-container
```

#### 3. Terminated State
```bash
# Container finished execution
# Exit code indicates success (0) or failure (non-zero)

kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state.terminated.exitCode}'
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state.terminated.reason}'
```

### CKA Exam Lifecycle Scenarios

#### Scenario 1: ImagePullBackOff
```bash
# Problem: Pod stuck in ImagePullBackOff
kubectl describe pod failing-pod
# Look for: Failed to pull image "nginx:nonexistent"

# Solution approaches:
# 1. Fix image name/tag
# 2. Add imagePullSecrets
# 3. Check registry connectivity
```

#### Scenario 2: CrashLoopBackOff
```bash
# Problem: Container keeps restarting
kubectl logs failing-pod --previous  # Check previous container logs
kubectl describe pod failing-pod | grep "Restart Count"

# Common causes:
# - Application configuration errors
# - Missing dependencies
# - Resource constraints
# - Liveness probe failures
```

## Task 2: Practice Multi-Container Patterns (45 minutes)

### What You'll Do
Master sidecar, adapter, and ambassador patterns commonly tested in the CKA exam.

### Sidecar Pattern

#### 1. Logging Sidecar
```yaml
# Create sidecar-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  - name: log-agent
    image: busybox
    command: ['sh', '-c', 'while true; do cat /var/log/nginx/access.log; sleep 30; done']
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  volumes:
  - name: log-volume
    emptyDir: {}
```

```bash
# Apply and test sidecar
kubectl apply -f sidecar-pod.yaml
kubectl logs sidecar-pod -c main-app
kubectl logs sidecar-pod -c log-agent

# Verify shared volume
kubectl exec sidecar-pod -c main-app -- ls -la /var/log/nginx
kubectl exec sidecar-pod -c log-agent -- ls -la /var/log/nginx
```

#### 2. Monitoring Sidecar
```yaml
# monitoring-sidecar.yaml
apiVersion: v1
kind: Pod
metadata:
  name: monitoring-pod
spec:
  containers:
  - name: web-server
    image: nginx
    ports:
    - containerPort: 80
  - name: metrics-collector
    image: busybox
    command: ['sh', '-c', 'while true; do wget -q -O- http://localhost:80/stub_status; sleep 60; done']
```

### Adapter Pattern

#### 1. Log Format Adapter
```yaml
# adapter-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  - name: log-adapter
    image: busybox
    command: ['sh', '-c', 'while true; do sed "s/ERROR/ERR/g" /var/log/nginx/error.log; sleep 30; done']
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  volumes:
  - name: log-volume
    emptyDir: {}
```

### Ambassador Pattern

#### 1. Database Ambassador
```yaml
# ambassador-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
spec:
  containers:
  - name: app
    image: nginx
    # App connects to localhost:3306
  - name: db-ambassador
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    ports:
    - containerPort: 3306
```

### Multi-Container Communication

#### 1. Shared Network (localhost)
```bash
# Containers in same pod share network namespace
# They can communicate via localhost

# Test network sharing
kubectl exec multi-container-pod -c container1 -- curl localhost:80
kubectl exec multi-container-pod -c container2 -- netstat -tulpn
```

#### 2. Shared Volumes
```bash
# Containers can share data via volumes
kubectl exec multi-container-pod -c container1 -- echo "data" > /shared/file.txt
kubectl exec multi-container-pod -c container2 -- cat /shared/file.txt
```

#### 3. Environment Variables
```yaml
# Containers can share environment variables
apiVersion: v1
kind: Pod
metadata:
  name: env-sharing-pod
spec:
  containers:
  - name: container1
    image: busybox
    env:
    - name: SHARED_CONFIG
      value: "production"
  - name: container2
    image: busybox
    env:
    - name: SHARED_CONFIG
      value: "production"
```

## Task 3: Implement Init Containers and Sidecar Patterns (15 minutes)

### What You'll Do
Practice init containers for setup tasks and validate multi-container communication.

### Init Containers

#### 1. Basic Init Container
```yaml
# init-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: init-setup
    image: busybox
    command: ['sh', '-c', 'echo "Initializing..." && sleep 5 && echo "Setup complete"']
  containers:
  - name: main-app
    image: nginx
```

```bash
# Apply and monitor init process
kubectl apply -f init-pod.yaml
kubectl get pod init-pod -w  # Watch pod phases
kubectl logs init-pod -c init-setup
kubectl describe pod init-pod | grep -A 10 "Init Containers"
```

#### 2. Multi-Init Container Setup
```yaml
# multi-init-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-init-pod
spec:
  initContainers:
  - name: init-database
    image: busybox
    command: ['sh', '-c', 'until nslookup database-service; do echo waiting for db; sleep 2; done;']
  - name: init-cache
    image: busybox
    command: ['sh', '-c', 'until nslookup cache-service; do echo waiting for cache; sleep 2; done;']
  containers:
  - name: web-app
    image: nginx
```

#### 3. Init Container with Shared Volume
```yaml
# init-volume-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-volume-pod
spec:
  initContainers:
  - name: setup-config
    image: busybox
    command: ['sh', '-c', 'echo "config=production" > /shared/app.conf']
    volumeMounts:
    - name: config-volume
      mountPath: /shared
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/app
  volumes:
  - name: config-volume
    emptyDir: {}
```

### CKA Exam Init Container Scenarios

#### Scenario 1: Service Dependency
```bash
# Problem: App needs database to be ready before starting
# Solution: Init container to check database connectivity

# Create init container that waits for service
kubectl run db-check --image=busybox --dry-run=client -o yaml -- sh -c "until nslookup database; do sleep 1; done" > init.yaml
# Modify to be initContainer, add main container
```

#### Scenario 2: Configuration Setup
```bash
# Problem: App needs config files before starting
# Solution: Init container to download/generate configs

# Init container downloads config
# Main container uses mounted volume with config
```

### Multi-Container Debugging

#### 1. Container-Specific Operations
```bash
# View logs from specific container
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> -c <container-name> --previous

# Execute in specific container
kubectl exec <pod-name> -c <container-name> -- <command>
kubectl exec -it <pod-name> -c <container-name> -- bash

# Describe specific container
kubectl describe pod <pod-name> | grep -A 20 "Container <container-name>"
```

#### 2. Container Status Checking
```bash
# Check all container statuses
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].name}'
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].ready}'

# Check init container status
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses[*].name}'
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses[*].state}'
```

## Common CKA Multi-Container Scenarios

### 1. Troubleshoot Init Container Failure
```bash
# Pod stuck in Init:0/1 state
kubectl describe pod stuck-pod
kubectl logs stuck-pod -c init-container-name

# Common fixes:
# - Service dependencies not ready
# - Network policy blocking communication
# - DNS resolution issues
```

### 2. Sidecar Container Not Starting
```bash
# Main container running, sidecar failing
kubectl get pod mixed-pod -o jsonpath='{.status.containerStatuses[*].state}'
kubectl logs mixed-pod -c sidecar-container

# Common issues:
# - Volume mount permissions
# - Resource constraints
# - Image pull issues
```

### 3. Container Communication Issues
```bash
# Containers can't communicate via localhost
kubectl exec pod -c container1 -- curl localhost:8080
kubectl exec pod -c container2 -- netstat -tulpn

# Debug network namespace sharing
kubectl exec pod -c container1 -- ip addr
kubectl exec pod -c container2 -- ip addr  # Should be identical
```

## Daily Goals Checklist

- [ ] Understand all pod phases and container states
- [ ] Can implement sidecar, adapter, and ambassador patterns
- [ ] Master init container usage and troubleshooting
- [ ] Can debug multi-container pod issues
- [ ] Practice container-specific kubectl operations
- [ ] Ready for deployment management

## Next Day Preparation

### What You'll Need for Day 10
- Pod lifecycle mastery ✓
- Multi-container pattern knowledge ✓
- Init container skills ✓
- Container debugging abilities ✓

### Coming Up: Deployments & ReplicaSets
Day 10 will cover deployment strategies, rolling updates, and replica management.

## Study Notes Section

**Pod Troubleshooting Workflow:**
```bash
1. kubectl get pods                    # Check status
2. kubectl describe pod <name>         # Get events
3. kubectl logs <name> -c <container>  # Check logs
4. kubectl get events | grep <pod>     # Related events
5. kubectl exec <name> -- <debug-cmd> # Debug inside
```

**Multi-Container Commands:**
```bash
# Container-specific operations
kubectl logs <pod> -c <container>
kubectl exec <pod> -c <container> -- <cmd>
kubectl describe pod <pod> | grep -A 10 "<container>"
```

**Personal Pod Debugging Notes:**
_Record specific debugging techniques and common issues encountered_