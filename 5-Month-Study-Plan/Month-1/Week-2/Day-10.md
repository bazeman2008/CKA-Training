# Day 10: Deployments & ReplicaSets

## Overview (1.5 hours)
Master deployment strategies, rolling updates, and replica management - core skills for 15% of the CKA exam.

## Why This Matters for the CKA Exam
- **Deployment management** is guaranteed to appear on the exam
- **Rolling updates/rollbacks** are common troubleshooting scenarios
- **Replica scaling** tests workload management skills
- **Deployment strategies** demonstrate production readiness

## Task 1: Practice Deployment Strategies and Rolling Updates (45 minutes)

### What You'll Do
Master deployment creation, scaling, updating, and rollback procedures.

### Basic Deployment Operations

#### 1. Creating Deployments
```bash
# Basic deployment creation
kubectl create deployment nginx --image=nginx:1.20

# Deployment with specific replicas
kubectl create deployment webapp --image=nginx:1.20 --replicas=3

# Generate deployment YAML
kubectl create deployment api --image=nginx:1.20 --dry-run=client -o yaml > deployment.yaml
```

#### 2. Scaling Deployments
```bash
# Scale deployment up
kubectl scale deployment nginx --replicas=5

# Scale deployment down
kubectl scale deployment nginx --replicas=2

# Autoscaling (if metrics server available)
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80
```

#### 3. Viewing Deployment Status
```bash
# Check deployment status
kubectl get deployments
kubectl get deploy nginx -o wide

# Detailed deployment information
kubectl describe deployment nginx

# Check replica sets
kubectl get replicasets
kubectl get rs -l app=nginx
```

### Rolling Updates

#### 1. Image Updates
```bash
# Update deployment image
kubectl set image deployment/nginx nginx=nginx:1.21

# Update with specific strategy
kubectl patch deployment nginx -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'

# Monitor rollout
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
```

#### 2. Configuration Updates
```bash
# Update environment variables
kubectl set env deployment/nginx ENVIRONMENT=production

# Update resource limits
kubectl patch deployment nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"limits":{"memory":"256Mi"}}}]}}}}'

# Update labels
kubectl patch deployment nginx -p '{"spec":{"template":{"metadata":{"labels":{"version":"v2"}}}}}'
```

#### 3. Rollout Management
```bash
# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx

# Check rollout status
kubectl rollout status deployment/nginx --timeout=300s
```

### CKA Exam Rolling Update Scenarios

#### Scenario 1: Zero-Downtime Update
```yaml
# deployment-rolling.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zero-downtime-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Ensures no downtime
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
        image: nginx:1.20
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Apply and test zero-downtime update
kubectl apply -f deployment-rolling.yaml
kubectl set image deployment/zero-downtime-app webapp=nginx:1.21
kubectl rollout status deployment/zero-downtime-app
```

#### Scenario 2: Quick Update with Some Downtime
```yaml
# Fast update strategy
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
```

## Task 2: Master Rollback Procedures (30 minutes)

### What You'll Do
Learn to quickly identify and fix failed deployments using rollback mechanisms.

### Rollback Operations

#### 1. View Rollout History
```bash
# Check rollout history
kubectl rollout history deployment/nginx

# Detailed history for specific revision
kubectl rollout history deployment/nginx --revision=2

# Annotate deployments for better tracking
kubectl annotate deployment/nginx deployment.kubernetes.io/revision-history-limit=10
```

#### 2. Rollback to Previous Version
```bash
# Rollback to previous revision
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Check rollback status
kubectl rollout status deployment/nginx
```

#### 3. Rollback Verification
```bash
# Verify rollback success
kubectl get deployment nginx -o jsonpath='{.spec.template.spec.containers[0].image}'
kubectl describe deployment nginx | grep Image
kubectl get pods -l app=nginx -o jsonpath='{.items[*].spec.containers[0].image}'
```

### Failed Deployment Scenarios

#### Scenario 1: Bad Image Update
```bash
# Simulate failed update
kubectl set image deployment/nginx nginx=nginx:nonexistent-tag

# Monitor failure
kubectl get pods -l app=nginx
kubectl describe deployment nginx
kubectl get events | grep nginx

# Rollback to fix
kubectl rollout undo deployment/nginx
kubectl rollout status deployment/nginx
```

#### Scenario 2: Resource Constraint Failure
```bash
# Update with impossible resource requirements
kubectl patch deployment nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"memory":"100Gi"}}}]}}}}'

# Observe failure
kubectl get pods -l app=nginx
kubectl describe pod <pending-pod>

# Rollback to working state
kubectl rollout undo deployment/nginx
```

### CKA Exam Rollback Strategies

#### 1. Quick Rollback (Emergency)
```bash
# Immediate rollback without investigation
kubectl rollout undo deployment/<name>
kubectl rollout status deployment/<name>
```

