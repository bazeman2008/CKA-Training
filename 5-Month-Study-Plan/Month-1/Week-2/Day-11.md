# Day 11: Services Introduction

## Overview (1.5 hours)
Master Kubernetes service types and networking fundamentals that comprise 20% of the CKA exam.

## Why This Matters for the CKA Exam
- **Service networking** accounts for 20% of the exam
- **Service troubleshooting** is critical for application connectivity
- **Service types** determine application accessibility patterns
- **Endpoint management** enables service debugging

## Task 1: Configure Service Types (45 minutes)

### What You'll Do
Master all Kubernetes service types and understand when to use each.

### ClusterIP Service (Default)

#### 1. Basic ClusterIP Service
```bash
# Create deployment first
kubectl create deployment nginx --image=nginx

# Expose with ClusterIP (default)
kubectl expose deployment nginx --port=80 --target-port=80

# Verify service
kubectl get service nginx
kubectl describe service nginx
```

#### 2. Custom ClusterIP Service
```yaml
# clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: ClusterIP
  selector:
    app: webapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  - name: https
    port: 443
    targetPort: 8443
    protocol: TCP
```

```bash
# Apply and test
kubectl apply -f clusterip-service.yaml
kubectl get service webapp-service -o wide

# Test internal connectivity
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- webapp-service:80
```

### NodePort Service

#### 1. Basic NodePort Service
```bash
# Expose deployment with NodePort
kubectl expose deployment nginx --port=80 --type=NodePort

# Check assigned NodePort
kubectl get service nginx -o jsonpath='{.spec.ports[0].nodePort}'

# Test external access
curl http://<node-ip>:<nodeport>
```

#### 2. Custom NodePort Service
```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Optional: specify nodePort
```

```bash
# Apply and verify external access
kubectl apply -f nodeport-service.yaml
curl http://$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}'):30080
```

### LoadBalancer Service

#### 1. LoadBalancer Service (Cloud Provider)
```yaml
# loadbalancer-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Apply and check external IP
kubectl apply -f loadbalancer-service.yaml
kubectl get service loadbalancer-service -w  # Wait for external IP

# Note: LoadBalancer requires cloud provider integration
# In local clusters, it will remain in Pending state
```

### ExternalName Service

#### 1. ExternalName for External Services
```yaml
# externalname-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
  ports:
  - port: 3306
```

```bash
# Apply and test DNS resolution
kubectl apply -f externalname-service.yaml
kubectl run test-pod --image=busybox --rm -it --restart=Never -- nslookup external-db
```

### CKA Exam Service Scenarios

#### Scenario 1: Service Not Accessible
```bash
# Problem: Cannot access service
# Troubleshooting steps:
kubectl get service <service-name>
kubectl get endpoints <service-name>
kubectl describe service <service-name>

# Check if pods match selector
kubectl get pods --show-labels
kubectl get pods -l <selector-from-service>
```

#### Scenario 2: NodePort Not Working
```bash
# Problem: NodePort not accessible from outside
# Check firewall rules on nodes
sudo iptables -L | grep <nodeport>
sudo firewall-cmd --list-ports | grep <nodeport>

# To open NodePort if needed:
# sudo firewall-cmd --permanent --add-port=<nodeport>/tcp
# sudo firewall-cmd --reload

# Verify service configuration
kubectl get service <service-name> -o yaml | grep nodePort
```

## Task 2: Practice Service Discovery and Endpoints (30 minutes)

### What You'll Do
Understand how services discover pods and manage endpoints automatically.

### Service Discovery Mechanisms

#### 1. DNS-Based Discovery
```bash
# Create service and test DNS resolution
kubectl create deployment webapp --image=nginx
kubectl expose deployment webapp --port=80

# Test DNS from another pod
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup webapp

# Full DNS names
# <service>.<namespace>.svc.cluster.local
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup webapp.default.svc.cluster.local
```

#### 2. Environment Variable Discovery
```bash
# Kubernetes injects service info as environment variables
kubectl run env-test --image=busybox --restart=Never -- env | grep WEBAPP

# Service discovery variables:
# WEBAPP_SERVICE_HOST
# WEBAPP_SERVICE_PORT
# WEBAPP_PORT_80_TCP_ADDR
# WEBAPP_PORT_80_TCP_PORT
```

### Endpoint Management

#### 1. Understanding Endpoints
```bash
# View service endpoints
kubectl get endpoints
kubectl get endpoints webapp -o yaml

# Endpoints are automatically created/updated
kubectl describe endpoints webapp

# Check endpoint addresses
kubectl get endpoints webapp -o jsonpath='{.subsets[*].addresses[*].ip}'
```

#### 2. Manual Endpoint Creation
```yaml
# manual-endpoint.yaml
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
  - ip: 192.168.1.100  # External server IP
  ports:
  - port: 80
```

```bash
# Apply and test external service
kubectl apply -f manual-endpoint.yaml
kubectl get endpoints external-service
```

#### 3. Headless Services
```yaml
# headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None  # Makes service headless
  selector:
    app: stateful-app
  ports:
  - port: 80
```

