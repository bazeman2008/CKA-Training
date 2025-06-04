# CKA Study Schedule

> **Related Documents**: 
> - [CKA-Training-Strategy.md](./CKA-Training-Strategy.md) - Learning methodology and study principles
> - [CKA-Testing-Strategies.md](./CKA-Testing-Strategies.md) - Exam-day tactics and tips

## 8-Week Intensive Schedule

This schedule assumes 2-3 hours of daily study time with additional weekend sessions.

### Week 1: Kubernetes Fundamentals
**Monday-Tuesday**: Core Concepts
- Kubernetes architecture and components
- Pods, ReplicaSets, Deployments
- Practice: Create 20 different pod configurations

**Wednesday-Thursday**: Services & Networking Basics
- Service types (ClusterIP, NodePort, LoadBalancer)
- Basic networking concepts
- Practice: Expose applications using different service types

**Friday**: Configuration
- ConfigMaps and Secrets
- Environment variables and volume mounts
- Practice: Deploy apps with external configuration

**Weekend**: Review & Practice
- Complete practice labs for all week 1 topics
- Time yourself creating resources imperatively
- Document common commands in personal cheat sheet

### Week 2: Workloads & Scheduling
**Monday-Tuesday**: Advanced Workloads
- DaemonSets, StatefulSets, Jobs, CronJobs
- Rolling updates and rollbacks
- Practice: Deploy each workload type

**Wednesday-Thursday**: Scheduling
- Node selectors and affinity
- Taints and tolerations
- Resource requests and limits
- Practice: Schedule pods with specific requirements

**Friday**: Multi-container Pods
- Sidecar, ambassador, adapter patterns
- Init containers
- Practice: Create pods with multiple containers

**Weekend**: Mock Exam #1
- Take first practice exam (untimed)
- Review all mistakes thoroughly
- Update study plan based on weak areas

### Week 3: Storage & Configuration
**Monday-Tuesday**: Storage Fundamentals
- Volumes, PersistentVolumes, PersistentVolumeClaims
- StorageClasses
- Practice: Create and mount different volume types

**Wednesday-Thursday**: Advanced Configuration
- Resource quotas and LimitRanges
- Pod security policies/standards
- Practice: Implement resource constraints

**Friday**: Application Lifecycle
- Deployments strategies
- Blue-green and canary deployments
- Practice: Perform various deployment scenarios

**Weekend**: Hands-on Storage Labs
- Set up stateful applications
- Practice backup and restore procedures
- Work with dynamic provisioning

### Week 4: Networking Deep Dive
**Monday-Tuesday**: Service Networking
- Service discovery and DNS
- Endpoints and EndpointSlices
- Practice: Troubleshoot service connectivity

**Wednesday-Thursday**: Network Policies
- Ingress and egress rules
- Policy scenarios
- Practice: Implement complex network policies

**Friday**: Ingress Controllers
- Ingress resources and rules
- TLS configuration
- Practice: Set up ingress for multiple services

**Weekend**: Mock Exam #2
- Timed practice exam (2 hours)
- Focus on time management
- Identify areas needing speed improvement

### Week 5: Security & RBAC
**Monday-Tuesday**: RBAC Fundamentals
- Roles, ClusterRoles, RoleBindings, ClusterRoleBindings
- Service accounts
- Practice: Create various RBAC scenarios

**Wednesday-Thursday**: Security Contexts
- Pod and container security contexts
- Capabilities and privileges
- Practice: Secure pod configurations

**Friday**: Secrets Management
- Creating and using secrets
- Encryption at rest
- Practice: Secure application deployments

**Weekend**: Security-focused Labs
- Implement least-privilege access
- Troubleshoot RBAC issues
- Practice security hardening

### Week 6: Cluster Maintenance
**Monday-Tuesday**: Node Management
- Cordon, drain, and uncordon nodes
- Node maintenance procedures
- Practice: Perform node operations

**Wednesday-Thursday**: Cluster Upgrades
- Upgrade procedures for control plane and nodes
- Version skew policies
- Practice: Simulate cluster upgrades

**Friday**: Backup and Restore
- ETCD backup and restore
- Disaster recovery procedures
- Practice: Perform backup operations

