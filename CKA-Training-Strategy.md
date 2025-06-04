# CKA Training Strategy

## Overview

This training strategy is designed to prepare you for the Certified Kubernetes Administrator (CKA) exam through a structured, hands-on approach that mirrors real-world scenarios and exam conditions.

## Core Training Principles

### 1. Hands-On Practice Over Theory
- **80/20 Rule**: Spend 80% of your time practicing kubectl commands and solving problems, 20% on reading documentation
- Set up a local Kubernetes cluster (minikube, kind, or kubeadm) for unlimited practice
- Create and destroy resources repeatedly until commands become muscle memory

### 2. Scenario-Based Learning
Focus on common exam scenarios:
- Deploy applications with specific requirements
- Troubleshoot broken clusters and applications
- Configure networking between pods
- Implement security policies
- Manage storage and persistent volumes
- Perform cluster maintenance tasks

### 3. Time-Boxed Practice Sessions
- Start with untimed practice to build confidence
- Progress to timed exercises matching exam conditions
- Practice full mock exams under 2-hour time limits
- Track your speed improvement over time

## Weekly Training Structure

### Week 1-2: Foundation Building
- Master basic kubectl commands
- Understand Kubernetes architecture
- Practice creating pods, deployments, services
- Learn YAML structure and editing

### Week 3-4: Core Concepts
- Networking (Services, Ingress, Network Policies)
- Storage (PV, PVC, StorageClasses)
- Configuration (ConfigMaps, Secrets)
- Resource management (requests, limits, quotas)

### Week 5-6: Advanced Topics
- RBAC and security contexts
- Scheduling (affinity, taints, tolerations)
- Logging and monitoring basics
- Etcd backup and restore

### Week 7-8: Exam Preparation
- Full-length practice exams
- Focus on weak areas identified
- Speed optimization techniques
- Documentation navigation practice

## Daily Practice Routine

### Morning (30-45 minutes)
- Review yesterday's mistakes
- Practice 3-5 imperative commands
- Solve one medium-complexity scenario

### Evening (60-90 minutes)
- Work through structured exercises
- Practice one exam domain thoroughly
- Document learnings and gotchas

### Weekend Sessions
- Take one full practice exam
- Review and understand all mistakes
- Practice troubleshooting scenarios

## Resource Utilization

### Primary Resources
1. **Official Kubernetes Documentation** - Your exam bible
2. **Killer.sh** - CKA simulator (comes with exam purchase)
3. **Kubernetes the Hard Way** - Deep understanding of components

### Practice Environments
- Local cluster for unlimited practice
- Cloud providers' free tiers for multi-node scenarios
- Container sandbox environments for quick experiments

## Common Pitfalls to Avoid

### During Training
- Don't rely on GUI tools - exam is CLI only
- Don't memorize YAML - learn to generate it
- Don't skip networking - it's heavily tested
- Don't ignore RBAC - it's complex but important

### Knowledge Gaps to Address
- Pod-to-pod communication
- Persistent volume lifecycles
- Service account usage
- Cluster upgrade procedures

## Progress Tracking

### Weekly Assessments
- Time yourself solving 5 random tasks
- Track improvement in speed and accuracy
- Identify recurring mistakes
- Adjust focus areas accordingly

### Skills Checklist
Maintain a checklist of all exam objectives:
- [ ] Can create resources imperatively
- [ ] Can troubleshoot pod failures
- [ ] Can configure network policies
- [ ] Can perform etcd backup
- [ ] Can configure RBAC
- [ ] Can troubleshoot cluster components

## Final Preparation

### Two Weeks Before Exam
- Daily full-length practice exams
- Focus only on weak areas
- Practice context switching
- Optimize keyboard shortcuts and aliases

### One Week Before Exam
- Light practice to maintain skills
- Review all documentation bookmarks
- Ensure comfortable with exam environment
- Mental preparation and stress management

---

**Remember**: The CKA is a practical exam. The more you practice with your hands on the keyboard, the better prepared you'll be. Consistency beats intensity - practice daily, even if just for 30 minutes.