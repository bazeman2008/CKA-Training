# Week 7: Advanced RBAC & Security

## Overview
Master Kubernetes security including advanced RBAC, security policies, network security, and admission controllers.

## Daily Tasks

### Day 43: RBAC Advanced (1.5 hours)
- [ ] Configure ClusterRoles and ClusterRoleBindings
- [ ] Manage service account tokens
- [ ] Practice complex permission scenarios

### Day 44: Security Policies (1.5 hours)
- [ ] Implement Pod Security Standards
- [ ] Troubleshoot security contexts
- [ ] Practice secure workload deployment

### Day 45: Network Security Advanced (1.5 hours)
- [ ] Configure complex network policies
- [ ] Implement default deny policies
- [ ] Practice multi-namespace security

### Day 46: Admission Controllers (1.5 hours)
- [ ] Understand admission controller concepts
- [ ] Troubleshoot admission controller issues
- [ ] Study cluster security mechanisms

### Day 47: Monitoring & Logging (1.5 hours)
- [ ] Set up cluster-level logging
- [ ] Practice event analysis and correlation
- [ ] Master troubleshooting with logs

## Weekend Session (3 hours)
- [ ] Security configuration practice
- [ ] RBAC troubleshooting scenarios
- [ ] **Goal Check:** Configure security policies efficiently

## Advanced RBAC Configurations

### ClusterRole and ClusterRoleBinding

```yaml
# ClusterRole for pod management across all namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]

# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-manager-binding
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: pod-manager-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: pod-manager
  apiGroup: rbac.authorization.k8s.io
```

### Service Account Management

```bash
# Create service account
kubectl create serviceaccount app-sa

# Get service account token
kubectl get secret $(kubectl get sa app-sa -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d

# Use service account in pod
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: nginx
```

### Aggregated ClusterRoles

```yaml
# Base ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-base
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
rules:
- apiGroups: [""]
  resources: ["pods", "nodes"]
  verbs: ["get", "list"]

# Aggregated ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-aggregated
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: [] # Rules are automatically filled in by the controller
```

## Pod Security Standards

```yaml
# Pod Security Policy (deprecated, but still in some exams)
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'

# Pod Security Standards (newer approach)
# Namespace labels for Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

## Complex Network Policies

```yaml
# Multi-rule network policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: complex-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: frontend
    - podSelector:
        matchLabels:
          role: api
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 3306
  - to:
    - podSelector:
        matchLabels:
          app: dns
    ports:
    - protocol: UDP
      port: 53

# Default deny all traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## RBAC Troubleshooting

```bash
# Check user permissions
kubectl auth can-i create pods --as john
kubectl auth can-i delete nodes --as john

# Check service account permissions
kubectl auth can-i list pods --as system:serviceaccount:default:app-sa

# Get role/clusterrole details
kubectl describe role pod-reader
kubectl describe clusterrole view

# Check bindings
kubectl get rolebindings
kubectl get clusterrolebindings

# Audit RBAC
kubectl get roles,rolebindings,clusterroles,clusterrolebindings --all-namespaces
```

## Security Audit Commands

```bash
# Check pod security context
kubectl get pod <pod-name> -o jsonpath='{.spec.securityContext}'

# Check container security context
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].securityContext}'

# List pods running as root
kubectl get pods -o json | jq '.items[] | select(.spec.securityContext.runAsUser == 0 or .spec.containers[].securityContext.runAsUser == 0) | .metadata.name'

# Check service accounts
kubectl get serviceaccounts --all-namespaces

# View admission controllers
kube-apiserver -h | grep enable-admission-plugins
```

## Monitoring and Logging Setup

```yaml
# Simple logging DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      serviceAccount: fluentd
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Learning Resources
- RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Pod Security Standards: https://kubernetes.io/docs/concepts/security/pod-security-standards/
- Network Policies: https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Admission Controllers: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/

## Success Criteria
- [ ] Can configure complex RBAC scenarios
- [ ] Understand service account token management
- [ ] Can implement pod security policies/standards
- [ ] Can create multi-rule network policies
- [ ] Can troubleshoot security configurations

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_