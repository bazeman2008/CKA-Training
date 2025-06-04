# Day 23: Volume Types

## Overview (1.5 hours)
Master different Kubernetes volume types including EmptyDir, HostPath, ConfigMap/Secret volumes, and understand their appropriate use cases for various storage scenarios.

## Why This Matters for the CKA Exam
- **Volume types appear in 15-20% of CKA questions**
- **EmptyDir and HostPath** are common for exam scenarios
- **ConfigMap/Secret volumes** frequently used for configuration
- **Volume troubleshooting** requires understanding type-specific behavior
- **Storage patterns** critical for real-world applications

## Task 1: EmptyDir Volume Usage (30 minutes)

### What You'll Do
Master EmptyDir volumes for temporary storage, shared data between containers, and memory-backed storage.

### Step-by-Step Instructions

#### 1. Basic EmptyDir Volume
```bash
# Create pod with EmptyDir volume
cat > emptydir-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
  labels:
    app: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "$(date): Writing data to shared volume" >> /shared/data.log
        sleep 10
      done
    volumeMounts:
    - name: shared-data
      mountPath: /shared
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  - name: reader
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        if [ -f /shared/data.log ]; then
          echo "=== Reading shared data ==="
          tail -5 /shared/data.log
          echo "=========================="
        fi
        sleep 15
      done
    volumeMounts:
    - name: shared-data
      mountPath: /shared
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: shared-data
    emptyDir: {}
EOF

# Apply and verify
kubectl apply -f emptydir-pod.yaml
kubectl logs emptydir-demo -c writer --tail=5
kubectl logs emptydir-demo -c reader --tail=5
```

#### 2. Memory-backed EmptyDir
```bash
# Create EmptyDir with memory backing
cat > memory-emptydir.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: memory-volume-demo
spec:
  containers:
  - name: memory-app
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: memory-storage
      mountPath: /tmp/memory
    - name: nginx-cache
      mountPath: /var/cache/nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
  volumes:
  - name: memory-storage
    emptyDir:
      medium: Memory
      sizeLimit: "100Mi"
  - name: nginx-cache
    emptyDir:
      sizeLimit: "200Mi"
EOF

# Apply and test memory volume
kubectl apply -f memory-emptydir.yaml
kubectl exec memory-volume-demo -- df -h | grep tmpfs
kubectl exec memory-volume-demo -- sh -c "echo 'test data' > /tmp/memory/test.txt && cat /tmp/memory/test.txt"
```

#### 3. EmptyDir for Init Container Data Sharing
```bash
# Create pod with init container using EmptyDir
cat > init-emptydir.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: init-shared-demo
spec:
  initContainers:
  - name: setup
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Initializing application data..."
      mkdir -p /shared/config /shared/data
      echo "server_name=app-server" > /shared/config/app.conf
      echo "debug=true" >> /shared/config/app.conf
      echo "port=8080" >> /shared/config/app.conf
      echo "Initialization complete" > /shared/data/init.status
    volumeMounts:
    - name: app-storage
      mountPath: /shared
  containers:
  - name: main-app
    image: nginx
    command:
    - sh
    - -c
    - |
      echo "Reading initialization data..."
      cat /app/config/app.conf
      cat /app/data/init.status
      nginx -g 'daemon off;'
    volumeMounts:
    - name: app-storage
      mountPath: /app
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  volumes:
  - name: app-storage
    emptyDir: {}
EOF

# Apply and verify init container data sharing
kubectl apply -f init-emptydir.yaml
kubectl logs init-shared-demo -c setup
kubectl exec init-shared-demo -- cat /app/config/app.conf
kubectl exec init-shared-demo -- cat /app/data/init.status
```

### CKA Exam Connection
- **Temporary Storage**: EmptyDir for scratch space and caches
- **Container Communication**: Sharing data between containers
- **Performance**: Memory-backed storage for high-speed access
- **Lifecycle Management**: Understanding EmptyDir cleanup behavior

### Validation Commands
```bash
# Verify EmptyDir functionality
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"
kubectl describe pod emptydir-demo | grep -A 5 "Volumes:"

# Check volume mounts
kubectl exec emptydir-demo -c writer -- ls -la /shared
kubectl exec memory-volume-demo -- mount | grep tmpfs
```

## Task 2: HostPath Volume Implementation (25 minutes)

### What You'll Do
Configure HostPath volumes for accessing host filesystem, system monitoring, and log collection scenarios.

### Step-by-Step Instructions

