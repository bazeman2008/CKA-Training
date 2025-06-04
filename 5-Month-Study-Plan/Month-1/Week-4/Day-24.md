# Day 24: Volume Access Modes & Storage Classes

## Overview (1.5 hours)
Master volume access modes, storage classes, and dynamic provisioning to understand how Kubernetes manages persistent storage across different scenarios and infrastructure types.

## Why This Matters for the CKA Exam
- **Access modes determine pod placement** and volume sharing capabilities
- **Storage classes enable dynamic provisioning** reducing manual PV management
- **Volume binding modes** affect scheduling and performance
- **Storage troubleshooting** requires understanding these advanced concepts
- **Production scenarios** frequently involve storage class configuration

## Task 1: Volume Access Modes Deep Dive (35 minutes)

### What You'll Do
Master the three volume access modes and understand their practical implications for pod scheduling and data sharing.

### Step-by-Step Instructions

#### 1. ReadWriteOnce (RWO) Access Mode
```bash
# Create PV with ReadWriteOnce access mode
cat > rwo-storage.yaml << 'EOF'
---
# ReadWriteOnce Persistent Volume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rwo-pv
  labels:
    type: local
    access: rwo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/rwo-storage
    type: DirectoryOrCreate
---
# PVC for ReadWriteOnce
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rwo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      access: rwo
EOF

# Create host directory and apply
sudo mkdir -p /data/rwo-storage
sudo chmod 755 /data/rwo-storage
kubectl apply -f rwo-storage.yaml
kubectl get pv,pvc -o custom-columns="NAME:.metadata.name,ACCESS:.spec.accessModes,STATUS:.status.phase"
```

#### 2. ReadWriteOnce Scheduling Behavior
```bash
# Create first pod using RWO volume
cat > rwo-pod1.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: rwo-pod-1
  labels:
    app: rwo-test
spec:
  containers:
  - name: writer
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Pod 1 writing to RWO volume on node: $(hostname)"
      while true; do
        echo "$(date): Pod-1 on $(hostname)" >> /data/rwo-log.txt
        sleep 10
      done
    volumeMounts:
    - name: rwo-storage
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: rwo-storage
    persistentVolumeClaim:
      claimName: rwo-pvc
EOF

# Create second pod to test RWO limitations
cat > rwo-pod2.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: rwo-pod-2
  labels:
    app: rwo-test
spec:
  containers:
  - name: writer
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Pod 2 attempting to write to RWO volume on node: $(hostname)"
      while true; do
        echo "$(date): Pod-2 on $(hostname)" >> /data/rwo-log.txt
        sleep 10
      done
    volumeMounts:
    - name: rwo-storage
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: rwo-storage
    persistentVolumeClaim:
      claimName: rwo-pvc
  # Force scheduling to different node
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: NotIn
            values:
            - $(kubectl get pod rwo-pod-1 -o jsonpath='{.spec.nodeName}' 2>/dev/null || echo "nonexistent")
EOF

# Apply and observe scheduling behavior
kubectl apply -f rwo-pod1.yaml
sleep 10
kubectl get pods -o wide

# Apply second pod (may fail scheduling if on different node)
kubectl apply -f rwo-pod2.yaml
kubectl get pods -o wide
kubectl describe pod rwo-pod-2 | grep -A 10 Events
```

#### 3. ReadOnlyMany (ROX) Access Mode
```bash
# Create ROX storage setup
cat > rox-storage.yaml << 'EOF'
---
# ReadOnlyMany Persistent Volume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rox-pv
  labels:
    type: local
    access: rox
spec:
  capacity:
    storage: 500Mi
  accessModes:
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/rox-storage
    type: DirectoryOrCreate
---
# PVC for ReadOnlyMany
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rox-pvc
spec:
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 500Mi
  selector:
    matchLabels:
      access: rox
EOF

# Prepare read-only data
sudo mkdir -p /data/rox-storage
echo "Shared configuration data" | sudo tee /data/rox-storage/config.txt
echo "Application version: 1.0.0" | sudo tee /data/rox-storage/version.txt
echo "Environment: production" | sudo tee /data/rox-storage/environment.txt
sudo chmod -R 644 /data/rox-storage/*

kubectl apply -f rox-storage.yaml
```

