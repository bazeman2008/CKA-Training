# Week 2: kubectl Mastery & Core Objects

## Overview
Master kubectl commands and work with core Kubernetes objects including pods, deployments, services, and configurations.

## Daily Tasks

### Day 8: kubectl Command Mastery (1.5 hours)
- [ ] Practice imperative vs declarative commands
- [ ] Master output formats, field selectors, label selectors
- [ ] Speed practice with `kubectl run` and `kubectl create`

### Day 9: Pods Deep Dive (1.5 hours)
- [ ] Study pod lifecycle and phases
- [ ] Practice multi-container patterns
- [ ] Implement init containers and sidecar patterns

### Day 10: Deployments & ReplicaSets (1.5 hours)
- [ ] Practice deployment strategies and rolling updates
- [ ] Master rollback procedures
- [ ] Practice scaling and managing deployments

### Day 11: Services Introduction (1.5 hours)
- [ ] Configure service types (ClusterIP, NodePort, LoadBalancer)
- [ ] Practice service discovery and endpoints
- [ ] Speed practice: expose applications quickly

### Day 12: ConfigMaps & Secrets (1.5 hours)
- [ ] Practice configuration management
- [ ] Mount configs and secrets in pods
- [ ] Troubleshoot configuration issues

## Weekend Session (2.5 hours)
- [ ] Create complete application stacks (pods + services + configs)
- [ ] Speed drills for basic object creation
- [ ] **Goal Check:** Deploy applications in under 10 minutes

## Key Commands to Practice

```bash
# kubectl output formats and selectors
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -l app=nginx
kubectl get pods --field-selector status.phase=Running

# Pod creation
kubectl run nginx --image=nginx
kubectl run busybox --image=busybox --rm -it -- sh

# Deployment management
kubectl create deployment nginx --image=nginx --replicas=3
kubectl scale deployment nginx --replicas=5
kubectl set image deployment/nginx nginx=nginx:1.19
kubectl rollout status deployment nginx
kubectl rollout undo deployment nginx

# Service creation
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl create service clusterip nginx --tcp=80:80

# ConfigMaps and Secrets
kubectl create configmap app-config --from-literal=key1=value1
kubectl create secret generic app-secret --from-literal=password=secret123
```

## Multi-Container Pod Examples

```yaml
# Sidecar pattern
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-app
    image: nginx
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Sidecar running"; sleep 30; done']

# Init container
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: init
    image: busybox
    command: ['sh', '-c', 'echo "Init complete" && sleep 5']
  containers:
  - name: main
    image: nginx
```

## Learning Resources
- kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- Pod Lifecycle: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
- Deployments: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- Services: https://kubernetes.io/docs/concepts/services-networking/service/

## Success Criteria
- [ ] Can create any basic resource imperatively within seconds
- [ ] Understand pod lifecycle and multi-container patterns
- [ ] Can manage deployments including updates and rollbacks
- [ ] Can expose applications using appropriate service types
- [ ] Can manage application configuration with ConfigMaps and Secrets

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_