#### 1. Basic HostPath Configuration
```bash
# Create directory on host first
sudo mkdir -p /var/hostpath-data
sudo chmod 755 /var/hostpath-data
echo "Host data content" | sudo tee /var/hostpath-data/host-file.txt

# Create pod with HostPath volume
cat > hostpath-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: host-accessor
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Accessing host directory..."
      ls -la /host-data/
      cat /host-data/host-file.txt
      echo "Pod data: $(date)" >> /host-data/pod-data.txt
      while true; do sleep 30; done
    volumeMounts:
    - name: host-storage
      mountPath: /host-data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: host-storage
    hostPath:
      path: /var/hostpath-data
      type: Directory
EOF

# Apply and test host access
kubectl apply -f hostpath-pod.yaml
kubectl exec hostpath-demo -- ls -la /host-data/
kubectl exec hostpath-demo -- cat /host-data/host-file.txt
sudo cat /var/hostpath-data/pod-data.txt
```

#### 2. HostPath for System Monitoring
```bash
# Create system monitoring pod
cat > system-monitor.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: system-monitor
  labels:
    app: monitoring
spec:
  containers:
  - name: monitor
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "=== System Info $(date) ==="
        echo "Load Average:"
        cat /host/proc/loadavg
        echo "Memory Info:"
        head -3 /host/proc/meminfo
        echo "Disk Usage:"
        df -h /host/root
        echo "Running Processes:"
        ps aux | head -5
        echo "=========================="
        sleep 60
      done
    volumeMounts:
    - name: proc
      mountPath: /host/proc
      readOnly: true
    - name: root-fs
      mountPath: /host/root
      readOnly: true
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
    securityContext:
      runAsUser: 0
  volumes:
  - name: proc
    hostPath:
      path: /proc
      type: Directory
  - name: root-fs
    hostPath:
      path: /
      type: Directory
  hostNetwork: true
  hostPID: true
EOF

# Apply and check system monitoring
kubectl apply -f system-monitor.yaml
kubectl logs system-monitor --tail=20
```

#### 3. HostPath Types and Security
```bash
# Create examples of different HostPath types
cat > hostpath-types.yaml << 'EOF'
---
# DirectoryOrCreate - creates directory if it doesn't exist
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-dir-create
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'Created directory' > /data/created.txt; sleep 300"]
    volumeMounts:
    - name: storage
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: storage
    hostPath:
      path: /var/auto-created-dir
      type: DirectoryOrCreate
---
# FileOrCreate - creates file if it doesn't exist
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-file-create
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /config/app.conf; sleep 300"]
    volumeMounts:
    - name: config-file
      mountPath: /config/app.conf
      subPath: app.conf
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: config-file
    hostPath:
      path: /var/app-config.conf
      type: FileOrCreate
EOF

# Apply and verify HostPath types
kubectl apply -f hostpath-types.yaml
sudo ls -la /var/auto-created-dir/
sudo ls -la /var/app-config.conf
```

#### 4. HostPath Security Considerations
```bash
# Create restricted HostPath example
cat > secure-hostpath.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secure-hostpath
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "ls -la /logs/; touch /logs/app.log; sleep 300"]
    volumeMounts:
    - name: log-storage
      mountPath: /logs
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
  volumes:
  - name: log-storage
    hostPath:
      path: /var/app-logs
      type: DirectoryOrCreate
EOF

# Create directory with proper permissions
sudo mkdir -p /var/app-logs
sudo chown 1000:1000 /var/app-logs
sudo chmod 755 /var/app-logs

# Apply and test security
kubectl apply -f secure-hostpath.yaml
kubectl logs secure-hostpath
```

### CKA Exam Connection
- **Host Integration**: Accessing host filesystem and resources
- **System Services**: Running system-level monitoring and logging
- **Security Awareness**: Understanding HostPath security implications
- **Troubleshooting**: Debugging node-level issues

### Validation Commands
```bash
# Verify HostPath mounts
kubectl describe pod hostpath-demo | grep -A 10 "Mounts:"
sudo ls -la /var/hostpath-data/

# Check security contexts
kubectl get pod secure-hostpath -o jsonpath='{.spec.securityContext}'
kubectl exec secure-hostpath -- id
```

## Task 3: ConfigMap and Secret Volumes (35 minutes)

### What You'll Do
Implement ConfigMap and Secret volumes for configuration management and secure data storage.

### Step-by-Step Instructions