#### 4. ReadOnlyMany Multi-Pod Access
```bash
# Create multiple pods accessing ROX volume
cat > rox-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rox-readers
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rox-reader
  template:
    metadata:
      labels:
        app: rox-reader
    spec:
      containers:
      - name: reader
        image: busybox
        command:
        - sh
        - -c
        - |
          echo "Reader pod starting on $(hostname)"
          while true; do
            echo "=== Configuration Read $(date) ==="
            echo "Config: $(cat /shared/config.txt)"
            echo "Version: $(cat /shared/version.txt)"
            echo "Environment: $(cat /shared/environment.txt)"
            echo "Reader Pod: $(hostname)"
            echo "================================="
            sleep 30
          done
        volumeMounts:
        - name: shared-config
          mountPath: /shared
          readOnly: true
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
      volumes:
      - name: shared-config
        persistentVolumeClaim:
          claimName: rox-pvc
EOF

# Apply and verify multi-pod access
kubectl apply -f rox-deployment.yaml
kubectl get pods -l app=rox-reader -o wide
kubectl logs -l app=rox-reader --tail=5
```

#### 5. ReadWriteMany (RWX) Simulation
```bash
# Note: RWX requires shared storage (NFS, CephFS, etc.)
# For learning purposes, simulate with multiple containers in one pod

cat > rwx-simulation.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: rwx-simulation
  labels:
    app: rwx-demo
spec:
  containers:
  - name: writer-1
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "$(date): Writer-1 entry" >> /shared/shared-log.txt
        sleep 15
      done
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  - name: writer-2
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "$(date): Writer-2 entry" >> /shared/shared-log.txt
        sleep 20
      done
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  - name: reader
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "=== Shared Log Content ==="
        tail -10 /shared/shared-log.txt 2>/dev/null || echo "Log file not yet created"
        echo "=========================="
        sleep 25
      done
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: shared-storage
    emptyDir: {}
EOF

# Apply and observe RWX behavior simulation
kubectl apply -f rwx-simulation.yaml
kubectl logs rwx-simulation -c reader --tail=15
```

### CKA Exam Connection
- **Pod Scheduling**: Access modes affect where pods can be scheduled
- **Data Sharing**: Understanding which access modes allow concurrent access
- **Storage Planning**: Choosing appropriate access modes for use cases
- **Troubleshooting**: Diagnosing volume access conflicts

### Validation Commands
```bash
# Check access mode assignments
kubectl get pv -o custom-columns="NAME:.metadata.name,ACCESS:.spec.accessModes,STATUS:.status.phase"
kubectl get pvc -o custom-columns="NAME:.metadata.name,ACCESS:.spec.accessModes,STATUS:.status.phase"

# Verify pod scheduling with volume constraints
kubectl get pods -o wide | grep rwo
kubectl describe pod rwo-pod-2 | grep -A 5 "Events:"
```

## Task 2: Storage Classes Configuration (35 minutes)

### What You'll Do
Create and configure storage classes for different storage requirements and understand dynamic provisioning.

### Step-by-Step Instructions

