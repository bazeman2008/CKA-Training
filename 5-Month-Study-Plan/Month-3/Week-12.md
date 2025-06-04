# Week 12: Final Preparation

## Overview
Final intensive preparation week focusing on weak areas and achieving exam readiness with comprehensive assessment.

## Daily Tasks

### Days 78-82: Weak Area Elimination (1.5 hours each)
- [ ] **Day 78:** Focus on lowest-scoring domain from mock exam
- [ ] **Day 79:** Intensive practice on problem areas
- [ ] **Day 80:** Speed drills on weak topics
- [ ] **Day 81:** Comprehensive domain review
- [ ] **Day 82:** Final troubleshooting scenarios

## Weekend Session - Final Mock Exam (4 hours)
- [ ] **MOCK EXAM:** Final comprehensive CKA simulation
- [ ] Final strategy refinement
- [ ] **Goal Check:** 85%+ score and exam confidence

## Domain-Specific Review

### Cluster Architecture, Installation & Configuration (25%)

#### Must-Know Tasks
```bash
# Cluster setup (must complete in 15-20 minutes)
kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# Install CNI
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Join worker nodes
kubeadm token create --print-join-command

# etcd backup (must complete in 5-10 minutes)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# etcd restore
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db --data-dir=/var/lib/etcd-backup
```

#### RBAC Quick Reference
```yaml
# Basic Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: jane
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Workloads & Scheduling (15%)

#### Deployment Management
```bash
# Create and manage deployments
kubectl create deployment nginx --image=nginx --replicas=3
kubectl scale deployment nginx --replicas=5
kubectl set image deployment/nginx nginx=nginx:1.19
kubectl rollout status deployment nginx
kubectl rollout undo deployment nginx

# Rolling update strategy
kubectl patch deployment nginx -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":1}}}}'
```

#### Scheduling Controls
```yaml
# Node affinity
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
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

# Taints and tolerations
kubectl taint nodes node1 key1=value1:NoSchedule
```

### Services & Networking (20%)

#### Service Types
```bash
# ClusterIP (default)
kubectl expose deployment nginx --port=80

# NodePort
kubectl expose deployment nginx --port=80 --type=NodePort

# LoadBalancer
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

#### Network Policies
```yaml
# Deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

# Allow specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Storage (10%)

#### PV/PVC Management
```yaml
# PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv-volume

# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

### Troubleshooting (30%)

#### Systematic Approach
1. **Identify the problem**
   - Read question carefully
   - Understand expected vs actual behavior
   - Check current context and namespace

2. **Gather information**
   ```bash
   kubectl get pods -o wide
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

3. **Analyze root cause**
   - Check resource constraints
   - Verify configuration
   - Test connectivity
   - Review permissions

4. **Implement fix**
   - Apply minimal necessary changes
   - Verify solution works
   - Document what was done

## Final Week Practice Schedule

### Day 78: Cluster Architecture Focus
- [ ] Practice cluster setup 3 times (target: <20 minutes each)
- [ ] etcd backup/restore drills (target: <10 minutes)
- [ ] RBAC scenarios (5 different user/permission combinations)
- [ ] Certificate management and renewal

### Day 79: Workloads & Scheduling Focus
- [ ] Complex deployment scenarios with rolling updates
- [ ] DaemonSet and Job configurations
- [ ] Pod scheduling with affinity/anti-affinity
- [ ] Resource management and QoS

### Day 80: Networking Focus
- [ ] Service exposure scenarios
- [ ] Network policy implementations
- [ ] Ingress configurations
- [ ] DNS troubleshooting

### Day 81: Storage & Security Focus
- [ ] PV/PVC configurations
- [ ] Security contexts and Pod Security Standards
- [ ] Volume mounting scenarios
- [ ] Storage troubleshooting

### Day 82: Troubleshooting Intensive
- [ ] Multi-component failure scenarios
- [ ] Node and cluster issues
- [ ] Application debugging
- [ ] Performance problem diagnosis

## Mock Exam Strategy

### Pre-Exam Setup (5 minutes)
```bash
# Set up environment
alias k=kubectl
export do='--dry-run=client -o yaml'
export now='--force --grace-period 0'

# Verify contexts
kubectl config get-contexts

# Test connectivity
kubectl get nodes
kubectl get pods --all-namespaces
```

### During Exam Approach
1. **Initial scan (3 minutes)**
   - Read all questions quickly
   - Note point values
   - Mark high-value, quick wins

2. **Execution phase (110 minutes)**
   - Start with highest value questions
   - Always verify context before starting
   - Document commands as you go
   - Skip if stuck for >10 minutes

3. **Review phase (7 minutes)**
   - Verify all solutions work
   - Check for partially completed tasks
   - Make final adjustments

### Critical Success Factors
- [ ] Always check/set correct context first
- [ ] Use imperative commands when possible
- [ ] Verify solutions actually work
- [ ] Manage time carefully
- [ ] Don't panic if something doesn't work immediately

## Final Checklist

### Technical Readiness
- [ ] Can set up cluster in <20 minutes
- [ ] Can perform etcd backup/restore in <10 minutes
- [ ] Can troubleshoot pod failures in <5 minutes
- [ ] Can configure RBAC correctly
- [ ] Can implement network policies
- [ ] Can expose services properly
- [ ] Can manage storage effectively

### Exam Readiness
- [ ] Scoring 85%+ on practice exams
- [ ] Completing within time limits
- [ ] Documentation navigation is fast
- [ ] Error recovery is quick
- [ ] Confident with all domains

### Mental Preparation
- [ ] Exam environment tested
- [ ] Documentation bookmarks organized
- [ ] Sleep schedule optimized
- [ ] Stress management techniques practiced
- [ ] Backup plans for technical issues

## Success Metrics
- [ ] Mock exam score: 85%+ 
- [ ] Time management: Complete with 15+ minutes remaining
- [ ] Error rate: <10% on attempted questions
- [ ] Confidence level: High across all domains

## Final Notes
Remember: The CKA exam tests practical skills, not theoretical knowledge. Focus on:
- Speed and accuracy
- Systematic troubleshooting
- Proper time management
- Staying calm under pressure

You've put in the work - trust your preparation and execute with confidence!

## Notes Section
_Use this space for final review notes and last-minute reminders_