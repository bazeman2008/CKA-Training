# Day 19: Workload Troubleshooting

## Overview (1.5 hours)
Master systematic troubleshooting of pod failures, container startup issues, and workload problems using kubectl describe, logs, events analysis, and diagnostic techniques essential for CKA exam success.

## Why This Matters for the CKA Exam
- **Troubleshooting comprises 30% of the CKA exam**
- **Pod failure diagnosis** is the most common troubleshooting scenario
- **Systematic approach** saves time and ensures thorough problem resolution
- **Event analysis** provides critical debugging information
- **Log interpretation** skills are essential for container issue resolution

## Task 1: Pod Failure Diagnosis Methodology (40 minutes)

### What You'll Do
Develop a systematic approach to diagnosing pod failures using a consistent troubleshooting methodology.

### Step-by-Step Instructions

#### 1. Establish Troubleshooting Framework
```bash
# Create troubleshooting namespace
kubectl create namespace troubleshoot-test

# Define systematic troubleshooting function
cat > troubleshoot-framework.sh << 'EOF'
#!/bin/bash
POD_NAME=$1
NAMESPACE=${2:-default}

echo "=== TROUBLESHOOTING POD: $POD_NAME ==="
echo "1. Basic Pod Information:"
kubectl get pod $POD_NAME -n $NAMESPACE -o wide

echo -e "\n2. Pod Status Details:"
kubectl describe pod $POD_NAME -n $NAMESPACE | grep -A 5 "Status:\|Conditions:"

echo -e "\n3. Container Status:"
kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.status.containerStatuses[*]}' | jq .

echo -e "\n4. Recent Events:"
kubectl get events -n $NAMESPACE --field-selector involvedObject.name=$POD_NAME --sort-by=.metadata.creationTimestamp | tail -10

echo -e "\n5. Pod Logs:"
kubectl logs $POD_NAME -n $NAMESPACE --tail=20

echo -e "\n6. Previous Logs (if restarted):"
kubectl logs $POD_NAME -n $NAMESPACE --previous --tail=10 2>/dev/null || echo "No previous logs available"
EOF

chmod +x troubleshoot-framework.sh
```

#### 2. Create Failing Pod Scenarios
```bash
# Scenario 1: Image pull failure
cat > failing-pods.yaml << 'EOF'
---
# ImagePullBackOff scenario
apiVersion: v1
kind: Pod
metadata:
  name: image-pull-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: nonexistent-registry.io/fake-image:v1.0
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
---
# CrashLoopBackOff scenario
apiVersion: v1
kind: Pod
metadata:
  name: crash-loop-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'Starting...'; sleep 5; exit 1"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
---
# Init container failure
apiVersion: v1
kind: Pod
metadata:
  name: init-fail
  namespace: troubleshoot-test
spec:
  initContainers:
  - name: init-setup
    image: busybox
    command: ["sh", "-c", "echo 'Init starting'; sleep 5; exit 1"]
  containers:
  - name: main-app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
---
# Resource constraint failure
apiVersion: v1
kind: Pod
metadata:
  name: resource-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: resource-hog
    image: nginx
    resources:
      requests:
        memory: "16Gi"  # Impossible memory request
        cpu: "32"       # Impossible CPU request
EOF

# Apply failing scenarios
kubectl apply -f failing-pods.yaml
sleep 10
```

#### 3. Practice Systematic Diagnosis
```bash
# Diagnose each failing pod systematically
echo "=== DIAGNOSING IMAGE PULL FAILURE ==="
./troubleshoot-framework.sh image-pull-fail troubleshoot-test

echo -e "\n=== DIAGNOSING CRASH LOOP FAILURE ==="
./troubleshoot-framework.sh crash-loop-fail troubleshoot-test

echo -e "\n=== DIAGNOSING INIT CONTAINER FAILURE ==="
./troubleshoot-framework.sh init-fail troubleshoot-test

echo -e "\n=== DIAGNOSING RESOURCE CONSTRAINT FAILURE ==="
./troubleshoot-framework.sh resource-fail troubleshoot-test
```

#### 4. Advanced Diagnostic Techniques
```bash
# Check node resource availability
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl top nodes

# Analyze events across namespace
kubectl get events -n troubleshoot-test --sort-by=.metadata.creationTimestamp

# Check for network policies affecting pods
kubectl get networkpolicies -n troubleshoot-test
kubectl describe networkpolicies -n troubleshoot-test

# Examine scheduler decisions
kubectl get events --field-selector reason=FailedScheduling -n troubleshoot-test
```

