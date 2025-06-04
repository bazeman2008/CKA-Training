# 5-Month CKA Study Plan 

> **Related Documents**: 
> - [CKA-Training-Strategy.md](./CKA-Training-Strategy.md) - Learning methodology and study principles
> - [CKA-Testing-Strategies.md](./CKA-Testing-Strategies.md) - Exam-day tactics and tips
> - [Detailed Weekly Guides](./5-Month-Study-Plan/Overview.md) - Week-by-week study documents

## Overview & Setup

- [ ] Set up 3 Linux VMs for cluster practice
- [ ] Install essential tools: vim, tmux, git, curl, wget
- [ ] Configure SSH keys between nodes
- [ ] Set up study schedule: 1.5 hours weekdays, 2-3 hours weekends
- [ ] Create study environment and bookmark kubernetes.io documentation

## Month 1: Foundation & Environment Setup

### Week 1: Environment & Container Fundamentals

#### Day 1: Linux Setup & CKA Environment (1.5 hours)
- [ ] Install Linux on 3 VMs
- [ ] Configure SSH and essential tools
- [ ] Set up bash aliases: `alias k=kubectl`
- [ ] Enable kubectl bash completion

#### Day 2: Container Runtime Foundation (1.5 hours)
- [ ] Install Podman and CRI-O
- [ ] Practice basic Podman commands: `run`, `ps`, `logs`, `exec`
- [ ] Understand container runtime in Kubernetes context

#### Day 3: Kubernetes Architecture Deep Dive (1.5 hours)
- [ ] Study control plane components (API server, etcd, scheduler)
- [ ] Study worker node components (kubelet, kube-proxy)
- [ ] Document architecture for troubleshooting reference

#### Day 4: kubectl Installation & Basics (1.5 hours)
- [ ] Install kubeadm, kubelet, kubectl on all nodes
- [ ] Practice basic kubectl commands and output formats
- [ ] Master `kubectl explain` for API reference

#### Day 5: First Cluster Setup (1.5 hours)
- [ ] Complete `kubeadm init` process
- [ ] Understand cluster certificates and tokens
- [ ] Document setup steps for speed improvement

#### Weekend Session 1 (3 hours)
- [ ] Complete cluster setup from scratch (multiple attempts)
- [ ] Join worker nodes and practice basic troubleshooting
- [ ] **Goal Check:** Cluster setup in under 30 minutes

### Week 2: kubectl Mastery & Core Objects

#### Day 8: kubectl Command Mastery (1.5 hours)
- [ ] Practice imperative vs declarative commands
- [ ] Master output formats, field selectors, label selectors
- [ ] Speed practice with `kubectl run` and `kubectl create`

#### Day 9: Pods Deep Dive (1.5 hours)
- [ ] Study pod lifecycle and phases
- [ ] Practice multi-container patterns
- [ ] Implement init containers and sidecar patterns

#### Day 10: Deployments & ReplicaSets (1.5 hours)
- [ ] Practice deployment strategies and rolling updates
- [ ] Master rollback procedures
- [ ] Practice scaling and managing deployments

#### Day 11: Services Introduction (1.5 hours)
- [ ] Configure service types (ClusterIP, NodePort, LoadBalancer)
- [ ] Practice service discovery and endpoints
- [ ] Speed practice: expose applications quickly

#### Day 12: ConfigMaps & Secrets (1.5 hours)
- [ ] Practice configuration management
- [ ] Mount configs and secrets in pods
- [ ] Troubleshoot configuration issues

#### Weekend Session 2 (2.5 hours)
- [ ] Create complete application stacks (pods + services + configs)
- [ ] Speed drills for basic object creation
- [ ] **Goal Check:** Deploy applications in under 10 minutes

### Week 3: Scheduling & Workload Management

#### Day 15: Advanced Scheduling (1.5 hours)
- [ ] Practice node selectors and node affinity
- [ ] Configure taints and tolerations
- [ ] Practice manual scheduling techniques

#### Day 16: DaemonSets & Jobs (1.5 hours)
- [ ] Configure DaemonSets for different use cases
- [ ] Create and manage Jobs and CronJobs
- [ ] Troubleshoot batch workloads