**Weekend**: Mock Exam #3
- Full exam simulation
- Strict time limit enforcement
- Practice exam environment setup

### Week 7: Troubleshooting & Monitoring
**Monday-Tuesday**: Application Troubleshooting
- Debugging pods and containers
- Analyzing logs
- Practice: Fix broken applications

**Wednesday-Thursday**: Cluster Troubleshooting
- Control plane component issues
- Networking problems
- Practice: Diagnose cluster problems

**Friday**: Monitoring and Logging
- Basic monitoring setup
- Log aggregation concepts
- Practice: Implement monitoring solutions

**Weekend**: Troubleshooting Marathon
- Work through troubleshooting scenarios
- Time-boxed problem solving
- Build troubleshooting methodology

### Week 8: Final Preparation
**Monday-Tuesday**: Weak Area Focus
- Review topics with lowest scores
- Targeted practice on difficult areas
- Speed optimization for common tasks

**Wednesday-Thursday**: Documentation Practice
- Navigate Kubernetes docs efficiently
- Build bookmark list
- Practice finding examples quickly

**Friday**: Final Review
- Review command aliases
- Practice context switching
- Mental preparation

**Weekend Before Exam**: Light Practice
- One final mock exam (Saturday)
- Review notes and command list (Sunday)
- Early rest and preparation

## Daily Time Allocation

### Weekdays (2-3 hours)
- **First 30 minutes**: Review previous day's topics
- **Next 90 minutes**: New topic study and practice
- **Final 30-60 minutes**: Hands-on labs

### Weekends (4-5 hours)
- **Saturday**: Practice exam or intensive labs
- **Sunday**: Review, documentation, and planning

## Progress Tracking & Milestones

### Weekly Assessment Methods
- **Time yourself** solving 5 random tasks each week
- **Track improvement** in speed and accuracy
- **Identify recurring mistakes** and address them
- **Adjust focus areas** based on performance

### Skills Progression Checklist
Track your mastery of each skill area:

#### Foundation Skills
- [ ] Can create resources imperatively without reference
- [ ] Understand YAML structure and can edit efficiently
- [ ] Comfortable with kubectl basic commands
- [ ] Can navigate Kubernetes documentation quickly

#### Intermediate Skills
- [ ] Can troubleshoot pod failures effectively
- [ ] Can configure network policies correctly
- [ ] Can implement persistent storage solutions
- [ ] Can manage ConfigMaps and Secrets

#### Advanced Skills
- [ ] Can perform etcd backup and restore
- [ ] Can configure RBAC policies
- [ ] Can troubleshoot cluster components
- [ ] Can perform cluster upgrades

### Milestone Checkpoints

### End of Week 2
- [ ] Comfortable with basic kubectl commands
- [ ] Can create any basic resource imperatively
- [ ] Understand pod lifecycle
- [ ] Complete first mock exam (baseline score)

### End of Week 4
- [ ] Networking concepts clear
- [ ] Can implement network policies
- [ ] Storage operations smooth
- [ ] Mock exam score improved by 20%+

### End of Week 6
- [ ] RBAC implementation confident
- [ ] Cluster maintenance procedures memorized
- [ ] Troubleshooting methodology developed
- [ ] Scoring 70%+ on mock exams

### End of Week 8
- [ ] Consistently finishing mock exams on time
- [ ] All exam objectives covered
- [ ] Scoring 80%+ on mock exams
- [ ] Ready for certification

### Performance Benchmarks
- **Week 2**: Complete simple tasks in < 5 minutes
- **Week 4**: Complete medium tasks in < 10 minutes
- **Week 6**: Complete complex tasks in < 15 minutes
- **Week 8**: Complete full mock exam in < 1:45 hours

## Adjustment Guidelines

- If falling behind, focus on high-weight exam topics
- If ahead of schedule, add more troubleshooting practice
- Adjust based on mock exam performance
- Don't skip topics even if they seem easy

---

**Note**: This schedule is intensive and requires dedication. Adjust the pace based on your prior Kubernetes experience and available study time. The key is consistent daily practice rather than cramming.