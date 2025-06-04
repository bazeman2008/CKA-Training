# Day 16: DaemonSets & Jobs

## Overview (1.5 hours)
Master DaemonSets for node-wide services and Jobs/CronJobs for batch processing workloads essential for CKA exam scenarios.

## Why This Matters for the CKA Exam
- **Workload types appear in 25-30% of exam questions**
- **DaemonSets** critical for cluster services and monitoring
- **Jobs** frequently tested for batch processing and maintenance tasks
- **CronJobs** essential for scheduled operations
- **Troubleshooting** batch workloads requires specific diagnostic skills

## Task 1: DaemonSet Configuration and Management (45 minutes)

### What You'll Do
Configure DaemonSets to run pods on every node and manage node-specific services.

### Step-by-Step Instructions

#### 1. Basic DaemonSet Creation
```bash
# Create simple DaemonSet for logging agent
cat > logging-daemonset.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-logging
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14-debian-1
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: dockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: dockercontainers
        hostPath:
          path: /var/lib/docker/containers
      terminationGracePeriodSeconds: 30
EOF

# Apply and verify
kubectl apply -f logging-daemonset.yaml
kubectl get daemonsets
kubectl get pods -l app=fluentd -o wide
```

#### 2. DaemonSet with Node Selector
```bash
# Create DaemonSet for worker nodes only
cat > monitoring-daemonset.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-monitoring
  labels:
    app: monitoring
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      nodeSelector:
        node-type: worker  # Only on labeled worker nodes
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          protocol: TCP
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      hostNetwork: true
      hostPID: true
EOF

# Label worker nodes and apply
kubectl label nodes worker-node-1 node-type=worker
kubectl label nodes worker-node-2 node-type=worker
kubectl apply -f monitoring-daemonset.yaml
kubectl get pods -l app=monitoring -o wide
```

#### 3. DaemonSet with Tolerations
```bash
# Create DaemonSet that runs on all nodes including master
cat > system-daemonset.yaml << 'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: system-monitoring
  labels:
    app: system-monitor
spec:
  selector:
    matchLabels:
      app: system-monitor
  template:
    metadata:
      labels:
        app: system-monitor
    spec:
      tolerations:
      # Allow running on master nodes
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: system-monitor
        image: busybox
        command: ["sh", "-c", "while true; do echo $(date): $(hostname): System check; sleep 30; done"]
        resources:
          requests:
            memory: "32Mi"
            cpu: "25m"
          limits:
            memory: "64Mi"
            cpu: "50m"
EOF

# Apply and verify on all nodes
kubectl apply -f system-daemonset.yaml
kubectl get pods -l app=system-monitor -o wide
kubectl logs -l app=system-monitor --tail=5
```

#### 4. DaemonSet Management Operations
```bash
# Update DaemonSet image
kubectl set image daemonset/fluentd-logging fluentd=fluentd:v1.15-debian-1
kubectl rollout status daemonset fluentd-logging

# Check rollout history
kubectl rollout history daemonset fluentd-logging

# Scale DaemonSet (remove from specific node)
kubectl patch node worker-node-1 -p '{"spec":{"unschedulable":true}}'
kubectl get pods -l app=fluentd -o wide

# Restore scheduling
kubectl patch node worker-node-1 -p '{"spec":{"unschedulable":false}}'
```

### CKA Exam Connection
- **Cluster Services**: Logging, monitoring, network plugins
- **Node Management**: Services required on every node
- **Security**: Running privileged containers on nodes
- **Troubleshooting**: Diagnosing node-level service issues

### Validation Commands
```bash
# Check DaemonSet status
kubectl get daemonsets -o wide
kubectl describe daemonset fluentd-logging

# Verify pod distribution
kubectl get pods -l app=fluentd --field-selector status.phase=Running
kubectl get nodes -o custom-columns="NAME:.metadata.name,PODS:.status.addresses[0].address"
```

## Task 2: Jobs and Batch Processing (35 minutes)

### What You'll Do
Create and manage Jobs for one-time tasks and batch processing workloads.

### Step-by-Step Instructions

#### 1. Basic Job Creation
```bash
# Create simple Job for data processing
cat > basic-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
spec:
  template:
    metadata:
      labels:
        app: data-processor
    spec:
      containers:
      - name: processor
        image: busybox
        command: ["sh", "-c", "echo 'Processing data...'; sleep 30; echo 'Data processing complete'"]
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
      restartPolicy: OnFailure
  backoffLimit: 3
  completions: 1
  parallelism: 1
EOF

# Apply and monitor
kubectl apply -f basic-job.yaml
kubectl get jobs
kubectl get pods -l app=data-processor
kubectl logs -l app=data-processor
```

#### 2. Parallel Job Processing
```bash
# Create Job with multiple parallel executions
cat > parallel-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-processor
spec:
  template:
    metadata:
      labels:
        app: parallel-processor
    spec:
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "echo 'Worker starting on' $(hostname); sleep $((RANDOM % 20 + 10)); echo 'Worker completed on' $(hostname)"]
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
      restartPolicy: OnFailure
  completions: 6      # Total completions needed
  parallelism: 3      # Concurrent pods
  backoffLimit: 2
EOF

# Apply and watch progress
kubectl apply -f parallel-job.yaml
kubectl get jobs -w &
kubectl get pods -l app=parallel-processor -w &
# Stop watching with Ctrl+C
```

