# Day 25: First Assessment - Storage & Troubleshooting

## Overview (2 hours)
Comprehensive assessment covering Week 4 storage concepts and previous weeks' troubleshooting skills. This assessment simulates CKA exam conditions and validates your readiness for Month 2.

## Why This Matters for the CKA Exam
- **Assessment validates 40% of CKA domains** covered so far
- **Storage questions appear in 10% of the exam**
- **Troubleshooting represents 30% of exam content**
- **Time management practice** for 2-hour exam duration
- **Confidence building** for advanced topics ahead

## Assessment Structure

### Time Allocation (120 minutes total)
- **Storage Tasks (45 minutes)**: PV/PVC, Volume types, Storage classes
- **Troubleshooting Tasks (35 minutes)**: Pod failures, Resource issues
- **Mixed Scenarios (25 minutes)**: Real-world problem solving
- **Review & Documentation (15 minutes)**: Answer verification

### Scoring Criteria
- **90-100%**: Excellent - Ready for Month 2 advanced topics
- **80-89%**: Good - Review weak areas before proceeding
- **70-79%**: Satisfactory - Additional practice recommended
- **Below 70%**: Needs improvement - Repeat Week 4 concepts

## Task 1: Storage Configuration Assessment (45 minutes)

### Scenario A: Multi-Application Storage Setup (15 minutes)

You are tasked with setting up storage for a multi-tier application with different storage requirements.

#### Requirements:
1. Create a database storage solution with 2Gi capacity, ReadWriteOnce access
2. Create shared configuration storage with 500Mi capacity, ReadOnlyMany access  
3. Create temporary cache storage using memory-backed volumes
4. Implement proper storage classes and volume binding

#### Implementation:
```bash
# Start timing - 15 minutes for this scenario
# Document your approach and commands

# Step 1: Create storage classes
cat > assessment-storage-classes.yaml << 'EOF'
# TODO: Create appropriate storage classes
# - database-storage: persistent, retention policy
# - shared-config: read-only optimized
# - cache-storage: memory-backed temporary
EOF

# Step 2: Create persistent volumes
cat > assessment-pvs.yaml << 'EOF'
# TODO: Create PVs matching your storage classes
# - Consider access modes carefully
# - Set appropriate reclaim policies
# - Use proper host paths
EOF

# Step 3: Create PVCs and test pods
cat > assessment-workload.yaml << 'EOF'
# TODO: Create:
# - Database pod with persistent storage
# - Config server pod with shared read-only storage  
# - Cache pod with memory-backed temporary storage
EOF

# Implementation commands:
# kubectl apply -f <your-files>
# kubectl get pv,pvc,pods -o wide
# kubectl describe pod <pod-names>
```

#### Success Criteria:
- [ ] All storage classes created with correct parameters
- [ ] PVs bound to appropriate PVCs
- [ ] Pods successfully start and mount volumes
- [ ] Demonstrate understanding of access modes
- [ ] Proper resource allocation and limits

#### Common Mistakes to Avoid:
- Incorrect access mode selection
- Missing storage class annotations
- Improper volume binding mode
- Inadequate resource requests/limits

### Scenario B: Storage Troubleshooting (15 minutes)

Diagnose and fix the provided broken storage configuration.

#### Broken Configuration:
```bash
# Apply this broken configuration and fix the issues
cat > broken-storage.yaml << 'EOF'
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: broken-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany  # Issue 1: Incorrect access mode for hostPath
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /nonexistent/path  # Issue 2: Path doesn't exist
    type: Directory  # Issue 3: Wrong type for nonexistent path
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: broken-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Issue 4: Mismatch with PV access mode
  resources:
    requests:
      storage: 2Gi  # Issue 5: Requesting more than PV provides
  storageClassName: nonexistent-class  # Issue 6: Storage class doesn't exist
---
apiVersion: v1
kind: Pod
metadata:
  name: broken-storage-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "50Mi"  # Issue 7: Too restrictive for nginx
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: wrong-pvc-name  # Issue 8: Wrong PVC name
EOF

kubectl apply -f broken-storage.yaml
```