#### 1. Basic Storage Class Creation
```bash
# Create local storage class (no dynamic provisioning)
cat > local-storageclass.yaml << 'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
reclaimPolicy: Delete
parameters:
  type: "local"
  performance: "standard"
EOF

# Create fast storage class for SSDs
cat > fast-storageclass.yaml << 'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: Immediate
allowVolumeExpansion: true
reclaimPolicy: Retain
parameters:
  type: "ssd"
  performance: "high"
  tier: "premium"
EOF

# Create slow storage class for archival
cat > slow-storageclass.yaml << 'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow-hdd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
parameters:
  type: "hdd"
  performance: "economy"
  tier: "standard"
  replication: "single"
EOF

# Apply storage classes
kubectl apply -f local-storageclass.yaml
kubectl apply -f fast-storageclass.yaml
kubectl apply -f slow-storageclass.yaml

# Verify storage classes
kubectl get storageclass
kubectl describe storageclass fast-ssd
```

#### 2. Storage Class Selection Testing
```bash
# Create PVCs with different storage classes
cat > storageclass-pvcs.yaml << 'EOF'
---
# PVC using default storage class
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: default-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  # storageClassName omitted - uses default
---
# PVC explicitly using fast storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: fast-ssd
---
# PVC using slow storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: slow-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: slow-hdd
---
# PVC with no storage class (won't bind without matching PV)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: no-class-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ""
EOF

# Apply and check PVC status
kubectl apply -f storageclass-pvcs.yaml
kubectl get pvc
kubectl describe pvc default-pvc | grep "StorageClass"
kubectl describe pvc fast-pvc | grep "StorageClass"
```

#### 3. Volume Binding Modes Testing
```bash
# Create PVs for different storage classes
cat > storage-class-pvs.yaml << 'EOF'
---
# PV for fast storage class
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fast-pv-1
  labels:
    type: ssd
    performance: high
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /data/fast-storage-1
    type: DirectoryOrCreate
---
# PV for slow storage class
apiVersion: v1
kind: PersistentVolume
metadata:
  name: slow-pv-1
  labels:
    type: hdd
    performance: economy
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: slow-hdd
  hostPath:
    path: /data/slow-storage-1
    type: DirectoryOrCreate
---
# PV for local storage class
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-1
  labels:
    type: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - $(hostname)
  hostPath:
    path: /data/local-storage-1
    type: DirectoryOrCreate
EOF

# Create directories and apply PVs
sudo mkdir -p /data/{fast-storage-1,slow-storage-1,local-storage-1}
kubectl apply -f storage-class-pvs.yaml
kubectl get pv,pvc
```

#### 4. Binding Mode Behavior Testing
```bash
# Test WaitForFirstConsumer binding mode
cat > wait-consumer-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: local-storage-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Pod using local storage on node: $(hostname)"
      echo "Writing to local volume..."
      echo "$(date): Local storage test" > /data/local-test.txt
      while true; do
        echo "$(date): Application running on $(hostname)" >> /data/app-log.txt
        sleep 30
      done
    volumeMounts:
    - name: local-volume
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: local-volume
    persistentVolumeClaim:
      claimName: local-pvc
EOF

# Create PVC for local storage (WaitForFirstConsumer)
cat > local-pvc.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
EOF

kubectl apply -f local-pvc.yaml
kubectl get pvc local-pvc  # Should be Pending

# Apply pod - should trigger binding
kubectl apply -f wait-consumer-pod.yaml
sleep 10
kubectl get pvc local-pvc  # Should be Bound
kubectl get pods -o wide
```

### CKA Exam Connection
- **Dynamic Provisioning**: Understanding automated storage allocation
- **Storage Tiers**: Matching storage performance to application needs
- **Binding Behavior**: How volume binding affects pod scheduling
- **Default Classes**: Managing cluster-wide storage defaults

### Validation Commands
```bash
# Check storage class configuration
kubectl get storageclass -o custom-columns="NAME:.metadata.name,PROVISIONER:.provisioner,BINDING:.volumeBindingMode,DEFAULT:.metadata.annotations.storageclass\.kubernetes\.io/is-default-class"

# Verify PV/PVC binding with storage classes
kubectl get pv -o custom-columns="NAME:.metadata.name,STORAGECLASS:.spec.storageClassName,STATUS:.status.phase"
kubectl get pvc -o custom-columns="NAME:.metadata.name,STORAGECLASS:.spec.storageClassName,STATUS:.status.phase"
```