#### 1. ConfigMap Volume Configuration
```bash
# Create ConfigMaps for different use cases
kubectl create configmap app-config \
  --from-literal=database.host=db-server \
  --from-literal=database.port=5432 \
  --from-literal=debug=true \
  --from-literal=log.level=info

kubectl create configmap nginx-config \
  --from-file=- <<EOF
server {
    listen 8080;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
    
    location /health {
        return 200 'healthy';
        add_header Content-Type text/plain;
    }
}
EOF

# Create pod using ConfigMap volumes
cat > configmap-volumes.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: app-config-volume
      mountPath: /etc/config
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d
      readOnly: true
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  volumes:
  - name: app-config-volume
    configMap:
      name: app-config
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx-config
        path: default.conf
EOF

# Apply and test ConfigMap volumes
kubectl apply -f configmap-volumes.yaml
kubectl exec configmap-demo -- ls -la /etc/config/
kubectl exec configmap-demo -- cat /etc/config/database.host
kubectl exec configmap-demo -- cat /etc/nginx/conf.d/default.conf
```

#### 2. Secret Volume Implementation
```bash
# Create Secrets for sensitive data
kubectl create secret generic database-secret \
  --from-literal=username=admin \
  --from-literal=password=supersecret \
  --from-literal=connection-string=postgresql://admin:supersecret@db:5432/app

kubectl create secret tls web-tls \
  --cert=/dev/null \
  --key=/dev/null \
  --dry-run=client -o yaml > web-tls-secret.yaml

# Create proper TLS secret for demo
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.com/O=example.com"

kubectl create secret tls web-tls \
  --cert=tls.crt --key=tls.key

# Create pod using Secret volumes
cat > secret-volumes.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Checking database credentials..."
      echo "Username: $(cat /etc/secrets/username)"
      echo "Password length: $(cat /etc/secrets/password | wc -c)"
      echo "Connection string available: $(ls -la /etc/secrets/connection-string)"
      echo "TLS certificate info:"
      ls -la /etc/tls/
      openssl x509 -in /etc/tls/tls.crt -text -noout | grep Subject:
      while true; do sleep 30; done
    volumeMounts:
    - name: database-secrets
      mountPath: /etc/secrets
      readOnly: true
    - name: tls-certs
      mountPath: /etc/tls
      readOnly: true
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: database-secrets
    secret:
      secretName: database-secret
      defaultMode: 0400
  - name: tls-certs
    secret:
      secretName: web-tls
      items:
      - key: tls.crt
        path: tls.crt
        mode: 0444
      - key: tls.key
        path: tls.key
        mode: 0400
EOF

# Apply and test Secret volumes
kubectl apply -f secret-volumes.yaml
kubectl exec secret-demo -- ls -la /etc/secrets/
kubectl exec secret-demo -- ls -la /etc/tls/
kubectl logs secret-demo
```

#### 3. ConfigMap and Secret Updates
```bash
# Test ConfigMap volume updates
kubectl patch configmap app-config --patch='{"data":{"debug":"false","new.feature":"enabled"}}'

# Check if volume is updated (may take up to 60 seconds)
sleep 60
kubectl exec configmap-demo -- cat /etc/config/debug
kubectl exec configmap-demo -- cat /etc/config/new.feature

# Create immutable ConfigMap
cat > immutable-config.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
data:
  app.version: "1.0.0"
  environment: "production"
immutable: true
EOF

kubectl apply -f immutable-config.yaml

# Try to update immutable ConfigMap (should fail)
kubectl patch configmap immutable-config --patch='{"data":{"app.version":"2.0.0"}}' || echo "Update failed as expected"
```

#### 4. Advanced Volume Configuration
```bash
# Create pod with mixed volume types
cat > mixed-volumes.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: mixed-volumes-demo
spec:
  containers:
  - name: web-app
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: /etc/app-config
    - name: secret-volume
      mountPath: /etc/app-secrets
      readOnly: true
    - name: temp-volume
      mountPath: /tmp/app-temp
    - name: logs-volume
      mountPath: /var/log/app
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
    env:
    - name: CONFIG_PATH
      value: "/etc/app-config"
    - name: SECRETS_PATH
      value: "/etc/app-secrets"
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: database-secret
  - name: temp-volume
    emptyDir:
      medium: Memory
      sizeLimit: "100Mi"
  - name: logs-volume
    hostPath:
      path: /var/log/app-logs
      type: DirectoryOrCreate
EOF

# Apply and verify mixed volumes
kubectl apply -f mixed-volumes.yaml
kubectl exec mixed-volumes-demo -- df -h
kubectl exec mixed-volumes-demo -- mount | grep -E "(tmpfs|app-config|app-secrets)"
```

