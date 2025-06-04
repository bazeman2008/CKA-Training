# Day 17: Resource Management

## Overview (1.5 hours)
Master Kubernetes resource management including requests and limits, Quality of Service classes, LimitRanges, and ResourceQuotas for optimal cluster resource utilization.

## Why This Matters for the CKA Exam
- **Resource management appears in 30-35% of exam questions**
- **QoS classes** determine pod eviction priority during resource pressure
- **LimitRanges** enforce resource constraints at namespace level
- **ResourceQuotas** control aggregate resource consumption
- **Troubleshooting** resource issues requires understanding these concepts

## Task 1: Resource Requests and Limits (40 minutes)

### What You'll Do
Configure resource requests and limits for containers and understand their impact on scheduling and runtime behavior.

### Step-by-Step Instructions

#### 1. Basic Resource Configuration
```bash
# Create pod with resource requests and limits
cat > resource-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
  labels:
    app: resource-demo
spec:
  containers:
  - name: web-server
    image: nginx
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "128Mi"
        cpu: "200m"
      limits:
        memory: "256Mi"
        cpu: "500m"
  - name: sidecar
    image: busybox
    command: ["sh", "-c", "while true; do echo $(date); sleep 30; done"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
EOF

# Apply and verify
kubectl apply -f resource-pod.yaml
kubectl describe pod resource-demo | grep -A 20 "Containers:"
kubectl top pod resource-demo --containers
```

#### 2. Resource Scenarios Testing
```bash
# Create pods with different resource patterns
cat > resource-scenarios.yaml << 'EOF'
---
# Guaranteed QoS (requests = limits)
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "256Mi"
        cpu: "500m"
---
# Burstable QoS (requests < limits)
apiVersion: v1
kind: Pod
metadata:
  name: burstable-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
---
# BestEffort QoS (no requests/limits)
apiVersion: v1
kind: Pod
metadata:
  name: besteffort-pod
spec:
  containers:
  - name: app
    image: nginx
EOF

# Apply and check QoS classes
kubectl apply -f resource-scenarios.yaml
kubectl get pods -o custom-columns="NAME:.metadata.name,QOS:.status.qosClass,STATUS:.status.phase"
kubectl describe pod guaranteed-pod | grep "QoS Class"
kubectl describe pod burstable-pod | grep "QoS Class"
kubectl describe pod besteffort-pod | grep "QoS Class"
```

#### 3. Deployment with Resource Management
```bash
# Create deployment with resource configuration
cat > resource-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: webapp
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
          limits:
            memory: "200Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
EOF

# Apply and monitor resource usage
kubectl apply -f resource-deployment.yaml
kubectl get pods -l app=web-app
kubectl top pods -l app=web-app
```

#### 4. Resource Monitoring and Analysis
```bash
# Check node resource allocation
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl top nodes

# Analyze pod resource consumption
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Get detailed resource information
kubectl get pods -o custom-columns="NAME:.metadata.name,CPU_REQ:.spec.containers[*].resources.requests.cpu,MEM_REQ:.spec.containers[*].resources.requests.memory"
```

### CKA Exam Connection
- **Resource Planning**: Proper cluster capacity planning
- **Scheduling**: Understanding resource-based scheduling decisions
- **Performance**: Preventing resource starvation
- **Troubleshooting**: Diagnosing resource-related pod failures

### Validation Commands
```bash
# Verify resource configuration
kubectl describe pod resource-demo | grep -A 10 "Requests\|Limits"
kubectl get pods -o yaml | grep -A 10 resources

# Monitor actual usage
kubectl top pods --all-namespaces
kubectl top nodes --use-protocol-buffers
```

## Task 2: Quality of Service (QoS) Classes (25 minutes)

### What You'll Do
Understand and implement the three QoS classes and their impact on pod scheduling and eviction.

### Step-by-Step Instructions

