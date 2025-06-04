# Week 11: Speed & Efficiency Training

## Overview
Focus on speed optimization, automation, and time management skills crucial for CKA exam success.

## Daily Tasks

### Day 71: kubectl Speed Mastery (1.5 hours)
- [ ] Imperative command speed drills
- [ ] Optimize documentation navigation
- [ ] Improve task completion efficiency

### Day 72: Common Task Automation (1.5 hours)
- [ ] Practice frequently used command combinations
- [ ] Optimize bash aliases and shortcuts
- [ ] Reduce repetitive task time

### Day 73: Documentation Mastery (1.5 hours)
- [ ] kubernetes.io navigation speed training
- [ ] Organize bookmarks and quick references
- [ ] Practice sub-5-minute documentation searches

### Day 74: Time Management Practice (1.5 hours)
- [ ] Timed task completion drills
- [ ] Practice question prioritization strategies
- [ ] Optimize exam time management

### Day 75: Error Recovery Techniques (1.5 hours)
- [ ] Practice quick mistake correction methods
- [ ] Learn partial credit maximization strategies
- [ ] Build efficiency under pressure

## Weekend Session (4 hours)
- [ ] **MOCK EXAM:** Full timed exam
- [ ] Speed and efficiency analysis
- [ ] **Goal Check:** 75-80% score with good time management

## Speed Optimization Techniques

### Essential Aliases and Shortcuts
```bash
# Core aliases (set these up immediately)
alias k=kubectl
alias kg='kubectl get'
alias kd='kubectl describe'
alias kdel='kubectl delete'
alias kl='kubectl logs'
alias ke='kubectl edit'
alias kex='kubectl exec'

# Output format shortcuts
export do='--dry-run=client -o yaml'
export now='--force --grace-period 0'

# Namespace shortcuts
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kgn='kubectl get nodes'

# Advanced aliases
alias kaf='kubectl apply -f'
alias kcf='kubectl create -f'
alias krm='kubectl delete'
alias klog='kubectl logs --tail=100'
```

### Imperative Command Mastery

#### Pod Creation Speed Drills
```bash
# Basic pod - Target: 10 seconds
kubectl run nginx --image=nginx

# Pod with port - Target: 15 seconds
kubectl run nginx --image=nginx --port=80

# Pod with environment variables - Target: 20 seconds
kubectl run nginx --image=nginx --env="VAR1=value1" --env="VAR2=value2"

# Pod with resource limits - Target: 25 seconds
kubectl run nginx --image=nginx --limits="cpu=200m,memory=128Mi" --requests="cpu=100m,memory=64Mi"

# Generate YAML quickly - Target: 15 seconds
kubectl run nginx --image=nginx $do > pod.yaml
```

#### Deployment Speed Drills
```bash
# Basic deployment - Target: 15 seconds
kubectl create deployment nginx --image=nginx

# Scale deployment - Target: 10 seconds
kubectl scale deployment nginx --replicas=3

# Update image - Target: 15 seconds
kubectl set image deployment/nginx nginx=nginx:1.19

# Expose deployment - Target: 20 seconds
kubectl expose deployment nginx --port=80 --type=NodePort

# Complete deployment with service - Target: 45 seconds
kubectl create deployment webapp --image=nginx --replicas=3
kubectl expose deployment webapp --port=80 --target-port=80 --type=ClusterIP
kubectl get all -l app=webapp
```

#### Service Creation Drills
```bash
# ClusterIP service - Target: 15 seconds
kubectl expose pod nginx --port=80

# NodePort service - Target: 20 seconds
kubectl expose deployment nginx --port=80 --type=NodePort

# Service from scratch - Target: 25 seconds
kubectl create service clusterip myapp --tcp=80:8080
```

### ConfigMap and Secret Speed Drills
```bash
# ConfigMap from literal - Target: 15 seconds
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Secret from literal - Target: 15 seconds
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=secret123

# ConfigMap from file - Target: 20 seconds
echo "key=value" > config.properties
kubectl create configmap file-config --from-file=config.properties

# Generate and edit - Target: 30 seconds
kubectl create configmap test-config --from-literal=test=value $do > config.yaml
vi config.yaml
kubectl apply -f config.yaml
```

### Documentation Navigation Optimization

#### Essential Bookmarks
1. **kubectl Cheat Sheet** - https://kubernetes.io/docs/reference/kubectl/cheatsheet/
2. **Pod Spec** - https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#pod-v1-core
3. **Service Spec** - https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#service-v1-core
4. **Deployment Spec** - https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#deployment-v1-apps
5. **NetworkPolicy Examples** - https://kubernetes.io/docs/concepts/services-networking/network-policies/
6. **RBAC Examples** - https://kubernetes.io/docs/reference/access-authn-authz/rbac/
7. **Volume Types** - https://kubernetes.io/docs/concepts/storage/volumes/
8. **Troubleshooting** - https://kubernetes.io/docs/tasks/debug/debug-application/