#### Day 17: Resource Management (1.5 hours)
- [ ] Configure resource requests and limits
- [ ] Understand QoS classes
- [ ] Implement LimitRanges and ResourceQuotas

#### Day 18: Static Pods (1.5 hours)
- [ ] Configure static pods
- [ ] Manage kubelet static pod directory
- [ ] Understand control plane static pods

#### Day 19: Workload Troubleshooting (1.5 hours)
- [ ] Practice pod failure diagnosis
- [ ] Debug container startup issues
- [ ] Master `kubectl describe`, `kubectl logs`, events analysis

#### Weekend Session 3 (3 hours)
- [ ] Practice complex scheduling scenarios
- [ ] Workload management speed tests
- [ ] **Goal Check:** Solve scheduling problems efficiently

### Week 4: Storage & First Assessment

#### Day 22: Persistent Volumes (1.5 hours)
- [ ] Configure PV and PVC lifecycle
- [ ] Practice static vs dynamic provisioning
- [ ] Configure storage classes

#### Day 23: Volume Types (1.5 hours)
- [ ] Configure hostPath, emptyDir, configMap, secret volumes
- [ ] Practice volume mounting and permissions
- [ ] Test various volume configurations

#### Day 24: Storage Troubleshooting (1.5 hours)
- [ ] Debug PVC binding issues
- [ ] Resolve permission problems
- [ ] Fix storage-related pod failures

#### Day 25: Security Contexts (1.5 hours)
- [ ] Configure pod and container security contexts
- [ ] Practice running pods as non-root
- [ ] Manage security capabilities

#### Day 26: Basic RBAC (1.5 hours)
- [ ] Create users and service accounts
- [ ] Configure roles and RoleBindings
- [ ] Practice basic permission management

#### Weekend Session 4 - First Mock Exam (3 hours)
- [ ] **MOCK EXAM:** 2-hour timed test covering Month 1 topics
- [ ] Review results and identify weak areas
- [ ] **Goal Check:** 40-50% score to proceed

## Month 2: Networking & Advanced Workloads

### Week 5: Networking Fundamentals

#### Day 29: Service Deep Dive (1.5 hours)
- [ ] Advanced service type configurations
- [ ] Manage Endpoints and EndpointSlices
- [ ] Practice service troubleshooting

#### Day 30: Ingress Configuration (1.5 hours)
- [ ] Configure ingress controllers and resources
- [ ] Set up HTTP routing and TLS termination
- [ ] Practice exposing applications via ingress

#### Day 31: Network Policies (1.5 hours)
- [ ] Control pod-to-pod communication
- [ ] Implement ingress and egress rules
- [ ] Practice network security and segmentation

#### Day 32: DNS & Service Discovery (1.5 hours)
- [ ] Configure and troubleshoot CoreDNS
- [ ] Practice service discovery mechanisms
- [ ] Master DNS debugging techniques

#### Day 33: CNI and Cluster Networking (1.5 hours)
- [ ] Configure network plugin (Calico/Flannel)
- [ ] Troubleshoot cluster-level networking
- [ ] Practice network connectivity testing

#### Weekend Session 5 (2.5 hours)
- [ ] End-to-end networking scenarios
- [ ] Network troubleshooting practice
- [ ] **Goal Check:** Diagnose network issues in under 15 minutes

### Week 6: Cluster Administration

#### Day 36: Node Management (1.5 hours)
- [ ] Practice drain, cordon, and uncordon procedures
- [ ] Troubleshoot node issues
- [ ] Master safe node maintenance practices

#### Day 37: Cluster Upgrades (1.5 hours)
- [ ] Practice kubeadm upgrade for control plane
- [ ] Upgrade worker node procedures
- [ ] Master version upgrade methodology

#### Day 38: etcd Backup & Restore (1.5 hours)
- [ ] **CRITICAL:** Practice etcd backup with etcdctl
- [ ] Practice cluster state restoration
- [ ] Speed drill: backup/restore procedures

#### Day 39: Certificate Management (1.5 hours)
- [ ] Check certificate expiration
- [ ] Practice certificate renewal with kubeadm
- [ ] Troubleshoot TLS configuration