#### 3. Job with Init Container
```bash
# Create Job with initialization step
cat > init-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: init-processor
spec:
  template:
    metadata:
      labels:
        app: init-processor
    spec:
      initContainers:
      - name: setup
        image: busybox
        command: ["sh", "-c", "echo 'Setting up environment...'; mkdir -p /shared/data; echo 'ready' > /shared/data/status"]
        volumeMounts:
        - name: shared-data
          mountPath: /shared
      containers:
      - name: main-processor
        image: busybox
        command: ["sh", "-c", "while [ ! -f /shared/data/status ]; do sleep 1; done; echo 'Processing with setup complete'; sleep 10"]
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
        volumeMounts:
        - name: shared-data
          mountPath: /shared
      volumes:
      - name: shared-data
        emptyDir: {}
      restartPolicy: OnFailure
  backoffLimit: 2
EOF

# Apply and monitor
kubectl apply -f init-job.yaml
kubectl get pods -l app=init-processor
kubectl logs -l app=init-processor -c setup
kubectl logs -l app=init-processor -c main-processor
```

#### 4. Job Management and Cleanup
```bash
# Check job completion
kubectl get jobs
kubectl describe job data-processor

# Manual job deletion
kubectl delete job data-processor

# Job with TTL (automatic cleanup)
cat > ttl-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: ttl-job
spec:
  ttlSecondsAfterFinished: 100  # Auto-delete after 100 seconds
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "echo 'Job with TTL'; sleep 5"]
      restartPolicy: OnFailure
EOF

kubectl apply -f ttl-job.yaml
```

### CKA Exam Connection
- **Maintenance Tasks**: Database backups, system updates
- **Data Processing**: ETL jobs, batch analytics
- **One-time Operations**: Cluster initialization, migration
- **Resource Management**: Controlling job resource usage

### Validation Commands
```bash
# Monitor job progress
kubectl get jobs --watch
kubectl describe job parallel-processor

# Check job logs
kubectl logs job/data-processor
kubectl logs -l job-name=parallel-processor --prefix=true
```

## Task 3: CronJobs for Scheduled Tasks (20 minutes)

### What You'll Do
Configure CronJobs for recurring tasks and scheduled maintenance operations.

### Step-by-Step Instructions

#### 1. Basic CronJob Creation
```bash
# Create CronJob for periodic backup
cat > backup-cronjob.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "*/2 * * * *"  # Every 2 minutes for testing
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: backup
        spec:
          containers:
          - name: backup
            image: busybox
            command: ["sh", "-c", "echo 'Backup started at' $(date); sleep 10; echo 'Backup completed at' $(date)"]
            resources:
              requests:
                memory: "32Mi"
                cpu: "50m"
              limits:
                memory: "64Mi"
                cpu: "100m"
          restartPolicy: OnFailure
      backoffLimit: 2
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
EOF

# Apply and monitor
kubectl apply -f backup-cronjob.yaml
kubectl get cronjobs
kubectl get jobs -w &
# Wait for 2-3 executions, then stop with Ctrl+C
```

#### 2. Advanced CronJob Configuration
```bash
# Create CronJob with concurrency control
cat > maintenance-cronjob.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: maintenance-cronjob
spec:
  schedule: "0 2 * * 0"  # Weekly at 2 AM Sunday
  concurrencyPolicy: Forbid  # Don't allow concurrent executions
  startingDeadlineSeconds: 300
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: maintenance
        spec:
          containers:
          - name: maintenance
            image: busybox
            command: ["sh", "-c", "echo 'Maintenance task starting'; sleep 30; echo 'Maintenance completed'"]
            resources:
              requests:
                memory: "64Mi"
                cpu: "100m"
              limits:
                memory: "128Mi"
                cpu: "200m"
          restartPolicy: OnFailure
      backoffLimit: 1
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 2
EOF

# Apply and check
kubectl apply -f maintenance-cronjob.yaml
kubectl describe cronjob maintenance-cronjob
```

#### 3. CronJob Management
```bash
# Suspend CronJob
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":true}}'
kubectl get cronjobs

# Resume CronJob
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":false}}'

# Manually trigger CronJob
kubectl create job --from=cronjob/backup-cronjob manual-backup
kubectl get jobs
```

### CronJob Schedule Examples
```bash
# Common schedule patterns:
# "0 0 * * *"       # Daily at midnight
# "0 */6 * * *"     # Every 6 hours
# "30 2 * * 1"      # Weekly Monday at 2:30 AM
# "0 0 1 * *"       # Monthly on 1st at midnight
# "*/15 * * * *"    # Every 15 minutes
```

### CKA Exam Connection
- **Automated Operations**: Regular backups, cleanup tasks
- **Maintenance Windows**: Scheduled updates and checks
- **Resource Management**: Preventing resource conflicts
- **Troubleshooting**: Debugging scheduled task failures

