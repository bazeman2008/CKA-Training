# Week 4: Storage & First Assessment

## Overview
Master Kubernetes storage concepts and complete your first comprehensive assessment.

## Daily Tasks

### Day 22: Persistent Volumes (1.5 hours)
- [ ] Configure PV and PVC lifecycle
- [ ] Practice static vs dynamic provisioning
- [ ] Configure storage classes

### Day 23: Volume Types (1.5 hours)
- [ ] Configure hostPath, emptyDir, configMap, secret volumes
- [ ] Practice volume mounting and permissions
- [ ] Test various volume configurations

### Day 24: Storage Troubleshooting (1.5 hours)
- [ ] Debug PVC binding issues
- [ ] Resolve permission problems
- [ ] Fix storage-related pod failures

### Day 25: Security Contexts (1.5 hours)
- [ ] Configure pod and container security contexts
- [ ] Practice running pods as non-root
- [ ] Manage security capabilities

### Day 26: Basic RBAC (1.5 hours)
- [ ] Create users and service accounts
- [ ] Configure roles and RoleBindings
- [ ] Practice basic permission management

## Weekend Session - First Mock Exam (3 hours)
- [ ] **MOCK EXAM:** 2-hour timed test covering Month 1 topics
- [ ] Review results and identify weak areas
- [ ] **Goal Check:** 40-50% score to proceed

## Storage Configurations

### Persistent Volumes and Claims

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

# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

### Volume Types Examples

```yaml
# Pod with multiple volume types
apiVersion: v1
kind: Pod
metadata:
  name: multi-volume-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
    - name: empty-volume
      mountPath: /cache
    - name: host-volume
      mountPath: /host-data
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: app-secret
  - name: empty-volume
    emptyDir: {}
  - name: host-volume
    hostPath:
      path: /data
      type: Directory
```

### Security Context Examples

```yaml
# Pod with security context
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
```

### Basic RBAC Configuration

```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa

# Role
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
- kind: ServiceAccount
  name: app-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Storage Troubleshooting Commands

```bash
# Check PV/PVC status
kubectl get pv
kubectl get pvc
kubectl describe pvc <pvc-name>

# Debug storage issues
kubectl describe pod <pod-name> | grep -A 10 Volumes
kubectl get events --field-selector involvedObject.name=<pvc-name>

# Check storage class
kubectl get storageclass
kubectl describe storageclass <storage-class-name>
```

## Mock Exam Topics Checklist

### Cluster Setup & Architecture
- [ ] Initialize cluster with kubeadm
- [ ] Join worker nodes
- [ ] Understand component roles

### Core Objects
- [ ] Create and manage pods
- [ ] Deploy and scale applications
- [ ] Expose services
- [ ] Configure applications

### Scheduling & Workloads
- [ ] Use node selectors and affinity
- [ ] Configure taints and tolerations
- [ ] Manage different workload types
- [ ] Set resource limits

### Storage
- [ ] Create and use PVs/PVCs
- [ ] Configure different volume types
- [ ] Troubleshoot storage issues

## Learning Resources
- Persistent Volumes: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- Security Context: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
- RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

## Success Criteria
- [ ] Can configure persistent storage for applications
- [ ] Understand different volume types and use cases
- [ ] Can implement basic security contexts
- [ ] Can set up basic RBAC
- [ ] Score 40-50% on first mock exam

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_