### CKA Exam Connection
- **Systematic Approach**: Consistent methodology prevents missing issues
- **Time Management**: Structured diagnosis saves exam time
- **Common Patterns**: Recognition of frequent failure modes
- **Documentation Skills**: Using kubectl efficiently for diagnosis

### Validation Commands
```bash
# Quick pod status overview
kubectl get pods -n troubleshoot-test -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,RESTARTS:.status.containerStatuses[0].restartCount"

# Event summary
kubectl get events -n troubleshoot-test --field-selector type=Warning
```

## Task 2: Container Startup Issue Diagnosis (30 minutes)

### What You'll Do
Focus specifically on container startup problems including command issues, environment problems, and configuration errors.

### Step-by-Step Instructions

#### 1. Command and Args Issues
```bash
# Create pods with startup command problems
cat > startup-issues.yaml << 'EOF'
---
# Wrong command syntax
apiVersion: v1
kind: Pod
metadata:
  name: wrong-command
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["invalid-command", "that-does-not-exist"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
---
# Missing environment variable
apiVersion: v1
kind: Pod
metadata:
  name: missing-env
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'Required var: '$REQUIRED_VAR; sleep 10"]
    env:
    - name: OPTIONAL_VAR
      value: "optional"
    # Missing REQUIRED_VAR causes application issues
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
---
# Volume mount issues
apiVersion: v1
kind: Pod
metadata:
  name: volume-mount-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: nonexistent-volume
      mountPath: /app/config
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  # volumes section missing - causes mount failure
EOF

# Apply and diagnose
kubectl apply -f startup-issues.yaml
sleep 10

# Diagnose startup issues
for pod in wrong-command missing-env volume-mount-fail; do
    echo "=== DIAGNOSING: $pod ==="
    kubectl describe pod $pod -n troubleshoot-test | grep -A 10 "Events:\|State:"
    echo ""
done
```

#### 2. Permission and Security Context Issues
```bash
# Create security context problems
cat > security-issues.yaml << 'EOF'
---
# Permission denied scenarios
apiVersion: v1
kind: Pod
metadata:
  name: permission-denied
  namespace: troubleshoot-test
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
  containers:
  - name: app
    image: nginx
    # nginx needs to bind to port 80 but running as non-root
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
---
# Read-only filesystem issues
apiVersion: v1
kind: Pod
metadata:
  name: readonly-fs-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'test' > /tmp/test.txt; sleep 60"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
    securityContext:
      readOnlyRootFilesystem: true
      # Missing tmpfs volume for /tmp
EOF

# Apply and analyze security issues
kubectl apply -f security-issues.yaml
sleep 15

# Check security-related failures
kubectl logs permission-denied -n troubleshoot-test
kubectl logs readonly-fs-fail -n troubleshoot-test
kubectl describe pod permission-denied -n troubleshoot-test | grep -A 5 Events
```

#### 3. Configuration and Dependency Issues
```bash
# Create configuration-related failures
cat > config-issues.yaml << 'EOF'
---
# ConfigMap dependency failure
apiVersion: v1
kind: Pod
metadata:
  name: configmap-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /config/app.conf; sleep 60"]
    volumeMounts:
    - name: config
      mountPath: /config
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: config
    configMap:
      name: nonexistent-configmap  # ConfigMap doesn't exist
---
# Secret dependency failure
apiVersion: v1
kind: Pod
metadata:
  name: secret-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'DB Password: '$DB_PASSWORD; sleep 60"]
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: nonexistent-secret  # Secret doesn't exist
          key: password
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# Apply and diagnose configuration issues
kubectl apply -f config-issues.yaml
sleep 10

# Analyze configuration failures
kubectl describe pod configmap-fail -n troubleshoot-test | grep -A 10 "Events:\|Volumes:"
kubectl describe pod secret-fail -n troubleshoot-test | grep -A 10 "Events:\|Environment:"
```

#### 4. Probes and Health Check Issues
```bash
# Create probe-related failures
cat > probe-issues.yaml << 'EOF'
---
# Failing readiness probe
apiVersion: v1
kind: Pod
metadata:
  name: readiness-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /nonexistent-endpoint
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
---
# Failing liveness probe
apiVersion: v1
kind: Pod
metadata:
  name: liveness-fail
  namespace: troubleshoot-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 30; while true; do sleep 1; done"]
    livenessProbe:
      exec:
        command:
        - test
        - -f
        - /tmp/healthy  # File that doesn't exist
      initialDelaySeconds: 10
      periodSeconds: 5
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# Apply and monitor probe failures
kubectl apply -f probe-issues.yaml
sleep 30

# Check probe-related events and status
kubectl describe pod readiness-fail -n troubleshoot-test | grep -A 5 "Readiness:\|Events:"
kubectl describe pod liveness-fail -n troubleshoot-test | grep -A 5 "Liveness:\|Events:"
```