## Task 3: Advanced Storage Patterns (25 minutes)

### What You'll Do
Implement advanced storage patterns including storage class inheritance, volume expansion, and complex binding scenarios.

### Step-by-Step Instructions

#### 1. Volume Expansion Testing
```bash
# Create expandable storage class
cat > expandable-storageclass.yaml << 'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: Immediate
allowVolumeExpansion: true
reclaimPolicy: Retain
parameters:
  type: "expandable"
EOF

# Create PV for expansion testing
cat > expandable-pv.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: expandable-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: expandable-storage
  hostPath:
    path: /data/expandable-storage
    type: DirectoryOrCreate
EOF

# Create PVC and pod for expansion
cat > expandable-setup.yaml << 'EOF'
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: expandable-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: expandable-storage
---
apiVersion: v1
kind: Pod
metadata:
  name: expansion-test
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "Current disk usage:"
        df -h /data
        echo "Writing test data..."
        echo "$(date): Expansion test data" >> /data/expansion-log.txt
        sleep 30
      done
    volumeMounts:
    - name: expandable-volume
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: expandable-volume
    persistentVolumeClaim:
      claimName: expandable-pvc
EOF

# Apply setup
sudo mkdir -p /data/expandable-storage
kubectl apply -f expandable-storageclass.yaml
kubectl apply -f expandable-pv.yaml
kubectl apply -f expandable-setup.yaml

# Test expansion (edit PVC)
kubectl patch pvc expandable-pvc -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
kubectl describe pvc expandable-pvc | grep -A 5 "Events\|Capacity"
```

#### 2. Multi-Zone Storage Simulation
```bash
# Simulate zone-aware storage classes
cat > zone-storage.yaml << 'EOF'
---
# Zone A storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: zone-a-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
parameters:
  zone: "zone-a"
  replication: "zone-local"
---
# Zone B storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: zone-b-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
parameters:
  zone: "zone-b"
  replication: "zone-local"
---
# Multi-zone storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: multi-zone-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: Immediate
allowVolumeExpansion: true
parameters:
  replication: "multi-zone"
  durability: "high"
EOF

kubectl apply -f zone-storage.yaml
kubectl get storageclass | grep zone
```

#### 3. Storage Class Priority and Selection
```bash
# Create application with storage preferences
cat > storage-preferences.yaml << 'EOF'
---
# High priority application - prefers fast storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: priority-app-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "fast-ssd"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-ssd
---
# Background job - uses slow storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: background-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  storageClassName: slow-hdd
---
# Development workload - uses default
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dev-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  # Uses default storage class
EOF

kubectl apply -f storage-preferences.yaml
kubectl get pvc -o custom-columns="NAME:.metadata.name,STORAGECLASS:.spec.storageClassName,SIZE:.spec.resources.requests.storage,STATUS:.status.phase"
```

#### 4. Storage Class Troubleshooting
```bash
# Create problematic storage scenarios
cat > storage-problems.yaml << 'EOF'
---
# PVC with non-existent storage class
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: missing-class-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nonexistent-class
---
# PVC requesting more than available
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: oversized-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi  # Larger than any available PV
  storageClassName: fast-ssd
EOF

# Apply and diagnose
kubectl apply -f storage-problems.yaml
kubectl get pvc | grep -E "(missing-class|oversized)"
kubectl describe pvc missing-class-pvc | grep -A 5 "Events"
kubectl describe pvc oversized-pvc | grep -A 5 "Events"
```

### CKA Exam Connection
- **Storage Operations**: Managing volume lifecycle and expansion
- **Zone Awareness**: Understanding storage locality and performance
- **Troubleshooting**: Diagnosing storage class and binding issues
- **Resource Planning**: Matching storage requirements to available resources

