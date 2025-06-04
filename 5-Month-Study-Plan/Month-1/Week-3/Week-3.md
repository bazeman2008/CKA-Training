# Week 3: Scheduling & Workload Management

## Overview
Master Kubernetes scheduling mechanisms, workload types, and resource management.

## Daily Tasks

### Day 15: Advanced Scheduling (1.5 hours)
- [ ] Practice node selectors and node affinity
- [ ] Configure taints and tolerations
- [ ] Practice manual scheduling techniques

### Day 16: DaemonSets & Jobs (1.5 hours)
- [ ] Configure DaemonSets for different use cases
- [ ] Create and manage Jobs and CronJobs
- [ ] Troubleshoot batch workloads

### Day 17: Resource Management (1.5 hours)
- [ ] Configure resource requests and limits
- [ ] Understand QoS classes
- [ ] Implement LimitRanges and ResourceQuotas

### Day 18: Static Pods (1.5 hours)
- [ ] Configure static pods
- [ ] Manage kubelet static pod directory
- [ ] Understand control plane static pods

### Day 19: Workload Troubleshooting (1.5 hours)
- [ ] Practice pod failure diagnosis
- [ ] Debug container startup issues
- [ ] Master `kubectl describe`, `kubectl logs`, events analysis

## Weekend Session (3 hours)
- [ ] Practice complex scheduling scenarios
- [ ] Workload management speed tests
- [ ] **Goal Check:** Solve scheduling problems efficiently

## Key Commands and Configurations

### Scheduling Examples

```yaml
# Node Selector
apiVersion: v1
kind: Pod
metadata:
  name: node-selector-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx

# Node Affinity
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
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
  containers:
  - name: nginx
    image: nginx
```

### Taints and Tolerations

```bash
# Add taint to node
kubectl taint nodes node1 key=value:NoSchedule

# Remove taint
kubectl taint nodes node1 key=value:NoSchedule-
```

```yaml
# Pod with toleration
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

### Workload Types

```yaml
# DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
      - name: monitoring
        image: monitoring:latest

# Job
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup:latest
      restartPolicy: OnFailure
  backoffLimit: 4

# CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: periodic-backup
spec:
  schedule: "*/30 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup:latest
          restartPolicy: OnFailure
```

### Resource Management

```yaml
# Pod with resources
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
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

# ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "1000m"
    requests.memory: "1Gi"
    limits.cpu: "2000m"
    limits.memory: "2Gi"
    persistentvolumeclaims: "2"

# LimitRange
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-limit
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
    type: Container
```

## Troubleshooting Commands

```bash
# Debug scheduling issues
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl logs <pod-name> -c <container-name>

# Check node capacity
kubectl describe nodes | grep -A 5 "Allocated resources"

# Static pod location
ls /etc/kubernetes/manifests/
```

## Learning Resources
- Pod Scheduling: https://kubernetes.io/docs/concepts/scheduling-eviction/
- Workload Resources: https://kubernetes.io/docs/concepts/workloads/
- Resource Management: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

## Success Criteria
- [ ] Can control pod placement using various scheduling techniques
- [ ] Understand all workload types and their use cases
- [ ] Can implement resource quotas and limits
- [ ] Can troubleshoot scheduling and workload issues
- [ ] Comfortable with static pod management

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_