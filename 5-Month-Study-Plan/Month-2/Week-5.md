# Week 5: Networking Fundamentals

## Overview
Master Kubernetes networking concepts including services, ingress, network policies, and DNS.

## Daily Tasks

### Day 29: Service Deep Dive (1.5 hours)
- [ ] Advanced service type configurations
- [ ] Manage Endpoints and EndpointSlices
- [ ] Practice service troubleshooting

### Day 30: Ingress Configuration (1.5 hours)
- [ ] Configure ingress controllers and resources
- [ ] Set up HTTP routing and TLS termination
- [ ] Practice exposing applications via ingress

### Day 31: Network Policies (1.5 hours)
- [ ] Control pod-to-pod communication
- [ ] Implement ingress and egress rules
- [ ] Practice network security and segmentation

### Day 32: DNS & Service Discovery (1.5 hours)
- [ ] Configure and troubleshoot CoreDNS
- [ ] Practice service discovery mechanisms
- [ ] Master DNS debugging techniques

### Day 33: CNI and Cluster Networking (1.5 hours)
- [ ] Configure network plugin (Calico/Flannel)
- [ ] Troubleshoot cluster-level networking
- [ ] Practice network connectivity testing

## Weekend Session (2.5 hours)
- [ ] End-to-end networking scenarios
- [ ] Network troubleshooting practice
- [ ] **Goal Check:** Diagnose network issues in under 15 minutes

## Networking Configurations

### Advanced Service Examples

```yaml
# Service with custom endpoints
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 10.10.10.10
  ports:
  - port: 80

# Headless service
apiVersion: v1
kind: Service
metadata:
  name: headless-svc
spec:
  clusterIP: None
  selector:
    app: stateful-app
  ports:
  - port: 80
    targetPort: 80
```

### Ingress Configuration

```yaml
# Basic Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
```

### Network Policy Examples

```yaml
# Deny all ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

# Allow specific pod-to-pod communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-to-db
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: webapp
    ports:
    - protocol: TCP
      port: 3306

# Egress control
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: limit-egress
spec:
  podSelector:
    matchLabels:
      app: restricted
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: allowed-namespace
  - to:
    - podSelector:
        matchLabels:
          app: dns
    ports:
    - protocol: UDP
      port: 53
```

## Network Troubleshooting Commands

```bash
# Service troubleshooting
kubectl get svc
kubectl get endpoints
kubectl describe svc <service-name>

# Test service connectivity
kubectl run test-pod --image=busybox --rm -it -- wget -qO- <service-name>:<port>

# DNS troubleshooting
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
kubectl logs -n kube-system -l k8s-app=kube-dns

# Network policy testing
kubectl run test-source --image=busybox --labels="app=source" -- sleep 3600
kubectl run test-target --image=nginx --labels="app=target"
kubectl exec test-source -- wget -qO- test-target

# Check CNI plugin
kubectl get pods -n kube-system | grep -E 'calico|flannel|weave'
kubectl logs -n kube-system <cni-pod>
```

## DNS Resolution Patterns

```bash
# Service DNS names
<service-name>.<namespace>.svc.cluster.local

# Pod DNS names (if using headless service)
<pod-name>.<service-name>.<namespace>.svc.cluster.local

# Test DNS resolution
kubectl exec <pod> -- nslookup <service-name>
kubectl exec <pod> -- dig <service-name>.default.svc.cluster.local
```

## Learning Resources
- Services: https://kubernetes.io/docs/concepts/services-networking/service/
- Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
- Network Policies: https://kubernetes.io/docs/concepts/services-networking/network-policies/
- DNS: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

## Success Criteria
- [ ] Can configure all service types and understand their use cases
- [ ] Can set up ingress for application routing
- [ ] Can implement network policies for security
- [ ] Can troubleshoot DNS and service discovery issues
- [ ] Understand CNI and cluster networking basics

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_