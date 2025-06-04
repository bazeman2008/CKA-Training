# Day 15: Advanced Scheduling

## Overview (1.5 hours)
Master Kubernetes advanced scheduling techniques including node selectors, node affinity, taints and tolerations, and manual scheduling methods essential for CKA exam success.

## Why This Matters for the CKA Exam
- **Scheduling questions appear in 20-25% of exam tasks**
- **Pod placement control** is critical for multi-node cluster scenarios
- **Taints and tolerations** frequently tested for node isolation
- **Manual scheduling** required when scheduler is unavailable
- **Node affinity** enables complex placement requirements

## Task 1: Node Selectors and Node Affinity (45 minutes)

### What You'll Do
Learn to control pod placement using node labels and advanced affinity rules.

### Step-by-Step Instructions

#### 1. Basic Node Selectors
```bash
# Label nodes for scheduling
kubectl label nodes worker-node-1 disktype=ssd
kubectl label nodes worker-node-2 disktype=hdd
kubectl label nodes worker-node-1 environment=production
kubectl label nodes worker-node-2 environment=development

# Verify node labels
kubectl get nodes --show-labels
kubectl describe node worker-node-1 | grep Labels -A 10
```

#### 2. Create Pod with Node Selector
```bash
# Create pod with node selector (imperative)
kubectl run nginx-ssd --image=nginx --dry-run=client -o yaml > nginx-ssd.yaml

# Edit to add node selector
cat >> nginx-ssd.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ssd
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
EOF

# Apply and verify placement
kubectl apply -f nginx-ssd.yaml
kubectl get pods -o wide
kubectl describe pod nginx-ssd | grep Node:
```

#### 3. Node Affinity Configuration
```bash
# Create pod with node affinity
cat > nginx-affinity.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nginx-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
            - nvme
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - production
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
EOF

# Apply and verify
kubectl apply -f nginx-affinity.yaml
kubectl get pods -o wide
kubectl describe pod nginx-affinity | grep Node -A 5
```

#### 4. Advanced Affinity Scenarios
```bash
# Create pod with anti-affinity (avoid same node)
cat > redis-antiaffinity.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cluster
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: redis
            topologyKey: kubernetes.io/hostname
      containers:
      - name: redis
        image: redis:6-alpine
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
EOF

# Apply and verify distribution
kubectl apply -f redis-antiaffinity.yaml
kubectl get pods -l app=redis -o wide
```

### CKA Exam Connection
- **Node Placement (25% of exam)**: Control where pods run
- **Resource Optimization**: Place workloads on appropriate nodes
- **High Availability**: Distribute pods across nodes
- **Troubleshooting**: Understand why pods don't schedule

### Validation Commands
```bash
# Check pod placement
kubectl get pods -o custom-columns="NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase"

# Verify scheduling decisions
kubectl describe pod nginx-ssd | grep Events -A 10
kubectl get events --field-selector involvedObject.name=nginx-affinity
```

## Task 2: Taints and Tolerations (35 minutes)

### What You'll Do
Configure node taints to repel pods and pod tolerations to allow scheduling on tainted nodes.

### Step-by-Step Instructions

#### 1. Understanding and Applying Taints
```bash
# View existing taints
kubectl describe nodes | grep Taints

# Add taints to nodes
kubectl taint nodes worker-node-1 dedicated=special-workloads:NoSchedule
kubectl taint nodes worker-node-2 maintenance=true:NoExecute

# Verify taints applied
kubectl describe node worker-node-1 | grep Taints
kubectl describe node worker-node-2 | grep Taints
```

#### 2. Test Taint Effects
```bash
# Try to schedule pod on tainted node (should fail)
kubectl run test-pod --image=nginx
kubectl get pods -o wide
kubectl describe pod test-pod | grep Events -A 10

# Check if pod is stuck in Pending
kubectl get pods --field-selector status.phase=Pending
```

