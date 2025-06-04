# Day 8: kubectl Command Mastery

## Overview (1.5 hours)
Master the distinction between imperative and declarative commands, and optimize kubectl usage for maximum CKA exam efficiency.

## Why This Matters for the CKA Exam
- **Every exam question** requires kubectl commands
- **Imperative commands** save 60-80% of time compared to YAML writing
- **Output formats** are essential for troubleshooting tasks
- **Field/label selectors** help find specific resources quickly

## Task 1: Practice Imperative vs Declarative Commands (45 minutes)

### What You'll Do
Learn when to use imperative commands for speed and declarative for complex configurations.

### Imperative Commands (Fast for Exam)

#### 1. Pod Creation
```bash
# Basic pod creation
kubectl run nginx --image=nginx
kubectl run busybox --image=busybox --command -- sleep 3600

# Pod with specific options
kubectl run nginx --image=nginx --port=80 --env="ENV=production"
kubectl run debug --image=busybox --rm -it --restart=Never -- sh

# Pod with resource limits
kubectl run nginx --image=nginx --limits="cpu=200m,memory=256Mi" --requests="cpu=100m,memory=128Mi"

# Pod with labels
kubectl run webapp --image=nginx --labels="app=web,tier=frontend"
```

#### 2. Deployment Creation
```bash
# Basic deployment
kubectl create deployment nginx --image=nginx

# Deployment with replicas
kubectl create deployment webapp --image=nginx --replicas=3

# Deployment with port
kubectl create deployment api --image=nginx --port=8080
```

#### 3. Service Creation
```bash
# Expose pod
kubectl expose pod nginx --port=80 --type=NodePort

# Expose deployment
kubectl expose deployment webapp --port=80 --target-port=8080

# Create service directly
kubectl create service clusterip webapp --tcp=80:8080
kubectl create service nodeport webapp --tcp=80:8080
```

#### 4. ConfigMap and Secret Creation
```bash
# ConfigMap from literals
kubectl create configmap app-config --from-literal=database_url=mongodb://localhost --from-literal=debug=true

# ConfigMap from file
echo "config=value" > app.properties
kubectl create configmap file-config --from-file=app.properties

# Secret from literals
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=secret123

# Secret for Docker registry
kubectl create secret docker-registry regcred --docker-server=registry.io --docker-username=user --docker-password=pass
```

### Declarative Commands (Complex Configurations)

#### 1. Generate YAML Templates
```bash
# Generate pod YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Generate deployment YAML
kubectl create deployment webapp --image=nginx --dry-run=client -o yaml > deployment.yaml

# Generate service YAML
kubectl expose deployment webapp --port=80 --dry-run=client -o yaml > service.yaml

# Generate configmap YAML
kubectl create configmap app-config --from-literal=key=value --dry-run=client -o yaml > configmap.yaml
```

#### 2. Apply Configurations
```bash
# Apply single file
kubectl apply -f pod.yaml

# Apply multiple files
kubectl apply -f deployment.yaml -f service.yaml

# Apply directory
kubectl apply -f ./manifests/

# Apply with record
kubectl apply -f deployment.yaml --record
```

### CKA Exam Strategy

#### When to Use Imperative
- ✅ Simple pod creation
- ✅ Basic deployments and services
- ✅ ConfigMaps and secrets
- ✅ Quick testing and validation
- ✅ Time-pressured scenarios

#### When to Use Declarative
- ✅ Complex pod specifications (init containers, volumes)
- ✅ Advanced deployment strategies
- ✅ Multi-container pods
- ✅ Custom resource definitions
- ✅ When modification is needed

### Practice Scenarios
```bash
# Scenario 1: Quick web server (imperative)
kubectl run nginx --image=nginx
kubectl expose pod nginx --port=80 --type=NodePort

# Scenario 2: Complex application (declarative)
kubectl run webapp --image=nginx --dry-run=client -o yaml > webapp.yaml
# Edit webapp.yaml to add init containers, volumes, etc.
kubectl apply -f webapp.yaml
```

## Task 2: Master Output Formats, Field Selectors, Label Selectors (30 minutes)

### What You'll Do
Learn to extract specific information quickly and filter resources effectively.

