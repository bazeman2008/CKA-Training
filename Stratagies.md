# CKA Exam Strategy Guide

## Time Management Strategy

You have 2 hours for 15-20 tasks worth different point values. **Always check the point values first** - tackle high-value questions early when you're fresh. If you're stuck on a low-value task for more than 5-10 minutes, flag it and move on. You can return to flagged questions if time permits.

## kubectl Efficiency

Set up aliases immediately when the exam starts:

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
```

Use imperative commands to generate YAML quickly rather than writing from scratch. For example, `k create deployment nginx --image=nginx $do > deploy.yaml` gives you a template to modify.

## Context Switching

The exam uses multiple clusters. **Always verify your current context** before starting each question with `kubectl config current-context`. Many candidates lose points by working on the wrong cluster. Make context switching automatic - read the question, switch context, then proceed.

## Documentation Usage

You can access kubernetes.io docs, but searching wastes time. Bookmark key pages beforehand if possible, or know exactly where to find examples for common tasks like creating persistent volumes, network policies, or RBAC configurations.

## Question Approach

Read each question completely before starting - the requirements are often more complex than they initially appear. Look for keywords like "NodePort," "specific namespace," or "particular node" that define exact requirements.

## Verification Habits

After completing each task, quickly verify it works. Run `kubectl get` commands to confirm objects were created correctly. For deployments, check that pods are running. This catches simple mistakes that cost easy points.

## Backup Strategy

If you modify existing resources, consider backing them up first with `kubectl get <resource> -o yaml > backup.yaml`. This lets you recover quickly if you break something.

---

**Key Takeaway:** The key is building these habits through practice so they become automatic under exam pressure.