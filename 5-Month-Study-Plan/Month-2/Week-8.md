# Week 8: Month 2 Assessment

## Overview
Intensive practice week focusing on weak areas identified from Month 1 and preparation for comprehensive assessment.

## Daily Tasks

### Day 50: Networking Weak Areas (1.5 hours)
- [ ] Focus on networking concepts that need improvement
- [ ] Practice service and ingress troubleshooting
- [ ] Speed drills on network policy creation

### Day 51: Cluster Maintenance Weak Areas (1.5 hours)
- [ ] Focus on cluster administration gaps
- [ ] Practice upgrade procedures
- [ ] etcd backup/restore speed improvement

### Day 52: Security Weak Areas (1.5 hours)
- [ ] Focus on RBAC troubleshooting
- [ ] Practice security policy implementation
- [ ] Review service account management

### Day 53: Speed Improvement Drills (1.5 hours)
- [ ] Timed exercises for common tasks
- [ ] Practice quick context switching
- [ ] Optimize kubectl command usage

### Day 54: Comprehensive Review (1.5 hours)
- [ ] Review all Month 1-2 topics
- [ ] Practice integrated scenarios
- [ ] Final preparation for mock exam

## Weekend Session - Major Mock Exam (4 hours)
- [ ] **MOCK EXAM:** Full CKA simulation covering Months 1-2
- [ ] Detailed review and improvement planning
- [ ] **Goal Check:** 60-70% score for Month 3 readiness

## Speed Drill Exercises

### Networking Speed Drills (Target: < 5 minutes each)

```bash
# 1. Create a service exposing deployment
kubectl create deployment nginx --image=nginx --replicas=3
kubectl expose deployment nginx --port=80 --type=NodePort

# 2. Create network policy allowing specific traffic
# Allow only pods with label app=frontend to access app=backend on port 8080

# 3. Debug service connectivity
# Pod cannot reach service - find and fix the issue

# 4. Configure ingress for two applications
# /app1 -> service1:80
# /app2 -> service2:80
```

### Cluster Administration Speed Drills (Target: < 10 minutes each)

```bash
# 1. Perform node maintenance
# - Safely drain node
# - Perform maintenance
# - Return node to service

# 2. Backup etcd
# Complete backup with verification

# 3. Upgrade cluster component
# Upgrade kubelet on one node

# 4. Troubleshoot failed control plane component
# API server is not responding - diagnose and fix
```

### Security Speed Drills (Target: < 5 minutes each)

```bash
# 1. Create RBAC for developer
# - Can manage pods/deployments in dev namespace
# - Can only view in prod namespace

# 2. Fix pod security issue
# Pod failing due to security context - diagnose and fix

# 3. Implement namespace isolation
# Prevent all traffic between namespaces

# 4. Create service account with specific permissions
# SA can only list and get pods across all namespaces
```

## Mock Exam Topics Distribution

### Cluster Architecture, Installation & Configuration (25%)
- [ ] Manage role-based access control (RBAC)
- [ ] Use Kubeadm to install a basic cluster
- [ ] Manage a highly-available Kubernetes cluster
- [ ] Provision underlying infrastructure to deploy a Kubernetes cluster
- [ ] Perform a version upgrade on a Kubernetes cluster using Kubeadm
- [ ] Implement etcd backup and restore

### Workloads & Scheduling (15%)
- [ ] Understand deployments and how to perform rolling update and rollbacks
- [ ] Use ConfigMaps and Secrets to configure applications
- [ ] Know how to scale applications
- [ ] Understand the primitives used to create robust, self-healing, application deployments
- [ ] Understand how resource limits can affect Pod scheduling
- [ ] Awareness of manifest management and common templating tools

### Services & Networking (20%)
- [ ] Understand host networking configuration on the cluster nodes
- [ ] Understand connectivity between Pods
- [ ] Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
- [ ] Know how to use Ingress controllers and Ingress resources
- [ ] Know how to configure and use CoreDNS
- [ ] Choose an appropriate container network interface plugin

### Storage (10%)
- [ ] Understand storage classes, persistent volumes
- [ ] Understand volume mode, access modes and reclaim policies for volumes
- [ ] Understand persistent volume claims primitive
- [ ] Know how to configure applications with persistent storage

### Troubleshooting (30%)
- [ ] Evaluate cluster and node logging
- [ ] Understand how to monitor applications
- [ ] Manage container stdout & stderr logs
- [ ] Troubleshoot application failure
- [ ] Troubleshoot cluster component failure
- [ ] Troubleshoot networking

## Practice Scenarios

### Scenario 1: Multi-tier Application Deployment
1. Deploy a frontend application (3 replicas)
2. Deploy a backend API (2 replicas)
3. Deploy a database (stateful, with persistent storage)
4. Configure services for each tier
5. Implement network policies for security
6. Configure ingress for external access

### Scenario 2: Cluster Maintenance
1. Perform rolling upgrade of worker nodes
2. Backup etcd before maintenance
3. Debug and fix a failing node
4. Renew expiring certificates
5. Restore cluster from backup if needed

### Scenario 3: Security Implementation
1. Create namespace isolation
2. Implement RBAC for different user roles
3. Configure pod security policies
4. Set up service accounts with minimal permissions
5. Audit and fix security violations

## Review Checklist

### Must Know Cold
- [ ] kubeadm init/join process
- [ ] etcd backup/restore commands
- [ ] Basic RBAC creation
- [ ] Service types and their differences
- [ ] Network policy structure
- [ ] PV/PVC lifecycle
- [ ] Node maintenance procedures
- [ ] Pod troubleshooting steps

### Common Pitfalls to Avoid
- [ ] Forgetting to verify context before working
- [ ] Not reading questions completely
- [ ] Spending too much time on low-value questions
- [ ] Not using kubectl imperative commands
- [ ] Forgetting to verify solutions work

## Learning Resources
- Review all bookmarked documentation pages
- Practice with killer.sh if available
- Review command cheat sheet
- Revisit weak areas from Months 1-2

## Success Criteria
- [ ] Complete all daily weak area improvements
- [ ] Score 60-70% on comprehensive mock exam
- [ ] Can complete common tasks within time targets
- [ ] Feel confident with all Month 1-2 topics
- [ ] Ready to tackle advanced troubleshooting in Month 3

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips from your assessment_