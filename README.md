# CKA Certification Study Guide 🚀

A comprehensive study repository for the Certified Kubernetes Administrator (CKA) exam, including test strategies, study materials, and practical scenarios.

## 📋 Repository Contents

- [Test Information & Format](./test-info.md) - Complete exam details and requirements
- [Study Strategy Guide](./study-strategy.md) - Effective exam preparation strategies
- [Test Environment & Scenarios](./test-environment.md) - What to expect during the exam
- [Practice Labs](./labs/) - Hands-on exercises organized by topic
- [Cheat Sheets](./cheat-sheets/) - Quick reference commands and YAML templates
- [Mock Exams](./mock-exams/) - Timed practice scenarios

## 🎯 CKA Exam Overview

The Certified Kubernetes Administrator exam is a **2-hour, hands-on, performance-based test** that validates your ability to manage Kubernetes clusters in production environments.

### Key Details
- **Format:** Terminal-based practical scenarios (no multiple choice)
- **Duration:** 2 hours
- **Questions:** 15-20 practical tasks
- **Passing Score:** 66%
- **Cost:** $395 (includes one free retake)
- **Validity:** 2 years

### Topic Breakdown
| Topic | Percentage | Focus Areas |
|-------|------------|-------------|
| **Troubleshooting** | 30% | Cluster debugging, application issues, networking problems |
| **Cluster Architecture** | 25% | RBAC, etcd backup/restore, cluster upgrades |
| **Services & Networking** | 20% | CNI, Ingress, Service types, CoreDNS |
| **Workloads & Scheduling** | 15% | Deployments, ConfigMaps, Secrets, resource management |
| **Storage** | 10% | PV/PVC, Storage classes, volume types |

## 🎓 Study Approach

### Phase 1: Foundation (2-3 weeks)
- [ ] Complete Kubernetes basics course
- [ ] Understand core concepts (Pods, Services, Deployments)
- [ ] Set up local Kubernetes environment
- [ ] Practice basic kubectl commands

### Phase 2: Hands-On Practice (3-4 weeks)
- [ ] Work through topic-specific labs
- [ ] Practice cluster administration tasks
- [ ] Master troubleshooting scenarios
- [ ] Complete RBAC and networking exercises

### Phase 3: Exam Preparation (1-2 weeks)
- [ ] Take timed mock exams
- [ ] Review weak areas
- [ ] Practice exam strategies
- [ ] Familiarize with kubernetes.io documentation layout

## ⚡ Essential Commands to Master

```bash
# Context Management (CRITICAL!)
kubectl config use-context <cluster-name>
kubectl config current-context

# Quick Object Creation
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl run test-pod --image=busybox --dry-run=client -o yaml

# Troubleshooting
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp

# Resource Management
kubectl top nodes
kubectl top pods
kubectl get pods -o wide --show-labels
```

## 🏆 Exam Success Strategies

### Time Management
1. **Check point values first** - tackle high-value questions early
2. **Don't spend >10 minutes** on low-value questions
3. **Flag and return** to difficult questions if time permits
4. **Always verify** your solutions work

### Technical Tips
- **Always switch context first** before starting each question
- **Use imperative commands** to generate YAML templates quickly
- **Set up aliases** for efficiency: `alias k=kubectl`
- **Bookmark important docs** for quick reference
- **Practice under time pressure** to build speed

### Common Pitfalls to Avoid
- ❌ Working on wrong cluster context
- ❌ Not reading question requirements completely  
- ❌ Spending too much time on low-value questions
- ❌ Forgetting to verify solutions
- ❌ Not using kubectl shortcuts

## 📚 Recommended Resources

### Official Documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [CKA Exam Curriculum](https://github.com/cncf/curriculum)

### Practice Platforms
- [Killer Shell CKA Simulator](https://killer.sh/)
- [KodeKloud CKA Course](https://kodekloud.com/)
- [A Cloud Guru](https://acloudguru.com/)

### Study Materials
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [CKA Study Guide](https://github.com/walidshaari/Kubernetes-Certified-Administrator)

## 🛠 Lab Environment Setup

### Option 1: Local Setup
```bash
# Using kind (Kubernetes in Docker)
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind create cluster --name cka-practice
```

### Option 2: Cloud Environment
- Google Kubernetes Engine (GKE)
- Amazon EKS
- Azure AKS
- DigitalOcean Kubernetes

### Option 3: Online Labs
- Play with Kubernetes
- Katacoda Kubernetes Playground
- KillerKoda scenarios

## 📅 Study Schedule Template

### Week 1-2: Fundamentals
- Day 1-3: Pods, ReplicaSets, Deployments
- Day 4-5: Services and Networking basics
- Day 6-7: ConfigMaps, Secrets, Volumes

### Week 3-4: Advanced Topics
- Day 1-2: RBAC and Security
- Day 3-4: Cluster maintenance and troubleshooting
- Day 5-7: Storage and persistent volumes

### Week 5-6: Practice & Review
- Day 1-3: Mock exams and timed practice
- Day 4-5: Review weak areas
- Day 6-7: Final preparation and documentation review

## 🤝 Contributing

Feel free to contribute additional practice scenarios, corrections, or improvements to this study guide. Please ensure all content is accurate and follows current Kubernetes best practices.

## 📝 License

This study guide is provided as-is for educational purposes. Kubernetes® is a registered trademark of The Linux Foundation.

---

**Good luck with your CKA certification journey! 🎉**

> Remember: The CKA exam tests practical skills, not theoretical knowledge. Focus on hands-on practice and real-world scenarios.
