# Week 10: Infrastructure Troubleshooting

## Overview
Master troubleshooting cluster infrastructure issues including nodes, control plane, and networking.

## Daily Tasks

### Day 64: Worker Node Issues (1.5 hours)
- [ ] Troubleshoot kubelet and kube-proxy
- [ ] Debug container runtime problems
- [ ] Resolve Node NotReady scenarios

### Day 65: Control Plane Debugging (1.5 hours)
- [ ] Fix API server connectivity and certificate issues
- [ ] Debug scheduler and controller-manager problems
- [ ] Resolve control plane component failures

### Day 66: Network Connectivity Issues (1.5 hours)
- [ ] Debug pod-to-pod communication
- [ ] Fix service-to-pod communication
- [ ] Resolve ingress and DNS problems

### Day 67: Cluster-Level Issues (1.5 hours)
- [ ] Troubleshoot etcd problems
- [ ] Debug cluster state issues
- [ ] Fix add-on and extension problems

### Day 68: Performance and Resource Issues (1.5 hours)
- [ ] Diagnose resource exhaustion
- [ ] Identify performance degradation
- [ ] Interpret monitoring and metrics

## Weekend Session (3 hours)
- [ ] Infrastructure troubleshooting practice
- [ ] Multi-layer problem resolution
- [ ] **Goal Check:** Systematic troubleshooting mastery

## Node Troubleshooting

### Node Status Issues
```bash
# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Check node conditions
kubectl get nodes -o custom-columns="NAME:.metadata.name,STATUS:.status.conditions[?(@.type=='Ready')].status"

# Node resource usage
kubectl top nodes
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

### Kubelet Troubleshooting
```bash
# Check kubelet service
systemctl status kubelet
journalctl -u kubelet -f

# Kubelet configuration
cat /var/lib/kubelet/config.yaml
ps aux | grep kubelet

# Container runtime issues
systemctl status containerd
crictl ps
crictl pods
```

### Common Node Problems

#### Node NotReady
```bash
# Check kubelet logs
journalctl -u kubelet --since "1 hour ago"

# Check disk space
df -h
du -sh /var/lib/* | sort -hr

# Check memory
free -h
cat /proc/meminfo

# Check network connectivity
ping <master-ip>
telnet <master-ip> 6443
```

#### Network Plugin Issues
```bash
# Check CNI plugin
ls /opt/cni/bin/
cat /etc/cni/net.d/*

# For Calico
kubectl get pods -n kube-system | grep calico
kubectl logs -n kube-system calico-node-<pod-id>

# For Flannel
kubectl get pods -n kube-system | grep flannel
kubectl logs -n kube-system kube-flannel-<pod-id>
```

## Control Plane Troubleshooting

### API Server Issues
```bash
# Check API server pod
kubectl get pods -n kube-system | grep apiserver
kubectl logs -n kube-system kube-apiserver-<node-name>

# Check API server process
ps aux | grep apiserver

# Test API server connectivity
curl -k https://localhost:6443/healthz

# Check certificates
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

### Scheduler Problems
```bash
# Check scheduler status
kubectl get pods -n kube-system | grep scheduler
kubectl logs -n kube-system kube-scheduler-<node-name>

# Check for pending pods
kubectl get pods --all-namespaces | grep Pending

# Scheduler configuration
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

### Controller Manager Issues
```bash
# Check controller manager
kubectl get pods -n kube-system | grep controller-manager
kubectl logs -n kube-system kube-controller-manager-<node-name>

# Check for unhealthy controllers
kubectl get componentstatuses
```

## etcd Troubleshooting

```bash
# Check etcd health
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  endpoint health

# Check etcd cluster status
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  endpoint status --cluster -w table

# etcd logs
kubectl logs -n kube-system etcd-<node-name>
journalctl -u etcd
```

## Network Troubleshooting

### Pod-to-Pod Communication
```bash
# Create test pods
kubectl run test-1 --image=busybox --rm -it -- sh
kubectl run test-2 --image=nginx

# Test connectivity
kubectl exec test-1 -- ping <test-2-ip>
kubectl exec test-1 -- wget -qO- <test-2-ip>

# Check network policies
kubectl get networkpolicy --all-namespaces
kubectl describe networkpolicy <policy-name>
```

### Service Discovery Issues
```bash
# Test service resolution
kubectl exec test-pod -- nslookup kubernetes.default
kubectl exec test-pod -- nslookup <service-name>.<namespace>.svc.cluster.local

# Check service endpoints
kubectl get endpoints
kubectl describe service <service-name>

# Debug CoreDNS
kubectl get pods -n kube-system | grep coredns
kubectl logs -n kube-system coredns-<pod-id>
```

### Ingress Problems
```bash
# Check ingress controller
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx ingress-nginx-controller-<pod-id>

# Check ingress resources
kubectl get ingress --all-namespaces
kubectl describe ingress <ingress-name>

# Test ingress connectivity
curl -H "Host: example.com" http://<ingress-ip>/
```

## Performance Troubleshooting

### Resource Exhaustion
```bash
# Check cluster resource usage
kubectl top nodes
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# Find resource-heavy pods
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.containers[].resources.requests.memory) | {namespace: .metadata.namespace, name: .metadata.name, memory: .spec.containers[].resources.requests.memory}'

# Check for resource quotas
kubectl get resourcequota --all-namespaces
kubectl describe resourcequota <quota-name>
```

### Node Pressure Conditions
```bash
# Check node conditions
kubectl describe nodes | grep -A 10 Conditions

# Memory pressure
kubectl get nodes -o custom-columns="NAME:.metadata.name,MEMORY:.status.conditions[?(@.type=='MemoryPressure')].status"

# Disk pressure
kubectl get nodes -o custom-columns="NAME:.metadata.name,DISK:.status.conditions[?(@.type=='DiskPressure')].status"

# PID pressure
kubectl get nodes -o custom-columns="NAME:.metadata.name,PID:.status.conditions[?(@.type=='PIDPressure')].status"
```

## Troubleshooting Scenarios

### Scenario 1: Node Becomes Unresponsive
1. SSH to affected node
2. Check system resources (CPU, memory, disk)
3. Review kubelet logs
4. Check container runtime status
5. Restart services if necessary
6. Re-join node to cluster if needed

### Scenario 2: Control Plane Component Fails
1. Identify failed component
2. Check static pod manifests
3. Review component logs
4. Verify certificates and connectivity
5. Restore from backup if necessary

### Scenario 3: Network Connectivity Loss
1. Test basic connectivity
2. Check CNI plugin status
3. Verify network policies
4. Test DNS resolution
5. Restart network components if needed

## Quick Diagnostic Commands

```bash
# Cluster health overview
kubectl get componentstatuses
kubectl get nodes
kubectl get pods --all-namespaces | grep -v Running

# System resource check
kubectl top nodes
kubectl describe nodes | grep -A 5 "System Info"

# Network diagnostics
kubectl get pods -n kube-system | grep -E "calico|flannel|weave|cilium"
kubectl get services --all-namespaces

# Certificate check
kubeadm certs check-expiration
```

## Learning Resources
- Troubleshooting Clusters: https://kubernetes.io/docs/tasks/debug/debug-cluster/
- Monitor Node Health: https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/
- Debug Services: https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/

## Success Criteria
- [ ] Can diagnose node issues systematically
- [ ] Can troubleshoot control plane components
- [ ] Can resolve network connectivity problems
- [ ] Can identify and fix etcd issues
- [ ] Can diagnose performance bottlenecks

## Notes Section
_Use this space to document infrastructure troubleshooting findings_