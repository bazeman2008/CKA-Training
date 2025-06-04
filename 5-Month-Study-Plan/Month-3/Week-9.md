# Week 9: Application Troubleshooting

## Overview
Master systematic approaches to troubleshooting application failures in Kubernetes environments.

## Daily Tasks

### Day 57: Pod Failure Diagnosis (1.5 hours)
- [ ] Practice systematic pod troubleshooting methodology
- [ ] Debug container startup and runtime issues
- [ ] Master `kubectl logs`, `describe`, events analysis

### Day 58: Resource Constraint Issues (1.5 hours)
- [ ] Identify CPU and memory constraints
- [ ] Understand QoS impact on scheduling and eviction
- [ ] Diagnose performance bottlenecks

### Day 59: Configuration Troubleshooting (1.5 hours)
- [ ] Debug ConfigMap and Secret mounting issues
- [ ] Fix environment variable problems
- [ ] Resolve configuration-related failures

### Day 60: Multi-Component Failures (1.5 hours)
- [ ] Practice complex scenarios with multiple failures
- [ ] Develop systematic approach to complex problems
- [ ] Build troubleshooting methodology under pressure

### Day 61: Storage Troubleshooting Advanced (1.5 hours)
- [ ] Fix PVC mounting failures and permissions
- [ ] Resolve storage driver problems
- [ ] Debug volume attachment issues

## Weekend Session (3 hours)
- [ ] Complex troubleshooting scenarios
- [ ] Speed improvement for troubleshooting tasks
- [ ] **Goal Check:** Solve application issues in under 12 minutes

## Troubleshooting Methodology

### Systematic Pod Diagnosis Steps
1. **Check pod status**
   ```bash
   kubectl get pod <pod-name> -o wide
   kubectl describe pod <pod-name>
   ```

2. **Analyze events**
   ```bash
   kubectl get events --field-selector involvedObject.name=<pod-name>
   kubectl get events --sort-by='.lastTimestamp'
   ```

3. **Review logs**
   ```bash
   # Current logs
   kubectl logs <pod-name>
   kubectl logs <pod-name> -c <container-name>
   
   # Previous container logs
   kubectl logs <pod-name> --previous
   
   # Follow logs
   kubectl logs <pod-name> -f
   
   # Logs with timestamps
   kubectl logs <pod-name> --timestamps
   ```

4. **Debug running containers**
   ```bash
   kubectl exec <pod-name> -- <command>
   kubectl exec -it <pod-name> -- /bin/bash
   ```

### Common Pod Failure Patterns

#### ImagePullBackOff
```bash
# Check image name and registry
kubectl describe pod <pod-name> | grep -A 5 "Events"

# Verify image exists
kubectl run test --image=<image-name> --dry-run=client -o yaml

# Check image pull secrets
kubectl get secret
kubectl describe secret <secret-name>
```

#### CrashLoopBackOff
```bash
# Check container exit code
kubectl describe pod <pod-name> | grep -A 10 "Last State"

# Review logs for crash reason
kubectl logs <pod-name> --previous

# Check liveness probe configuration
kubectl get pod <pod-name> -o yaml | grep -A 10 "livenessProbe"
```

#### Pending Pods
```bash
# Check for scheduling issues
kubectl describe pod <pod-name> | grep -A 10 "Events"

# Verify node resources
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check for PVC issues
kubectl get pvc
kubectl describe pvc <pvc-name>
```

### Resource Constraint Troubleshooting

```yaml
# Pod with resource issues
apiVersion: v1
kind: Pod
metadata:
  name: resource-test
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```bash
# Check resource usage
kubectl top pod <pod-name>
kubectl top pod <pod-name> --containers

# Check QoS class
kubectl get pod <pod-name> -o jsonpath='{.status.qosClass}'

# Find pods exceeding limits
kubectl get pods -o json | jq -r '.items[] | select(.status.phase=="Running") | select(.spec.containers[].resources.limits.memory) | .metadata.name'
```

### Configuration Issues

```bash
# ConfigMap not mounting
kubectl describe pod <pod-name> | grep -A 10 "Volumes"
kubectl get configmap
kubectl describe configmap <configmap-name>

# Secret mounting issues
kubectl get secret
kubectl describe secret <secret-name>

# Environment variable problems
kubectl exec <pod-name> -- env | grep <var-name>
kubectl get pod <pod-name> -o yaml | grep -A 10 "env:"
```

### Storage Troubleshooting

```bash
# PVC not binding
kubectl get pv
kubectl get pvc
kubectl describe pvc <pvc-name>

# Check storage class
kubectl get storageclass
kubectl describe storageclass <storage-class>

# Volume mount issues
kubectl describe pod <pod-name> | grep -A 10 "Mount"
kubectl exec <pod-name> -- ls -la /mount/path

# Permission problems
kubectl exec <pod-name> -- ls -la /
kubectl exec <pod-name> -- id
```

## Troubleshooting Exercises

### Exercise 1: Debug Failed Deployment
```yaml
# Create a broken deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: broken
  template:
    metadata:
      labels:
        app: broken
    spec:
      containers:
      - name: app
        image: nginx:latst  # Typo in image tag
        resources:
          requests:
            memory: "10Gi"  # Excessive memory request
        volumeMounts:
        - name: config
          mountPath: /etc/config
      volumes:
      - name: config
        configMap:
          name: missing-config  # ConfigMap doesn't exist
```

### Exercise 2: Fix Resource Constraints
1. Deploy application with insufficient resources
2. Identify OOMKilled containers
3. Adjust resource limits appropriately
4. Implement proper health checks

### Exercise 3: Resolve Configuration Issues
1. Create ConfigMap with wrong data
2. Mount Secret with incorrect permissions
3. Fix environment variable conflicts
4. Resolve volume mount problems

## Quick Reference Commands

```bash
# Pod debugging one-liners
kubectl get pods --all-namespaces | grep -v Running
kubectl get events --all-namespaces --sort-by='.lastTimestamp'
kubectl logs -l app=myapp --all-containers --prefix

# Resource analysis
for pod in $(kubectl get pods -o name); do kubectl top $pod; done

# Find pods in error state
kubectl get pods -A -o json | jq '.items[] | select(.status.phase != "Running" and .status.phase != "Succeeded") | {namespace: .metadata.namespace, name: .metadata.name, phase: .status.phase}'
```

## Learning Resources
- Troubleshooting Applications: https://kubernetes.io/docs/tasks/debug/debug-application/
- Debug Pods: https://kubernetes.io/docs/tasks/debug/debug-pod/
- Resource Management: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

## Success Criteria
- [ ] Can diagnose pod failures within 3 minutes
- [ ] Can identify and fix resource constraints
- [ ] Can resolve configuration mounting issues
- [ ] Can troubleshoot multi-component failures
- [ ] Can debug storage-related problems efficiently

## Notes Section
_Use this space to document common issues and their solutions_