#### 3. Create Pods with Tolerations
```bash
# Pod that tolerates NoSchedule taint
cat > toleration-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "special-workloads"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "128Mi"
        cpu: "100m"
  nodeSelector:
    kubernetes.io/hostname: worker-node-1
EOF

# Apply and verify
kubectl apply -f toleration-pod.yaml
kubectl get pods -o wide
```

#### 4. Advanced Toleration Scenarios
```bash
# Pod with multiple tolerations
cat > multi-toleration.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: multi-toleration
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "special-workloads"
    effect: "NoSchedule"
  - key: "maintenance"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
    tolerationSeconds: 300
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "25m"
      limits:
        memory: "64Mi"
        cpu: "50m"
EOF

# Apply and test
kubectl apply -f multi-toleration.yaml
kubectl get pods -o wide
```

#### 5. Managing Taints
```bash
# Remove taints
kubectl taint nodes worker-node-1 dedicated=special-workloads:NoSchedule-
kubectl taint nodes worker-node-2 maintenance=true:NoExecute-

# Verify removal
kubectl describe nodes | grep Taints
```

### CKA Exam Connection
- **Node Isolation**: Dedicated nodes for specific workloads
- **Maintenance Operations**: Safe node maintenance with NoExecute
- **Resource Management**: Control workload distribution
- **Troubleshooting**: Understanding why pods can't schedule

### Validation Commands
```bash
# Check taint status
kubectl get nodes -o custom-columns="NAME:.metadata.name,TAINTS:.spec.taints[*].key"

# Verify pod tolerations
kubectl describe pod toleration-pod | grep Tolerations -A 5
```

## Task 3: Manual Scheduling Techniques (20 minutes)

### What You'll Do
Practice manual pod scheduling when the scheduler is unavailable or for specific placement requirements.

### Step-by-Step Instructions

#### 1. Basic Manual Scheduling
```bash
# Create pod with nodeName specified
cat > manual-schedule.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: manual-nginx
spec:
  nodeName: worker-node-1  # Manual assignment
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "128Mi"
        cpu: "100m"
EOF

# Apply and verify
kubectl apply -f manual-schedule.yaml
kubectl get pods -o wide
```

#### 2. Simulate Scheduler Failure Scenario
```bash
# Check scheduler status
kubectl get pods -n kube-system | grep scheduler

# Create pod without scheduler (pending)
kubectl run pending-pod --image=nginx
kubectl get pods

# Manually assign the pending pod
kubectl get pod pending-pod -o yaml > pending-pod.yaml
# Edit pending-pod.yaml to add nodeName: worker-node-2
kubectl delete pod pending-pod
# Modify the yaml and reapply with nodeName specified
```

#### 3. Advanced Manual Scheduling
```bash
# Create pod with specific scheduling constraints
cat > constrained-manual.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: constrained-manual
spec:
  nodeName: worker-node-1
  containers:
  - name: webapp
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
    env:
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
EOF

# Apply and test
kubectl apply -f constrained-manual.yaml
kubectl exec constrained-manual -- env | grep NODE_NAME
```

### CKA Exam Connection
- **Emergency Scheduling**: When scheduler fails
- **Specific Placement**: Critical workload placement
- **Troubleshooting**: Understanding scheduling decisions
- **Control Plane Issues**: Manual intervention scenarios

### Validation Commands
```bash
# Verify manual scheduling
kubectl get pods -o custom-columns="NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase"

# Check for scheduler events
kubectl get events --field-selector reason=Scheduled
```

## Task 4: Scheduling Troubleshooting Practice (10 minutes)

### What You'll Do
Practice diagnosing and resolving common scheduling problems.

### Common Scheduling Issues

#### 1. Insufficient Resources
```bash
# Create pod with impossible resource requirements
cat > impossible-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: impossible-pod
spec:
  containers:
  - name: resource-hog
    image: nginx
    resources:
      requests:
        memory: "64Gi"  # More than available
        cpu: "32"       # More than available
EOF

# Apply and diagnose
kubectl apply -f impossible-pod.yaml
kubectl get pods
kubectl describe pod impossible-pod | grep Events -A 10
```