#### Day 40: Control Plane Troubleshooting (1.5 hours)
- [ ] Debug API server issues
- [ ] Troubleshoot scheduler and controller-manager
- [ ] Practice static pod troubleshooting

#### Weekend Session 6 (3 hours)
- [ ] Complete cluster maintenance scenarios
- [ ] etcd backup/restore speed drills
- [ ] **Goal Check:** Complete backup/restore in under 10 minutes

### Week 7: Advanced RBAC & Security

#### Day 43: RBAC Advanced (1.5 hours)
- [ ] Configure ClusterRoles and ClusterRoleBindings
- [ ] Manage service account tokens
- [ ] Practice complex permission scenarios

#### Day 44: Security Policies (1.5 hours)
- [ ] Implement Pod Security Standards
- [ ] Troubleshoot security contexts
- [ ] Practice secure workload deployment

#### Day 45: Network Security Advanced (1.5 hours)
- [ ] Configure complex network policies
- [ ] Implement default deny policies
- [ ] Practice multi-namespace security

#### Day 46: Admission Controllers (1.5 hours)
- [ ] Understand admission controller concepts
- [ ] Troubleshoot admission controller issues
- [ ] Study cluster security mechanisms

#### Day 47: Monitoring & Logging (1.5 hours)
- [ ] Set up cluster-level logging
- [ ] Practice event analysis and correlation
- [ ] Master troubleshooting with logs

#### Weekend Session 7 (3 hours)
- [ ] Security configuration practice
- [ ] RBAC troubleshooting scenarios
- [ ] **Goal Check:** Configure security policies efficiently

### Week 8: Month 2 Assessment

#### Days 50-54: Intensive Practice Week (1.5 hours each)
- [ ] **Day 50:** Focus on networking weak areas
- [ ] **Day 51:** Focus on cluster maintenance weak areas
- [ ] **Day 52:** Focus on security weak areas
- [ ] **Day 53:** Speed improvement drills
- [ ] **Day 54:** Comprehensive review

#### Weekend Session 8 - Major Mock Exam (4 hours)
- [ ] **MOCK EXAM:** Full CKA simulation covering Months 1-2
- [ ] Detailed review and improvement planning
- [ ] **Goal Check:** 60-70% score for Month 3 readiness

## Month 3: Troubleshooting Mastery

### Week 9: Application Troubleshooting

#### Day 57: Pod Failure Diagnosis (1.5 hours)
- [ ] Practice systematic pod troubleshooting methodology
- [ ] Debug container startup and runtime issues
- [ ] Master `kubectl logs`, `describe`, events analysis

#### Day 58: Resource Constraint Issues (1.5 hours)
- [ ] Identify CPU and memory constraints
- [ ] Understand QoS impact on scheduling and eviction
- [ ] Diagnose performance bottlenecks

#### Day 59: Configuration Troubleshooting (1.5 hours)
- [ ] Debug ConfigMap and Secret mounting issues
- [ ] Fix environment variable problems
- [ ] Resolve configuration-related failures

#### Day 60: Multi-Component Failures (1.5 hours)
- [ ] Practice complex scenarios with multiple failures
- [ ] Develop systematic approach to complex problems
- [ ] Build troubleshooting methodology under pressure

#### Day 61: Storage Troubleshooting Advanced (1.5 hours)
- [ ] Fix PVC mounting failures and permissions
- [ ] Resolve storage driver problems
- [ ] Debug volume attachment issues

#### Weekend Session 9 (3 hours)
- [ ] Complex troubleshooting scenarios
- [ ] Speed improvement for troubleshooting tasks
- [ ] **Goal Check:** Solve application issues in under 12 minutes

### Week 10: Infrastructure Troubleshooting

#### Day 64: Worker Node Issues (1.5 hours)
- [ ] Troubleshoot kubelet and kube-proxy
- [ ] Debug container runtime problems
- [ ] Resolve Node NotReady scenarios

#### Day 65: Control Plane Debugging (1.5 hours)
- [ ] Fix API server connectivity and certificate issues
- [ ] Debug scheduler and controller-manager problems
- [ ] Resolve control plane component failures