### CKA Exam Connection
- **Configuration Management**: Separating config from container images
- **Security**: Proper handling of sensitive data
- **Updates**: Understanding volume update mechanisms
- **Integration**: Combining different volume types effectively

### Validation Commands
```bash
# Verify ConfigMap and Secret volumes
kubectl describe pod configmap-demo | grep -A 10 "Volumes:"
kubectl describe pod secret-demo | grep -A 10 "Volumes:"

# Check volume content and permissions
kubectl exec secret-demo -- ls -la /etc/secrets/ /etc/tls/
kubectl exec configmap-demo -- ls -la /etc/config/
```

## Task 4: Volume Troubleshooting and Best Practices (20 minutes)

### What You'll Do
Practice troubleshooting common volume issues and understand best practices for volume usage.

### Step-by-Step Instructions

#### 1. Volume Mount Troubleshooting
```bash
# Create problematic volume configurations
cat > volume-problems.yaml << 'EOF'
---
# Problem 1: Nonexistent ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: missing-configmap
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config
      mountPath: /etc/config
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  volumes:
  - name: config
    configMap:
      name: nonexistent-config
---
# Problem 2: Permission issues
apiVersion: v1
kind: Pod
metadata:
  name: permission-issue
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'test' > /restricted/file.txt; sleep 300"]
    volumeMounts:
    - name: restricted-volume
      mountPath: /restricted
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  volumes:
  - name: restricted-volume
    hostPath:
      path: /root  # Restricted directory
      type: Directory
EOF

# Apply and diagnose issues
kubectl apply -f volume-problems.yaml

# Troubleshoot missing ConfigMap
kubectl describe pod missing-configmap | grep -A 10 "Events:"
kubectl get events --field-selector involvedObject.name=missing-configmap

# Troubleshoot permission issues
kubectl describe pod permission-issue | grep -A 10 "Events:"
kubectl logs permission-issue
```

#### 2. Volume Performance Analysis
```bash
# Create performance test pod
cat > volume-performance.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: volume-performance
spec:
  containers:
  - name: performance-test
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Testing volume performance..."
      echo "EmptyDir performance:"
      time dd if=/dev/zero of=/emptydir/test.file bs=1M count=100
      echo "Memory volume performance:"
      time dd if=/dev/zero of=/memory/test.file bs=1M count=100
      echo "HostPath performance:"
      time dd if=/dev/zero of=/hostpath/test.file bs=1M count=100
      echo "Performance test completed"
      sleep 300
    volumeMounts:
    - name: emptydir-vol
      mountPath: /emptydir
    - name: memory-vol
      mountPath: /memory
    - name: hostpath-vol
      mountPath: /hostpath
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  volumes:
  - name: emptydir-vol
    emptyDir: {}
  - name: memory-vol
    emptyDir:
      medium: Memory
  - name: hostpath-vol
    hostPath:
      path: /tmp/volume-perf-test
      type: DirectoryOrCreate
EOF

# Apply and run performance test
kubectl apply -f volume-performance.yaml
kubectl logs volume-performance -f
```

#### 3. Volume Best Practices Implementation
```bash
# Create best practices example
cat > volume-best-practices.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: volume-best-practices
  annotations:
    description: "Example of volume best practices"
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 8080
    volumeMounts:
    # Read-only configuration
    - name: app-config
      mountPath: /etc/nginx/conf.d
      readOnly: true
    # Read-only secrets
    - name: tls-certs
      mountPath: /etc/ssl/certs
      readOnly: true
    # Writable temporary storage
    - name: temp-storage
      mountPath: /tmp
    # Writable cache with size limit
    - name: cache-storage
      mountPath: /var/cache/nginx
    # Logs with proper permissions
    - name: log-storage
      mountPath: /var/log/nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
  volumes:
  # Configuration from ConfigMap
  - name: app-config
    configMap:
      name: nginx-config
      defaultMode: 0444
  # Secrets with restricted permissions
  - name: tls-certs
    secret:
      secretName: web-tls
      defaultMode: 0400
  # Temporary storage
  - name: temp-storage
    emptyDir:
      sizeLimit: "100Mi"
  # Cache storage
  - name: cache-storage
    emptyDir:
      sizeLimit: "200Mi"
  # Persistent logs
  - name: log-storage
    hostPath:
      path: /var/log/nginx-app
      type: DirectoryOrCreate
EOF

# Create required ConfigMap and Secret if not exists
kubectl get configmap nginx-config || kubectl create configmap nginx-config --from-literal=nginx.conf="# nginx config"
kubectl get secret web-tls || kubectl create secret generic web-tls --from-literal=tls.crt="cert" --from-literal=tls.key="key"

# Apply best practices example
kubectl apply -f volume-best-practices.yaml
kubectl describe pod volume-best-practices | grep -A 20 "Volumes:"
```