#### 2. Investigated Rollback
```bash
# Check what changed
kubectl rollout history deployment/<name>
kubectl rollout history deployment/<name> --revision=<current>
kubectl rollout history deployment/<name> --revision=<previous>

# Targeted rollback
kubectl rollout undo deployment/<name> --to-revision=<specific>
```

## Task 3: Practice Scaling and Managing Deployments (15 minutes)

### What You'll Do
Master scaling operations and deployment management for optimal resource usage.

### Scaling Operations

#### 1. Manual Scaling
```bash
# Scale up for increased load
kubectl scale deployment webapp --replicas=10

# Scale down for resource conservation
kubectl scale deployment webapp --replicas=2

# Scale to zero (maintenance)
kubectl scale deployment webapp --replicas=0
```

#### 2. Conditional Scaling
```bash
# Scale only if current replicas match condition
kubectl scale deployment webapp --current-replicas=3 --replicas=5

# Verify scaling
kubectl get deployment webapp
kubectl get pods -l app=webapp
```

#### 3. Horizontal Pod Autoscaler (HPA)
```bash
# Create HPA (requires metrics server)
kubectl autoscale deployment webapp --min=2 --max=10 --cpu-percent=70

# Check HPA status
kubectl get hpa
kubectl describe hpa webapp

# Test scaling behavior
kubectl run load-generator --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://webapp-service; done"
```

### Deployment Management

#### 1. Deployment Strategies
```yaml
# Recreate strategy (downtime acceptable)
spec:
  strategy:
    type: Recreate

# Rolling update (zero downtime)
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

#### 2. Deployment Labels and Selectors
```bash
# Update deployment labels
kubectl label deployment webapp environment=production

# Change selector (requires deployment recreation)
kubectl patch deployment webapp -p '{"spec":{"selector":{"matchLabels":{"app":"webapp","version":"v2"}}}}'
```

#### 3. Deployment Annotations
```bash
# Add deployment annotations
kubectl annotate deployment webapp deployment.kubernetes.io/change-cause="Updated to nginx 1.21"

# View annotations
kubectl get deployment webapp -o jsonpath='{.metadata.annotations}'
```

### ReplicaSet Understanding

#### 1. ReplicaSet Relationship
```bash
# View ReplicaSets created by deployment
kubectl get rs -l app=webapp
kubectl describe rs <replicaset-name>

# Check ReplicaSet ownership
kubectl get rs <rs-name> -o jsonpath='{.metadata.ownerReferences}'
```

#### 2. Manual ReplicaSet Operations
```bash
# Scale ReplicaSet directly (not recommended)
kubectl scale rs <rs-name> --replicas=5

# Delete ReplicaSet (deployment will recreate)
kubectl delete rs <rs-name>
```

## Common CKA Deployment Scenarios

### Scenario 1: Deployment Not Progressing
```bash
# Problem: Deployment stuck in progressing state
kubectl get deployment webapp
kubectl describe deployment webapp | grep -A 10 "Conditions"
kubectl get events | grep webapp

# Common causes and solutions:
# - Insufficient resources: Check node capacity
# - Image pull issues: Verify image name/registry
# - Pod security policies: Check security contexts
```

### Scenario 2: Partial Update Failure
```bash
# Problem: Some pods updated, some still old version
kubectl get pods -l app=webapp -o custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[0].image"

# Troubleshoot mixed versions
kubectl rollout status deployment/webapp
kubectl describe deployment webapp
kubectl get rs -l app=webapp

# Force complete rollout
kubectl rollout restart deployment/webapp
```

### Scenario 3: Performance Issues After Update
```bash
# Problem: Application slow after deployment update
kubectl top pods -l app=webapp
kubectl describe deployment webapp | grep -A 5 "resource"
kubectl logs -l app=webapp --tail=100

# Rollback if performance is unacceptable
kubectl rollout undo deployment/webapp
```

## Daily Goals Checklist

- [ ] Master deployment creation and management
- [ ] Can perform rolling updates and rollbacks efficiently
- [ ] Understand deployment strategies and their implications
- [ ] Can troubleshoot deployment issues systematically
- [ ] Practice scaling operations and HPA concepts
- [ ] Ready for service networking

## Next Day Preparation

### What You'll Need for Day 11
- Deployment management skills ✓
- Rolling update/rollback proficiency ✓
- Scaling operation knowledge ✓
- Troubleshooting capabilities ✓

### Coming Up: Services Introduction
Day 11 will cover service types, endpoints, and service discovery mechanisms.

## Study Notes Section

**Deployment Quick Commands:**
```bash
# Create and manage
kubectl create deployment <name> --image=<image>
kubectl scale deployment <name> --replicas=<num>
kubectl set image deployment/<name> <container>=<new-image>

# Updates and rollbacks
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout history deployment/<name>

# Troubleshooting
kubectl describe deployment <name>
kubectl get rs -l app=<name>
kubectl get events | grep <name>
```

**Personal Deployment Strategies:**
_Record preferred deployment patterns and troubleshooting approaches_