#### Your Task:
Identify and fix all 8 issues in the configuration above. Document your findings and create corrected configuration.

#### Solution Template:
```bash
# Issue Analysis:
# 1. ________________________________
# 2. ________________________________  
# 3. ________________________________
# 4. ________________________________
# 5. ________________________________
# 6. ________________________________
# 7. ________________________________
# 8. ________________________________

# Create fixed configuration
cat > fixed-storage.yaml << 'EOF'
# Your corrected configuration here
EOF
```

#### Success Criteria:
- [ ] All 8 issues identified correctly
- [ ] Fixed configuration applies successfully
- [ ] Pod starts and mounts volume properly
- [ ] Demonstrates systematic troubleshooting approach

### Scenario C: Dynamic Storage Management (15 minutes)

Implement a storage solution that demonstrates volume expansion and multiple storage tiers.

#### Requirements:
1. Create a storage class that allows volume expansion
2. Create a PVC that starts at 1Gi and expand it to 3Gi
3. Demonstrate different storage tiers (fast, standard, slow)
4. Show proper volume lifecycle management

#### Implementation Template:
```bash
# Create expandable storage solution
cat > expandable-solution.yaml << 'EOF'
# TODO: Implement expandable storage class and related resources
EOF

# Demonstrate expansion process
# TODO: Document expansion commands and verification

# Create storage tier demonstration
cat > storage-tiers.yaml << 'EOF'
# TODO: Create fast, standard, and slow storage classes
# TODO: Create workloads using different tiers
EOF
```

#### Success Criteria:
- [ ] Volume expansion works correctly
- [ ] Different storage tiers properly configured
- [ ] Appropriate storage class selection demonstrated
- [ ] Volume lifecycle properly managed

## Task 2: Troubleshooting Assessment (35 minutes)

### Scenario D: Pod Startup Failures (15 minutes)

Multiple pods are failing to start. Use systematic troubleshooting to identify and resolve issues.

#### Broken Pods Configuration:
```bash
# Apply these failing pods and troubleshoot
cat > failing-pods.yaml << 'EOF'
---
# Pod 1: Image issue
apiVersion: v1
kind: Pod
metadata:
  name: image-issue-pod
spec:
  containers:
  - name: app
    image: nonexistent-registry.io/fake-image:v1.0
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
---
# Pod 2: Resource issue
apiVersion: v1
kind: Pod
metadata:
  name: resource-issue-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "32Gi"  # Impossible memory request
        cpu: "16"       # Impossible CPU request
---
# Pod 3: Configuration issue
apiVersion: v1
kind: Pod
metadata:
  name: config-issue-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /config/app.conf; sleep 300"]
    volumeMounts:
    - name: config
      mountPath: /config
  volumes:
  - name: config
    configMap:
      name: missing-configmap
---
# Pod 4: Security issue
apiVersion: v1
kind: Pod
metadata:
  name: security-issue-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: app
    image: nginx  # nginx needs to bind to port 80 as root
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
EOF

kubectl apply -f failing-pods.yaml
```

#### Your Troubleshooting Task:
For each failing pod, use systematic troubleshooting to:
1. Identify the root cause
2. Explain why the pod is failing  
3. Provide the correct solution
4. Implement the fix

#### Troubleshooting Framework:
```bash
# Use this systematic approach for each pod
for pod in image-issue-pod resource-issue-pod config-issue-pod security-issue-pod; do
    echo "=== TROUBLESHOOTING: $pod ==="
    # TODO: Add your diagnostic commands
    # kubectl get pod $pod
    # kubectl describe pod $pod
    # kubectl logs $pod
    # kubectl get events --field-selector involvedObject.name=$pod
    echo ""
done
```

#### Success Criteria:
- [ ] All pod failure causes identified correctly
- [ ] Systematic troubleshooting approach demonstrated
- [ ] Correct solutions provided and implemented
- [ ] Understanding of different failure types shown

