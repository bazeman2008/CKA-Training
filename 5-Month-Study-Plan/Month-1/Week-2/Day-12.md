# Day 12: ConfigMaps & Secrets

## Overview (1.5 hours)
Master application configuration management using ConfigMaps and Secrets, essential for workload management questions on the CKA exam.

## Why This Matters for the CKA Exam
- **Configuration management** appears in 15% workload questions
- **Secret handling** is critical for security scenarios
- **Environment injection** is commonly tested
- **Volume mounting** combines storage and configuration knowledge

## Task 1: Practice Configuration Management (45 minutes)

### What You'll Do
Learn to externalize application configuration using ConfigMaps and manage sensitive data with Secrets.

### ConfigMap Creation and Usage

#### 1. Creating ConfigMaps from Literals
```bash
# Basic ConfigMap creation
kubectl create configmap app-config --from-literal=database_url=postgresql://localhost:5432/app --from-literal=debug_mode=true

# Multiple key-value pairs
kubectl create configmap web-config \
  --from-literal=server_name=web-server \
  --from-literal=port=8080 \
  --from-literal=log_level=info \
  --from-literal=cache_enabled=true

# View ConfigMap contents
kubectl get configmap app-config -o yaml
kubectl describe configmap app-config
```

#### 2. Creating ConfigMaps from Files
```bash
# Create configuration files
echo "port=8080" > app.properties
echo "host=localhost" >> app.properties
echo "debug=true" >> app.properties

cat > nginx.conf << 'EOF'
server {
    listen 80;
    server_name localhost;
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
EOF

# Create ConfigMaps from files
kubectl create configmap app-properties --from-file=app.properties
kubectl create configmap nginx-config --from-file=nginx.conf

# Create ConfigMap from directory
mkdir config-dir
echo "key1=value1" > config-dir/config1.txt
echo "key2=value2" > config-dir/config2.txt
kubectl create configmap dir-config --from-file=config-dir/
```

#### 3. Creating ConfigMaps from YAML
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-config
data:
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
    player.initial-lives=3
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

```bash
# Apply ConfigMap
kubectl apply -f configmap.yaml
kubectl get configmap game-config -o yaml
```

### Using ConfigMaps in Pods

#### 1. Environment Variables from ConfigMap
```yaml
# pod-env-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'env | grep -E "(DATABASE|DEBUG)" && sleep 3600']
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: DEBUG_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: debug_mode
```

```bash
# Apply and test
kubectl apply -f pod-env-configmap.yaml
kubectl logs env-configmap-pod
kubectl exec env-configmap-pod -- env | grep -E "(DATABASE|DEBUG)"
```

#### 2. All ConfigMap Keys as Environment Variables
```yaml
# pod-envfrom-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-configmap-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'env && sleep 3600']
    envFrom:
    - configMapRef:
        name: web-config
```

```bash
# Apply and verify all variables loaded
kubectl apply -f pod-envfrom-configmap.yaml
kubectl logs envfrom-configmap-pod | grep -E "(server_name|port|log_level)"
```

#### 3. ConfigMap as Volume Mount
```yaml
# pod-volume-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-configmap-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: app-properties
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

```bash
# Apply and verify file mounting
kubectl apply -f pod-volume-configmap.yaml
kubectl exec volume-configmap-pod -- ls -la /etc/config
kubectl exec volume-configmap-pod -- cat /etc/config/app.properties
kubectl exec volume-configmap-pod -- ls -la /etc/nginx/conf.d
```

### Secret Creation and Usage

#### 1. Creating Secrets from Literals
```bash
# Basic Secret creation
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123 \
  --from-literal=api-key=abc123xyz

# Database credentials
kubectl create secret generic db-credentials \
  --from-literal=username=dbuser \
  --from-literal=password=dbpass123 \
  --from-literal=host=database.example.com

# View Secret (values are base64 encoded)
kubectl get secret app-secret -o yaml
kubectl describe secret app-secret
```

#### 2. Creating Secrets from Files
```bash
# Create credential files
echo -n 'admin' > username.txt
echo -n 'secret123' > password.txt

# Create Secret from files
kubectl create secret generic file-secret --from-file=username.txt --from-file=password.txt

# TLS Secret
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=example.com"
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
```

#### 3. Docker Registry Secrets
```bash
# Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# Use in pod spec
```yaml
spec:
  imagePullSecrets:
  - name: regcred
```

### Using Secrets in Pods

#### 1. Environment Variables from Secret
```yaml
# pod-env-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-secret-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo "Username: $USERNAME, Password: $PASSWORD" && sleep 3600']
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

#### 2. Secret as Volume Mount
```yaml
# pod-volume-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-secret-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'cat /etc/secret/* && sleep 3600']
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

```bash
# Apply and verify secret mounting
kubectl apply -f pod-volume-secret.yaml
kubectl exec volume-secret-pod -- ls -la /etc/secret
kubectl exec volume-secret-pod -- cat /etc/secret/username
```

## Task 2: Mount Configs and Secrets in Pods (30 minutes)

### What You'll Do
Practice complex mounting scenarios combining ConfigMaps and Secrets.

### Advanced Mounting Scenarios