#### 1. Understanding QoS Classes
```bash
# Check existing pod QoS classes
kubectl get pods -o custom-columns="NAME:.metadata.name,QOS:.status.qosClass"

# Detailed QoS analysis
for pod in $(kubectl get pods -o jsonpath='{.items[*].metadata.name}'); do
    echo "Pod: $pod"
    kubectl get pod $pod -o jsonpath='{.status.qosClass}'
    echo ""
done
```

#### 2. Create Pods for Each QoS Class
```bash
# Create comprehensive QoS examples
cat > qos-examples.yaml << 'EOF'
---
# Guaranteed QoS - highest priority, last to be evicted
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-critical
spec:
  containers:
  - name: critical-app
    image: nginx
    resources:
      requests:
        memory: "200Mi"
        cpu: "200m"
      limits:
        memory: "200Mi"
        cpu: "200m"
---
# Burstable QoS - medium priority
apiVersion: v1
kind: Pod
metadata:
  name: burstable-normal
spec:
  containers:
  - name: normal-app
    image: nginx
    resources:
      requests:
        memory: "100Mi"
        cpu: "100m"
      limits:
        memory: "400Mi"
        cpu: "500m"
---
# BestEffort QoS - lowest priority, first to be evicted
apiVersion: v1
kind: Pod
metadata:
  name: besteffort-batch
spec:
  containers:
  - name: batch-app
    image: nginx
EOF

# Apply and verify QoS assignments
kubectl apply -f qos-examples.yaml
kubectl get pods -o custom-columns="NAME:.metadata.name,QOS:.status.qosClass,STATUS:.status.phase"
```

#### 3. QoS Impact Simulation
```bash
# Create high memory pressure pod to trigger eviction
cat > memory-pressure.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: memory-hog
spec:
  containers:
  - name: memory-consumer
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "1G", "--vm-hang", "1"]
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "1Gi"
EOF

# Apply and monitor eviction behavior
kubectl apply -f memory-pressure.yaml
kubectl get pods -w &
# Monitor for evictions (BestEffort pods evicted first)
# Stop monitoring with Ctrl+C after observation
```

#### 4. QoS Best Practices
```bash
# Production workload patterns
cat > production-qos.yaml << 'EOF'
---
# Critical system component (Guaranteed)
apiVersion: v1
kind: Pod
metadata:
  name: database
spec:
  containers:
  - name: database
    image: postgres:13
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    env:
    - name: POSTGRES_PASSWORD
      value: "password"
---
# Web application (Burstable)
apiVersion: v1
kind: Pod
metadata:
  name: web-frontend
spec:
  containers:
  - name: frontend
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
---
# Background job (BestEffort for development)
apiVersion: v1
kind: Pod
metadata:
  name: background-job
spec:
  containers:
  - name: job
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
EOF

kubectl apply -f production-qos.yaml
```

### CKA Exam Connection
- **Eviction Policies**: Understanding pod removal order
- **Cluster Stability**: Protecting critical workloads
- **Resource Allocation**: Optimal resource distribution
- **Troubleshooting**: Identifying why pods were evicted

### Validation Commands
```bash
# Check QoS distribution
kubectl get pods --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,QOS:.status.qosClass"

# Monitor eviction events
kubectl get events --field-selector reason=Evicted
kubectl describe node | grep -A 10 "Events"
```

## Task 3: LimitRanges Configuration (15 minutes)

### What You'll Do
Implement LimitRanges to enforce resource constraints at the namespace level.

### Step-by-Step Instructions

#### 1. Create Namespace with LimitRange
```bash
# Create test namespace
kubectl create namespace resource-test

# Create comprehensive LimitRange
cat > limit-range.yaml << 'EOF'
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: resource-test
spec:
  limits:
  - type: Container
    default:
      cpu: "200m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "1000m"
      memory: "1Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
  - type: Pod
    max:
      cpu: "2000m"
      memory: "2Gi"
  - type: PersistentVolumeClaim
    max:
      storage: "10Gi"
    min:
      storage: "1Gi"
EOF

# Apply and verify
kubectl apply -f limit-range.yaml
kubectl describe limitrange resource-limits -n resource-test
```