### Validation Commands
```bash
# Check storage class functionality
kubectl get pv,pvc -o wide
kubectl describe storageclass expandable-storage | grep "AllowVolumeExpansion"

# Monitor storage expansion
kubectl get events --field-selector reason=VolumeResizeSuccessful
```

## Task 4: Storage Best Practices and Cleanup (15 minutes)

### What You'll Do
Implement storage best practices and perform comprehensive cleanup of demo resources.

### Step-by-Step Instructions

#### 1. Storage Best Practices Summary
```bash
# Create best practices documentation
cat > storage-best-practices.md << 'EOF'
# Storage Best Practices Summary

## Access Mode Selection
- ReadWriteOnce (RWO): Single node access, most common
- ReadOnlyMany (ROX): Multiple nodes read-only, configuration data
- ReadWriteMany (RWX): Multiple nodes read-write, requires shared storage

## Storage Class Design
- Use descriptive names (fast-ssd, slow-hdd, local-storage)
- Set appropriate default storage class
- Consider volume binding modes for performance
- Enable expansion where supported

## Volume Management
- Use PVCs instead of direct PV references
- Implement proper resource requests
- Consider reclaim policies carefully
- Monitor storage usage and expansion

## Security Considerations
- Use appropriate access modes
- Implement proper RBAC for storage resources
- Consider encryption for sensitive data
- Regular backup and disaster recovery

## Performance Guidelines
- Match storage type to workload requirements
- Use local storage for temporary data
- Consider node affinity for local volumes
- Monitor I/O patterns and bottlenecks
EOF

cat storage-best-practices.md
```

#### 2. Storage Resource Summary
```bash
# Create summary script for storage resources
cat > storage-summary.sh << 'EOF'
#!/bin/bash
echo "=== STORAGE RESOURCE SUMMARY ==="
echo ""
echo "Storage Classes:"
kubectl get storageclass -o custom-columns="NAME:.metadata.name,PROVISIONER:.provisioner,BINDING:.volumeBindingMode,EXPANSION:.allowVolumeExpansion,DEFAULT:.metadata.annotations.storageclass\.kubernetes\.io/is-default-class"

echo ""
echo "Persistent Volumes:"
kubectl get pv -o custom-columns="NAME:.metadata.name,CAPACITY:.spec.capacity.storage,ACCESS:.spec.accessModes,STORAGECLASS:.spec.storageClassName,STATUS:.status.phase,CLAIM:.spec.claimRef.name"

echo ""
echo "Persistent Volume Claims:"
kubectl get pvc -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,VOLUME:.spec.volumeName,CAPACITY:.status.capacity.storage,ACCESS:.spec.accessModes,STORAGECLASS:.spec.storageClassName"

echo ""
echo "Pods Using Storage:"
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName" | grep -v "No resources found"

echo ""
echo "Storage Events:"
kubectl get events --field-selector type=Warning | grep -i -E "(volume|storage|pv|pvc)" | tail -5
EOF

chmod +x storage-summary.sh
./storage-summary.sh
```

#### 3. Comprehensive Cleanup
```bash
# Clean up all demo resources in order
echo "Starting comprehensive storage cleanup..."

# Delete pods first
kubectl delete pods --all --timeout=60s

# Delete PVCs (this will release PVs)
kubectl delete pvc --all

# Delete PVs
kubectl delete pv --all

# Delete storage classes (except system ones)
kubectl delete storageclass local-storage fast-ssd slow-hdd expandable-storage zone-a-storage zone-b-storage multi-zone-storage

# Clean up host directories
echo "Cleaning up host directories..."
sudo rm -rf /data/rwo-storage /data/rox-storage /data/fast-storage-1 /data/slow-storage-1 /data/local-storage-1 /data/expandable-storage

# Clean up generated files
rm -f rwo-storage.yaml rwo-pod1.yaml rwo-pod2.yaml
rm -f rox-storage.yaml rox-deployment.yaml
rm -f rwx-simulation.yaml
rm -f local-storageclass.yaml fast-storageclass.yaml slow-storageclass.yaml
rm -f storageclass-pvcs.yaml storage-class-pvs.yaml
rm -f wait-consumer-pod.yaml local-pvc.yaml
rm -f expandable-storageclass.yaml expandable-pv.yaml expandable-setup.yaml
rm -f zone-storage.yaml storage-preferences.yaml storage-problems.yaml
rm -f storage-best-practices.md storage-summary.sh

echo "Storage cleanup completed!"
```