#### Day 66: Network Connectivity Issues (1.5 hours)
- [ ] Debug pod-to-pod communication
- [ ] Fix service-to-pod communication
- [ ] Resolve ingress and DNS problems

#### Day 67: Cluster-Level Issues (1.5 hours)
- [ ] Troubleshoot etcd problems
- [ ] Debug cluster state issues
- [ ] Fix add-on and extension problems

#### Day 68: Performance and Resource Issues (1.5 hours)
- [ ] Diagnose resource exhaustion
- [ ] Identify performance degradation
- [ ] Interpret monitoring and metrics

#### Weekend Session 10 (3 hours)
- [ ] Infrastructure troubleshooting practice
- [ ] Multi-layer problem resolution
- [ ] **Goal Check:** Systematic troubleshooting mastery

### Week 11: Speed & Efficiency Training

#### Day 71: kubectl Speed Mastery (1.5 hours)
- [ ] Imperative command speed drills
- [ ] Optimize documentation navigation
- [ ] Improve task completion efficiency

#### Day 72: Common Task Automation (1.5 hours)
- [ ] Practice frequently used command combinations
- [ ] Optimize bash aliases and shortcuts
- [ ] Reduce repetitive task time

#### Day 73: Documentation Mastery (1.5 hours)
- [ ] kubernetes.io navigation speed training
- [ ] Organize bookmarks and quick references
- [ ] Practice sub-5-minute documentation searches

#### Day 74: Time Management Practice (1.5 hours)
- [ ] Timed task completion drills
- [ ] Practice question prioritization strategies
- [ ] Optimize exam time management

#### Day 75: Error Recovery Techniques (1.5 hours)
- [ ] Practice quick mistake correction methods
- [ ] Learn partial credit maximization strategies
- [ ] Build efficiency under pressure

#### Weekend Session 11 (4 hours)
- [ ] **MOCK EXAM:** Full timed exam
- [ ] Speed and efficiency analysis
- [ ] **Goal Check:** 75-80% score with good time management

### Week 12: Final Preparation

#### Days 78-82: Weak Area Elimination (1.5 hours each)
- [ ] **Day 78:** Focus on lowest-scoring domain from mock exam
- [ ] **Day 79:** Intensive practice on problem areas
- [ ] **Day 80:** Speed drills on weak topics
- [ ] **Day 81:** Comprehensive domain review
- [ ] **Day 82:** Final troubleshooting scenarios

#### Weekend Session 12 - Final Mock Exam (4 hours)
- [ ] **MOCK EXAM:** Final comprehensive CKA simulation
- [ ] Final strategy refinement
- [ ] **Goal Check:** 85%+ score and exam confidence

## Month 4: CKA Exam Mastery

### Week 13: Perfect Practice

#### Daily Sessions (1.5 hours each)
- [ ] **Day 85:** Cluster Architecture mastery
- [ ] **Day 86:** Workloads & Scheduling mastery
- [ ] **Day 87:** Services & Networking mastery
- [ ] **Day 88:** Storage mastery
- [ ] **Day 89:** Troubleshooting mastery

#### Weekend Session 13 (3 hours)
- [ ] **MOCK EXAM:** Domain-specific testing
- [ ] Performance analysis and optimization

### Week 14: Exam Simulation

#### Daily Sessions (1.5 hours each)
- [ ] **Day 92:** Speed optimization training
- [ ] **Day 93:** Documentation efficiency training
- [ ] **Day 94:** Error recovery practice
- [ ] **Day 95:** Time management optimization
- [ ] **Day 96:** Confidence building exercises

#### Weekend Session 14 (3 hours)
- [ ] **MOCK EXAM:** Final exam simulation
- [ ] Mental preparation and strategy finalization

### Week 15: Final Preparation

#### Daily Sessions (1.5 hours each)
- [ ] **Day 99:** Environment testing and setup
- [ ] **Day 100:** Light review and mental preparation
- [ ] **Day 101:** Equipment check and final prep
- [ ] **Day 102:** Rest day - no intensive study
- [ ] **Day 103:** Final confidence review