#### 2. Test LimitRange Enforcement
```bash
# Test pod without resource specification (should get defaults)
kubectl run test-default --image=nginx -n resource-test
kubectl describe pod test-default -n resource-test | grep -A 10 "Requests\|Limits"

# Test pod exceeding limits (should be rejected)
cat > exceed-limits.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: exceed-limits
  namespace: resource-test
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "2000m"  # Exceeds max
        memory: "2Gi"  # Exceeds max
EOF

kubectl apply -f exceed-limits.yaml
# Should see error about exceeding limits
```

#### 3. LimitRange for Different Resource Types
```bash
# Create storage-focused LimitRange
cat > storage-limits.yaml << 'EOF'
apiVersion: v1
kind: LimitRange
metadata:
  name: storage-limits
  namespace: resource-test
spec:
  limits:
  - type: PersistentVolumeClaim
    max:
      storage: "5Gi"
    min:
      storage: "100Mi"
    default:
      storage: "1Gi"
EOF

kubectl apply -f storage-limits.yaml
kubectl get limitranges -n resource-test
```

### CKA Exam Connection
- **Resource Governance**: Preventing resource abuse
- **Default Policies**: Ensuring minimum resource allocation
- **Multi-tenancy**: Isolating namespace resources
- **Cost Control**: Managing cluster resource costs

### Validation Commands
```bash
# Check LimitRange enforcement
kubectl describe limitrange -n resource-test
kubectl get pods -n resource-test -o custom-columns="NAME:.metadata.name,CPU_REQ:.spec.containers[*].resources.requests.cpu"
```

## Task 4: ResourceQuotas Implementation (20 minutes)

### What You'll Do
Configure ResourceQuotas to control aggregate resource consumption within namespaces.

### Step-by-Step Instructions

#### 1. Basic ResourceQuota Configuration
```bash
# Create ResourceQuota for namespace
cat > resource-quota.yaml << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: resource-test
spec:
  hard:
    requests.cpu: "2000m"
    requests.memory: "4Gi"
    limits.cpu: "4000m"
    limits.memory: "8Gi"
    requests.storage: "10Gi"
    persistentvolumeclaims: "3"
    pods: "10"
    services: "5"
    secrets: "10"
    configmaps: "10"
EOF

# Apply and verify
kubectl apply -f resource-quota.yaml
kubectl describe resourcequota compute-quota -n resource-test
```

#### 2. Test Quota Enforcement
```bash
# Create deployment that consumes quota
cat > quota-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quota-test
  namespace: resource-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: quota-test
  template:
    metadata:
      labels:
        app: quota-test
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            cpu: "300m"
            memory: "512Mi"
          limits:
            cpu: "500m"
            memory: "1Gi"
EOF

# Apply and check quota usage
kubectl apply -f quota-deployment.yaml
kubectl describe resourcequota compute-quota -n resource-test
```

#### 3. Advanced ResourceQuota Scenarios
```bash
# Create quota with scope selectors
cat > scoped-quota.yaml << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: best-effort-quota
  namespace: resource-test
spec:
  hard:
    pods: "5"
  scopes:
  - BestEffort
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: non-best-effort-quota
  namespace: resource-test
spec:
  hard:
    requests.cpu: "1000m"
    requests.memory: "2Gi"
  scopes:
  - NotBestEffort
EOF

# Apply and test different QoS classes
kubectl apply -f scoped-quota.yaml
kubectl get resourcequotas -n resource-test
```