#### 1. Multiple ConfigMaps and Secrets
```yaml
# complex-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: complex-config-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    volumeMounts:
    - name: app-config-volume
      mountPath: /etc/app-config
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: app-config-volume
    configMap:
      name: app-properties
  - name: secret-volume
    secret:
      secretName: app-secret
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

#### 2. Selective Key Mounting
```yaml
# selective-mount-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: selective-mount-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: selective-config
      mountPath: /etc/config
  volumes:
  - name: selective-config
    configMap:
      name: game-config
      items:
      - key: game.properties
        path: game.conf
      - key: ui.properties
        path: ui.conf
```

#### 3. File Permissions and Ownership
```yaml
# permissions-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: permissions-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
      defaultMode: 0400  # Read-only for owner
      items:
      - key: username
        path: user
        mode: 0444  # Read-only for all
```

### Deployment with ConfigMaps and Secrets

#### 1. Deployment Using Configuration
```yaml
# webapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
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
        image: nginx
        env:
        - name: ENV
          valueFrom:
            configMapKeyRef:
              name: web-config
              key: log_level
        envFrom:
        - configMapRef:
            name: web-config
        - secretRef:
            name: app-secret
        volumeMounts:
        - name: config-volume
          mountPath: /etc/app
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
      - name: secret-volume
        secret:
          secretName: app-secret
```

## Task 3: Troubleshoot Configuration Issues (15 minutes)

### What You'll Do
Learn to debug common ConfigMap and Secret issues in CKA scenarios.

### Common Configuration Issues

#### 1. ConfigMap/Secret Not Found
```bash
# Problem: Pod references non-existent ConfigMap/Secret
kubectl describe pod failing-pod | grep -A 5 "Warning"

# Check if ConfigMap/Secret exists
kubectl get configmap
kubectl get secret

# Create missing configuration
kubectl create configmap missing-config --from-literal=key=value
```

#### 2. Key Not Found in ConfigMap/Secret
```bash
# Problem: Referenced key doesn't exist
kubectl get configmap app-config -o jsonpath='{.data}'
kubectl get secret app-secret -o jsonpath='{.data}'

# Add missing key
kubectl patch configmap app-config -p '{"data":{"missing-key":"value"}}'
```

#### 3. Mount Path Conflicts
```bash
# Problem: Multiple volumes mounted to same path
kubectl describe pod conflicted-pod | grep -A 10 "Mounts"

# Check volume definitions
kubectl get pod conflicted-pod -o yaml | grep -A 20 "volumes"
```

#### 4. Permission Issues
```bash
# Problem: Cannot read mounted secret files
kubectl exec pod -- ls -la /etc/secrets
kubectl exec pod -- cat /etc/secrets/password

# Check secret mode and ownership
kubectl get secret app-secret -o yaml | grep -A 5 "mode"
```

### CKA Debugging Workflow

#### 1. Configuration Troubleshooting Steps
```bash
# 1. Check pod status and events
kubectl describe pod <pod-name>

# 2. Verify ConfigMap/Secret exists
kubectl get configmap <config-name>
kubectl get secret <secret-name>

# 3. Check key existence
kubectl get configmap <config-name> -o jsonpath='{.data}'

# 4. Verify environment variables
kubectl exec <pod-name> -- env | grep <expected-var>

# 5. Check mounted files
kubectl exec <pod-name> -- ls -la <mount-path>
kubectl exec <pod-name> -- cat <mount-path>/<file>
```

#### 2. Configuration Update Scenarios
```bash
# Update ConfigMap
kubectl patch configmap app-config -p '{"data":{"new-key":"new-value"}}'

# Update Secret
kubectl patch secret app-secret -p '{"data":{"new-key":"bmV3LXZhbHVl"}}'  # base64 encoded

# Restart pods to pick up changes
kubectl rollout restart deployment webapp
```

## Daily Goals Checklist

- [ ] Master ConfigMap creation from literals, files, and YAML
- [ ] Can create and manage Secrets securely
- [ ] Practice mounting ConfigMaps and Secrets as volumes
- [ ] Understand environment variable injection methods
- [ ] Can troubleshoot configuration issues effectively
- [ ] Ready for weekend practice session

## Weekend Session Preparation

### What You'll Practice This Weekend
- Complete application stack deployment
- Multi-tier application with configuration
- Speed drills for basic object creation
- Integration of all Week 2 concepts

### Skills to Demonstrate
- Deploy applications in under 10 minutes
- Troubleshoot configuration issues quickly
- Use imperative commands effectively
- Apply declarative configurations correctly

## Study Notes Section

**Configuration Quick Commands:**
```bash
# ConfigMap creation
kubectl create configmap <name> --from-literal=key=value
kubectl create configmap <name> --from-file=file.txt

# Secret creation
kubectl create secret generic <name> --from-literal=key=value
kubectl create secret docker-registry <name> --docker-server=...

# Debugging
kubectl describe pod <name> | grep -A 10 "Events"
kubectl exec <pod> -- env | grep <var>
kubectl exec <pod> -- cat <mount-path>/<file>
```

**Personal Configuration Patterns:**
_Record preferred ConfigMap/Secret usage patterns and troubleshooting steps_