### Output Formats

#### 1. YAML Output (-o yaml)
```bash
# Get complete resource definition
kubectl get pod nginx -o yaml

# Get multiple resources
kubectl get pods,services -o yaml

# Useful for backup and analysis
kubectl get deployment webapp -o yaml > webapp-backup.yaml
```

#### 2. JSON Output (-o json)
```bash
# Get JSON format
kubectl get pod nginx -o json

# Extract specific fields with jq
kubectl get pods -o json | jq '.items[].metadata.name'
kubectl get nodes -o json | jq '.items[].status.nodeInfo.kubeletVersion'
```

#### 3. JSONPath Output (-o jsonpath)
```bash
# Extract pod names
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Extract pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Extract node information
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Complex JSONPath queries
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'
```

#### 4. Custom Columns (-o custom-columns)
```bash
# Pod information
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName"

# Node information
kubectl get nodes -o custom-columns="NAME:.metadata.name,STATUS:.status.conditions[?(@.type=='Ready')].status,VERSION:.status.nodeInfo.kubeletVersion"

# Service information
kubectl get services -o custom-columns="NAME:.metadata.name,TYPE:.spec.type,CLUSTER-IP:.spec.clusterIP,PORT:.spec.ports[0].port"
```

#### 5. Wide Output (-o wide)
```bash
# Extended pod information
kubectl get pods -o wide

# Extended node information
kubectl get nodes -o wide

# Extended service information
kubectl get services -o wide
```

### Field Selectors

#### 1. Basic Field Selection
```bash
# Filter by status
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector status.phase=Pending

# Filter by node
kubectl get pods --field-selector spec.nodeName=worker-node-1

# Filter by namespace
kubectl get pods --field-selector metadata.namespace=kube-system
```

#### 2. Advanced Field Selection
```bash
# Multiple conditions (AND)
kubectl get pods --field-selector status.phase=Running,spec.nodeName=worker-node-1

# Get events for specific object
kubectl get events --field-selector involvedObject.name=nginx

# Filter by creation timestamp
kubectl get pods --field-selector metadata.creationTimestamp\>2023-01-01T00:00:00Z
```

### Label Selectors

#### 1. Basic Label Selection
```bash
# Single label
kubectl get pods -l app=nginx
kubectl get pods -l tier=frontend

# Show all labels
kubectl get pods --show-labels

# Add labels to existing resources
kubectl label pod nginx env=production
kubectl label deployment webapp version=v1.2
```

#### 2. Advanced Label Selection
```bash
# Multiple labels (AND)
kubectl get pods -l app=nginx,env=production

# Label existence
kubectl get pods -l app  # Pods that have 'app' label
kubectl get pods -l '!app'  # Pods that don't have 'app' label

# Set-based selectors
kubectl get pods -l 'env in (production,staging)'
kubectl get pods -l 'tier notin (cache,queue)'

# Inequality
kubectl get pods -l version!=v1.0
```

### CKA Exam Applications

#### 1. Troubleshooting Scenarios
```bash
# Find all failing pods
kubectl get pods --field-selector status.phase=Failed

# Find pods on specific node
kubectl get pods -o wide --field-selector spec.nodeName=worker-node-1

# Get events for troubleshooting
kubectl get events --sort-by=.metadata.creationTimestamp | tail -10
```

#### 2. Information Extraction
```bash
# Get all pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}' | tr ' ' '\n'

# Get node resource information
kubectl get nodes -o custom-columns="NAME:.metadata.name,CPU:.status.capacity.cpu,MEMORY:.status.capacity.memory"

# Check service endpoints
kubectl get endpoints -o custom-columns="NAME:.metadata.name,ENDPOINTS:.subsets[*].addresses[*].ip"
```

## Task 3: Speed Practice with kubectl run and kubectl create (15 minutes)

### What You'll Do
Build muscle memory for the most common kubectl operations with timed exercises.

### Speed Drill 1: Basic Resource Creation (5 minutes)
```bash
# Target: Complete all in under 2 minutes
kubectl run nginx --image=nginx
kubectl create deployment webapp --image=httpd --replicas=3
kubectl expose deployment webapp --port=80
kubectl create configmap app-config --from-literal=env=prod
kubectl create secret generic app-secret --from-literal=key=secret
kubectl get all
kubectl delete deployment webapp
kubectl delete pod nginx
kubectl delete configmap app-config
kubectl delete secret app-secret
```

