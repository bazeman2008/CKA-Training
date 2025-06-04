# Day 22: Persistent Volumes

## Overview (1.5 hours)
Master Kubernetes persistent storage concepts that comprise 10% of the CKA exam and are critical for stateful application management.

## Why This Matters for the CKA Exam
- **Storage domain** accounts for 10% of the exam
- **PV/PVC troubleshooting** is common in real scenarios
- **Storage classes** determine dynamic provisioning behavior
- **Volume lifecycle** knowledge enables storage debugging

## Task 1: Configure PV and PVC Lifecycle (45 minutes)

### What You'll Do
Understand the complete persistent volume lifecycle and master volume provisioning patterns.

### Understanding Persistent Volume Lifecycle

#### 1. Volume Phases
```bash
# PV Phases:
# Available - Volume is available and not yet bound
# Bound - Volume is bound to a claim
# Released - Claim has been deleted but not yet reclaimed
# Failed - Volume failed automatic reclamation

# Check PV status
kubectl get pv
kubectl describe pv <pv-name>
```

#### 2. Reclaim Policies
```yaml
# Three reclaim policies:
# Retain - Manual reclamation required
# Recycle - Basic scrub (deprecated)
# Delete - Associated storage asset deleted

# Example PV with Retain policy
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-retain
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv-retain
```

### Static Provisioning

#### 1. Creating Persistent Volumes
```yaml
# hostpath-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/hostpath-pv
    type: DirectoryOrCreate
```

```bash
# Create directory on node first
ssh worker-node-1 "sudo mkdir -p /data/hostpath-pv"

# Apply PV
kubectl apply -f hostpath-pv.yaml
kubectl get pv hostpath-pv
kubectl describe pv hostpath-pv
```

#### 2. Creating Multiple PVs
```yaml
# multiple-pvs.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv-1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-2
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /data/pv-2
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-3
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv-3
```

### Creating Persistent Volume Claims

#### 1. Basic PVC
```yaml
# basic-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: basic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
# Apply PVC and check binding
kubectl apply -f basic-pvc.yaml
kubectl get pvc basic-pvc
kubectl describe pvc basic-pvc

# Check which PV was bound
kubectl get pv -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,CLAIM:.spec.claimRef.name"
```

#### 2. PVC with Specific Requirements
```yaml
# specific-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: specific-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  selector:
    matchLabels:
      type: local
```

#### 3. PVC with Storage Class
```yaml
# storageclass-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storageclass-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-storage
```

### Using PVCs in Pods

#### 1. Pod with PVC
```yaml
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-storage
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: basic-pvc
```

```bash
# Apply and test persistence
kubectl apply -f pod-with-pvc.yaml

# Create file in mounted volume
kubectl exec pod-with-storage -- sh -c "echo 'Hello from PVC' > /usr/share/nginx/html/index.html"

# Delete and recreate pod
kubectl delete pod pod-with-storage
kubectl apply -f pod-with-pvc.yaml

# Verify data persistence
kubectl exec pod-with-storage -- cat /usr/share/nginx/html/index.html
```

#### 2. Deployment with PVC
```yaml
# deployment-with-pvc.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-storage
spec:
  replicas: 1  # Note: ReadWriteOnce allows only one pod
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx
        volumeMounts:
        - name: data-volume
          mountPath: /data
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: basic-pvc
```

## Task 2: Practice Static vs Dynamic Provisioning (30 minutes)

### What You'll Do
Compare static and dynamic provisioning approaches and understand when to use each.

### Static Provisioning Deep Dive

#### 1. Pre-created Volume Matching
```bash
# Static provisioning workflow:
# 1. Admin creates PV
# 2. User creates PVC
# 3. Kubernetes binds PVC to suitable PV

# Check available PVs
kubectl get pv --sort-by=.spec.capacity.storage

# Create PVC that matches existing PV
kubectl create -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: matched-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Verify binding
kubectl get pvc matched-pvc
kubectl get pv -o wide
```

#### 2. Volume Selector Usage
```yaml
# selective-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: selective-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      environment: production
    matchExpressions:
    - key: tier
      operator: In
      values: ["frontend", "backend"]
```

### Dynamic Provisioning Introduction

#### 1. Storage Classes
```yaml
# Create a simple storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-local
provisioner: kubernetes.io/no-provisioner  # Local volumes
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

```bash
# Apply storage class
kubectl apply -f storageclass.yaml
kubectl get storageclass
kubectl describe storageclass fast-local
```

#### 2. PVC with Storage Class
```yaml
# dynamic-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-local
```

### CKA Exam Scenarios

#### Scenario 1: PVC Pending (No Suitable PV)
```bash
# Problem: PVC stuck in Pending state
kubectl get pvc
kubectl describe pvc pending-pvc

