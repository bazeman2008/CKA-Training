# Week 6: Cluster Administration

## Overview
Master critical cluster administration tasks including node management, upgrades, etcd backup/restore, and certificate management.

## Daily Tasks

### Day 36: Node Management (1.5 hours)
- [ ] Practice drain, cordon, and uncordon procedures
- [ ] Troubleshoot node issues
- [ ] Master safe node maintenance practices

### Day 37: Cluster Upgrades (1.5 hours)
- [ ] Practice kubeadm upgrade for control plane
- [ ] Upgrade worker node procedures
- [ ] Master version upgrade methodology

### Day 38: etcd Backup & Restore (1.5 hours)
- [ ] **CRITICAL:** Practice etcd backup with etcdctl
- [ ] Practice cluster state restoration
- [ ] Speed drill: backup/restore procedures

### Day 39: Certificate Management (1.5 hours)
- [ ] Check certificate expiration
- [ ] Practice certificate renewal with kubeadm
- [ ] Troubleshoot TLS configuration

### Day 40: Control Plane Troubleshooting (1.5 hours)
- [ ] Debug API server issues
- [ ] Troubleshoot scheduler and controller-manager
- [ ] Practice static pod troubleshooting

## Weekend Session (3 hours)
- [ ] Complete cluster maintenance scenarios
- [ ] etcd backup/restore speed drills
- [ ] **Goal Check:** Complete backup/restore in under 10 minutes

## Node Management Commands

```bash
# Cordon node (prevent new pods)
kubectl cordon <node-name>

# Drain node (evict pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon node (allow scheduling)
kubectl uncordon <node-name>

# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Node troubleshooting
journalctl -u kubelet
systemctl status kubelet
```

## Cluster Upgrade Process

```bash
# Check current version
kubectl get nodes
kubeadm version

# Upgrade control plane
# 1. Update kubeadm
apt-get update && apt-get install -y kubeadm=1.XX.X-00

# 2. Check upgrade plan
kubeadm upgrade plan

# 3. Apply upgrade
kubeadm upgrade apply v1.XX.X

# 4. Upgrade kubelet and kubectl
apt-get update && apt-get install -y kubelet=1.XX.X-00 kubectl=1.XX.X-00
systemctl daemon-reload
systemctl restart kubelet

# Upgrade worker nodes
# 1. Drain node from control plane
kubectl drain <node-name> --ignore-daemonsets

# 2. On worker node, upgrade kubeadm
apt-get update && apt-get install -y kubeadm=1.XX.X-00

# 3. Upgrade node configuration
kubeadm upgrade node

# 4. Upgrade kubelet and kubectl
apt-get update && apt-get install -y kubelet=1.XX.X-00 kubectl=1.XX.X-00
systemctl daemon-reload
systemctl restart kubelet

# 5. Uncordon node from control plane
kubectl uncordon <node-name>
```

## etcd Backup and Restore

```bash
# Set etcdctl API version
export ETCDCTL_API=3

# etcd backup
etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
etcdctl snapshot status /backup/etcd-snapshot.db

# etcd restore
# 1. Stop kube-apiserver
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp

# 2. Restore etcd
etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore \
  --initial-cluster=master=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380 \
  --name=master \
  --initial-cluster-token=etcd-cluster-1

# 3. Update etcd manifest to use new data directory
vim /etc/kubernetes/manifests/etcd.yaml
# Change --data-dir to /var/lib/etcd-restore

# 4. Move kube-apiserver back
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

## Certificate Management

```bash
# Check certificate expiration
kubeadm certs check-expiration

# Renew all certificates
kubeadm certs renew all

# Renew specific certificate
kubeadm certs renew apiserver

# Manual certificate location
ls /etc/kubernetes/pki/

# View certificate details
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

## Control Plane Troubleshooting

```bash
# Check control plane pods
kubectl get pods -n kube-system

# Check static pod manifests
ls /etc/kubernetes/manifests/

# API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>
journalctl -u kube-apiserver

# Scheduler logs
kubectl logs -n kube-system kube-scheduler-<node-name>

# Controller manager logs
kubectl logs -n kube-system kube-controller-manager-<node-name>

# kubelet logs
journalctl -u kubelet -f

# Check component status
kubectl get componentstatuses
```

## Troubleshooting Checklist

### Node Not Ready
1. Check kubelet status: `systemctl status kubelet`
2. Check kubelet logs: `journalctl -u kubelet`
3. Check container runtime: `systemctl status containerd`
4. Check disk space: `df -h`
5. Check memory: `free -h`

### Control Plane Issues
1. Check static pod manifests
2. Verify certificates are valid
3. Check etcd health
4. Verify network connectivity
5. Check resource availability

## Learning Resources
- Cluster Administration: https://kubernetes.io/docs/tasks/administer-cluster/
- kubeadm Upgrade: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- etcd Admin: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- Certificate Management: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

## Success Criteria
- [ ] Can safely drain and maintain nodes
- [ ] Can perform cluster upgrades without downtime
- [ ] Can backup and restore etcd quickly (< 10 minutes)
- [ ] Can manage and renew certificates
- [ ] Can troubleshoot control plane components

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_