### Speed Drill 2: Complex Scenarios (5 minutes)
```bash
# Target: Complete in under 3 minutes
# Create pod with resource limits
kubectl run limited-pod --image=nginx --limits="cpu=100m,memory=128Mi" --requests="cpu=50m,memory=64Mi"

# Create deployment and scale
kubectl create deployment scaled-app --image=nginx
kubectl scale deployment scaled-app --replicas=5

# Create service and verify
kubectl expose deployment scaled-app --port=80 --type=NodePort
kubectl get service scaled-app -o wide

# Clean up
kubectl delete deployment scaled-app
kubectl delete service scaled-app
kubectl delete pod limited-pod
```

### Speed Drill 3: Information Extraction (5 minutes)
```bash
# Target: Complete all queries in under 2 minutes
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get nodes -o custom-columns="NAME:.metadata.name,STATUS:.status.conditions[?(@.type=='Ready')].status"
kubectl get pods --field-selector status.phase=Running
kubectl get all -l app=nginx
kubectl describe node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') | head -20
```

### Common kubectl Patterns for CKA

#### 1. Quick Resource Operations
```bash
# Create and expose pattern
kubectl create deployment nginx --image=nginx && kubectl expose deployment nginx --port=80

# Generate, edit, apply pattern
kubectl run pod --image=nginx --dry-run=client -o yaml > pod.yaml
# Edit pod.yaml as needed
kubectl apply -f pod.yaml

# Backup and restore pattern
kubectl get deployment webapp -o yaml > webapp-backup.yaml
kubectl delete deployment webapp
kubectl apply -f webapp-backup.yaml
```

#### 2. Debugging Workflows
```bash
# Standard debugging sequence
kubectl get pods
kubectl describe pod <problematic-pod>
kubectl logs <problematic-pod>
kubectl get events --field-selector involvedObject.name=<pod-name>

# Resource investigation
kubectl get all --show-labels
kubectl get pods -o wide
kubectl top pods
```

#### 3. Resource Management
```bash
# Scaling operations
kubectl scale deployment webapp --replicas=0  # Scale down
kubectl scale deployment webapp --replicas=3  # Scale up

# Rolling updates
kubectl set image deployment/webapp nginx=nginx:1.19
kubectl rollout status deployment webapp
kubectl rollout undo deployment webapp
```

## Time Management for CKA

### Command Efficiency Tips
- **Use aliases**: `k` instead of `kubectl`
- **Tab completion**: Let bash complete resource names
- **Copy-paste**: From previous commands or terminal history
- **Dry-run templates**: Generate YAML quickly with `--dry-run=client -o yaml`

### Exam Strategy
- **Start imperative**: Use imperative commands for simple tasks
- **Switch declarative**: When complex configuration needed
- **Verify quickly**: Use `kubectl get` to confirm resources created
- **Clean as you go**: Delete test resources to avoid confusion

## Daily Goals Checklist

- [ ] Master imperative vs declarative command decision making
- [ ] Comfortable with all major output formats
- [ ] Can use field and label selectors effectively
- [ ] Complete speed drills within target times
- [ ] Understand when each approach is optimal
- [ ] Ready for core object manipulation

## Next Day Preparation

### What You'll Need for Day 9
- kubectl command proficiency ✓
- Output format mastery ✓
- Selector usage skills ✓
- Speed and efficiency ✓

### Coming Up: Pods Deep Dive
Day 9 will explore pod lifecycle, multi-container patterns, and advanced pod configurations.

## Study Notes Section

**Speed Reference Commands:**
```bash
# Quick creation
kubectl run <name> --image=<image>
kubectl create deployment <name> --image=<image>
kubectl expose deployment <name> --port=<port>

# Quick info
kubectl get pods -o wide
kubectl describe pod <name>
kubectl logs <name>

# Quick templates
kubectl create <resource> --dry-run=client -o yaml
```

**Personal Command Optimizations:**
_Record your preferred command patterns and shortcuts_