# Common causes:
# 1. No PV with sufficient capacity
# 2. Access mode mismatch
# 3. Storage class not found
# 4. No available PVs in cluster

# Troubleshooting steps:
kubectl get pv  # Check available volumes
kubectl get pv -o custom-columns="NAME:.metadata.name,CAPACITY:.spec.capacity.storage,STATUS:.status.phase,CLAIM:.spec.claimRef.name"
```

#### Scenario 2: PVC Bound to Wrong PV
```bash
# Problem: PVC bound to unsuitable PV
kubectl get pvc my-pvc -o jsonpath='{.spec.volumeName}'
kubectl describe pv <volume-name>

# Solution: Use selectors for specific binding
# Or create PV with specific labels
```

## Task 3: Configure Storage Classes (20 minutes)

### What You'll Do
Create and manage storage classes for different storage requirements.

### Storage Class Configuration

#### 1. Local Storage Class
```yaml
# local-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
reclaimPolicy: Delete
```

#### 2. Fast Storage Class
```yaml
# fast-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: Immediate
reclaimPolicy: Retain
allowVolumeExpansion: true
```

#### 3. Slow Storage Class
```yaml
# slow-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow-hdd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
parameters:
  type: hdd
  tier: cold
```

### Using Storage Classes

#### 1. Default Storage Class
```bash
# Set default storage class
kubectl patch storageclass fast-ssd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Remove default annotation
kubectl patch storageclass fast-ssd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

# Check default storage class
kubectl get storageclass | grep default
```

#### 2. PVC without Storage Class (uses default)
```yaml
# default-sc-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: default-sc-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  # storageClassName omitted - uses default
```

## Task 4: Volume Lifecycle Management (5 minutes)

### What You'll Do
Practice complete volume lifecycle operations and cleanup procedures.

### Volume Cleanup

#### 1. PVC Deletion and Reclaim
```bash
# Delete PVC (PV behavior depends on reclaim policy)
kubectl delete pvc basic-pvc

# Check PV status after PVC deletion
kubectl get pv

# For Retain policy - PV becomes Released
# For Delete policy - PV is deleted
```

#### 2. Manual PV Cleanup
```bash
# For Released PVs with Retain policy
kubectl get pv -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"

# Manual cleanup (remove claimRef)
kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'

# PV becomes Available again
```

#### 3. Complete Storage Cleanup
```bash
# Clean up all test resources
kubectl delete pvc --all
kubectl delete pv --all
kubectl delete storageclass fast-ssd slow-hdd local-storage

# Clean up node directories
ssh worker-node-1 "sudo rm -rf /data/pv-*"
ssh worker-node-2 "sudo rm -rf /data/pv-*"
```

## Common CKA Storage Issues

### Issue 1: Volume Mount Failures
```bash
# Problem: Pod stuck in ContainerCreating
kubectl describe pod <pod-name> | grep -A 10 Events

# Common causes:
# - PVC not bound
# - Volume not available on node
# - Permission issues
```

### Issue 2: Storage Capacity Issues
```bash
# Problem: Insufficient storage
kubectl describe pvc <pvc-name>
kubectl get pv -o custom-columns="NAME:.metadata.name,CAPACITY:.spec.capacity.storage,AVAILABLE:.status.phase"
```

### Issue 3: Access Mode Conflicts
```bash
# Problem: PVC can't bind due to access mode
# ReadWriteOnce - Single node read/write
# ReadOnlyMany - Multiple nodes read-only
# ReadWriteMany - Multiple nodes read/write

kubectl describe pvc <pvc-name> | grep "access modes"
```

## Daily Goals Checklist

- [ ] Understand PV/PVC lifecycle completely
- [ ] Can create and manage persistent volumes
- [ ] Master static vs dynamic provisioning concepts
- [ ] Can configure and use storage classes
- [ ] Practice volume troubleshooting scenarios
- [ ] Ready for volume types exploration

## Next Day Preparation

### What You'll Need for Day 23
- PV/PVC creation skills ✓
- Storage class understanding ✓
- Volume lifecycle knowledge ✓
- Troubleshooting capabilities ✓

### Coming Up: Volume Types
Day 23 will explore different volume types and their specific use cases.

## Study Notes Section

**Storage Quick Commands:**
```bash
# PV/PVC management
kubectl get pv,pvc
kubectl describe pv <name>
kubectl describe pvc <name>

# Storage classes
kubectl get storageclass
kubectl describe storageclass <name>

# Troubleshooting
kubectl get events | grep -i volume
kubectl describe pod <name> | grep -A 10 Events
```

**Personal Storage Notes:**
_Record storage patterns and troubleshooting experiences_