### CKA Exam Connection
- **Container Runtime**: Understanding startup sequences
- **Security**: Diagnosing permission and security context issues
- **Configuration Management**: Troubleshooting config dependencies
- **Health Monitoring**: Understanding probe failure impacts

### Validation Commands
```bash
# Check container startup issues
kubectl get pods -n troubleshoot-test -o custom-columns="NAME:.metadata.name,READY:.status.containerStatuses[0].ready,STARTED:.status.containerStatuses[0].started"

# Analyze startup failures
kubectl get events -n troubleshoot-test --field-selector reason=Failed
```

## Task 3: Advanced Troubleshooting with Events and Logs (25 minutes)

### What You'll Do
Master advanced techniques for analyzing events, logs, and system state to diagnose complex workload issues.

### Step-by-Step Instructions

#### 1. Comprehensive Events Analysis
```bash
# Create event analysis helper script
cat > analyze-events.sh << 'EOF'
#!/bin/bash
NAMESPACE=${1:-troubleshoot-test}

echo "=== EVENT ANALYSIS FOR NAMESPACE: $NAMESPACE ==="

echo "1. All Events (chronological):"
kubectl get events -n $NAMESPACE --sort-by=.metadata.creationTimestamp

echo -e "\n2. Warning Events Only:"
kubectl get events -n $NAMESPACE --field-selector type=Warning

echo -e "\n3. Events by Reason:"
kubectl get events -n $NAMESPACE -o custom-columns="TIME:.firstTimestamp,REASON:.reason,OBJECT:.involvedObject.name,MESSAGE:.message"

echo -e "\n4. Failed Scheduling Events:"
kubectl get events -n $NAMESPACE --field-selector reason=FailedScheduling

echo -e "\n5. Pull/Mount Errors:"
kubectl get events -n $NAMESPACE | grep -E "(Failed|Error|Pull|Mount)"
EOF

chmod +x analyze-events.sh
./analyze-events.sh troubleshoot-test
```

#### 2. Advanced Log Analysis Techniques
```bash
# Create multi-container pod for log analysis
cat > multi-container-debug.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-debug
  namespace: troubleshoot-test
spec:
  containers:
  - name: main-app
    image: nginx
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  - name: sidecar-logger
    image: busybox
    command: ["sh", "-c", "while true; do echo $(date): Sidecar logging; sleep 10; done"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  - name: failing-sidecar
    image: busybox
    command: ["sh", "-c", "echo 'Starting failing sidecar'; sleep 15; exit 1"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

kubectl apply -f multi-container-debug.yaml
sleep 20

# Advanced log analysis commands
echo "=== MULTI-CONTAINER LOG ANALYSIS ==="

# All containers logs
kubectl logs multi-container-debug -n troubleshoot-test --all-containers=true

# Specific container logs
kubectl logs multi-container-debug -n troubleshoot-test -c main-app
kubectl logs multi-container-debug -n troubleshoot-test -c sidecar-logger
kubectl logs multi-container-debug -n troubleshoot-test -c failing-sidecar

# Previous logs for restarting containers
kubectl logs multi-container-debug -n troubleshoot-test -c failing-sidecar --previous

# Follow logs with timestamps
kubectl logs multi-container-debug -n troubleshoot-test -c sidecar-logger -f --timestamps &
sleep 10
# Stop following with Ctrl+C
```

#### 3. System-Level Debugging
```bash
# Node and cluster-level debugging
echo "=== SYSTEM-LEVEL DEBUGGING ==="

# Check node conditions
kubectl describe nodes | grep -A 10 "Conditions:"

# Check system pods
kubectl get pods -n kube-system | grep -v Running

# Check resource pressure
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check for system events
kubectl get events --all-namespaces --field-selector type=Warning --sort-by=.metadata.creationTimestamp | tail -20

# Check kubelet logs on current node
sudo journalctl -u kubelet --since "10 minutes ago" | grep -i error | tail -10
```