#### Weekend Session 15
- [ ] **CKA EXAM DAY:** Take the official CKA exam
- [ ] Post-exam analysis and planning

### Week 16: Post-Exam

- [ ] **If Passed:** Celebrate and plan next certifications (CKAD/CKS)
- [ ] **If Retake Needed:** Analyze results and create focused improvement plan
- [ ] Update resume and LinkedIn with CKA certification
- [ ] Plan advanced Kubernetes learning path

## Month 5: Advanced Skills & Career Development

### Week 17-20: Choose Your Path

#### If CKA Passed - Advanced Topics (1.5 hours daily)
- [ ] **Week 17:** CKAD preparation planning
- [ ] **Week 18:** CKS (security) preparation planning
- [ ] **Week 19:** Production deployment patterns
- [ ] **Week 20:** Career advancement and job search

#### If Retake Needed - Intensive Improvement (1.5 hours daily)
- [ ] **Week 17:** Focus on failed exam domains
- [ ] **Week 18:** Intensive practice on weak areas
- [ ] **Week 19:** Speed and accuracy improvement
- [ ] **Week 20:** Final preparation for retake

## Progress Tracking & Milestones

### Weekly Goals Checklist
- [ ] **Week 1:** Environment setup complete
- [ ] **Week 2:** Basic kubectl mastery
- [ ] **Week 3:** Scheduling and workload competency
- [ ] **Week 4:** First mock exam 40-50%
- [ ] **Week 5:** Networking fundamentals
- [ ] **Week 6:** Cluster administration skills
- [ ] **Week 7:** Security and RBAC competency
- [ ] **Week 8:** Second mock exam 60-70%
- [ ] **Week 9:** Application troubleshooting mastery
- [ ] **Week 10:** Infrastructure troubleshooting mastery
- [ ] **Week 11:** Speed and efficiency optimization
- [ ] **Week 12:** Third mock exam 75-80%
- [ ] **Week 13-14:** Exam mastery 85%+
- [ ] **Week 15:** CKA exam success
- [ ] **Week 16-20:** Advanced skills development

### Key Milestone Checklist
- [ ] **Month 1 Complete:** Foundation skills established
- [ ] **Month 2 Complete:** Core CKA competency achieved
- [ ] **Month 3 Complete:** Troubleshooting mastery achieved
- [ ] **Month 4 Complete:** CKA certification earned
- [ ] **Month 5 Complete:** Advanced skills and career progression

### Critical Skills Checklist (Must Master for CKA)
- [ ] Cluster setup with kubeadm in under 20 minutes
- [ ] etcd backup and restore in under 10 minutes
- [ ] Pod troubleshooting with logs and events
- [ ] Service and ingress configuration
- [ ] Network policy implementation
- [ ] RBAC configuration and troubleshooting
- [ ] Node maintenance (drain/cordon/uncordon)
- [ ] Resource management and quotas
- [ ] Certificate management and renewal
- [ ] Storage configuration and troubleshooting

### Performance Benchmarks
- **Week 2:** Complete simple tasks in < 5 minutes
- **Week 4:** Complete medium tasks in < 10 minutes
- **Week 6:** Complete complex tasks in < 15 minutes
- **Week 8:** Complete full mock exam in < 1:45 hours
- **Week 12:** Achieve 80%+ accuracy with time to spare

### Study Habit Tracking
- [ ] **Week 1:** Established daily 1.5-hour routine
- [ ] **Week 2:** Consistent weekend sessions
- [ ] **Week 4:** First progress assessment
- [ ] **Week 8:** Mid-program evaluation
- [ ] **Week 12:** Final preparation phase
- [ ] **Week 15:** Exam readiness achieved

## Resources Checklist
- [ ] Bookmark kubernetes.io documentation
- [ ] Set up kubectl aliases and bash completion
- [ ] Create personal command cheat sheet
- [ ] Join Kubernetes Slack community
- [ ] Follow CKA exam updates and announcements
- [ ] Set up practice environment automation scripts
- [ ] Create backup study materials and references

---

**Note:** For detailed week-by-week study guides with commands, exercises, and examples, see the [5-Month Study Plan directory](./5-Month-Study-Plan/).