#### Fast Search Techniques
```bash
# Use kubectl explain for quick API reference
kubectl explain pod.spec
kubectl explain deployment.spec.strategy
kubectl explain service.spec

# Quick example generation
kubectl create deployment test --image=nginx $do
kubectl create service clusterip test --tcp=80:80 $do
kubectl create networkpolicy deny-all $do
```

### Time Management Strategies

#### Question Prioritization Matrix
| Points | Estimated Time | Priority |
|--------|---------------|----------|
| 8-12   | 15-20 min    | HIGH     |
| 5-7    | 10-15 min    | MEDIUM   |
| 2-4    | 5-10 min     | LOW      |

#### Time Boxing Approach
1. **Quick scan (2 minutes)**: Read all questions, note point values
2. **High-value tasks first (60 minutes)**: Tackle 8-12 point questions
3. **Medium tasks (40 minutes)**: Complete 5-7 point questions
4. **Low tasks (15 minutes)**: Quick wins on 2-4 point questions
5. **Review time (3 minutes)**: Verify solutions work

### Error Recovery Techniques

#### Common Mistake Patterns
```bash
# Wrong context - Quick fix
kubectl config get-contexts
kubectl config use-context <correct-context>

# Typo in commands - Prevention
# Use tab completion aggressively
kubectl get po<TAB>
kubectl describe no<TAB>

# YAML syntax errors - Quick debugging
kubectl apply -f file.yaml --dry-run=client

# Resource already exists - Force recreation
kubectl delete pod nginx $now
kubectl run nginx --image=nginx
```

#### Backup Strategies
```bash
# Before modifying existing resources
kubectl get deploy nginx -o yaml > backup-deploy.yaml

# Quick rollback for deployments
kubectl rollout undo deployment nginx

# Resource recreation from backup
kubectl delete -f original.yaml
kubectl apply -f backup.yaml
```

## Speed Training Exercises

### Exercise 1: Complete Application Stack (Target: 5 minutes)
1. Create namespace "webapp"
2. Create deployment "frontend" with nginx:1.19, 3 replicas
3. Expose frontend with ClusterIP service on port 80
4. Create ConfigMap with app configuration
5. Create Secret with database credentials
6. Verify all resources are running

### Exercise 2: RBAC Setup (Target: 4 minutes)
1. Create service account "developer"
2. Create role with pod read/write permissions
3. Create RoleBinding connecting SA to role
4. Test permissions with `kubectl auth can-i`

### Exercise 3: Network Policy (Target: 3 minutes)
1. Create two test pods with different labels
2. Create network policy allowing only specific pod communication
3. Test connectivity between pods

### Exercise 4: Storage Configuration (Target: 4 minutes)
1. Create PersistentVolume with hostPath
2. Create PersistentVolumeClaim
3. Create pod using the PVC
4. Verify volume is mounted correctly

## Advanced Efficiency Tips

### Vim Speed Techniques
```bash
# Essential vim commands for YAML editing
:set number          # Show line numbers
:set expandtab       # Use spaces instead of tabs
:set shiftwidth=2    # 2-space indentation

# Quick navigation
gg                   # Go to top
G                    # Go to bottom
:20                  # Go to line 20
/searchterm          # Search forward
n                    # Next search result

# Quick editing
dd                   # Delete line
yy                   # Copy line
p                    # Paste
u                    # Undo
Ctrl+r              # Redo
```

### Bash Productivity
```bash
# History search
Ctrl+r              # Reverse search
!!                  # Repeat last command
!k                  # Repeat last command starting with 'k'

# Quick navigation
cd -                # Go to previous directory
pushd /path         # Save current dir and go to path
popd                # Return to saved directory

# Command substitution
kubectl delete pod $(kubectl get pods -l app=test -o name)
```

### Context and Namespace Management
```bash
# Quick context switching
kubectl config get-contexts
kubectl config use-context cluster1

# Namespace shortcuts
kubectl config set-context --current --namespace=production

# Multi-cluster management
kubectl --context=cluster1 get pods
kubectl --context=cluster2 get nodes
```

## Mock Exam Preparation

### 15-Minute Practice Scenarios
1. **Cluster maintenance**: Drain node, upgrade kubelet, uncordon
2. **Application deployment**: Deploy app with service, configmap, secret
3. **Troubleshooting**: Fix broken pod, service connectivity issue
4. **Security**: Set up RBAC, network policies
5. **Storage**: Configure PV/PVC, fix mounting issues

### Performance Metrics to Track
- [ ] Commands per minute (target: 15-20)
- [ ] Time to complete basic tasks (target: <2 minutes)
- [ ] Documentation lookup time (target: <30 seconds)
- [ ] Error correction time (target: <1 minute)

## Success Criteria
- [ ] Can complete basic tasks in under 2 minutes
- [ ] Can navigate documentation efficiently (<30 seconds)
- [ ] Can recover from errors quickly (<1 minute)
- [ ] Can prioritize and manage time effectively
- [ ] Can score 75-80% on timed mock exams

## Notes Section
_Track your speed improvements and efficiency gains here_