#### 4. Quota Monitoring and Management
```bash
# Monitor quota usage across namespaces
kubectl get resourcequota --all-namespaces
kubectl top pods -n resource-test

# Create helper script for quota monitoring
cat > quota-monitor.sh << 'EOF'
#!/bin/bash
echo "=== ResourceQuota Status ==="
kubectl get resourcequota -n resource-test -o custom-columns="NAME:.metadata.name,USED_CPU:.status.used.requests\.cpu,HARD_CPU:.status.hard.requests\.cpu,USED_MEMORY:.status.used.requests\.memory,HARD_MEMORY:.status.hard.requests\.memory"
echo ""
echo "=== Pod Resource Usage ==="
kubectl top pods -n resource-test
EOF

chmod +x quota-monitor.sh
./quota-monitor.sh
```

### CKA Exam Connection
- **Resource Control**: Preventing namespace resource overconsumption
- **Multi-tenancy**: Isolating team/project resources
- **Capacity Planning**: Understanding cluster resource distribution
- **Troubleshooting**: Diagnosing quota-related deployment failures

### Validation Commands
```bash
# Check quota status
kubectl describe resourcequota -n resource-test
kubectl get resourcequota -n resource-test -o yaml

# Monitor resource consumption
kubectl top pods -n resource-test --sort-by=cpu
```

## Common Issues and Solutions

### Issue 1: Pod Stuck in Pending Due to Resources
**Problem**: Pod can't be scheduled due to insufficient resources
**Solution**: 
```bash
# Check cluster capacity
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl top nodes

# Check pod resource requirements
kubectl describe pod <pod-name> | grep -A 10 "Requests\|Events"

# Adjust resource requests or add nodes
```

### Issue 2: LimitRange Violations
**Problem**: Pods rejected due to LimitRange constraints
**Solution**:
```bash
# Check LimitRange settings
kubectl describe limitrange -n <namespace>

# Verify pod resource specifications
kubectl describe pod <pod-name> | grep -A 10 Resources

# Adjust pod resources or modify LimitRange
```

### Issue 3: ResourceQuota Exceeded
**Problem**: Cannot create resources due to quota limits
**Solution**:
```bash
# Check quota usage
kubectl describe resourcequota -n <namespace>

# Find resource-consuming pods
kubectl top pods -n <namespace> --sort-by=cpu

# Scale down or increase quota
kubectl scale deployment <name> --replicas=<lower-number> -n <namespace>
```

## Time Management Tips

### For CKA Exam Success
- **Quick Resource Check**: Use `kubectl top` commands for immediate resource view
- **Resource Patterns**: Memorize common resource request/limit combinations
- **QoS Quick Check**: Use `kubectl get pods -o custom-columns` for QoS overview
- **Quota Monitoring**: Use `kubectl describe resourcequota` for usage analysis

### Daily Practice Goals
- [ ] Configure resource requests/limits in under 3 minutes
- [ ] Create LimitRange with appropriate constraints
- [ ] Set up ResourceQuota for namespace control
- [ ] Troubleshoot resource-related issues efficiently

## Next Day Preparation

### Review Before Day 18
- [ ] Resource request vs limit concepts
- [ ] QoS class implications for eviction
- [ ] LimitRange enforcement mechanisms
- [ ] ResourceQuota scope and monitoring

### What's Coming Next
Day 18 will cover Static Pods, which connects to resource management:
- Static pods use same resource specifications ✓
- Understanding kubelet resource management ✓  
- Control plane component resource allocation ✓

## Study Notes Section
_Use this space to document resource patterns specific to your environment_

**Resource Patterns:**
- Small workload: cpu=100m, memory=128Mi
- Medium workload: ________________
- Large workload: _________________
- Database: ______________________

**QoS Guidelines:**
- Critical services: Guaranteed QoS
- Web applications: _______________
- Batch jobs: ____________________

**Quota Templates:**
```yaml
# Development namespace quota:
requests.cpu: ____
requests.memory: ____
# Production namespace quota:
requests.cpu: ____
requests.memory: ____
```

**Personal Resource Management Patterns:**
_Record your preferred resource configurations and monitoring approaches_