```bash
# Test headless service DNS resolution
kubectl apply -f headless-service.yaml
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup headless-service

# Returns individual pod IPs instead of service IP
```

### Service Debugging

#### 1. Service Connectivity Testing
```bash
# Test service from inside cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>:<port>

# Test service from specific namespace
kubectl run test-pod -n <namespace> --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>.<target-namespace>:80
```

#### 2. Endpoint Troubleshooting
```bash
# Check if endpoints exist
kubectl get endpoints <service-name>

# If no endpoints, check:
# 1. Pod selector matches
kubectl get pods --show-labels
kubectl get service <service-name> -o jsonpath='{.spec.selector}'

# 2. Pods are ready
kubectl get pods -l <selector> -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}'

# 3. Container ports match targetPort
kubectl get pods -l <selector> -o jsonpath='{.items[*].spec.containers[*].ports[*].containerPort}'
```

## Task 3: Speed Practice - Expose Applications Quickly (15 minutes)

### What You'll Do
Build speed for quickly exposing applications during the CKA exam.

### Speed Drill 1: Basic Service Creation (5 minutes)
```bash
# Target: Complete in under 2 minutes
kubectl create deployment quick-app --image=nginx
kubectl expose deployment quick-app --port=80 --type=ClusterIP
kubectl expose deployment quick-app --port=80 --type=NodePort --name=quick-app-np
kubectl get services
kubectl get endpoints

# Test connectivity
kubectl run test --image=busybox --rm -it --restart=Never -- wget -qO- quick-app:80

# Cleanup
kubectl delete deployment quick-app
kubectl delete service quick-app quick-app-np
```

### Speed Drill 2: Multi-Port Service (5 minutes)
```bash
# Target: Complete in under 3 minutes
# Create service with multiple ports
kubectl create deployment multi-port --image=nginx
kubectl create service clusterip multi-port --tcp=80:80,443:443

# Verify multi-port service
kubectl get service multi-port -o yaml | grep -A 10 ports

# Test both ports
kubectl run test --image=busybox --rm -it --restart=Never -- telnet multi-port 80
kubectl run test --image=busybox --rm -it --restart=Never -- telnet multi-port 443

# Cleanup
kubectl delete deployment multi-port
kubectl delete service multi-port
```

### Speed Drill 3: External Service Mapping (5 minutes)
```bash
# Target: Complete in under 2 minutes
# Create ExternalName service
kubectl create service externalname google --external-name=google.com

# Test external service resolution
kubectl run test --image=busybox --rm -it --restart=Never -- nslookup google

# Create manual endpoint service
kubectl create service clusterip manual-svc --tcp=80:80
kubectl create -f - <<EOF
apiVersion: v1
kind: Endpoints
metadata:
  name: manual-svc
subsets:
- addresses:
  - ip: 8.8.8.8
  ports:
  - port: 80
EOF

# Cleanup
kubectl delete service google manual-svc
kubectl delete endpoints manual-svc
```

## Common CKA Service Issues

### Issue 1: Service Selector Mismatch
```bash
# Problem: Service has no endpoints
kubectl get endpoints <service-name>  # Shows no addresses

# Solution: Check selector matches pod labels
kubectl get service <service-name> -o jsonpath='{.spec.selector}'
kubectl get pods --show-labels | grep <expected-labels>

# Fix: Update service selector or pod labels
kubectl patch service <service-name> -p '{"spec":{"selector":{"app":"correct-label"}}}'
```

### Issue 2: Port Configuration Problems
```bash
# Problem: Connection refused when accessing service
kubectl describe service <service-name> | grep Port

# Check container ports
kubectl get pods -l <selector> -o jsonpath='{.items[*].spec.containers[*].ports}'

# Fix: Update targetPort to match container port
kubectl patch service <service-name> -p '{"spec":{"ports":[{"port":80,"targetPort":8080}]}}'
```

### Issue 3: DNS Resolution Failures
```bash
# Problem: Cannot resolve service name
kubectl run debug --image=busybox --rm -it --restart=Never -- nslookup <service-name>

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Verify service exists
kubectl get service <service-name>
```

## Daily Goals Checklist

- [ ] Master all service types and their use cases
- [ ] Understand service discovery mechanisms (DNS, env vars)
- [ ] Can troubleshoot service connectivity issues
- [ ] Practice endpoint management and debugging
- [ ] Complete speed drills within target times
- [ ] Ready for configuration management

## Next Day Preparation

### What You'll Need for Day 12
- Service creation and management skills ✓
- Service discovery understanding ✓
- Troubleshooting capabilities ✓
- Speed and efficiency ✓

### Coming Up: ConfigMaps & Secrets
Day 12 will cover application configuration management using ConfigMaps and Secrets.

## Study Notes Section

**Service Quick Commands:**
```bash
# Service creation
kubectl expose deployment <name> --port=<port> --type=<type>
kubectl create service <type> <name> --tcp=<port>:<targetport>

# Service debugging
kubectl get service <name>
kubectl get endpoints <name>
kubectl describe service <name>

# Connectivity testing
kubectl run test --image=busybox --rm -it -- wget -qO- <service>:<port>
```

**Personal Service Patterns:**
_Record preferred service configurations and troubleshooting steps_