#### 4. Debugging Workflow Automation
```bash
# Create comprehensive debugging script
cat > debug-workload.sh << 'EOF'
#!/bin/bash
POD_NAME=$1
NAMESPACE=${2:-default}

if [ -z "$POD_NAME" ]; then
    echo "Usage: $0 <pod-name> [namespace]"
    exit 1
fi

echo "=== COMPREHENSIVE WORKLOAD DEBUG: $POD_NAME ==="

# 1. Basic status
echo "1. Pod Status:"
kubectl get pod $POD_NAME -n $NAMESPACE -o wide

# 2. Resource usage
echo -e "\n2. Resource Usage:"
kubectl top pod $POD_NAME -n $NAMESPACE --containers 2>/dev/null || echo "Metrics not available"

# 3. Detailed description
echo -e "\n3. Pod Description:"
kubectl describe pod $POD_NAME -n $NAMESPACE

# 4. Events
echo -e "\n4. Related Events:"
kubectl get events -n $NAMESPACE --field-selector involvedObject.name=$POD_NAME

# 5. Logs for all containers
echo -e "\n5. Container Logs:"
CONTAINERS=$(kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.spec.containers[*].name}')
for container in $CONTAINERS; do
    echo "--- Container: $container ---"
    kubectl logs $POD_NAME -n $NAMESPACE -c $container --tail=10
    echo ""
done

# 6. Previous logs if containers restarted
echo -e "\n6. Previous Logs (if restarted):"
for container in $CONTAINERS; do
    kubectl logs $POD_NAME -n $NAMESPACE -c $container --previous --tail=5 2>/dev/null && echo "Previous logs for $container" || echo "No previous logs for $container"
done

# 7. Security context and permissions
echo -e "\n7. Security Context:"
kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.spec.securityContext}' | jq . 2>/dev/null || echo "No pod security context"
kubectl get pod $POD_NAME -n $NAMESPACE -o jsonpath='{.spec.containers[*].securityContext}' | jq . 2>/dev/null || echo "No container security context"
EOF

chmod +x debug-workload.sh

# Test the comprehensive debugging script
./debug-workload.sh crash-loop-fail troubleshoot-test
```

### CKA Exam Connection
- **Efficient Diagnosis**: Systematic approach to complex problems
- **Multi-Container**: Understanding sidecar and init container issues
- **System Integration**: Connecting pod issues to cluster state
- **Time Management**: Automated debugging workflows

### Validation Commands
```bash
# Quick troubleshooting overview
kubectl get pods -n troubleshoot-test --field-selector status.phase!=Running
kubectl get events -n troubleshoot-test --field-selector type=Warning --sort-by=.metadata.creationTimestamp | tail -10
```

## Task 4: Troubleshooting Performance and Resource Issues (15 minutes)

### What You'll Do
Practice diagnosing performance problems and resource-related issues affecting workload operation.

### Step-by-Step Instructions

#### 1. Resource Exhaustion Scenarios
```bash
# Create resource exhaustion test
cat > resource-exhaustion.yaml << 'EOF'
---
# Memory exhaustion
apiVersion: v1
kind: Pod
metadata:
  name: memory-exhausted
  namespace: troubleshoot-test
spec:
  containers:
  - name: memory-hog
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
    resources:
      requests:
        memory: "100Mi"
        cpu: "100m"
      limits:
        memory: "150Mi"  # Less than what stress tries to allocate
        cpu: "200m"
---
# CPU throttling
apiVersion: v1
kind: Pod
metadata:
  name: cpu-throttled
  namespace: troubleshoot-test
spec:
  containers:
  - name: cpu-hog
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "2", "--timeout", "60s"]
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"  # Limited CPU
        memory: "128Mi"
EOF

kubectl apply -f resource-exhaustion.yaml
sleep 15

# Monitor resource exhaustion
kubectl top pods -n troubleshoot-test --sort-by=memory
kubectl describe pod memory-exhausted -n troubleshoot-test | grep -A 5 "State:\|Events:"
```

#### 2. Performance Monitoring and Analysis
```bash
# Create performance monitoring script
cat > monitor-performance.sh << 'EOF'
#!/bin/bash
NAMESPACE=troubleshoot-test

echo "=== PERFORMANCE MONITORING ==="

echo "1. Resource Usage by Pod:"
kubectl top pods -n $NAMESPACE --sort-by=cpu

echo -e "\n2. Node Resource Usage:"
kubectl top nodes

echo -e "\n3. Resource Requests vs Limits:"
kubectl get pods -n $NAMESPACE -o custom-columns="NAME:.metadata.name,CPU_REQ:.spec.containers[*].resources.requests.cpu,CPU_LIM:.spec.containers[*].resources.limits.cpu,MEM_REQ:.spec.containers[*].resources.requests.memory,MEM_LIM:.spec.containers[*].resources.limits.memory"

echo -e "\n4. QoS Classes:"
kubectl get pods -n $NAMESPACE -o custom-columns="NAME:.metadata.name,QOS:.status.qosClass"

echo -e "\n5. Container Restart Counts:"
kubectl get pods -n $NAMESPACE -o custom-columns="NAME:.metadata.name,RESTARTS:.status.containerStatuses[*].restartCount"
EOF

chmod +x monitor-performance.sh
./monitor-performance.sh
```