### Scenario E: Resource and Performance Issues (20 minutes)

Diagnose and resolve cluster resource and performance problems.

#### Problem Setup:
```bash
# Create resource pressure scenario
cat > resource-pressure.yaml << 'EOF'
---
# High memory usage pod
apiVersion: v1
kind: Pod
metadata:
  name: memory-hog
  labels:
    type: resource-test
spec:
  containers:
  - name: memory-consumer
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "512M", "--vm-hang", "1"]
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "1Gi"
        cpu: "500m"
---
# CPU intensive pod
apiVersion: v1
kind: Pod
metadata:
  name: cpu-hog
  labels:
    type: resource-test
spec:
  containers:
  - name: cpu-consumer
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "2", "--timeout", "300s"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "1000m"
---
# Deployment with inappropriate resource settings
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-problems
spec:
  replicas: 5
  selector:
    matchLabels:
      app: resource-problems
  template:
    metadata:
      labels:
        app: resource-problems
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            memory: "2Gi"  # Too high for cluster
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
EOF

kubectl apply -f resource-pressure.yaml
```

#### Your Performance Analysis Task:
1. Monitor resource usage and identify bottlenecks
2. Analyze pod scheduling issues
3. Implement proper resource management
4. Demonstrate performance optimization

#### Analysis Framework:
```bash
# Resource monitoring commands
echo "=== RESOURCE ANALYSIS ==="

# TODO: Add your monitoring and analysis commands
# kubectl top nodes
# kubectl top pods --sort-by=memory
# kubectl describe nodes | grep -A 5 "Allocated resources"
# kubectl get events --field-selector type=Warning

# TODO: Identify issues and provide solutions
```

#### Success Criteria:
- [ ] Resource bottlenecks identified correctly
- [ ] Scheduling issues diagnosed and resolved
- [ ] Appropriate resource limits implemented
- [ ] Performance optimization demonstrated

## Task 3: Mixed Real-World Scenarios (25 minutes)

### Scenario F: Production Storage Migration (25 minutes)

You need to migrate a running application to a new storage solution while maintaining data integrity and minimal downtime.

#### Current Setup:
```bash
# Existing application with basic storage
cat > current-app.yaml << 'EOF'
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: old-storage-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/old-storage
    type: DirectoryOrCreate
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: old-storage-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: app
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: old-storage-pvc
EOF

# Create initial data
sudo mkdir -p /data/old-storage
echo "<h1>Important Production Data</h1>" | sudo tee /data/old-storage/index.html
echo "Critical data file" | sudo tee /data/old-storage/important.txt

kubectl apply -f current-app.yaml
```

#### Migration Requirements:
1. **Create new storage solution** with better performance characteristics
2. **Migrate data** from old storage to new storage without data loss
3. **Update application** to use new storage
4. **Verify data integrity** after migration
5. **Clean up old resources** safely

#### Migration Plan Template:
```bash
# Phase 1: Prepare new storage
cat > new-storage.yaml << 'EOF'
# TODO: Create improved storage solution
# - Better storage class
# - Larger capacity
# - Appropriate access modes
EOF

# Phase 2: Data migration strategy
cat > migration-plan.sh << 'EOF'
#!/bin/bash
# TODO: Document your migration steps
# 1. Create new storage
# 2. Copy data safely
# 3. Update application
# 4. Verify integrity
# 5. Clean up old resources
EOF

# Phase 3: Verification and cleanup
# TODO: Add verification commands and cleanup procedures
```

#### Success Criteria:
- [ ] New storage solution properly designed
- [ ] Data migration completed without loss
- [ ] Application successfully updated
- [ ] Data integrity verified
- [ ] Old resources cleaned up safely
- [ ] Minimal downtime achieved

#### Challenge Considerations:
- How do you ensure data consistency during migration?
- What's your rollback plan if migration fails?
- How do you minimize application downtime?
- How do you verify data integrity?

## Task 4: Assessment Review & Documentation (15 minutes)