### Validation Commands
```bash
# Check CronJob status
kubectl get cronjobs -o wide
kubectl describe cronjob backup-cronjob

# View job history
kubectl get jobs --sort-by=.metadata.creationTimestamp
kubectl logs job/backup-cronjob-<timestamp>
```

## Task 4: Troubleshooting Batch Workloads (10 minutes)

### What You'll Do
Practice diagnosing and resolving common issues with DaemonSets, Jobs, and CronJobs.

### Common Issues and Solutions

#### 1. DaemonSet Not Running on All Nodes
```bash
# Diagnose missing DaemonSet pods
kubectl get daemonsets
kubectl get pods -l app=fluentd -o wide
kubectl describe nodes | grep Taints

# Check for node taints blocking DaemonSet
kubectl describe daemonset fluentd-logging | grep -A 10 Tolerations

# Fix by adding tolerations or removing taints
kubectl taint nodes worker-node-1 special=true:NoSchedule-
```

#### 2. Job Stuck or Failing
```bash
# Diagnose job issues
kubectl get jobs
kubectl describe job data-processor
kubectl get pods -l job-name=data-processor
kubectl logs -l job-name=data-processor

# Common fixes:
# - Increase backoffLimit
# - Fix container command/image
# - Check resource constraints
kubectl patch job data-processor -p '{"spec":{"backoffLimit":5}}'
```

#### 3. CronJob Not Executing
```bash
# Check CronJob status
kubectl get cronjobs
kubectl describe cronjob backup-cronjob

# Verify schedule format
kubectl get cronjob backup-cronjob -o yaml | grep schedule

# Check for suspension
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":false}}'
```

#### 4. Resource Constraints
```bash
# Check cluster resources
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Adjust resource requests
kubectl patch daemonset fluentd-logging -p '{"spec":{"template":{"spec":{"containers":[{"name":"fluentd","resources":{"requests":{"memory":"64Mi","cpu":"50m"}}}]}}}}'
```

### Diagnostic Commands
```bash
# Comprehensive workload diagnosis
kubectl get all --all-namespaces | grep -E "(daemonset|job|cronjob)"
kubectl get events --sort-by=.metadata.creationTimestamp | tail -20
kubectl logs -l app=<workload-label> --previous
```

## Common Issues and Solutions

### Issue 1: DaemonSet Pod Not Scheduled
**Problem**: DaemonSet missing pods on some nodes
**Solution**: 
```bash
# Check node conditions and taints
kubectl describe nodes | grep -E "(Conditions|Taints)" -A 5
kubectl get nodes -o custom-columns="NAME:.metadata.name,STATUS:.status.conditions[?(@.type=='Ready')].status"

# Add necessary tolerations to DaemonSet
```

### Issue 2: Job Never Completes
**Problem**: Job runs but never finishes
**Solution**:
```bash
# Check container exit codes
kubectl describe pods -l job-name=<job-name>
kubectl logs -l job-name=<job-name> --previous

# Verify restartPolicy is OnFailure or Never
kubectl get job <job-name> -o yaml | grep restartPolicy
```

### Issue 3: CronJob Creates Too Many Jobs
**Problem**: CronJob creating excessive job instances
**Solution**:
```bash
# Check concurrencyPolicy and history limits
kubectl describe cronjob <cronjob-name>

# Adjust settings
kubectl patch cronjob <name> -p '{"spec":{"concurrencyPolicy":"Forbid","successfulJobsHistoryLimit":3}}'
```

## Time Management Tips

### For CKA Exam Success
- **DaemonSet Quick Check**: Use `kubectl get daemonsets` and `kubectl get pods -o wide`
- **Job Monitoring**: Use `kubectl get jobs` and `kubectl logs job/<name>`
- **CronJob Testing**: Change schedule to `*/1 * * * *` for quick testing
- **Resource Patterns**: Use standard resource requests for batch workloads

### Daily Practice Goals
- [ ] Create DaemonSet with tolerations in under 5 minutes
- [ ] Configure parallel Job efficiently
- [ ] Set up CronJob with proper scheduling
- [ ] Troubleshoot workload issues systematically

## Next Day Preparation

### Review Before Day 17
- [ ] DaemonSet scheduling behavior and tolerations
- [ ] Job completion patterns and parallelism
- [ ] CronJob schedule syntax and concurrency control
- [ ] Resource management for batch workloads

### What's Coming Next
Day 17 will cover Resource Management, which builds on workload concepts:
- Resource requests and limits for Jobs/DaemonSets ✓
- QoS classes affecting workload scheduling ✓  
- LimitRanges controlling workload resources ✓

## Study Notes Section
_Use this space to document workload configurations specific to your environment_

**DaemonSet Patterns:**
- Logging: ________________
- Monitoring: ______________
- Network: ________________

**Job Configurations:**
- Backup Commands: ________________
- Processing Patterns: ______________
- Resource Allocations: ____________

**CronJob Schedules:**
```bash
# Common patterns for your environment
# Daily backup: "0 2 * * *"
# Weekly maintenance: _______________
# Monthly cleanup: __________________
```

**Personal Workload Patterns:**
_Record your preferred configurations and troubleshooting approaches_