#### 4. Volume Cleanup and Management
```bash
# Clean up all demo resources
echo "Cleaning up volume demo resources..."
kubectl delete pods --all
kubectl delete configmap app-config nginx-config immutable-config
kubectl delete secret database-secret web-tls

# Clean up host directories
sudo rm -rf /var/hostpath-data /var/auto-created-dir /var/app-logs /var/app-config.conf /tmp/volume-perf-test /var/log/nginx-app

# Clean up generated files
rm -f emptydir-pod.yaml memory-emptydir.yaml init-emptydir.yaml
rm -f hostpath-pod.yaml system-monitor.yaml hostpath-types.yaml secure-hostpath.yaml
rm -f configmap-volumes.yaml secret-volumes.yaml mixed-volumes.yaml immutable-config.yaml
rm -f volume-problems.yaml volume-performance.yaml volume-best-practices.yaml
rm -f tls.crt tls.key web-tls-secret.yaml

echo "Volume demos cleanup completed"
```

### CKA Exam Connection
- **Troubleshooting Skills**: Diagnosing volume-related pod failures
- **Security Practices**: Proper permissions and access patterns
- **Performance Understanding**: Choosing appropriate volume types
- **Operational Knowledge**: Volume lifecycle management

### Validation Commands
```bash
# Check volume troubleshooting
kubectl get events --sort-by=.metadata.creationTimestamp | tail -10
kubectl describe pod volume-best-practices | grep -A 5 "SecurityContext"

# Verify performance characteristics
kubectl exec volume-performance -- df -h | grep -E "(emptydir|memory|hostpath)"
```

## Common Issues and Solutions

### Issue 1: ConfigMap/Secret Not Found
**Problem**: Pod fails to start due to missing ConfigMap or Secret
**Solution**: 
```bash
# Check if ConfigMap/Secret exists
kubectl get configmap,secret
kubectl describe pod <pod-name> | grep -A 10 Events

# Create missing resources or fix references
```

### Issue 2: Permission Denied on Volume Mount
**Problem**: Container cannot write to mounted volume
**Solution**:
```bash
# Check volume permissions and security context
kubectl describe pod <pod-name> | grep -A 5 "Security Context"
kubectl exec <pod-name> -- ls -la /mounted/path

# Adjust fsGroup, runAsUser, or volume permissions
```

### Issue 3: Volume Mount Path Conflicts
**Problem**: Multiple volumes mounted to same path
**Solution**:
```bash
# Check volume mount configuration
kubectl describe pod <pod-name> | grep -A 20 "Mounts:"

# Fix conflicting mount paths in pod spec
```

## Daily Goals Checklist

- [ ] Understand EmptyDir use cases and configuration
- [ ] Master HostPath security implications and patterns
- [ ] Configure ConfigMap and Secret volumes properly
- [ ] Troubleshoot common volume mounting issues
- [ ] Implement volume best practices
- [ ] Ready for Volume Access Modes exploration

## Next Day Preparation

### What You'll Need for Day 24
- Volume types understanding ✓
- Mount configuration skills ✓
- Security context knowledge ✓
- Troubleshooting capabilities ✓

### Coming Up: Volume Access Modes & Storage Classes
Day 24 will explore advanced storage patterns and dynamic provisioning.

## Study Notes Section

**Volume Type Quick Reference:**
```bash
# EmptyDir - temporary storage
emptyDir: {}
emptyDir:
  medium: Memory
  sizeLimit: "1Gi"

# HostPath - host filesystem access
hostPath:
  path: /var/data
  type: DirectoryOrCreate

# ConfigMap - configuration files
configMap:
  name: my-config
  defaultMode: 0644

# Secret - sensitive data
secret:
  secretName: my-secret
  defaultMode: 0400
```

**Personal Volume Patterns:**
_Record your preferred volume configurations and troubleshooting approaches_