#### 4. Knowledge Verification
```bash
# Quick verification that cleanup was successful
echo "=== CLEANUP VERIFICATION ==="
echo "Remaining PVs:"
kubectl get pv || echo "No PVs found"

echo ""
echo "Remaining PVCs:"
kubectl get pvc || echo "No PVCs found"

echo ""
echo "Remaining Storage Classes:"
kubectl get storageclass

echo ""
echo "Host directories:"
sudo ls -la /data/ 2>/dev/null || echo "No /data directory"

echo ""
echo "Cleanup verification complete!"
```

### CKA Exam Connection
- **Resource Management**: Proper cleanup prevents resource conflicts
- **Best Practices**: Understanding optimal storage configurations
- **Documentation**: Maintaining clear storage policies
- **Monitoring**: Regular assessment of storage resource usage

### Validation Commands
```bash
# Final validation
kubectl get all
kubectl get pv,pvc,storageclass
ls -la *.yaml 2>/dev/null || echo "All demo files cleaned up"
```

## Common Issues and Solutions

### Issue 1: PVC Stuck in Pending
**Problem**: PVC remains in Pending state indefinitely
**Solution**: 
```bash
# Check if storage class exists and has available PVs
kubectl get storageclass
kubectl get pv --field-selector=status.phase=Available
kubectl describe pvc <pvc-name> | grep Events

# Common causes: No matching PV, wrong access mode, insufficient capacity
```

### Issue 2: Volume Binding Mode Issues
**Problem**: Pod fails to start due to volume binding
**Solution**:
```bash
# Check binding mode and node scheduling
kubectl describe storageclass <storage-class>
kubectl describe pod <pod-name> | grep Events
kubectl get pvc <pvc-name> -o yaml | grep volumeBindingMode

# WaitForFirstConsumer requires pod to be scheduled first
```

### Issue 3: Access Mode Conflicts
**Problem**: Pod can't mount volume due to access mode restrictions
**Solution**:
```bash
# Check current volume usage and access modes
kubectl get pv <pv-name> -o jsonpath='{.spec.accessModes}'
kubectl get pod -o wide | grep <pv-name>

# RWO allows only one node, ROX is read-only, RWX needs shared storage
```

## Daily Goals Checklist

- [ ] Understand all three volume access modes thoroughly
- [ ] Configure storage classes for different use cases
- [ ] Master volume binding mode behaviors
- [ ] Implement storage expansion scenarios
- [ ] Troubleshoot common storage issues
- [ ] Ready for Storage Assessment

## Next Day Preparation

### What You'll Need for Day 25
- Access mode behavior understanding ✓
- Storage class configuration skills ✓
- Volume binding troubleshooting ✓
- Best practices knowledge ✓

### Coming Up: Storage Assessment & Practice
Day 25 will be the First Assessment focused on storage concepts.

## Study Notes Section

**Access Mode Quick Reference:**
- **ReadWriteOnce (RWO)**: Single node, read-write
- **ReadOnlyMany (ROX)**: Multiple nodes, read-only  
- **ReadWriteMany (RWX)**: Multiple nodes, read-write (requires shared storage)

**Storage Class Parameters:**
```yaml
# Basic storage class template
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
parameters:
  type: "ssd"
```

**Personal Storage Patterns:**
_Record your preferred storage configurations and troubleshooting approaches_