#### 2. Node Selector Issues
```bash
# Create pod with non-existent node selector
cat > bad-selector.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: bad-selector
spec:
  nodeSelector:
    nonexistent: label
  containers:
  - name: nginx
    image: nginx
EOF

# Apply and diagnose
kubectl apply -f bad-selector.yaml
kubectl describe pod bad-selector | grep Events -A 10
```

#### 3. Taint Issues
```bash
# Apply taint and try to schedule incompatible pod
kubectl taint nodes worker-node-1 special=true:NoSchedule
kubectl run incompatible --image=nginx
kubectl describe pod incompatible | grep Events -A 10

# Clean up
kubectl delete pod incompatible
kubectl taint nodes worker-node-1 special=true:NoSchedule-
```

### Diagnostic Commands
```bash
# Comprehensive scheduling diagnosis
kubectl get pods --field-selector status.phase=Pending
kubectl get events --sort-by=.metadata.creationTimestamp | tail -10
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl top nodes
```

## Common Issues and Solutions

### Issue 1: Pod Stuck in Pending State
**Problem**: Pod remains in Pending status
**Solution**: 
```bash
# Diagnostic steps
kubectl describe pod <pod-name> | grep Events -A 10
kubectl get nodes -o wide
kubectl describe nodes | grep -A 5 "Conditions\|Taints\|Allocated"

# Common fixes
kubectl get pods --field-selector status.phase=Pending
kubectl delete pod <pending-pod>  # If configuration error
```

### Issue 2: Node Affinity Not Working
**Problem**: Pods not scheduling according to affinity rules
**Solution**:
```bash
# Check node labels
kubectl get nodes --show-labels
kubectl describe node <node-name> | grep Labels -A 10

# Verify affinity syntax
kubectl describe pod <pod-name> | grep -A 10 "Node-Selectors\|Tolerations"
```

### Issue 3: Taint/Toleration Mismatch
**Problem**: Pod with toleration still not scheduling
**Solution**:
```bash
# Check exact taint values
kubectl describe nodes | grep Taints -A 3

# Verify toleration syntax
kubectl get pod <pod-name> -o yaml | grep -A 10 tolerations

# Check for exact key/value/effect match
```

## Time Management Tips

### For CKA Exam Success
- **Quick Label Check**: Use `kubectl get nodes --show-labels` first
- **Imperative + Edit**: Generate YAML then modify for affinity
- **Taint Patterns**: Remember `key=value:effect` format
- **Manual Scheduling**: Add `nodeName` to existing pod spec

### Daily Practice Goals
- [ ] Complete node selector tasks in under 10 minutes
- [ ] Configure node affinity rules efficiently
- [ ] Apply and remove taints quickly
- [ ] Diagnose scheduling issues systematically

## Next Day Preparation

### Review Before Day 16
- [ ] Node labeling and selector syntax
- [ ] Node affinity vs pod affinity differences
- [ ] Taint effects: NoSchedule, PreferNoSchedule, NoExecute
- [ ] Manual scheduling with nodeName

### What's Coming Next
Day 16 will cover DaemonSets and Jobs, which build on scheduling concepts:
- DaemonSets use node selectors and tolerations ✓
- Jobs require resource management understanding ✓  
- Workload troubleshooting uses scheduling diagnostics ✓

## Study Notes Section
_Use this space to document scheduling configurations specific to your environment_

**Node Configuration:**
- Master Node Labels: ________________
- Worker Node 1 Labels: ______________
- Worker Node 2 Labels: ______________
- Common Taint Patterns: _____________

**Quick Reference Commands:**
```bash
# Essential scheduling commands
kubectl label nodes <node> <key>=<value>
kubectl taint nodes <node> <key>=<value>:<effect>
kubectl get pods -o wide
kubectl describe pod <name> | grep Events -A 10
```

**Personal Scheduling Patterns:**
_Record your preferred scheduling configurations and shortcuts_