#### 3. Diagnostic Summary and Cleanup
```bash
# Create final diagnostic summary
echo "=== TROUBLESHOOTING SUMMARY ==="
echo "Failed Pods:"
kubectl get pods -n troubleshoot-test | grep -v Running | grep -v Completed

echo -e "\nPod States:"
kubectl get pods -n troubleshoot-test -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,READY:.status.containerStatuses[*].ready,RESTARTS:.status.containerStatuses[*].restartCount"

echo -e "\nCritical Events:"
kubectl get events -n troubleshoot-test --field-selector type=Warning | head -10

# Clean up troubleshooting resources
echo -e "\nCleaning up troubleshooting resources..."
kubectl delete namespace troubleshoot-test
rm -f troubleshoot-framework.sh analyze-events.sh debug-workload.sh monitor-performance.sh
rm -f failing-pods.yaml startup-issues.yaml security-issues.yaml config-issues.yaml probe-issues.yaml multi-container-debug.yaml resource-exhaustion.yaml
```

## Common Issues and Solutions

### Issue 1: Pod Stuck in Pending
**Problem**: Pod remains in Pending state
**Solution**: 
```bash
# Check scheduling issues
kubectl describe pod <pod-name> | grep Events -A 10
kubectl get nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Common causes: resource constraints, node selectors, taints
```

### Issue 2: Pod in CrashLoopBackOff
**Problem**: Container keeps restarting
**Solution**:
```bash
# Check container logs and previous logs
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name> | grep -A 10 "Last State"

# Common causes: application errors, missing dependencies, wrong commands
```

### Issue 3: Pod Running but Not Ready
**Problem**: Pod shows Running but 0/1 Ready
**Solution**:
```bash
# Check readiness probes
kubectl describe pod <pod-name> | grep -A 5 "Readiness"
kubectl logs <pod-name>

# Common causes: failing readiness probes, application startup time
```

### Issue 4: ImagePullBackOff
**Problem**: Cannot pull container image
**Solution**:
```bash
# Check image name and registry access
kubectl describe pod <pod-name> | grep -A 5 "Events"
kubectl get events --field-selector reason=Failed

# Common causes: typos in image name, registry permissions, network issues
```

## Time Management Tips

### For CKA Exam Success
- **Systematic Approach**: Always follow the same diagnostic sequence
- **Quick Commands**: Use aliases and shortcuts for common debugging tasks
- **Event Focus**: Events often contain the most critical information
- **Log Efficiency**: Use `--tail` and `--previous` flags to focus log analysis

### Daily Practice Goals
- [ ] Complete pod diagnosis in under 5 minutes
- [ ] Identify failure root cause quickly
- [ ] Use systematic troubleshooting methodology
- [ ] Efficiently analyze logs and events

## Next Day Preparation

### Review Before Next Week
- [ ] Systematic troubleshooting approach mastered
- [ ] kubectl describe and logs usage efficient
- [ ] Event analysis skills developed
- [ ] Common failure patterns recognized

### What's Coming Next
You've completed Week 3! Next week will cover Storage & First Assessment:
- Troubleshooting skills will be essential for storage issues ✓
- Event analysis applies to storage problems ✓  
- Systematic approach works for all Kubernetes components ✓

## Study Notes Section
_Use this space to document troubleshooting patterns specific to your environment_

**Troubleshooting Checklist:**
1. [ ] kubectl get pods -o wide
2. [ ] kubectl describe pod <name>
3. [ ] kubectl logs <name> --previous
4. [ ] kubectl get events --field-selector involvedObject.name=<name>
5. [ ] Check node resources and conditions
6. [ ] Verify image, config, and dependency availability

**Common Error Patterns:**
- ImagePullBackOff: ________________
- CrashLoopBackOff: _______________
- Pending: _______________________
- Init:Error: ____________________

**Quick Diagnostic Commands:**
```bash
# Pod overview
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,READY:.status.containerStatuses[*].ready"

# Events summary
kubectl get events --sort-by=.metadata.creationTimestamp | tail -10

# Resource check
kubectl top pods --sort-by=memory
```

**Personal Troubleshooting Patterns:**
_Record your preferred diagnostic approaches and common issues in your environment_