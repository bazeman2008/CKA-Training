# Complete CKA Test Guide

## What is the CKA Test?

The Certified Kubernetes Administrator (CKA) exam is a hands-on, performance-based certification test that validates your ability to perform the responsibilities of a Kubernetes administrator. Unlike traditional multiple-choice exams, you'll work directly with live Kubernetes clusters through a command-line interface.

## Exam Format & Structure

- **Duration:** 2 hours
- **Format:** Performance-based (hands-on tasks)
- **Questions:** 15-20 practical scenarios
- **Passing Score:** 66%
- **Environment:** Browser-based terminal with multiple Kubernetes clusters
- **Documentation:** Full access to kubernetes.io documentation

## Topics and Percentage Breakdown

### 1. Storage (10%)
- Understanding storage classes, persistent volumes (PV), and persistent volume claims (PVC)
- Know how to configure applications with persistent storage
- Configure volume types and access modes

### 2. Troubleshooting (30%)
- Evaluate cluster and node logging
- Understand how to monitor applications
- Manage container stdout & stderr logs
- Troubleshoot application failure
- Troubleshoot cluster component failure
- Troubleshoot networking

### 3. Workloads & Scheduling (15%)
- Understand deployments and how to perform rolling updates and rollbacks
- Use ConfigMaps and Secrets to configure applications
- Know how to scale applications
- Understand the primitives used to create robust, self-healing, application deployments
- Understand how resource limits can affect Pod scheduling
- Awareness of manifest management and common templating tools

### 4. Cluster Architecture, Installation & Configuration (25%)
- Manage role-based access control (RBAC)
- Use Kubeadm to install a basic cluster
- Manage a highly-available Kubernetes cluster
- Provision underlying infrastructure to deploy a Kubernetes cluster
- Perform a version upgrade on a Kubernetes cluster using Kubeadm
- Implement etcd backup and restore

### 5. Services & Networking (20%)
- Understand host networking configuration on the cluster nodes
- Understand connectivity between Pods
- Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
- Know how to use Ingress controllers and Ingress resources
- Know how to configure and use CoreDNS
- Choose an appropriate container network interface plugin

## Important Test-Taking Information

### Before the Exam

**System Requirements:**
- Chrome or Chromium browser (latest version)
- Stable internet connection
- Webcam and microphone for proctoring
- Government-issued photo ID

**Setup Requirements:**
- Clean, private room
- Clear desk surface
- No additional monitors (single screen only)
- Remove all personal items from workspace

**Registration:**
- Schedule through Linux Foundation portal
- Exam costs $395 (includes one free retake)
- Valid for 2 years from pass date

### During the Exam

**Environment:**
- You'll work with 6-8 different Kubernetes clusters
- Each question specifies which cluster context to use
- Terminal-based interface with kubectl pre-installed
- Copy/paste functionality available

**Key Commands You Must Know:**
```bash
# Context switching (critical!)
kubectl config use-context <cluster-name>
kubectl config current-context

# Quick object creation
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl run pod-name --image=nginx --dry-run=client -o yaml > pod.yaml

# Essential troubleshooting
kubectl describe <resource> <name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

**Allowed Resources:**
- Official Kubernetes documentation (kubernetes.io)
- GitHub repository docs for Kubernetes
- Blog posts and tutorials are NOT allowed

### Scoring and Results

- Automatic scoring system evaluates your cluster configurations
- Results available within 24 hours
- Detailed score breakdown provided
- Certificate valid for 2 years
- Digital badge provided for LinkedIn/resume

### Pro Tips for Success

**Time Management:**
- Check point values for each question first
- Tackle high-value questions when you're fresh
- Don't spend more than 10 minutes on any single low-value question
- Flag difficult questions and return if time permits

**Technical Preparation:**
- Practice with real clusters, not just theory
- Master imperative kubectl commands for speed
- Set up bash aliases and environment variables
- Practice context switching between clusters
- Learn to read and modify YAML efficiently

**Common Pitfalls to Avoid:**
- Working on wrong cluster context
- Not reading question requirements completely
- Spending too much time on low-value questions
- Forgetting to verify your work
- Not using imperative commands to save time

**Verification Strategy:**
Always verify your work with commands like:
```bash
kubectl get pods -o wide
kubectl describe deployment <name>
kubectl get svc
kubectl get pv,pvc
```

## Recommended Study Path

1. **Foundation:** Complete Kubernetes basics course
2. **Hands-on:** Set up your own clusters using kubeadm
3. **Practice:** Work through scenario-based labs
4. **Mock Exams:** Take timed practice tests
5. **Documentation:** Familiarize yourself with kubernetes.io layout

The CKA is challenging but achievable with proper hands-on practice. Focus on building muscle memory with kubectl commands and understanding real-world Kubernetes administration scenarios.