### Knowledge Validation Checklist

Complete this checklist to validate your understanding:

#### Storage Concepts:
- [ ] Can create and configure PVs and PVCs correctly
- [ ] Understand all volume access modes and their implications
- [ ] Can configure storage classes for different use cases
- [ ] Understand volume binding modes and their effects
- [ ] Can implement volume expansion scenarios
- [ ] Understand different volume types and their use cases

#### Troubleshooting Skills:
- [ ] Can systematically diagnose pod startup failures
- [ ] Understand resource-related issues and solutions
- [ ] Can analyze and resolve storage mounting problems
- [ ] Can identify and fix configuration errors
- [ ] Can monitor and optimize resource usage
- [ ] Can handle real-world migration scenarios

#### Practical Skills:
- [ ] Can work efficiently under time pressure
- [ ] Demonstrate good kubectl command knowledge
- [ ] Can create complex multi-resource configurations
- [ ] Understand security implications of storage decisions
- [ ] Can document solutions and approaches clearly

### Assessment Scoring

#### Calculate Your Score:
- **Task 1 (Storage)**: ___/45 points
  - Scenario A: ___/15 points
  - Scenario B: ___/15 points  
  - Scenario C: ___/15 points

- **Task 2 (Troubleshooting)**: ___/35 points
  - Scenario D: ___/15 points
  - Scenario E: ___/20 points

- **Task 3 (Real-world)**: ___/25 points
  - Scenario F: ___/25 points

- **Task 4 (Documentation)**: ___/15 points
  - Knowledge validation: ___/15 points

**Total Score**: ___/120 points
**Percentage**: ___% 

### Performance Analysis

#### Time Management Review:
- **Planned vs Actual Time**: Did you complete within 2 hours?
- **Task Efficiency**: Which tasks took longer than expected?
- **Command Speed**: Are you fluent with kubectl commands?
- **Problem-Solving**: How quickly did you identify issues?

#### Areas for Improvement:
```bash
# Self-assessment notes
echo "Areas needing improvement:"
# TODO: Document specific areas for focused study

echo "Strengths demonstrated:"
# TODO: Document your strong areas

echo "Next study priorities:"
# TODO: Plan your Month 2 preparation focus
```

### Month 2 Readiness Assessment

Based on your performance, determine your readiness for Month 2 advanced topics:

#### Ready for Month 2 (85%+ score):
- Strong foundational knowledge demonstrated
- Efficient troubleshooting skills
- Good time management
- Can proceed to networking and advanced concepts

#### Need Additional Review (70-84% score):
- Review specific weak areas identified
- Practice more hands-on scenarios
- Focus on speed and efficiency
- Consider additional storage practice

#### Repeat Week 4 (Below 70% score):
- Fundamental concepts need reinforcement
- More practice with basic storage operations
- Focus on systematic troubleshooting approach
- Build command familiarity before proceeding

## Cleanup and Preparation for Month 2

### Assessment Cleanup:
```bash
# Clean up all assessment resources
kubectl delete pods,deployments,pvc,pv,storageclass --all
sudo rm -rf /data/old-storage /data/new-storage
rm -f *.yaml *.sh *.md

echo "Assessment cleanup completed!"
```

### Month 2 Preparation:
```bash
# Prepare for Month 2 topics
echo "Month 2 Preview: Networking & Security"
echo "Topics coming up:"
echo "- Cluster networking fundamentals"
echo "- Service types and ingress"
echo "- Network policies"
echo "- Security contexts and RBAC"
echo "- Cluster maintenance and upgrades"
```

## Study Notes Section

**Assessment Lessons Learned:**
_Document key insights from this assessment_

**Time Management Insights:**
_Record timing strategies that worked well_

**Common Mistake Patterns:**
_Note frequent errors to avoid in future_

**Command Efficiency Improvements:**
_Document kubectl shortcuts and efficiency gains_

**Personal CKA Preparation Notes:**
_Record specific areas for continued focus_